# 📋 Changelog

Todas as mudanças notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
e este projeto adere ao [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [1.0.0] — 2026-04-13

### Adicionado
- Captura de áudio em tempo real via microfone (Web Audio API)
- Carregamento de arquivos de áudio (WAV, MP3, OGG, FLAC)
- Quatro modos de visualização:
  - **Cascata 3D** — evolução temporal com perspectiva
  - **Barras** — amplitude instantânea por parcial
  - **Espectrograma** — mapa de calor parciais × tempo
  - **Temporal** — curvas de amplitude ao longo do tempo
- Detecção de pitch por autocorrelação com interpolação parabólica
- Exibição de nota musical, F₀ e desvio em cents
- Extração automática de parciais harmônicos baseada na F₀
- Fallback para bandas espectrais em sons sem pitch definido
- Controles interativos:
  - Número de parciais (4–32)
  - Suavização FFT (0.00–0.98)
  - Ganho (−20 dB a +30 dB)
  - Piso de ruído (−120 dB a −30 dB)
  - Tamanho da FFT (2048, 4096, 8192, 16384)
  - Toggle de escala logarítmica
- Quatro paletas de cores (Aurora, Fogo, Oceano, Neon)
- Painel lateral com lista de parciais em tempo real
- Interface responsiva (desktop e tablet)
- Aplicativo single-file sem dependências externas
- Documentação completa em português
- Configuração para GitHub Pages
