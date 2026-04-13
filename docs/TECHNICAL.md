# 🔧 Documentação Técnica

Documentação detalhada da implementação do Visualizador de Parciais Harmônicos.

---

## Visão geral da arquitetura

O aplicativo é um **single-file HTML** que combina HTML, CSS e JavaScript sem dependências externas (exceto Google Fonts). Isso garante portabilidade total — basta servir o `index.html` para ter o app funcional.

### Pipeline de áudio

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   getUserMedia   │────▶│  AudioContext │────▶│   AnalyserNode  │
│   ou FileReader  │     │              │     │  (FFT nativa)   │
└─────────────────┘     └──────────────┘     └────────┬────────┘
                                                       │
                                    ┌──────────────────┼──────────────────┐
                                    ▼                  ▼                  ▼
                          getFloatFrequency    getFloatTimeDomain    smoothingTime
                               Data()              Data()            Constant
                                    │                  │
                                    ▼                  ▼
                          Extração de parciais   Autocorrelação
                          (bins FFT → amps)      (pitch detect)
                                    │                  │
                                    ▼                  ▼
                              state.partial       state.f0
                              Amplitudes[]        (freq Hz)
                                    │                  │
                                    └────────┬─────────┘
                                             ▼
                                     requestAnimation
                                        Frame()
                                             │
                                             ▼
                                      Canvas 2D draw
                                      (4 view modes)
```

---

## Objeto de estado (`state`)

Toda a aplicação gira em torno de um único objeto de estado:

```javascript
const state = {
  running: false,          // Captura ativa?
  view: 'waterfall',       // Modo de visualização atual
  numPartials: 16,         // Nº de parciais a exibir (4–32)
  smoothing: 0.85,         // Suavização da FFT (0.00–0.98)
  gain: 0,                 // Ganho em dB (−20 a +30)
  floor: -80,              // Piso de ruído em dB (−120 a −30)
  fftSize: 4096,           // Tamanho da FFT
  logScale: false,         // Escala logarítmica?
  colorScheme: 0,          // Índice da paleta de cores (0–3)
  f0: 0,                   // Frequência fundamental detectada (Hz)
  partialAmplitudes: [],   // Amplitudes normalizadas [0,1] de cada parcial
  history: [],             // Histórico de frames (para cascata e temporal)
  historyLength: 80,       // Nº de frames no histórico
  spectrogramData: [],     // Dados acumulados para espectrograma
  audioContext: null,       // Referência ao AudioContext
  analyser: null,           // Referência ao AnalyserNode
  sourceNode: null,         // Referência ao MediaStreamSource ou BufferSource
  isFile: false,            // Entrada é arquivo (true) ou microfone (false)?
};
```

---

## Detecção de pitch — autocorrelação

### Algoritmo

A detecção de F₀ usa autocorrelação no domínio do tempo, um método robusto para sinais monofônicos com conteúdo harmônico claro.

**Passos:**

1. **Cálculo de RMS**: se o RMS do buffer está abaixo de 0.01, o sinal é considerado silêncio e retorna −1 (sem pitch).

2. **Trimming**: remove as bordas do buffer onde a amplitude é menor que 0.2 (20% do pico). Isso reduz artefatos de borda.

3. **Autocorrelação**: para cada lag `τ`, calcula:

   ```
   R(τ) = Σ x(n) · x(n + τ)
   ```

   onde `x(n)` é o sinal trimado.

4. **Busca do pico**: a partir do primeiro zero-crossing descendente de R(τ), encontra o máximo global. O lag correspondente é o período T₀.

5. **Interpolação parabólica**: ajusta uma parábola nos três pontos ao redor do pico para obter resolução sub-amostra:

   ```
   δ = −b / (2a)
   onde a = (R[T₀−1] + R[T₀+1] − 2·R[T₀]) / 2
         b = (R[T₀+1] − R[T₀−1]) / 2
   ```

6. **Frequência**: `F₀ = sampleRate / (T₀ + δ)`

### Conversão para nota musical

```javascript
noteNum = 12 × log₂(F₀ / 440) + 69   // MIDI note number
cents = (noteNum − round(noteNum)) × 100
```

### Limitações

- Funciona bem para sinais monofônicos com harmônicos claros (voz, flauta, violão)
- Pode falhar com sinais polifônicos, muito ruidosos ou fortemente inarmônicos
- A resolução em frequência depende do tamanho do buffer (fftSize)

---

## Extração de parciais harmônicos

### Com F₀ detectada (sons harmônicos)

Para cada parcial `n` de 1 a N:

1. **Frequência esperada**: `f_n = F₀ × n`
2. **Bin FFT correspondente**: `bin = round(f_n / binWidth)` onde `binWidth = sampleRate / fftSize`
3. **Busca de pico**: varre uma janela ao redor do bin esperado. A largura da janela é proporcional à frequência (`spread = max(2, 3 × f_n / 1000)`) para acomodar vibrato e desvios de afinação.
4. **Normalização**: converte de dB para amplitude linear relativa ao piso de ruído:
   ```
   amp = max(0, (maxDb − floor) / (−floor)) × gainLinear
   ```

### Sem F₀ (sons inarmônicos, ruído, percussão)

Divide o espectro em N bandas equidistantes (ou logarítmicas se `logScale` ativo) e mostra a energia máxima de cada banda. Isso permite visualizar a distribuição espectral mesmo sem conteúdo harmônico.

---

## Modos de visualização

### Cascata 3D (`waterfall`)

Renderiza o histórico de frames como "fatias" empilhadas em perspectiva, semelhante às análises clássicas de Grey (1977) e Risset.

- Frames antigos aparecem ao fundo (menor e mais transparente)
- Frames recentes aparecem na frente (maior e mais opaco)
- A perspectiva é simulada via offsets lineares em X e Y
- Gradiente de preenchimento usa as primeiras 6 cores da paleta
- Linha de contorno com opacidade proporcional à posição temporal

### Barras (`bars`)

Exibe barras verticais para cada parcial com:
- Altura proporcional à amplitude
- Gradiente vertical (cor → transparente)
- Efeito de glow (box-shadow via Canvas) proporcional à amplitude
- Labels com número do parcial e frequência em Hz

### Espectrograma (`spectrogram`)

Mapa de calor com:
- Eixo X = tempo (frames acumulados)
- Eixo Y = número do parcial (invertido, grave embaixo)
- Cor = paleta ativa, opacidade = amplitude
- Acumula até 200 frames

### Temporal (`timeline`)

Gráfico de linhas sobrepostas mostrando a evolução temporal dos 8 primeiros parciais:
- Cada parcial é uma linha colorida
- A fundamental (P1) é desenhada por último, com traço mais grosso
- Labels à direita indicam cada parcial

---

## Paletas de cores

| Paleta | Estilo | Uso sugerido |
|--------|--------|-------------|
| **Aurora** | Multicolorida vibrante | Uso geral, apresentações |
| **Fogo** | Amarelos/laranjas/vermelhos | Ênfase em energia, ataques |
| **Oceano** | Azuis/cyans | Tons suaves, sons de flauta |
| **Neon** | Alto contraste saturado | Demonstrações em projetor |

Cada paleta contém 32 cores para suportar o número máximo de parciais.

---

## Parâmetros e seus efeitos

| Parâmetro | Range | Efeito |
|-----------|-------|--------|
| **Nº de Parciais** | 4–32 | Mais parciais = mais detalhe do timbre, mais custo de renderização |
| **Suavização** | 0.00–0.98 | Valores altos = movimento suave mas lento; valores baixos = resposta rápida mas nervosa |
| **Ganho** | −20 a +30 dB | Amplifica ou atenua visualmente. Útil para sinais fracos de microfone |
| **Piso** | −120 a −30 dB | Define o que é considerado "silêncio". Abaixar revela mais detalhe em sinais fracos |
| **FFT Size** | 2048–16384 | Maior = melhor resolução em frequência, pior em tempo. Para F₀ < 100 Hz, use 8192+ |
| **Escala log** | on/off | Afeta apenas o modo de bandas espectrais (sem F₀). Comprime as altas frequências |

---

## Compatibilidade de navegadores

| Navegador | Microfone | Arquivo | Notas |
|-----------|-----------|---------|-------|
| Chrome 66+ | ✅ | ✅ | Referência principal |
| Firefox 76+ | ✅ | ✅ | Funcional |
| Edge 79+ | ✅ | ✅ | Baseado em Chromium |
| Safari 14.1+ | ✅ | ✅ | Requer gesto do usuário para AudioContext |
| Safari iOS | ⚠️ | ✅ | Microfone pode exigir HTTPS |
| Chrome Android | ✅ | ✅ | Layout responsivo |

> **HTTPS é necessário** para acesso ao microfone em todos os navegadores (exceto localhost).

---

## Performance

- O loop de animação usa `requestAnimationFrame` (~60 fps)
- A FFT é computada nativamente pelo `AnalyserNode` (implementação C++ do navegador)
- A autocorrelação é O(n²) no tamanho do buffer — para fftSize=16384 pode causar lag em dispositivos lentos
- O Canvas 2D é suficiente para esta aplicação; WebGL seria necessário apenas para displays 4K+ com muitas parciais

### Otimizações futuras

- Web Workers para autocorrelação em thread separada
- OffscreenCanvas para renderização fora da thread principal
- Throttle do cálculo de pitch (não precisa ser a cada frame)

---

## Referências técnicas

- [Web Audio API — W3C Spec](https://www.w3.org/TR/webaudio/)
- [AnalyserNode — MDN](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode)
- [Autocorrelation pitch detection](https://www.audiolabs-erlangen.de/resources/MIR/FMP/C3/C3S3_PitchTracking_Autocorrelation.html)
- McLeod, P. & Wyvill, G. (2005). "A Smarter Way to Find Pitch". *ICMC*.
