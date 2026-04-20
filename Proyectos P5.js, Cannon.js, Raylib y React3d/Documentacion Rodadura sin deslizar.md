# Documentación — Rodadura sin Deslizar
## Proyectos 7 y 8 · Universidad Cuauhtémoc · Métodos Numéricos
**Integrantes:** Víctor · Jesús · Diego · Mario  
**Referencia:** Dobre, A. & Bhatt, D. (2012). *Physics for JavaScript Games, Animation & Simulations*. Cap. 13, pp. 336–355.

---

## Proyecto 7 — Rodadura sin Deslizar (Raylib)

### ¿Qué hace?

Simula una esfera, cilindro o disco rodando por un plano inclinado en 2D.
La física es la del libro (Cap. 13): aceleración analítica que depende del
momento de inercia del objeto, fricción estática que produce la rotación y
la condición `v = r·ω` que verifica rodadura pura. Todo se dibuja con
**Canvas 2D puro** en estilo visual Raylib — paleta de colores plana,
tipografía monospace y HUD lateral con datos en tiempo real.

### Controles

| Acción | Efecto |
|--------|--------|
| `ESPACIO` | Iniciar / Pausar la simulación |
| `R` | Reiniciar desde el inicio del plano |
| `↑ / ↓` | Aumentar / reducir el ángulo del plano |
| `A / D` | Reducir / aumentar el deslizamiento (slip) |
| `1` | Cambiar a esfera — `I = 2/5·mr²` |
| `2` | Cambiar a cilindro — `I = 1/2·mr²` |
| `3` | Cambiar a disco — `I = 1/2·mr²` |

### Física implementada

Cada frame se calculan aceleración, velocidad angular y fricción con las
fórmulas del libro:

```
a     = g·sin(θ) / (1 + I/mr²)
v    += a · dt
ω     = (v / r) · slip
x    += v · dt
```

El parámetro `slip` modifica la relación entre velocidad lineal y angular.
Con `slip = 1.0` se cumple exactamente `v = r·ω` (rodadura pura). Valores
distintos simulan deslizamiento en hielo (`slip > 1`) o barro (`slip < 1`).

La fricción que mantiene la rodadura:

```
f = m·g·sin(θ) / (1 + mr²/I)
```

### Energías mostradas en el HUD

```
E_traslacion = 1/2 · m · v²
E_rotacion   = 1/2 · I · ω²
E_total      = E_traslacion + E_rotacion
```

### Vectores de fuerza visuales

Se dibujan tres flechas sobre el objeto en tiempo real:

| Flecha | Color | Fuerza |
|--------|:-----:|--------|
| Gravedad | Rojo | `m·g` hacia abajo |
| Normal | Verde | `m·g·cos(θ)` perpendicular al plano |
| Fricción | Amarillo | `f` cuesta arriba del plano |

### Gráfica de velocidad

En la parte inferior del HUD se dibuja en tiempo real la comparación entre
`v` (azul) y `r·ω` (naranja). Cuando se superponen exactamente, se cumple
la condición de rodadura pura del libro.

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| Canvas 2D (Raylib style) | Nativo | Render 2D, HUD, gráfica de velocidad |

---

## Proyecto 8 — Rodadura sin Deslizar (React3D)

### ¿Qué hace?

La misma simulación física del Proyecto 13 pero en **3D completo** con
interfaz construida en **React** y escena renderizada con **Three.js**.
La cámara es orbital. El panel lateral es un componente React con sliders
y botones que actualizan el estado en tiempo real sin recargar la escena.

### Arquitectura del proyecto

```
App (React)
 ├── ThreeCanvas  ← Three.js: renderer, scene, luces, mesh, flechas
 └── Panel        ← React: sliders, datos, botones, ecuaciones
```

El estado físico (`v`, `ω`, `x`, `angleRot`) vive en React con `useState`
y se comparte entre los dos componentes. Three.js lee ese estado en cada
frame de animación para posicionar y rotar el mesh.

### Física implementada

Idéntica al Proyecto 13. El loop de animación de React usa
`requestAnimationFrame` para actualizar el estado cada frame:

```javascript
const a      = g·sin(θ) / (1 + I/mr²)
newV         = v + a · dt
newOmega     = (newV / r) · slip
newX         = x + newV · dt
newAngleRot  = angleRot + newV · dt / r
```

### Integración React → Three.js

Three.js no gestiona el estado — solo dibuja lo que React le indica:

```javascript
// React actualiza el estado
setState({ v: newV, omega: newOmega, x: newX, angleRot: newRot })

// Three.js lo lee en el loop de animación
ball.position.copy(slopeToWorld(state.x, theta, params.radius))
ball.rotation.z = -state.angleRot   // esfera
ball.rotation.x =  state.angleRot   // cilindro / disco
```

### Tipos de objeto en 3D

| Tipo | Mesh Three.js | Material | Color |
|------|--------------|----------|:-----:|
| Esfera | `SphereGeometry` con 6 toroides decorativos | `MeshStandardMaterial` metalness 0.5 | Cian |
| Cilindro | `CylinderGeometry` rotado 90° | `MeshStandardMaterial` roughness 0.4 | Naranja |
| Disco | `CylinderGeometry` plano rotado 90° | `MeshStandardMaterial` roughness 0.4 | Verde |

Los tres tienen una franja roja (`TorusGeometry`) para hacer visible la rotación.

### Panel React — controles

| Control | Efecto |
|---------|--------|
| Slider **Ángulo** | Modifica la inclinación y reinicia la simulación |
| Slider **Masa** | Cambia `m` — afecta fricción y energías |
| Slider **Radio** | Cambia `r` — reconstruye el mesh |
| Slider **Deslizamiento** | Igual que en el Proyecto 13 |
| Botones **Esfera / Cilindro / Disco** | Cambia el tipo y reconstruye el mesh |
| **▶ Iniciar / ⏸ Pausar** | Controla el loop de física |
| **↺ Reset** | Vuelve al estado inicial |
| **Vectores ON/OFF** | Muestra u oculta las flechas 3D |

### Vectores 3D

Las flechas se construyen con `THREE.CylinderGeometry` (tronco) +
`THREE.ConeGeometry` (punta) y se escalan y rotan cada frame según
los valores de fuerza calculados por React:

| Flecha | Color | Representa |
|--------|:-----:|-----------|
| Gravedad | Rojo `#ff2244` | `m·g` hacia abajo |
| Normal | Verde `#22dd44` | `m·g·cos(θ)` perpendicular al plano |
| Fricción | Amarillo `#ffdd00` | Fricción de rodadura cuesta arriba |

### Diferencias clave entre proyectos 13 y 14

| Aspecto | Proyecto 13 (Raylib) | Proyecto 14 (React3D) |
|---------|----------------------|-----------------------|
| Dimensión | 2D | 3D con cámara orbital |
| Render | Canvas 2D nativo | Three.js WebGL |
| UI | HUD dibujado sobre canvas | Componente React independiente |
| Estado | Variables globales JS | `useState` de React |
| Sliders | `keydown` del teclado | Sliders HTML en panel React |
| Gráfica velocidad | Sí — dibujada en canvas | No |

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| React | 18.2.0 | Gestión de estado, UI del panel |
| Three.js | r128 | Render 3D, meshes, luces, sombras |
| Babel | 7.23.2 | Transpilación de JSX en el navegador |

---

*Universidad Cuauhtémoc · Métodos Numéricos · Víctor · Jesús · Diego · Mario*
