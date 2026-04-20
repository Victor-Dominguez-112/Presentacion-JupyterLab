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

# Documentación — Péndulo Cónico Giratorio
## Proyectos 3 y 4 · Universidad Cuauhtémoc · Métodos Numéricos

## Proyecto 3 — Péndulo Cónico (p5.js)

### ¿Qué hace?

Simula un péndulo cónico: una masa (bob) colgada de un pivote por una cuerda
que gira describiendo un cono imaginario. El bob traza un círculo horizontal
continuo mientras la cuerda permanece tensa en un ángulo fijo respecto a la
vertical. Incluye un efecto de **precesión de Foucault** que hace que el plano
de rotación derive lentamente con el tiempo, igual que un péndulo de Foucault real.

La visualización es 3D proyectada manualmente: no usa WebGL ni Three.js, sino
una función `project()` propia que convierte coordenadas 3D a píxeles 2D con
perspectiva y elevación de cámara controlable.

### Controles

| Control | Efecto |
|---------|--------|
| Slider **Longitud L** | Cambia la longitud de la cuerda |
| Slider **Ángulo cónico θ** | Abre o cierra el cono de rotación |
| Slider **Vel. angular ω** | Más rápido o más lento |
| Slider **Efecto Foucault** | Intensidad de la precesión del plano |
| Slider **Elevación vista** | Cambia el ángulo de la cámara hacia arriba/abajo |
| Arrastrar mouse | Rotar la cámara horizontalmente |
| **⏸ Pausa** | Congela la simulación |
| **〰 Estela** | Activa o desactiva el rastro de la trayectoria |
| **🔄 Sentido** | Invierte la dirección de rotación |
| **🗑 Limpiar** | Borra la estela y reinicia el ángulo |

### Física implementada

La posición del bob se calcula geométricamente a partir del ángulo de rotación
acumulado cada frame:

```
R  = L · sin(θ)         ← radio del círculo horizontal
dy = L · cos(θ)         ← descenso vertical de la cuerda

phi      += ω · dt      ← avanza el ángulo de rotación
phiFouc  += fouc · dt   ← precesión del plano (Foucault)

bx = R · cos(phi + phiFouc)
bz = R · sin(phi + phiFouc)
by = dy
```

La velocidad instantánea del bob se calcula por diferenciación:

```
vx = −R · ω · sin(totalPhi)
vz =  R · ω · cos(totalPhi)
speed = √(vx² + vz²)
```

No hay integración de fuerzas: la trayectoria circular es impuesta
geométricamente, lo que garantiza un movimiento perfectamente estable.

### Proyección 3D manual

La función `project(x3, y3, z3)` aplica dos rotaciones sucesivas:
primero la rotación horizontal de cámara (`camAngle`) y luego la
elevación vertical (`elev`), seguido de una proyección perspectiva simple:

```
dep = FOV / (FOV + profundidad + 900)
sx  = cx + rx · dep
sy  = cy + ry · dep
```

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| p5.js | 1.9.0 | Dibujo 2D del canvas, animación, eventos de mouse |

---

## Proyecto 4 — Péndulo Cónico (p5.js + Cannon.js)

### ¿Qué hace?

Es la versión con física real del mismo péndulo cónico. En lugar de calcular
la trayectoria geométricamente, **Cannon.js** simula el bob como un cuerpo
rígido con masa, velocidad y fuerzas reales. La cuerda se modela como una
restricción de distancia fija entre el bob y el ancla. La gravedad, la
inercia y las colisiones son calculadas por el motor de física en cada frame.

La diferencia visual más notable es que el bob puede **perder energía**
(amortiguación), recibir **impulsos externos** y verse afectado por la
**fuerza de Coriolis** implementada manualmente sobre el cuerpo de Cannon.

### Diferencias clave con el Proyecto 3

| Aspecto | Proyecto 9 (p5.js puro) | Proyecto 10 (Cannon.js) |
|---------|------------------------|------------------------|
| Física | Geométrica, perfecta | Motor de cuerpo rígido |
| Trayectoria | Siempre circular | Puede variar con impulsos |
| Amortiguación | No existe | `linearDamping` real |
| Impulsos | No soportado | Botón 💥 aplica fuerza aleatoria |
| Coriolis | Offset angular acumulado | Fuerza de Coriolis calculada frame a frame |
| Energía cinética | No se mide | Se muestra en el HUD en tiempo real |

### Controles adicionales

| Control | Efecto |
|---------|--------|
| Slider **Amortiguación** | `linearDamping` de Cannon: la órbita pierde energía gradualmente |
| **💥 Impulso** | Aplica una fuerza aleatoria al bob para perturbarlo |
| **🔄 Reset** | Reconstruye el mundo Cannon con los parámetros actuales |

### Física de Cannon.js

El mundo se construye con `rebuild()` cada vez que cambia un slider:

```javascript
world.gravity.set(0, -9.81, 0)

// Bob: cuerpo esférico con masa 1 kg
bobBody = new CANNON.Body({ mass: 1.0, linearDamping: getDamp() })
bobBody.position.set(R, -dy, 0)
bobBody.velocity.set(0, 0, vt)   // velocidad tangencial inicial

// Cuerda: restricción de distancia fija
ropeConstraint = new CANNON.DistanceConstraint(bobBody, anchorBody, Lm)
```

### Fuerza de Coriolis (Efecto Foucault)

Se aplica cada frame directamente sobre el cuerpo de Cannon mediante
la fórmula `F = −2m(Ω × v)` con Ω vertical:

```javascript
// Ω × v con Ω = (0, fouc, 0)
fx =  2 · m · fouc · vz
fz = -2 · m · fouc · vx
bobBody.applyForce(new CANNON.Vec3(fx, 0, fz), bobBody.position)
```

Esto produce una deriva real del plano de oscilación, idéntica al
comportamiento físico de un péndulo de Foucault.

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| p5.js | 1.9.0 | Dibujo 2D, animación, eventos de mouse |
| Cannon.js | 0.6.2 | Motor de física 3D: gravedad, restricciones, fuerzas |

---

# Documentación — Torque y Dinámica Rotacional 3D
## Proyectos 5 y 6 · Universidad Cuauhtémoc · Métodos Numéricos

---

## Proyecto 5 — Torque 3D (p5.js WEBGL)

### ¿Qué hace?

Tres simulaciones de dinámica rotacional en 3D dentro de un solo archivo,
organizadas en tabs. Todo el render y toda la física son calculados por
**p5.js en modo WEBGL** — sin ningún motor externo. La cámara es orbital:
arrastrar para rotar, scroll para zoom.

### Tabs

| Tab | Simulación | Física |
|-----|-----------|--------|
| 1 | Cubo 3D con torque | `τ = I·α` sobre eje X, Y, Z o los tres |
| 2 | Tres turbinas 3D | `α = τ/I` — diferente inercia, diferente respuesta |
| 3 | Bola rodando en plano inclinado | Fórmula analítica del libro `f = mg·sin(θ)/(1+mr²/I)` |

### Tab 1 — Cubo con Torque (pp. 345–347)

Cubo rígido con wireframe y ejes XYZ visibles que gira bajo un torque
aplicado sobre el eje seleccionado. Incluye amortiguamiento angular que
lleva al cubo a una velocidad angular terminal:

```
torque_neto = τ − k · ω
α  = torque_neto / I
ω += α · dt
θ += ω · dt
```

Velocidad terminal analítica: `ω_terminal = τ / k`

### Tab 2 — Turbinas 3D (pp. 347–349)

Tres turbinas con poste, hub y tres paletas cada una. Sus momentos de
inercia son `I = 1`, `I = 5` e `I = 81`. Clic sostenido en el viewport
aplica el torque máximo configurado. La diferencia de arranque y frenado
entre turbinas demuestra visualmente que `T = Iα`.

### Tab 3 — Rodadura sin Deslizar (pp. 350–354)

La física se calcula con las fórmulas analíticas del libro directamente
en p5.js. No hay motor externo: la fricción, la aceleración y la
aceleración angular se calculan frame a frame:

```
f     = mg·sin(θ) / (1 + mr²/I)
accel = g·sin(θ)  / (1 + I/mr²)
α     = accel / r
```

La esfera tiene meridianos dibujados para hacer visible la rotación.
Una flecha roja muestra el vector velocidad en tiempo real.

### Controles comunes

| Acción | Efecto |
|--------|--------|
| Arrastrar mouse | Rotar la cámara orbital |
| Scroll | Acercar / alejar |
| Sliders del panel | Modificar parámetros en tiempo real |
| ↺ Reset | Reinicia la simulación activa |

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| p5.js WEBGL | 1.9.0 | Render 3D, física, cámara orbital, luces |

---

## Proyecto 6 — Torque 3D (p5.js WEBGL + Cannon.js)

### ¿Qué hace?

Mismas tres simulaciones del Proyecto 11, pero la **Tab 3 (Rodadura)**
usa **Cannon.js** como motor de física real en lugar de las fórmulas
analíticas. Las Tabs 1 y 2 son idénticas al Proyecto 11.

### Diferencia clave: Tab 3

| Aspecto | Proyecto 11 (p5.js puro) | Proyecto 12 (Cannon.js) |
|---------|--------------------------|-------------------------|
| Física | Fórmulas analíticas del libro | Motor de cuerpo rígido 3D |
| Fricción | Calculada con `f = mg·sin(θ)/(1+mr²/I)` | Definida como `ContactMaterial` |
| Rotación | Actualizada con `angZ += angAcc·dt` | Quaternion de Cannon sincronizado a p5 |
| Rastro | No disponible | Línea 3D con fade acumulada |
| Vectores | Solo velocidad (flecha roja) | Velocidad (roja) + eje angular (cian) |

### Cómo funciona la integración Cannon → p5.js

Cannon.js calcula la posición y rotación del cuerpo en cada frame.
p5.js solo lee esos valores y los aplica al mesh:

```javascript
// Cannon avanza la física
cannonWorld.step(1/60);

// p5.js lee la posición y la escala a píxeles
var SC = 280;
var bx = cannonBall.position.x * SC;
var by = -cannonBall.position.y * SC;   // eje Y invertido

// Rotación: quaternion de Cannon → eje-ángulo para p5.rotate()
var angle = 2 * Math.acos(q.w);
var s = Math.sqrt(1 - q.w * q.w);
p.rotate(angle, [q.x/s, q.y/s, q.z/s]);
```

### Construcción del mundo Cannon (buildCannon)

Se reconstruye cada vez que cambia el ángulo o el tipo de esfera:

```javascript
world.gravity.set(0, -14, 0)

// Esfera con momento de inercia personalizado
cannonBall = new CANNON.Body({ mass: 1 })
cannonBall.inertia.set(I, I, I)

// Plano inclinado (caja estática rotada)
cannonPlane.quaternion.setFromAxisAngle(
    new CANNON.Vec3(0,0,1), -angulo
)

// Material de contacto con fricción configurable
new CANNON.ContactMaterial(bMat, pMat, { friction: fric })
```

### Elementos visuales exclusivos del Proyecto 6

- **Rastro luminoso** — línea 3D en cian que traza la trayectoria real
  calculada por Cannon, con desvanecimiento gradual
- **Flecha velocidad** (roja) — dirección y magnitud del vector `v`
  del cuerpo de Cannon
- **Flecha eje angular** (cian) — dirección del vector `ω` de Cannon,
  muestra alrededor de qué eje está girando la esfera en cada instante
- **Tag CANNON.js** — indicador visual en la esquina del viewport que
  aparece solo en la Tab 3

### Tecnologías

| Biblioteca | Versión | Uso |
|------------|:-------:|-----|
| p5.js WEBGL | 1.9.0 | Render 3D, Tabs 1 y 2, cámara orbital |
| Cannon.js | 0.6.2 | Física de Tab 3: gravedad, colisiones, fricción, rodadura |

---

# Documentación — Rodadura sin Deslizar
## Proyectos 7 y 8 · Universidad Cuauhtémoc · Métodos Numéricos 
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

### Diferencias clave entre proyectos 7 y 8

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