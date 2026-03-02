Original prompt: Eliminar bandas laterales negras en móvil mediante: (1) acortar la tierra al mínimo y fusionarla visualmente con el fondo de los botones, (2) extender el fondo hasta arriba y superponer el título sobre el canvas.

## Cambios implementados (branch: claude/admiring-merkle)

### 1. Título superpuesto al canvas-wrap
- `<h1>` y `#subtitle` movidos DENTRO de `#canvas-wrap` dentro de un `<div id="title-overlay">`
- CSS: `#title-overlay { position: absolute; top: 0; left: 0; right: 0; z-index: 10; pointer-events: none; }`
- `#canvas-wrap` ahora tiene `position: relative`
- Libera ~40–45px de altura vertical para el canvas en mobile

### 2. Canvas reducido: CV_H 960→780, GROUND_Y 820→760
- Tierra recortada al mínimo (20px de tierra visible en canvas, solo franja de pasto)
- La tierra del canvas coincide visualmente con el fondo marrón `#2e1506` del HUD/controles
- `drawSky()` ahora calcula las bandas del gradiente de forma proporcional a `GROUND_Y` (en vez de píxeles hardcodeados)
- `align-items: flex-end` en canvas-wrap: la base del canvas toca el HUD directamente, el espacio libre queda arriba (donde flota el título)

### 3. Resultado en mobile (390px ancho)
- Antes: canvas 290px ancho → bandas ~50px cada lado
- Después: canvas 390px ancho → bandas ~0px ✅

### Comportamiento por dispositivo
- **Mobile portrait (≤390px)**: canvas llena el ancho completo, sin bandas ✅
- **Desktop landscape**: persisten bandas laterales (inherente a un canvas portrait en pantalla landscape; fuera del alcance de este fix)

## Estado actual
- Sin errores de consola
- Juego funcional (disparo, impacto, fallo, HUD)
- Playwright tests corriendo con NODE_PATH=/Users/juanchi/.codex/skills/develop-web-game/node_modules

## TODOs / sugerencias para el siguiente agente
- Considerar añadir soporte landscape en mobile (`@media (orientation: landscape)`) con canvas horizontal diferente
- El viewport height exacto del iPhone con Safari depende del estado de las barras de navegación; el fix funciona con dvh
- Los assets de estrellas están hardcodeados en píxeles hasta y≈755, algunos quedarán bajo el suelo (GROUND_Y=760) pero drawGround() los tapa
