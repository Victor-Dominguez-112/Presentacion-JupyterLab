# Documentación — Torque y Dinámica Rotacional 3D
## Proyectos 5 y 6 · Universidad Cuauhtémoc · Métodos Numéricos
**Integrantes:** Víctor · Jesús · Diego · Mario  
**Referencia:** Dobre, A. & Bhatt, D. (2012). *Physics for JavaScript Games, Animation & Simulations*. Cap. 13, pp. 336–355.

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

*Universidad Cuauhtémoc · Métodos Numéricos · Víctor · Jesús · Diego · Mario*
