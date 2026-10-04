# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Tetris en HTML5 Canvas + JS vanilla. Sin build, sin tests, sin linter, sin `package.json`. Comentarios, UI y README en español; mantener.

## Ejecutar

Abrir `index.html` en el navegador, o servir desde esta carpeta: `npx serve .` / `python3 -m http.server 8000`.

## Arquitectura

Tres archivos: `index.html` (canvas `#board` 300x600, `#next-canvas`, HUD, overlay), `style.css` (tema oscuro retro), `game.js` (toda la lógica, estado en variables globales de módulo).

- Tablero: matriz `ROWS x COLS`; celda `0` (vacía) o índice 1–7 en `COLORS`. `PIECES` son matrices cuadradas; `rotateCW` = transponer + invertir filas. `tryRotate` prueba wall kicks `[0, -1, 1, -2, 2]`.
- `collide(shape, ox, oy)` es la única comprobación de colisión: movimiento, rotación, ghost y spawn. Colisión al hacer spawn llama `endGame()`.
- Flujo de pieza: `lockPiece` = `merge` → `clearLines` → `spawn`. `clearLines` actualiza `level` y `dropInterval` (`max(100, 1000 - (level - 1) * 90)`; nivel sube cada 10 líneas).
- Puntos: `LINE_SCORES[n] * level`; hard drop +2 por celda, soft drop +1 por fila.
- Bucle `loop(ts)` con `requestAnimationFrame`; acumula en `dropAccum` contra `dropInterval`. `init()` también sirve de reinicio (botón `#restart-btn`).

## Gotchas

- Tamaño del canvas hardcodeado en `index.html`: si cambia `COLS`, `ROWS` o `BLOCK` en `game.js`, actualizar `width`/`height` de `<canvas id="board">` (`COLS * BLOCK` x `ROWS * BLOCK`).
- `endGame()` hace `cancelAnimationFrame(animId)`, pero cuando se dispara desde `loop` (vía `lockPiece` → `spawn`), `loop` termina con `animId = requestAnimationFrame(loop)` y el bucle sigue vivo tras game over. Los controles se bloquean con `gameOver`, pero el bucle sigue acumulando y llamando `lockPiece`; tenerlo en cuenta al tocar el ciclo de vida del bucle.
- Pausa: `togglePause` cancela el frame y al reanudar reinicia `lastTime` para evitar un `dt` enorme.
