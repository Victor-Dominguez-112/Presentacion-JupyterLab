# Documentación — Sistema Solar 3D
## Proyectos 1 y 2 · Universidad Cuauhtémoc · Métodos Numéricos
**Integrantes:** Víctor Domínguez · Mario Rangel · Jesús Macias · Diego Vázquez

---

## Proyecto 1 — Sistema Solar 3D (Raylib)

### ¿Qué hace?

Simula un sistema solar en 3D con el Sol en el centro y cuatro planetas orbitando
a su alrededor. Cada planeta sigue la **Ley de Gravitación Universal** de Newton:
la fuerza que el Sol ejerce sobre cada planeta apunta siempre hacia el centro y
su magnitud depende de la distancia. El resultado es una órbita elíptica estable.

La escena incluye un fondo de 3000 estrellas generadas aleatoriamente, anillos de
órbita para cada planeta y una estela que va trazando la trayectoria recorrida.

### Controles

| Acción | Efecto |
|--------|--------|
| Arrastrar el mouse | Rotar la cámara orbital |
| Scroll | Acercar / alejar |
| **+ Planeta** | Agrega un planeta nuevo en posición y tamaño aleatorio |
| **Reset** | Vuelve a los 4 planetas iniciales |
| **Pausa** | Congela la simulación |
| **Estela** | Activa o desactiva el rastro de trayectoria |

### Física implementada

La aceleración de cada planeta se calcula cada frame apuntando hacia el Sol:

```
acc = −pos / |pos|  ×  (G × M_sol / |pos|²)
vel += acc × dt
pos += vel × dt
```

Esto es integración de Euler sobre la Ley de Gravitación Universal. La velocidad
orbital inicial se calcula para que la órbita sea circular:

```
v_orbital = √(G × M_sol / r)
```

### Estructura del código

- **Renderer / Scene / Camera** — configuración inicial de Three.js
- **Estrellas** — 3000 puntos aleatorios en `BufferGeometry`
- **`makePlanet(r, theta, incl, size, color)`** — crea un planeta con su mesh,
  anillo de órbita, trail y estado físico inicial
- **`stepPhysics()`** — aplica la gravedad y actualiza posición de cada planeta
- **`animate()`** — loop principal: física → cámara → render

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| Raylib (Three.js) | 0.160.0 | Render 3D, geometrías, luces |

---

## Proyecto 2 — Sistema Solar 3D (Raylib + React3D)

### ¿Qué hace?

Es una versión extendida del Proyecto 7 con un **panel de control lateral** completo.
Agrega sliders para modificar la física en tiempo real, checkboxes para activar
elementos visuales y una lista de cuerpos celestes con su distancia al Sol actualizada
en cada frame.

### Controles adicionales respecto al Proyecto 1

| Control | Efecto |
|---------|--------|
| Slider **Gravedad G** | Modifica la constante gravitacional en tiempo real |
| Slider **Masa Sol** | Aumenta o reduce la masa del Sol |
| Slider **Velocidad** | Multiplica la velocidad de simulación (1× a 10×) |
| Slider **Estela largo** | Controla cuántos puntos conserva el rastro |
| Checkbox **Anillos órbita** | Muestra u oculta los anillos de referencia |
| Checkbox **Vectores velocidad** | Muestra flechas `ArrowHelper` sobre cada planeta |
| Checkbox **Grilla** | Muestra u oculta la grilla de referencia del plano |
| Lista de cuerpos | Muestra nombre y distancia al Sol de cada planeta en vivo |

### Diferencias técnicas con el Proyecto 1

**Velocidad de simulación variable:** el loop ejecuta `simSpeed` pasos de física
por frame en lugar de uno solo:

```javascript
const steps = Math.max(1, Math.round(simSpeed));
for (let s = 0; s < steps; s++) {
    stepPhysics();
}
```

**Vectores de velocidad:** usa `THREE.ArrowHelper` para dibujar la dirección y
sentido del vector velocidad sobre cada planeta. Se actualizan cada frame con
`arrow.setDirection(vel.normalize())`.

**Contador de FPS:** mide el tiempo entre frames y actualiza el panel lateral
cada medio segundo.

**Panel React3D:** el panel lateral es HTML puro con sliders y checkboxes que
modifican variables del módulo ES6 mediante `Object.defineProperty` para exponer
las variables internas del módulo al scope global de `window`.

### Estructura del código

- **`makePlanet(..., name)`** — igual que el Proyecto 7 pero agrega flecha de
  velocidad y nombre visible en el panel
- **`stepPhysics()`** — ejecuta múltiples pasos por frame según `simSpeed`
- **`updateList()`** — regenera el HTML del panel con la distancia actual de
  cada planeta
- **`animate()`** — loop con contador de FPS y actualización del panel cada 8 frames

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| Raylib (Three.js) | 0.160.0 | Render 3D, geometrías, luces, flechas |
| React3D (HTML/CSS) | — | Panel lateral de controles interactivos |

---

