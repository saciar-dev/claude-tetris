# CLAUDE.md

Este archivo brinda guía a Claude Code (claude.ai/code) al trabajar con código en este repositorio.

## Proyecto

Tetris en JavaScript vanilla. Sin dependencias, sin proceso de build, sin package.json, sin suite de tests. Tres archivos: `index.html`, `style.css`, `game.js`.

## Ejecución

Simplemente abre `index.html` en un navegador, o sírvelo de forma estática:

```bash
python3 -m http.server 8000   # o: npx serve .
```

No existen comandos de build, lint ni test en este repositorio.

## Arquitectura

Toda la lógica del juego vive en `game.js` (~300 líneas, archivo único, sin módulos):

- **Tablero**: matriz de `ROWS × COLS` (20×10); `0` = celda vacía, `1–7` = índice de color de la pieza. `BLOCK` = 30px por celda, debe coincidir con `width`/`height` del canvas en `index.html` si se modifica.
- **Piezas**: el array `PIECES` contiene los 7 tetrominós como matrices cuadradas; `COLORS` mapea el índice de pieza a un color. La rotación es `rotateCW` (transposición + inversión), y `tryRotate` aplica los desplazamientos de wall-kick `[0, -1, 1, -2, 2]` hasta encontrar una posición sin colisión.
- **Colisión**: `collide(shape, ox, oy)` verifica los límites del tablero y los bloques ya fijados.
- **Bucle de juego**: `loop(ts)` se ejecuta vía `requestAnimationFrame`, acumula el tiempo transcurrido en `dropAccum` y baja la pieza una fila cuando `dropAccum >= dropInterval`.
- **Fijado y puntuación**: `lockPiece` → `merge` (escribe la pieza en `board`) → `clearLines` (recorre de abajo hacia arriba, elimina filas completas e inserta filas vacías arriba). La puntuación usa `LINE_SCORES = [0,100,300,500,800]` × `level`; el hard drop suma 2 pts/celda, el soft drop 1 pt/fila. El nivel sube cada 10 líneas; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Pieza fantasma**: `ghostY()` proyecta la pieza actual hacia abajo hasta su fila de aterrizaje; se dibuja con `globalAlpha = 0.2`.
- **Renderizado**: `draw()` limpia y redibuja la grilla, el tablero, el fantasma y la pieza actual en cada frame sobre el canvas `#board`; `drawNext()` renderiza la pieza de vista previa en el canvas separado `#next-canvas`.
- Todas las referencias al DOM/canvas y el estado mutable del juego (`board`, `current`, `next`, `score`, etc.) son variables globales a nivel de módulo — no hay un contenedor de estado ni clases.

El manejo de teclado se hace con un único listener `keydown` al final de `game.js` (flechas para mover/rotar/soft-drop, Espacio para hard drop, P para pausar). El click en `restartBtn` llama a `init()` para reiniciar todo el estado.

Al cambiar `COLS`, `ROWS` o `BLOCK`, también hay que actualizar los atributos `width`/`height` del canvas `#board` en `index.html` para que coincidan (`COLS × BLOCK`, `ROWS × BLOCK`).
