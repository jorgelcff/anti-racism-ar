# Anti-Racism AR — WebAR com A‑Frame + AR.js

Este projeto é uma cena WebAR que utiliza A‑Frame 1.2.0 e AR.js (aframe-ar.js) para exibir um modelo 3D em cima de um marcador `.patt`. Ele inclui:

- Detecção de marcador e feedback visual.
- Rotação suave do modelo 3D quando o marcador é detectado (15°/s no eixo Y).
- Estrutura de fala (áudio) com componente customizado, preparada para futura sincronização labial.
- Botão de fallback para tocar áudio manualmente (caso o navegador bloqueie autoplay).
- Menu de 3 pontos (⋮) com painel que permite:
  - Carregar dinamicamente um arquivo `.glb` e um áudio opcional.
  - Ajustar escala, posição (X/Y/Z) e rotação Y do modelo.
  - Acionar motions (Idle/Acenar/Parar) e configurar mapeamentos de clipes/bones/morphs.

## Arquivos principais

- `index.html` — Cena A‑Frame + lógica AR.js + componentes customizados + UI de upload.
- `pattern-marker.patt` — Arquivo do marcador usado pela cena.
- `model1.glb` — Modelo 3D padrão (pode ser substituído via upload).
- `audio1.m4a` (opcional) — Áudio padrão da personagem (pode ser substituído via upload).

## Funcionalidades

- **Rotação contínua**: Quando o marcador é detectado (`markerFound`), o modelo (`#modelo-personagem`) ativa uma rotação suave no eixo Y, controlada pelo componente `rotating-when-visible`. A rotação é pausada em `markerLost`.
- **Fala da personagem**: O componente `character-speech` prepara a reprodução de áudio e um método placeholder `syncMouth()` para futura sincronização labial com WebAudio/morph targets/bones. A reprodução do áudio é iniciada manualmente pelo botão "Ouvir fala".
- **Fallback de áudio**: Como navegadores podem bloquear autoplay, ao tentar tocar áudio e falhar, o botão "Ouvir fala" aparece para o usuário autorizar a reprodução manualmente.
- **Upload de conteúdo**: É possível enviar um novo `.glb` e um áudio opcional pela UI. O GLB substitui o modelo atual via Blob URL; o áudio substitui o `src` do elemento `<audio id="fala1">`.
- **Motion adaptável**: Componente `character-motion` que usa clipes embutidos no GLB (via AnimationMixer) quando disponíveis, ou aplica adapters mínimos via bones/morphs quando não houver clipes. API simples: `playMotion`, `stopAll`, `setMorph`, `setBoneRotation`, `setConfig`.
- **Lip-sync básico (opcional)**: Quando habilitado no menu, o áudio é analisado via WebAudio (RMS) e mapeado para o morph `MouthOpen`, animando a boca durante a fala.
- **Indicadores de estado**: Painel de status mostra o estado do áudio (Reproduzindo/Parado/Bloqueado/Erro) e eventos do marcador.

## Como executar localmente

É recomendado servir os arquivos via um servidor local para evitar problemas de
permissões/CORS. Com Python instalado, dentro da pasta do projeto:

```powershell
python -m http.server 8000
```

Depois, abra no navegador: `http://localhost:8000`.

Permita o acesso à câmera quando solicitado e aponte a webcam para o `pattern-marker.patt`.

## Uso da cena

1. **Detecção do marcador**: Ao detectar, um retângulo verde aparece e some após 2 segundos. O modelo começa a girar suavemente.
2. **Reprodução do áudio**: Após os 2s, o botão "Ouvir fala" aparece no canto inferior direito. Clique para tocar o áudio.
3. **Menu (⋮)**: Clique no botão de três pontinhos no canto superior direito para abrir o painel:

- Upload de **Modelo (.glb)** e **Áudio (opcional)**.
- **Transformar Modelo**: Escala, Posição X/Y/Z, Rotação Y.
- **Motion**: Idle, Acenar, Parar.
- **Mapeamento**: renomeie `clips` (Idle/Wave/Talk), `bones` (Jaw/RightHand) e `morphs` (MouthOpen) conforme seu GLB; clique em “Aplicar Mapeamento”.
- **Lip-sync**: marque a opção “Lip-sync” para ativar animação de boca baseada no áudio.

4. **Perda do marcador**: A rotação é pausada, o áudio é interrompido e o botão é ocultado automaticamente.

## Componentes customizados

### `rotating-when-visible`

- **Schema**: `speed` (number), graus por segundo. Default: `15`.
- **Métodos**: `start()`, `stop()`.
- **Comportamento**: Em `tick()`, incrementa o ângulo Y: `speed * (timeDelta/1000)` com wrap em `360`.

Uso no HTML:

- No elemento do modelo: `rotating-when-visible="speed: 15"`
- A ativação é feita em `markerFound` chamando `start()` e a pausa em `markerLost` chamando `stop()`.

### `character-speech`

- **Schema**: `audioSelector` (selector). Default: `#fala1`.
- **Métodos**: `playSpeech()`, `stopSpeech()`, `syncMouth()` (placeholder para animação labial).
- **Comportamento**:
  - Tenta tocar o áudio do elemento selecionado. Se o navegador bloquear autoplay, o botão "Ouvir fala" é exibido.
  - Em `timeupdate` do áudio, chama `syncMouth()` (TODO: implementar análise do som e animação da boca).

Uso no HTML:

- No elemento do modelo: `character-speech="audioSelector: #fala1"`
- Reprodução do áudio é feita ao clicar no botão "Ouvir fala".

### `character-motion`

- **Função**: gerencia animações do modelo com dois níveis: clipes embutidos (AnimationMixer) e adapters mínimos para `bones`/`morphs`.
- **Config padrão**: `clips: { idle: 'Idle', wave: 'Wave', talk: 'Talk' }`, `bones: { jaw: 'Jaw', handR: 'RightHand' }`, `morphs: { mouthOpen: 'MouthOpen' }`.
- **API**:
  - `playMotion(nome, { loop, fade })`: executa clipe (ou adapter) com crossfade.
  - `stopMotion(nome)` e `stopAll()`: interrompe ações.
  - `setMorph(chave, valor)`: ajusta influência de morphs (0..1), ex.: `mouthOpen`.
  - `setBoneRotation(chave, {x,y,z})`: ajusta rotação de bones.
  - `setConfig({ clips, bones, morphs })`: aplica mapeamento customizado.
- **Uso**:
  - No HTML: adicionar `character-motion` ao `a-gltf-model`.
  - Em `markerFound`, iniciar `idle` se existir; em `markerLost`, `stopAll()`.

## UI e elementos

- `#detectado`: retângulo verde de confirmação da detecção do marcador; removido 2s após `markerFound`.
- `#btn-play-fala`: botão discreto para iniciar a fala manualmente (inferior direito). É ocultado quando o áudio começa a tocar ou quando o marcador é perdido.
- Botão **⋮**: abre/fecha o painel de menu.
- **Uploads**: enviar `.glb` e áudio (opcional).
- **Transformar Modelo**: escala (slider), posição X/Y/Z (inputs), rotação Y (slider).
- **Motion**: botões Idle/Acenar/Parar.
- **Mapeamento**: campos para nomes de `clips`, `bones` e `morphs`; botão “Aplicar Mapeamento”.
- `#msg`: área de mensagens de status (ex.: "Modelo carregado", "Falha ao carregar", etc.).
- `#audioState`: indicador textual do estado do áudio.

## Troca de modelo (GLB) via upload

- Aceita apenas `.glb`.
- Ao selecionar o arquivo, o código cria uma Blob URL (`URL.createObjectURL`) e define no `src` do `a-gltf-model`.
- Blob URLs anteriores são revogadas (`URL.revokeObjectURL`) para evitar vazamentos.

## Troca de áudio via upload

- Aceita formatos suportados pelo navegador (ex.: `m4a`, `mp3`).
- Ao selecionar o arquivo, o código cria uma Blob URL e a usa como `src` do elemento `<audio id="fala1">`.
- O áudio é reproduzido apenas quando o usuário clica em "Ouvir fala".

## Lip-sync básico (opcional)

- Habilite a opção “Lip-sync” no menu (⋮).
- Ao reproduzir o áudio, o sistema analisa o sinal com WebAudio e calcula o RMS a cada frame.
- O valor RMS é mapeado para o morph `MouthOpen`, abrindo/fechando a boca de forma suave.
- Para resultados melhores, ajuste o mapeamento de `morphs` e use modelos com targets apropriados.

## Marcador

- O projeto usa `pattern-marker.patt`. Você pode baixar/imprimir o marcador pelo link na tela.
- O `a-marker` está configurado com `emitevents="true"` para usar `markerFound`/`markerLost`.
- Além disso vocë pode verificar como criar o seu próprio em https://medium.com/arjs/how-to-create-your-own-marker-44becbec1105

## Dicas e solução de problemas

- **Câmera bloqueada**: Se a câmera não abrir, verifique permissões do navegador e sistema.
- **Autoplay bloqueado**: Se o som não tocar automaticamente, use o botão "Ouvir fala".
- **Formato GLB**: Se o modelo não aparecer, confirme que é `.glb` válido (GLTF binário). `.gltf` com recursos externos não é suportado por este uploader.
- **Escala/posição**: O modelo tem escala padrão `0.4`; se seu GLB for muito grande/pequeno, ajuste manualmente os atributos `scale`/`position` no HTML ou peça a implementação de controles de escala.
- **Servidor local**: Evite abrir o `index.html` direto do sistema de arquivos; use um servidor (ex.: `python -m http.server`).

## Como customizar

- Ajuste a velocidade da rotação: `rotating-when-visible="speed: 10"` ou outro valor.
- Troque o seletor do áudio: `character-speech="audioSelector: #meu-audio"` e atualize o elemento `<audio>` correspondente.
- Altere o comportamento de UI (tempo de desaparecimento do retângulo, posição/estilo do botão) no CSS/JS do `index.html`.

## Licenças e créditos

- A‑Frame e AR.js são projetos open-source. Confira suas respectivas licenças nos repositórios oficiais.
- Modelos/áudios utilizados devem respeitar as licenças de uso.
