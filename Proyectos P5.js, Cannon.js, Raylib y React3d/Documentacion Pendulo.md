# Documentación — Péndulo Cónico Giratorio
## Proyectos 3 y 4 · Universidad Cuauhtémoc · Métodos Numéricos
**Integrantes:** Víctor · Mario · Jesús · Diego

---

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

*Universidad Cuauhtémoc · Métodos Numéricos · Víctor · Jesús · Diego · Mario*
