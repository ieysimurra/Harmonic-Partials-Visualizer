# 🎵 Parciais Harmônicos — Visualizador Dinâmico

[![GitHub Pages](https://img.shields.io/badge/Demo-GitHub%20Pages-6ee7b7?style=for-the-badge&logo=github)](https://SEU-USUARIO.github.io/harmonic-partials-visualizer/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
[![Made with Web Audio API](https://img.shields.io/badge/Web%20Audio-API-f472b6?style=for-the-badge)]()

> Aplicativo web interativo para análise e visualização em tempo real do comportamento dinâmico de parciais harmônicos, utilizando captura de áudio por microfone ou arquivo.

Desenvolvido para uso didático na disciplina de **Acústica Musical** do curso de Música da UNICAMP (NICS — Núcleo Interdisciplinar de Comunicação Sonora).

<p align="center">
  <img src="docs/img/screenshot-waterfall.png" alt="Visualização Cascata 3D" width="800">
</p>

---

## ✨ Funcionalidades

### 🎙️ Entrada de Áudio
- **Microfone em tempo real** — captura direta via `getUserMedia`
- **Arquivo de áudio** — carregamento de `.wav`, `.mp3`, `.ogg`, `.flac` etc. com reprodução em loop

### 📊 Quatro Modos de Visualização

| Modo | Descrição |
|------|-----------|
| **Cascata 3D** | Evolução temporal dos parciais em perspectiva tridimensional (inspirado em análises clássicas de Risset/Grey) |
| **Barras** | Amplitude instantânea de cada parcial com indicação de frequência |
| **Espectrograma** | Mapa de calor parciais × tempo com codificação por cor |
| **Temporal** | Curvas de amplitude dos 8 primeiros parciais ao longo do tempo |

### 🎯 Detecção de Pitch
- Algoritmo de autocorrelação com interpolação parabólica
- Exibição de **nota musical**, **frequência fundamental (F₀)** e **desvio em cents**
- Extração automática de parciais harmônicos baseada na F₀ detectada
- Fallback para bandas espectrais quando não há pitch definido (sons inarmônicos/ruído)

### 🎛️ Controles Interativos
- **Número de parciais**: 4 a 32
- **Suavização FFT** (`smoothingTimeConstant`): 0.00 a 0.98
- **Ganho**: −20 dB a +30 dB
- **Piso de ruído**: −120 dB a −30 dB
- **Tamanho da FFT**: 2048, 4096, 8192 ou 16384 amostras
- **Escala logarítmica** (toggle)
- **4 paletas de cores**: Aurora, Fogo, Oceano, Neon

### 📋 Painel de Parciais
- Lista em tempo real com frequência e barra de amplitude para cada parcial
- Codificação por cor sincronizada com a visualização

---

## 🚀 Demo Online

**👉 [Acesse a demo ao vivo no GitHub Pages](https://SEU-USUARIO.github.io/harmonic-partials-visualizer/)**

> Necessário um navegador moderno com suporte a Web Audio API (Chrome, Firefox, Edge, Safari 14.1+).

---

## 📦 Instalação e Uso

### Opção 1 — GitHub Pages (recomendado)

Não é necessário instalar nada. Acesse o link da demo acima e use diretamente no navegador.

### Opção 2 — Execução local

```bash
# Clone o repositório
git clone https://github.com/SEU-USUARIO/harmonic-partials-visualizer.git
cd harmonic-partials-visualizer

# Abra diretamente no navegador
open index.html
# ou
xdg-open index.html    # Linux
start index.html        # Windows
```

### Opção 3 — Servidor local (necessário para algumas funcionalidades de áudio)

```bash
# Com Python 3
python3 -m http.server 8000

# Com Node.js
npx serve .

# Depois acesse http://localhost:8000
```

> **Nota**: Alguns navegadores exigem HTTPS ou localhost para acesso ao microfone. Se a captura não funcionar abrindo o arquivo diretamente, use um servidor local.

---

## 🎓 Uso Pedagógico

Este aplicativo foi projetado para apoiar o ensino de Acústica Musical. Algumas atividades sugeridas:

### Atividade 1 — Estrutura Espectral de Instrumentos
Peça aos alunos que capturem sons de diferentes instrumentos (voz, flauta, violão, clarinete) e comparem a distribuição de parciais. Discuta a relação entre **timbre** e **espectro harmônico**.

### Atividade 2 — Transientes de Ataque
Usando o modo **Cascata 3D** ou **Temporal**, observe como as parciais se comportam nos primeiros milissegundos de um som. Compare o ataque de um piano com o de uma flauta.

### Atividade 3 — Harmônicos e Inharmonicidade
Compare sons com parciais harmônicos (voz, cordas) versus sons inarmônicos (sinos, pratos). No modo de bandas espectrais (sem F₀ detectado), observe como a energia se distribui de forma diferente.

### Atividade 4 — Relação Frequência × Nota Musical
Use a detecção de pitch para verificar a F₀ de diferentes notas cantadas ou tocadas. Observe o desvio em cents e discuta afinação temperada versus natural.

### Atividade 5 — Efeito do Tamanho da FFT
Altere o tamanho da FFT entre 2048 e 16384. Discuta o compromisso entre **resolução em frequência** e **resolução temporal** (princípio da incerteza de Heisenberg aplicado ao sinal).

---

## 🏗️ Arquitetura Técnica

### Stack

| Camada | Tecnologia |
|--------|-----------|
| Captura de áudio | Web Audio API (`AudioContext`, `AnalyserNode`) |
| Processamento | FFT nativa do `AnalyserNode` + autocorrelação em JS |
| Renderização | Canvas 2D com `requestAnimationFrame` |
| Interface | HTML + CSS (Grid/Flexbox), zero dependências externas |
| Tipografia | DM Sans + Space Mono (Google Fonts) |

### Fluxo de Dados

```
Microfone/Arquivo
      │
      ▼
  AudioContext
      │
      ▼
  AnalyserNode ──► getFloatFrequencyData() ──► Extração de parciais
      │                                              │
      ▼                                              ▼
  getFloatTimeDomainData()                   Amplitudes por parcial
      │                                              │
      ▼                                              ▼
  Autocorrelação ──► F₀                    Histórico temporal
      │                                              │
      ▼                                              ▼
  Nota + Cents                              Canvas 2D (4 modos)
```

### Detecção de Pitch

O algoritmo utiliza **autocorrelação no domínio do tempo** com os seguintes passos:

1. Cálculo do RMS para descarte de silêncio
2. Trimming do buffer por threshold de amplitude
3. Função de autocorrelação
4. Busca do primeiro pico após a região inicial
5. Interpolação parabólica para refinamento sub-amostra

### Extração de Parciais

Quando uma F₀ é detectada:
- Calcula a frequência esperada de cada parcial: `F₀ × n` (n = 1, 2, ..., N)
- Localiza o bin FFT correspondente
- Busca o máximo numa janela ao redor do bin (largura proporcional à frequência)
- Normaliza a amplitude em relação ao piso de ruído

Quando não há F₀ (sons inarmônicos):
- Divide o espectro em N bandas (lineares ou logarítmicas)
- Mostra a energia máxima de cada banda

---

## ⚙️ Configuração do GitHub Pages

### Passo a passo

1. **Crie o repositório** no GitHub (pode ser público ou privado com GitHub Pro)

2. **Faça o push dos arquivos**:
   ```bash
   git init
   git add .
   git commit -m "feat: visualizador de parciais harmônicos"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/harmonic-partials-visualizer.git
   git push -u origin main
   ```

3. **Ative o GitHub Pages**:
   - Vá em **Settings** → **Pages**
   - Em "Source", selecione **Deploy from a branch**
   - Branch: `main`, pasta: `/ (root)`
   - Clique em **Save**

4. **Aguarde o deploy** (1–2 minutos). O site estará disponível em:
   ```
   https://SEU-USUARIO.github.io/harmonic-partials-visualizer/
   ```

### Domínio personalizado (opcional)

Crie um arquivo `CNAME` na raiz com o domínio:
```
parciais.seudominio.com.br
```

Configure o DNS do seu domínio apontando um CNAME para `SEU-USUARIO.github.io`.

---

## 📁 Estrutura do Repositório

```
harmonic-partials-visualizer/
├── index.html              # Aplicativo completo (single-file)
├── README.md               # Esta documentação
├── LICENSE                  # Licença MIT
├── CONTRIBUTING.md          # Guia de contribuição
├── CHANGELOG.md            # Histórico de versões
├── .github/
│   └── ISSUE_TEMPLATE.md   # Template para issues
└── docs/
    ├── TECHNICAL.md         # Documentação técnica detalhada
    └── img/                 # Screenshots para documentação
        └── screenshot-waterfall.png
```

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Consulte o [CONTRIBUTING.md](CONTRIBUTING.md) para orientações.

### Ideias para contribuição

- [ ] Exportação de dados (CSV com amplitudes das parciais ao longo do tempo)
- [ ] Modo de gravação (captura de snapshots do espectro)
- [ ] Suporte a entrada MIDI para F₀ externa
- [ ] Modo de comparação lado a lado (dois sons)
- [ ] Visualização WebGL para performance em displays de alta resolução
- [ ] Suporte offline (Service Worker / PWA)
- [ ] Internacionalização (inglês, espanhol)
- [ ] Testes automatizados com Web Audio API mock

---

## 📚 Referências

- Henrique, L. L. (2014). *Acústica Musical*. Fundação Calouste Gulbenkian.
- Risset, J.-C. & Wessel, D. L. (1999). "Exploration of Timbre by Analysis and Synthesis". Em *The Psychology of Music*, Academic Press.
- Grey, J. M. (1977). "Multidimensional perceptual scaling of musical timbres". *JASA*, 61(5).
- Roads, C. (1996). *The Computer Music Tutorial*. MIT Press.
- Web Audio API Specification — [W3C](https://www.w3.org/TR/webaudio/)
- MDN Web Docs — [AnalyserNode](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode)

---

## 📄 Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

---

## 👤 Autor

**Prof. Ivan Simurra**
NICS — Núcleo Interdisciplinar de Comunicação Sonora
Universidade Estadual de Campinas (UNICAMP)

---

<p align="center">
  <sub>Feito com 🎵 para o ensino de Acústica Musical · NICS/UNICAMP</sub>
</p>
