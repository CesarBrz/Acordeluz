# AcordeLuz – Instrumento musical visual via MIDI Bluetooth (ESP32 + WS2812)

Assista o vídeo: [https://www.instagram.com/p/CgN-LxKgQs7/]

## Português

### Visão geral
O AcordeLuz é uma obra de arte interativa que funciona como um instrumento musical visual. Ele recebe notas e controles MIDI por Bluetooth (BLE MIDI) e traduz esses eventos em padrões de luz em um painel de LEDs WS2812. O código foi projetado para painéis grandes (12 linhas × 75 colunas = 900 LEDs) e pode operar com dois módulos ESP32 controlando metades do painel (superior e inferior), anunciados como “AcordeLuz A” e “AcordeLuz B”.

### Principais recursos
- Visualizações responsivas a Notas MIDI (Note On) e Controles (Control Change)
- Conexão MIDI via Bluetooth Low Energy (BLE) diretamente no ESP32
- Mapeamento de painel em zig‑zag com função de remapeamento dedicada
- Pós-processamento: fade, “escorrer” vertical e reforço de cores primárias/secundárias
- Alternância de algoritmos ao vivo via MIDI CC ou porta serial
- Mistura de múltiplos algoritmos simultaneamente (pincéis) para compor o quadro em tempo real

### Hardware e requisitos
- Placa: ESP32 (BLE integrado)
- LEDs: WS2812/WS2812B (matriz 12×75 por módulo; total 900 LEDs por módulo)
- Pino de dados dos LEDs: `LED_DATA_PIN 14`
- Fonte de alimentação adequada (5V com corrente suficiente; para 900 LEDs, planeje vários pontos de injeção e GND comum)

### Bibliotecas
- FastLED
- BLEMidi (ESP32 BLE MIDI)

Instale via Arduino IDE (Library Manager) ou PlatformIO. Compile para “ESP32 Dev Module”.

### Configuração rápida (no código)
- `NUM_LEDS` = 900
- `LedsPorLinha` = 75
- `ModuloESP` = 0 controla metade de cima; `1` controla metade de baixo
- Nomes BLE: “AcordeLuz A” (ModuloESP 0) e “AcordeLuz B” (ModuloESP 1)

### Como usar
1) Alimente o painel e o ESP32; compartilhe o GND entre fonte e placa.
2) Faça o pareamento BLE MIDI a partir do seu instrumento/DAW com “AcordeLuz A/B”.
3) Envie notas MIDI; as visualizações serão geradas em tempo real.
4) Use MIDI CC no canal 16 para alternar algoritmos e pós-processamento.
5) Combine vários algoritmos ativos ao mesmo tempo para criar composições visuais (“pincéis”).

### Mapeamento MIDI
- Canal: controles são lidos no canal 16 (internamente `channel == 15`).
- Note On: dispara o algoritmo ativo (Note Off é ignorado).
- Control Change:
  - CC 65 → Alterna Algoritmo 1 (Barras randômicas)
  - CC 66 → Alterna Algoritmo 2 (Distância do centro)
  - CC 67 → Alterna Algoritmo 3 (Máscara – variação 4)
  - CC 68 → Alterna Algoritmo 4 (Máscara – variação 5)
  - CC 69 → Alterna Algoritmo 5 (Anéis – sólido)
  - CC 64 → Alterna Algoritmo 6 (Anéis – randômico)
  - CC 80 → Alterna Algoritmo 7 (Teste 1 – máscara padrão 1)
  - CC 81 → Alterna Algoritmo 8 (Teste 2 – máscara padrão 2)
  - CC 82 → Alterna Algoritmo 9 (reservado)
  - CC 83 → Alterna Algoritmo 10 (reservado)
  - CC 1  → Define semente de ruído (intensidade randômica)
  - CC 7  → Alterna pós-processamento: valor > 68 ativa Fade, senão ativa Escorrer

Você pode manter vários algoritmos ligados ao mesmo tempo; cada CC alterna um algoritmo de forma independente. Os efeitos se somam no quadro final (pincéis).

Observação: por padrão, o algoritmo 8 vem ativo no boot. O laço principal aplica `CoresSecundarias` a cada quadro e, conforme flags, `FadeNotas` e/ou `EscorreNotas`.

### Sincronização entre módulos (Serial)
Duas placas ESP32 podem operar em conjunto (metade superior e inferior do painel). Elas se comunicam por uma porta Serial dedicada para manter os efeitos sincronizados e compartilhar flags de estado conforme necessário. Essa ligação é simples (troca de mensagens/flags de baixa latência) e permite que ambos os lados componham o mesmo “quadro” visual de forma coesa.

### Recursos utilizados pelos algoritmos
- Bitmasks/tabelas de lookup para padrões: máscaras padrão e por‑nota (ex.: `MascaraPadrao1`, `MascaraPadrao2`, `MascaraAlgo4`, `MascaraAlgo5`)
- Máscaras de anéis 22×22 por grau/oitava para `GeraAneis` (funções `DesenhaAneisSolido` e `DesenhaAneisRandom`)
- Máscaras auxiliares (ex.: `MascaraBumbo`, `MascaraFormas`) para compor elementos rítmicos/geométricos
- Espelhamentos e duplicação de linhas para criar simetrias e reforço de nota/oitava
- Bias de cor e ruído controlado por seed (via CC1) para variações dinâmicas

### O que foi implementado manualmente
- Remapeamento físico `Remap()` para tiras WS2812 em zig‑zag (garantindo coordenadas lineares confiáveis)
- Conjunto de máscaras (bitmasks) elaborado e calibrado manualmente, incluindo padrões derivados de prototipagem visual (ex.: referências/estudos no Blender) e arrays otimizados para 900 px por módulo
- Pós‑processadores autorais: `FadeNotas` (esmaecimento com limiar/velocidade) e `EscorreNotas` (arraste vertical com atenuação)
- Integração MIDI BLE com callbacks dedicados (`onNoteOn`, `onNoteOff`, `onControlChange`) e mapeamento de CCs para alternância de algoritmos em tempo real
- Mecanismo de composição por “pincéis”: múltiplos algoritmos ativos pintam sobre o mesmo buffer de LEDs antes do `FastLED.show()`
- Controle via Serial para depuração e alternância rápida (menu e limpeza do painel)

### Comandos via Serial
- `1`..`10`: alterna algoritmos 1 a 10
- `E`/`e`: alterna Escorrer
- `F`/`f`: alterna Fade
- `C`/`c`: limpa o painel

### Dicas de energia e montagem
- Injete 5V ao longo do painel; use cabos dimensionados; GND comum entre fonte e ESP32.
- Evite acender 100% branco em todo o painel sem fonte dimensionada; correntes podem ser muito altas.

### Estrutura do código (resumo)
- `setup()`: inicializa FastLED, anuncia BLE MIDI (“AcordeLuz A/B”), pisca status e registra callbacks.
- `loop()`: aplica pós-processamento e atualiza o frame (`FastLED.show()`).
- Callbacks BLE: `onNoteOn`, `onNoteOff`, `onControlChange` mapeiam MIDI para visualizações e flags.
- Algoritmos: `GeraBarrasRandom`, `GeraDistanciaCentro`, `GeraAneis` (sólido/aleatório), `ALGO3`, `ALGO4`, `ALGOTeste1/2`.
- Pós: `FadeNotas`, `EscorreNotas`, `CoresSecundarias`.
- Mapeamento físico: `Remap()` corrige o zig‑zag das tiras.

---

## English

###Summary
AcordeLuz turns Bluetooth MIDI into light on a WS2812 panel, creating reactive and immersive real‑time musical visualizations.

### Overview
AcordeLuz is an interactive artwork that acts as a visual musical instrument. It receives MIDI notes and controls over Bluetooth (BLE MIDI) and renders light patterns on a WS2812 LED panel. The code targets large panels (12 rows × 75 columns = 900 LEDs) and can run with two ESP32 modules controlling the top and bottom halves, advertised as “AcordeLuz A” and “AcordeLuz B”.

### Highlights
- Real‑time visual response to MIDI Note and Control events
- Bluetooth Low Energy MIDI directly on ESP32
- Zig‑zag panel mapping via dedicated remap function
- Post‑processing: fade, vertical “drip” and primary/secondary color reinforcement
- Live algorithm toggling via MIDI CC or serial
- Mix multiple algorithms simultaneously (brushes) to compose each frame in real time

### Hardware & requirements
- Board: ESP32 (with BLE)
- LEDs: WS2812/WS2812B (matrix 12×75 per module; 900 LEDs per module)
- LED data pin: `LED_DATA_PIN 14`
- Adequate 5V power supply (multiple injection points and shared ground)

### Libraries
- FastLED
- BLEMidi (ESP32 BLE MIDI)

Install with Arduino IDE (Library Manager) or PlatformIO. Build for “ESP32 Dev Module”.

### Quick configuration (in code)
- `NUM_LEDS` = 900
- `LedsPorLinha` = 75
- `ModuloESP` = 0 top half; `1` bottom half
- BLE names: “AcordeLuz A” (ModuloESP 0) and “AcordeLuz B” (ModuloESP 1)

### How to use
1) Power the panel and ESP32; share ground between PSU and board.
2) Pair from your instrument/DAW to “AcordeLuz A/B” over BLE MIDI.
3) Send MIDI notes; visuals are rendered in real time.
4) Use MIDI CC on channel 16 to toggle algorithms and post‑processing.
5) Combine multiple active algorithms at once to create richer compositions (brushes).

### MIDI mapping
- Channel: controls are read on channel 16 (`channel == 15` internally).
- Note On: triggers the active algorithm (Note Off is ignored).
- Control Change:
  - CC 65 → Toggle Algorithm 1 (Random bars)
  - CC 66 → Toggle Algorithm 2 (Distance from center)
  - CC 67 → Toggle Algorithm 3 (Mask – variant 4)
  - CC 68 → Toggle Algorithm 4 (Mask – variant 5)
  - CC 69 → Toggle Algorithm 5 (Rings – solid)
  - CC 64 → Toggle Algorithm 6 (Rings – random)
  - CC 80 → Toggle Algorithm 7 (Test 1 – standard mask 1)
  - CC 81 → Toggle Algorithm 8 (Test 2 – standard mask 2)
  - CC 82 → Toggle Algorithm 9 (reserved)
  - CC 83 → Toggle Algorithm 10 (reserved)
  - CC 1  → Set noise seed (random intensity)
  - CC 7  → Post‑processing: value > 68 enables Fade, otherwise Drip

You can keep multiple algorithms enabled simultaneously; each CC toggles an algorithm independently. Effects accumulate on the final frame (brushes).

Note: algorithm 8 is enabled by default on boot. The main loop applies `CoresSecundarias` every frame and, based on flags, `FadeNotas` and/or `EscorreNotas`.

### Module synchronization (Serial)
Two ESP32 boards can run together (top and bottom halves). They communicate over a dedicated Serial link to keep effects in sync and share state flags when needed. The protocol is lightweight (low‑latency flag/messages), allowing both halves to compose a coherent visual frame.

### Algorithm building blocks
- Bitmasks/lookup tables for patterns: standard and per‑note masks (e.g., `MascaraPadrao1`, `MascaraPadrao2`, `MascaraAlgo4`, `MascaraAlgo5`)
- 22×22 ring masks per degree/octave for `GeraAneis` (`DesenhaAneisSolido`, `DesenhaAneisRandom`)
- Auxiliary masks (e.g., `MascaraBumbo`, `MascaraFormas`) to add rhythmic/geometric elements
- Mirroring and line duplication to build symmetry and reinforce degree/octave
- Color bias and noise seeding (via CC1) for dynamic variation

### Hand‑crafted engineering
- Physical remapping `Remap()` for zig‑zag WS2812 strips (robust linear coordinates)
- Curated mask sets (bitmasks), including patterns prototyped in Blender and optimized arrays for 900 px per module
- Custom post‑processors: `FadeNotas` (threshold/speed‑aware fade) and `EscorreNotas` (vertical drag with attenuation)
- BLE MIDI integration with dedicated callbacks (`onNoteOn`, `onNoteOff`, `onControlChange`) and CC mapping for live toggling
- Brush‑based composition: multiple active algorithms paint on the same LED buffer before `FastLED.show()`
- Serial control for debugging and quick toggling (menu and panel clear)

### Serial commands
- `1`..`10`: toggle algorithms 1 to 10
- `E`/`e`: toggle Drip
- `F`/`f`: toggle Fade
- `C`/`c`: clear panel

### Power and assembly tips
- Inject 5V along the panel; use proper gauge wiring; common ground with ESP32.
- Avoid full‑white at 100% without proper PSU; current can be very high.

### Code structure (short)
- `setup()`: init FastLED, start BLE MIDI (“AcordeLuz A/B”), blink status, register callbacks.
- `loop()`: apply post‑processing and update frame (`FastLED.show()`).
- BLE callbacks: `onNoteOn`, `onNoteOff`, `onControlChange` map MIDI to visuals and flags.
- Algorithms: `GeraBarrasRandom`, `GeraDistanciaCentro`, `GeraAneis` (solid/random), `ALGO3`, `ALGO4`, `ALGOTeste1/2`.
- Post: `FadeNotas`, `EscorreNotas`, `CoresSecundarias`.
- Physical mapping: `Remap()` handles zig‑zag strips.



