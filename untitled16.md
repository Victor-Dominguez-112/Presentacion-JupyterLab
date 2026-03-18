# Documentación 
## Simulaciones de Física Computacional con p5.js en JupyterLab

---

> **Universidad Cuauhtémoc**  
> **Materia:** Métodos Numéricos  
> **Integrantes:** Víctor Dominguez · Jesús Macias · Diego Vazquez · Mario Rangel  
> **Referencia bibliográfica:** Dobre, A. & Bhatt, D. (2012). *Physics for JavaScript Games, Animation & Simulations*. Apress.


---

## 1. Introducción y Objetivos

El presente documento describe el desarrollo de cuatro proyectos de simulación computacional elaborados como parte de la materia de **Métodos Numéricos** en la Universidad Cuauhtémoc. Cada proyecto implementa conceptos fundamentales de física y matemáticas mediante la biblioteca gráfica **p5.js**, ejecutada dentro del entorno interactivo **JupyterLab**.

La motivación central es la demostración práctica de que las leyes físicas, formuladas matemáticamente, pueden traducirse directamente en código para producir comportamientos visuales realistas. Como señala el libro de referencia:

> *"Simple laws of motion and simple force laws can give rise to complex motions. It is therefore generally easier to code the laws rather than the motions."*  
> — Capítulo 1, p. 12

### Objetivos Generales

- Comprender la diferencia conceptual entre **animación** (reproducir el efecto) y **simulación** (reproducir la causa).
- Implementar el ciclo de simulación numérica de cuatro pasos propuesto por el libro: identificar la física, formular ecuaciones, desarrollar algoritmos y escribir el código.
- Aplicar las Leyes de Newton, la conservación del momento y principios de geometría computacional dentro de un entorno reproducible y documentado.

### Objetivos Específicos

| # | Proyecto | Objetivo específico |
|---|----------|---------------------|
| 1 | Pelota que Rebota | Implementar cinemática 2D con gravedad y rebote inelástico |
| 2 | Segunda Ley de Newton | Aplicar F = ma con fuerzas múltiples y visualizar vectores |
| 3 | Conservación del Momento | Simular colisiones con coeficiente de restitución variable |
| 4 | Editor de Cuadros | Implementar interacción gráfica y detección geométrica |

---

## 2. Entorno Tecnológico

Todos los proyectos fueron desarrollados bajo la misma arquitectura técnica para garantizar consistencia y reproducibilidad.

| Componente | Descripción |
|------------|-------------|
| **JupyterLab** | Entorno de cuadernos interactivos. Permite combinar celdas de código, texto y salidas multimedia |
| **Kernel Python 3** | Motor de ejecución. Cada celda de código es interpretada por Python |
| **IPython.display.HTML** | Módulo de Python que inyecta HTML, CSS y JavaScript en la salida de una celda |
| **p5.js v1.9.0** | Biblioteca JavaScript de gráficos y animación, cargada desde CDN de Cloudflare |
| **Modo instanced de p5.js** | Patrón que encapsula cada simulación en su propio ámbito sin contaminar el espacio global |

El kernel de Python actúa exclusivamente como mensajero: entrega el bloque HTML al navegador. Una vez entregado, p5.js toma el control completo. Esto permite usar JavaScript dentro de un entorno Python sin instalar kernels adicionales.

---

## 3. Marco Teórico

### 3.1 Cinemática y Simulación Numérica

La cinemática estudia el movimiento de los cuerpos sin considerar sus causas. Las variables que describen el estado de una partícula en 2D son posición $(x, y)$, velocidad $(v_x, v_y)$ y aceleración $(a_x, a_y)$.

El proyecto adopta el **método de integración de Euler hacia adelante**, en el que el tiempo avanza en pasos discretos $\Delta t$ correspondientes a un frame:

$$\vec{v}_{n+1} = \vec{v}_n + \vec{a}_n \cdot \Delta t$$

$$\vec{r}_{n+1} = \vec{r}_n + \vec{v}_{n+1} \cdot \Delta t$$

Este esquema es suficientemente preciso para animaciones a 60 fps, donde $\Delta t \approx 0.016$ s.

### 3.2 Las Leyes de Newton

**Primera Ley — Principio de Inercia:**

$$\vec{F} = \vec{0} \implies \vec{a} = \vec{0}$$

**Segunda Ley — Forma para masa constante:**

$$\vec{F} = m\vec{a} \qquad \Longrightarrow \qquad \vec{a} = \frac{\vec{F}}{m}$$

**Tercera Ley — Acción y Reacción:**

$$\vec{F}_{A \to B} = -\vec{F}_{B \to A}$$

### 3.3 El Objeto Forces

El libro propone (pp. 116–117) encapsular las leyes de fuerza en un objeto estático `Forces`. Las implementaciones usadas son:

**Gravedad constante** — near-Earth gravity:

$$\vec{F}_g = m \cdot g \,\hat{j}$$

**Drag lineal** — resistencia viscosa a bajas velocidades:

$$\vec{F}_d = -k\vec{v}$$

**Fuerza resultante** — suma vectorial de todas las fuerzas activas:

$$\vec{F}_{total} = \vec{F}_g + \vec{F}_d$$

### 3.4 Ciclo de Simulación por Frame

| Función | Responsabilidad | Fórmula |
|---------|----------------|---------|
| `calcForce()` | Calcular la fuerza resultante | $\vec{F} = \sum \vec{F}_i$ |
| `updateAccel()` | Segunda Ley de Newton | $\vec{a} = \vec{F}/m$ |
| `updateVelo()` | Integración de Euler | $\vec{v} \mathrel{+}= \vec{a} \cdot \Delta t$ |
| `moveObject()` | Integración de Euler | $\vec{r} \mathrel{+}= \vec{v} \cdot \Delta t$ |

### 3.5 Conservación del Momento Lineal

$$m_1 v_1 + m_2 v_2 = m_1 v_1' + m_2 v_2' = \text{constante}$$

**Velocidades post-colisión** con coeficiente de restitución $e$:

$$v_1' = \frac{(m_1 - e \cdot m_2)\,u_1 + (1+e)\,m_2\,u_2}{m_1 + m_2}$$

$$v_2' = \frac{(m_2 - e \cdot m_1)\,u_2 + (1+e)\,m_1\,u_1}{m_1 + m_2}$$

| Valor de $e$ | Tipo de colisión | Energía cinética |
|:---:|---|---|
| $e = 1$ | Perfectamente elástica | Se conserva totalmente |
| $0 < e < 1$ | Parcialmente inelástica | Pérdida parcial |
| $e = 0$ | Perfectamente inelástica | Pérdida máxima; los cuerpos se unen |

### 3.6 Geometría Computacional Interactiva

Detección de colisión punto-rectángulo. Un punto $(p_x, p_y)$ está dentro de un rectángulo si:

$$x_{min} \leq p_x \leq x_{max} \quad \text{AND} \quad y_{min} \leq p_y \leq y_{max}$$

Donde $x_{min} = \min(x, x+w)$ y $x_{max} = \max(x, x+w)$. El uso de mínimo y máximo permite dibujar en cualquier dirección.

---

## Proyecto 1 — Editor de Cuadros Interactivo

**Referencia:** Geometría computacional 2D | **Archivo:** `editor_cuadros_p5js.ipynb`

### Descripción

Herramienta de dibujo interactivo que aplica geometría computacional y manejo de estado. El usuario crea rectángulos mediante clics, visualiza dimensiones al hacer hover y elimina cuadros con la tecla Delete.

### Máquina de Estados Finitos

**Estado 0 — ESPERANDO:** el sistema está listo para el primer clic. Al hacer clic, registra el punto de inicio y transiciona al Estado 1.

**Estado 1 — DIBUJANDO:** el sistema actualiza las dimensiones del cuadro temporal en cada movimiento del ratón. Al hacer el segundo clic, guarda el cuadro y regresa al Estado 0.

### Detección Punto-Rectángulo

Al mover el ratón en Estado 0, el sistema recorre el arreglo en orden inverso y verifica la condición de contención para cada cuadro. La iteración inversa garantiza que, ante superposición, el cuadro visualmente superior sea seleccionado primero.

### Funcionalidades

| Acción | Descripción |
|--------|-------------|
| **Clic izquierdo** | Inicia el cuadro en la posición del cursor |
| **Movimiento del ratón** | Actualiza dimensiones en tiempo real |
| **Segundo clic izquierdo** | Confirma y guarda el cuadro |
| **Hover sobre cuadro** | Muestra dimensiones en px y resalta en rojo |
| **Tecla Delete / Supr** | Elimina el cuadro bajo el cursor |
| **Botón "Borrar todo"** | Limpia el canvas completamente |
| **Color picker** | Selecciona el color de relleno |
| **Slider de opacidad** | Controla la transparencia del relleno |

``` # Celda 1 — Cargar p5.js
from IPython.display import HTML
HTML("""
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
<p style="color:#4caf50;font-weight:bold;">✅ p5.js cargado</p>
""")

```

``` # Celda 2 — Editor de Cuadros Interactivo
from IPython.display import HTML

HTML("""
<div style="
  font-family: sans-serif;
  background-color: #1c2333;
  padding: 20px;
  border-radius: 12px;
  display: inline-block;
  color: white;
">

  <!-- Título igual al screenshot -->
  <div style="background:#1c2333; padding:10px 16px; border-radius:8px;
              font-size:15px; font-weight:bold; margin-bottom:12px; color:#c9d1d9;">
    Página 12 a 14 Editor de cuadros
  </div>

  
  <p style="margin:0 0 12px 0; font-size:14px;">
     <b>Clic</b> para iniciar el cuadro · <b>Mueve</b> el ratón · <b>Clic</b> para terminar · <b>Delete/Supr</b> para borrar
  </p>

  <!-- Botones de control -->
  <div style="margin-bottom:10px; display:flex; gap:10px; align-items:center; flex-wrap:wrap;">
    <button onclick="borrarTodo()"
      style="background:#e94560;color:white;border:none;padding:7px 16px;border-radius:8px;cursor:pointer;font-size:13px;font-weight:bold;">
       Borrar todo
    </button>
    <label style="font-size:13px; display:flex; align-items:center; gap:6px; cursor:pointer;">
      Color:
      <input type="color" id="colorPicker" value="#0000ff"
        style="width:36px; height:28px; border:none; border-radius:6px; cursor:pointer;">
    </label>
    <label style="font-size:13px; display:flex; align-items:center; gap:6px;">
      Opacidad:
      <input type="range" id="opacitySlider" min="0.1" max="1" step="0.05" value="0.4"
        style="width:100px;"
        oninput="document.getElementById('opacVal').innerText=parseFloat(this.value).toFixed(2)">
      <span id="opacVal" style="font-size:12px;">0.40</span>
    </label>
    <span id="contadorSpan" style="font-size:12px; color:#aaa;">Cuadros: 0</span>
  </div>

  <!-- Canvas -->
  <div id="editor-canvas"
    style="background:#ffffff; border:2px solid #000; cursor:crosshair; display:inline-block;">
  </div>

  <!-- Info en tiempo real -->
  <div id="info-cuadros"
    style="margin-top:8px; font-size:12px; color:#cccccc; font-family:monospace; min-height:18px;">
  </div>
</div>

<script>
var editorSketch = function(p) {

  var W = 700, H = 500;

  // ── Estado del editor ──────────────────────────────────
  var cuadros        = [];
  var estadoDibujo   = 0;      // 0 = esperando, 1 = dibujando
  var inicioX, inicioY;
  var cuadroTemporal  = null;
  var cuadroHover     = null;

  // Leer color y opacidad del panel
  function getColor() {
    var hex  = document.getElementById('colorPicker').value;
    var opac = parseFloat(document.getElementById('opacitySlider').value);
    var r = parseInt(hex.slice(1,3),16);
    var g = parseInt(hex.slice(3,5),16);
    var b = parseInt(hex.slice(5,7),16);
    return { r:r, g:g, b:b, a:opac };
  }

  // Exponer funciones globales para los botones HTML
  window.borrarTodo = function() {
    cuadros = [];
    cuadroHover = null;
    document.getElementById('contadorSpan').innerText = 'Cuadros: 0';
  };

  // ── Setup ──────────────────────────────────────────────
  p.setup = function() {
    var cnv = p.createCanvas(W, H);
    cnv.parent('editor-canvas');
    p.noLoop();   // solo redibujamos cuando hay eventos
  };

  // ── Draw ───────────────────────────────────────────────
  p.draw = function() {
    p.background(255);

    // Cuadrícula sutil
    p.stroke(220);
    p.strokeWeight(1);
    for (var gx = 0; gx < W; gx += 40) p.line(gx, 0, gx, H);
    for (var gy = 0; gy < H; gy += 40) p.line(0, gy, W, gy);

    // Dibujar cuadros guardados
    for (var i = 0; i < cuadros.length; i++) {
      var c = cuadros[i];
      var esHover = (c === cuadroHover);

      p.fill(c.r, c.g, c.b, c.a * 255);
      p.stroke(esHover ? p.color(255,80,80) : p.color(0,0,180));
      p.strokeWeight(esHover ? 2.5 : 1.5);
      p.rect(c.x, c.y, c.ancho, c.alto);

      // Etiqueta de medidas al hacer hover
      if (esHover) {
        var lx = c.ancho < 0 ? c.x + c.ancho : c.x;
        var ly = c.alto  < 0 ? c.y + c.alto  : c.y;
        p.fill(0);
        p.noStroke();
        p.textSize(11);
        p.textAlign(p.LEFT);
        p.text(Math.abs(c.ancho) + ' × ' + Math.abs(c.alto) + ' px', lx + 4, ly + 14);
      }
    }

    // Cuadro temporal (mientras dibuja)
    if (cuadroTemporal) {
      var ct = cuadroTemporal;
      p.fill(ct.r, ct.g, ct.b, ct.a * 255);
      p.stroke(220, 50, 50);
      p.strokeWeight(2);
      p.rect(ct.x, ct.y, ct.ancho, ct.alto);

      // Medidas en tiempo real
      p.fill(0);
      p.noStroke();
      p.textSize(12);
      p.textAlign(p.LEFT);
      p.text('Ancho: ' + Math.abs(ct.ancho) + 'px   Alto: ' + Math.abs(ct.alto) + 'px',
             12, 20);
    }
  };

  // ── Clic izquierdo — iniciar / confirmar cuadro ────────
  p.mouseClicked = function() {
    if (p.mouseX < 0 || p.mouseX > W || p.mouseY < 0 || p.mouseY > H) return;

    if (estadoDibujo === 0) {
      // Primer clic: marca el inicio
      inicioX = p.mouseX;
      inicioY = p.mouseY;
      var col = getColor();
      cuadroTemporal = { x:inicioX, y:inicioY, ancho:0, alto:0,
                         r:col.r, g:col.g, b:col.b, a:col.a };
      estadoDibujo = 1;

    } else if (estadoDibujo === 1) {
      // Segundo clic: guarda el cuadro
      if (Math.abs(cuadroTemporal.ancho) > 3 && Math.abs(cuadroTemporal.alto) > 3) {
        cuadros.push(cuadroTemporal);
        document.getElementById('contadorSpan').innerText = 'Cuadros: ' + cuadros.length;
      }
      cuadroTemporal = null;
      estadoDibujo = 0;
    }
    p.redraw();
  };

  // ── Movimiento — actualiza tamaño o detecta hover ──────
  p.mouseMoved = function() {
    if (estadoDibujo === 1 && cuadroTemporal) {
      cuadroTemporal.ancho = p.mouseX - inicioX;
      cuadroTemporal.alto  = p.mouseY - inicioY;
    } else {
      cuadroHover = null;
      for (var i = cuadros.length - 1; i >= 0; i--) {
        var c = cuadros[i];
        var minX = Math.min(c.x, c.x + c.ancho);
        var maxX = Math.max(c.x, c.x + c.ancho);
        var minY = Math.min(c.y, c.y + c.alto);
        var maxY = Math.max(c.y, c.y + c.alto);
        if (p.mouseX >= minX && p.mouseX <= maxX &&
            p.mouseY >= minY && p.mouseY <= maxY) {
          cuadroHover = c;
          break;
        }
      }
      // Info panel
      if (cuadroHover) {
        document.getElementById('info-cuadros').innerText =
          'Hover: Ancho = ' + Math.abs(cuadroHover.ancho) + 'px  ·  Alto = ' +
          Math.abs(cuadroHover.alto) + 'px  (Delete / Supr para eliminar)';
      } else {
        document.getElementById('info-cuadros').innerText = '';
      }
    }
    p.redraw();
  };

  // ── Delete / Supr — eliminar cuadro bajo el cursor ─────
  p.keyPressed = function() {
    if (p.keyCode === p.DELETE && estadoDibujo === 0 && cuadroHover) {
      var idx = cuadros.indexOf(cuadroHover);
      if (idx > -1) {
        cuadros.splice(idx, 1);
        cuadroHover = null;
        document.getElementById('contadorSpan').innerText = 'Cuadros: ' + cuadros.length;
        document.getElementById('info-cuadros').innerText = '';
        p.redraw();
      }
    }
  };
};

new p5(editorSketch);
</script>
""")

```

## Proyecto 2 — Segunda Ley de Newton con Vectores de Fuerza

**Referencia:** Capítulo 5, páginas 112–117 | **Archivo:** `newton_forces_p5js.ipynb`

### Descripción

Implementa el sistema modular de fuerzas del Capítulo 5. Una partícula se mueve bajo gravedad constante y drag lineal simultáneamente. Se visualizan en tiempo real cuatro vectores sobre la partícula, haciendo visible la física que normalmente permanece abstracta.

### Vectores Visualizados

| Vector | Color | Fórmula | Comportamiento observable |
|--------|:---:|---------|--------------------------|
| $\vec{F}_g$ — Gravedad | 🔴 Rojo | $m \cdot g \,\hat{j}$ | Siempre apunta abajo; magnitud aumenta con la masa |
| $\vec{F}_d$ — Drag | 🩵 Cian | $-k\vec{v}$ | Cambia dirección con la velocidad |
| $\vec{F}$ — Resultante | 🟡 Amarillo | $\vec{F}_g + \vec{F}_d$ | Determina la aceleración del sistema |
| $\vec{v}$ — Velocidad | 🟣 Morado | estado actual | Dirección y rapidez de la partícula |

### Experimentos con los Controles

| Configuración | Resultado esperado |
|---|---|
| Drag $k = 0$ | Trayectoria parabólica pura |
| Drag $k$ alto | Partícula alcanza velocidad terminal $v_t = mg/k$ |
| Masa doble, misma fuerza | Aceleración a la mitad ($a = F/m$) |

### Relación con Métodos Numéricos

La función `updateAccel()` implementa directamente $\vec{a} = \vec{F}/m$. Las funciones `updateVelo()` y `moveObject()` implementan las dos integraciones de Euler sucesivas que relacionan fuerza, velocidad y posición.

---

``` # Celda 2 — Simulación: Leyes de Newton con objeto Forces
# Basado en: Physics for JS Games — Capítulo 5, páginas 112-117
from IPython.display import HTML

HTML("""
  
<div id="newton-root" style="font-family:'Segoe UI',sans-serif; background:#0d1117; padding:20px; border-radius:14px; display:inline-block; color:#e0e0e0;">

  <!-- Título igual al screenshot -->
  <div style="background:#1c2333; padding:10px 16px; border-radius:8px;
              font-size:15px; font-weight:bold; margin-bottom:12px; color:#c9d1d9;">
    Página 112 a 117 Leyes de Newton — Objeto Forces
  </div>

  <!-- ── CONTROLES ───────────────────────────────────────── -->
  <div style="display:flex; gap:18px; flex-wrap:wrap; margin-bottom:14px; align-items:flex-end;">

    <div>
      <div style="font-size:12px; color:#8b949e; margin-bottom:4px;">Masa (m)</div>
      <input type="range" id="massSlider" min="0.5" max="5" step="0.5" value="1"
             style="width:110px;" oninput="document.getElementById('massVal').innerText=parseFloat(this.value).toFixed(1)+' kg'">
      <span id="massVal" style="font-size:12px; margin-left:6px;">1.0 kg</span>
    </div>

    <div>
      <div style="font-size:12px; color:#8b949e; margin-bottom:4px;">Gravedad (g)</div>
      <input type="range" id="gravSlider" min="0.05" max="0.8" step="0.05" value="0.2"
             style="width:110px;" oninput="document.getElementById('gravVal').innerText=parseFloat(this.value).toFixed(2)">
      <span id="gravVal" style="font-size:12px; margin-left:6px;">0.20</span>
    </div>

    <div>
      <div style="font-size:12px; color:#8b949e; margin-bottom:4px;">Drag (k)</div>
      <input type="range" id="dragSlider" min="0" max="0.1" step="0.005" value="0.02"
             style="width:110px;" oninput="document.getElementById('dragVal').innerText=parseFloat(this.value).toFixed(3)">
      <span id="dragVal" style="font-size:12px; margin-left:6px;">0.020</span>
    </div>

    <div style="display:flex; gap:8px;">
      <button onclick="nwtReset()"
        style="background:#e94560;color:#fff;border:none;padding:7px 16px;border-radius:8px;cursor:pointer;font-size:12px;font-weight:bold;">🔄 Reiniciar</button>
      <button onclick="nwtToggle()" id="nwtPauseBtn"
        style="background:#161b22;color:#fff;border:1px solid #30363d;padding:7px 16px;border-radius:8px;cursor:pointer;font-size:12px;font-weight:bold;">⏸ Pausar</button>
    </div>

    <div style="display:flex; gap:8px; align-items:center;">
      <label style="font-size:12px; color:#8b949e; display:flex; align-items:center; gap:5px; cursor:pointer;">
        <input type="checkbox" id="showVectors" checked> Mostrar vectores
      </label>
      <label style="font-size:12px; color:#8b949e; display:flex; align-items:center; gap:5px; cursor:pointer;">
        <input type="checkbox" id="showTrail" checked> Trayectoria
      </label>
    </div>
  </div>

  <!-- ── CANVAS ──────────────────────────────────────────── -->
  <div id="newton-canvas"></div>

  <!-- ── LEYENDA DE VECTORES ──────────────────────────────── -->
  <div style="display:flex; gap:20px; margin-top:10px; font-size:12px;">
    <span><span style="color:#ff6b6b;">●</span> F_gravedad (mg ↓)</span>
    <span><span style="color:#4ecdc4;">●</span> F_drag (−kv)</span>
    <span><span style="color:#ffe66d;">●</span> F_total (resultante)</span>
    <span><span style="color:#a29bfe;">●</span> Velocidad (v)</span>
  </div>

  <!-- ── PANEL DE DATOS ───────────────────────────────────── -->
  <div id="nwt-data" style="font-family:monospace; font-size:11px; color:#8b949e; margin-top:8px; line-height:1.8;"></div>
</div>

<script>
// ═══════════════════════════════════════════════════════════
//  Vector2D  —  clase auxiliar (equivale a la del libro)
// ═══════════════════════════════════════════════════════════
function Vector2D(x, y) { this.x = x; this.y = y; }
Vector2D.prototype.add = function(v) { return new Vector2D(this.x + v.x, this.y + v.y); };
Vector2D.prototype.addScaled = function(v, s) { return new Vector2D(this.x + v.x*s, this.y + v.y*s); };
Vector2D.prototype.multiply = function(s) { return new Vector2D(this.x*s, this.y*s); };
Vector2D.prototype.mag = function() { return Math.sqrt(this.x*this.x + this.y*this.y); };

// ═══════════════════════════════════════════════════════════
//  Forces  —  objeto del libro (pp. 116-117)
// ═══════════════════════════════════════════════════════════
function Forces() {}

Forces.zeroForce = function() {
  return new Vector2D(0, 0);                      // p.117
};

Forces.constantGravity = function(m, g) {
  return new Vector2D(0, m * g);                  // p.117  F_g = mg hacia abajo
};

Forces.linearDrag = function(k, velo) {
  return velo.multiply(-k);                       // p.117  F_d = -kv
};

// ═══════════════════════════════════════════════════════════
//  Sketch p5.js
// ═══════════════════════════════════════════════════════════
var nwtSketch = function(p) {

  var W = 560, H = 360;
  var dt = 1;                  // timestep por frame
  var radius = 18;
  var paused = false;

  // Estado de la partícula
  var pos, velo, acc;
  var force, fGrav, fDrag;    // fuerzas individuales para visualizar
  var trail = [];
  var bounceCount = 0;

  // Leer sliders
  function cfg() {
    return {
      m: parseFloat(document.getElementById('massSlider').value),
      g: parseFloat(document.getElementById('gravSlider').value),
      k: parseFloat(document.getElementById('dragSlider').value)
    };
  }

  function reset() {
    pos   = new Vector2D(80, 60);
    velo  = new Vector2D(3, 0);
    acc   = new Vector2D(0, 0);
    force = Forces.zeroForce();
    fGrav = Forces.zeroForce();
    fDrag = Forces.zeroForce();
    trail = [];
    bounceCount = 0;
  }

  window.nwtReset  = reset;
  window.nwtToggle = function() {
    paused = !paused;
    document.getElementById('nwtPauseBtn').innerText = paused ? '▶ Reanudar' : '⏸ Pausar';
  };

  // ── Ciclo del libro (p.115) ──────────────────────────────
  function calcForce(c) {
    fGrav = Forces.constantGravity(c.m, c.g);   // F_g = m·g
    fDrag = Forces.linearDrag(c.k, velo);        // F_d = −k·v
    force = fGrav.add(fDrag);                    // resultante
  }

  function updateAccel(c) {
    acc = force.multiply(1 / c.m);               // a = F/m  (p.115)
  }

  function updateVelo() {
    velo = velo.addScaled(acc, dt);              // Δv = a·dt (p.115)
  }

  function moveObject() {
    pos = pos.addScaled(velo, dt);               // Δpos = v·dt (p.115)
  }

  // ── Rebote en paredes ────────────────────────────────────
  function handleBounds() {
    if (pos.y > H - radius) {
      pos.y = H - radius;
      velo.y *= -0.75;
      bounceCount++;
    }
    if (pos.x > W - radius) { pos.x = W - radius; velo.x *= -0.85; }
    if (pos.x < radius)     { pos.x = radius;     velo.x *= -0.85; }
    if (pos.y < radius)     { pos.y = radius;      velo.y *= -0.75; }
  }

  // ── Dibujar flecha de vector ─────────────────────────────
  function drawArrow(ox, oy, vx, vy, col, label, scale) {
    scale = scale || 1;
    var ex = ox + vx * scale;
    var ey = oy + vy * scale;
    var len = Math.sqrt(vx*vx + vy*vy) * scale;
    if (len < 1) return;

    p.stroke(col);
    p.strokeWeight(2.2);
    p.line(ox, oy, ex, ey);

    // Punta de flecha
    var angle = Math.atan2(ey - oy, ex - ox);
    var hs = 9;
    p.fill(col);
    p.noStroke();
    p.push();
    p.translate(ex, ey);
    p.rotate(angle);
    p.triangle(0, 0, -hs, -hs*0.45, -hs, hs*0.45);
    p.pop();

    // Etiqueta
    p.noStroke();
    p.fill(col);
    p.textSize(11);
    p.textAlign(p.LEFT);
    p.text(label, ex + 5, ey + 4);
  }

  // ────────────────────────────────────────────────────────
  p.setup = function() {
    var cnv = p.createCanvas(W, H);
    cnv.parent('newton-canvas');
    p.frameRate(60);
    reset();
  };

  p.draw = function() {
    // Fondo
    p.background(13, 17, 23);

    // Cuadrícula
    p.stroke(255, 255, 255, 12);
    p.strokeWeight(1);
    for (var gx = 0; gx < W; gx += 40) p.line(gx, 0, gx, H);
    for (var gy = 0; gy < H; gy += 40) p.line(0, gy, W, gy);

    // Suelo
    p.stroke(233, 69, 96, 160);
    p.strokeWeight(2);
    p.line(0, H - radius, W, H - radius);

    var c = cfg();

    if (!paused) {
      // ── Ciclo F=ma del libro ────────────────────────────
      calcForce(c);
      updateAccel(c);
      updateVelo();
      moveObject();
      handleBounds();
      // ───────────────────────────────────────────────────

      trail.push({x: pos.x, y: pos.y});
      if (trail.length > 120) trail.shift();
    }

    // Trayectoria
    if (document.getElementById('showTrail').checked) {
      p.noFill();
      for (var i = 1; i < trail.length; i++) {
        var a = p.map(i, 0, trail.length, 0, 200);
        p.stroke(162, 155, 254, a);
        p.strokeWeight(p.map(i, 0, trail.length, 0.5, 2));
        p.line(trail[i-1].x, trail[i-1].y, trail[i].x, trail[i].y);
      }
    }

    // ── Vectores de fuerza ──────────────────────────────────
    if (document.getElementById('showVectors').checked) {
      var sc = 60;   // escala visual de los vectores
      var vsc = 20;

      // Velocidad (morado)
      drawArrow(pos.x, pos.y, velo.x, velo.y,
        p.color(162, 155, 254), 'v', vsc);

      // F_gravedad (rojo)
      drawArrow(pos.x - 20, pos.y, fGrav.x, fGrav.y,
        p.color(255, 107, 107), 'Fg', sc);

      // F_drag (cian)
      drawArrow(pos.x, pos.y, fDrag.x, fDrag.y,
        p.color(78, 205, 196), 'Fd', sc);

      // F_total / resultante (amarillo)
      drawArrow(pos.x + 20, pos.y, force.x, force.y,
        p.color(255, 230, 109), 'F', sc);
    }

    // ── Partícula ───────────────────────────────────────────
    // Sombra
    p.noStroke(); p.fill(0, 0, 0, 70);
    p.ellipse(pos.x + 4, pos.y + 4, radius*2, radius*2);

    // Cuerpo
    p.stroke(200, 200, 255, 180); p.strokeWeight(2);
    p.fill(100, 160, 255);
    p.ellipse(pos.x, pos.y, radius*2, radius*2);

    // Brillo
    p.noStroke(); p.fill(255, 255, 255, 110);
    p.ellipse(pos.x - radius*0.3, pos.y - radius*0.35, radius*0.55, radius*0.38);

    // ── Panel de datos ──────────────────────────────────────
    var spd = velo.mag().toFixed(2);
    var fmag = force.mag().toFixed(3);
    document.getElementById('nwt-data').innerHTML =
      '<b style="color:#e0e0e0;">Partícula:</b> ' +
      'pos = (' + pos.x.toFixed(1) + ', ' + pos.y.toFixed(1) + ')' +
      ' &nbsp;|&nbsp; ' +
      'v = (' + velo.x.toFixed(2) + ', ' + velo.y.toFixed(2) + ')  |speed| = ' + spd +
      '<br>' +
      '<b style="color:#ff6b6b;">F_grav</b> = (0, ' + fGrav.y.toFixed(3) + ')' +
      ' &nbsp;|&nbsp; ' +
      '<b style="color:#4ecdc4;">F_drag</b> = (' + fDrag.x.toFixed(3) + ', ' + fDrag.y.toFixed(3) + ')' +
      ' &nbsp;|&nbsp; ' +
      '<b style="color:#ffe66d;">F_total</b> = (' + force.x.toFixed(3) + ', ' + force.y.toFixed(3) + ')  |F| = ' + fmag +
      '<br>' +
      '<b style="color:#a29bfe;">a</b> = (' + acc.x.toFixed(4) + ', ' + acc.y.toFixed(4) + ')' +
      ' &nbsp;|&nbsp; m = ' + c.m.toFixed(1) + ' kg' +
      ' &nbsp;|&nbsp; rebotes = ' + bounceCount;

    // FPS
    p.fill(255,255,255,60); p.noStroke(); p.textSize(10);
    p.textAlign(p.RIGHT);
    p.text('FPS: ' + Math.round(p.frameRate()), W - 6, 14);
  };
};

new p5(nwtSketch);
</script>
""")

```


## Proyecto 3 — Pelota que Rebota

**Referencia:** Capítulo 1, páginas 12–14 | **Archivo:** `Leyes de Newton Objeto Forces.ipynb`

### Descripción

Implementa el primer ejemplo del libro: una pelota 2D bajo la acción de la gravedad que rebota en el suelo con pérdida de energía. En lugar de programar la trayectoria curva, se programan las causas y el movimiento emerge de forma natural.

### Variables del Sistema

| Variable | Valor inicial | Descripción |
|----------|:---:|-------------|
| `x`, `y` | 50, 50 | Coordenadas de la pelota (px) |
| `vx`, `vy` | 3, 0 | Velocidad horizontal y vertical (px/frame) |
| `g` | 0.30 | Aceleración gravitacional (px/frame²) |

### Física Implementada

Gravedad — incrementa la velocidad vertical cada frame:

$$v_y \leftarrow v_y + g$$

Actualización de posición:

$$x \leftarrow x + v_x \qquad y \leftarrow y + v_y$$

Rebote en el suelo con factor de restitución $e = 0.8$:

$$v_y \leftarrow -0.8 \cdot v_y$$

### Relación con Métodos Numéricos

Cada línea de actualización discretiza una EDO. La instrucción `vy += g` corresponde a $\frac{dv_y}{dt} = g$, y `y += vy` corresponde a $\frac{dy}{dt} = v_y$. Ambas aplican el método de Euler hacia adelante con $\Delta t = 1$ frame.

### Funcionalidades

- Simulación en tiempo real a 60 fps
- Rastro de trayectoria con desvanecimiento gradual
- Panel de datos: posición, velocidad y contador de rebotes
- Controles: gravedad, factor de rebote y velocidad inicial horizontal
- Botones de pausa y reinicio

---

``` # Celda 2 — SIMULACIÓN 1: Segunda Ley de Newton F = ma  (p. 123)
# calcForce() → updateAccel() → updateVelo() → moveObject()
from IPython.display import HTML

HTML("""
<div style="font-family:'Segoe UI',sans-serif; background:#0d1117; padding:18px;
            border-radius:12px; display:inline-block; color:#e0e0e0;">

  <!-- Título igual al screenshot -->
  <div style="background:#1c2333; padding:10px 16px; border-radius:8px;
              font-size:15px; font-weight:bold; margin-bottom:12px; color:#c9d1d9;">
    Página 123: Segunda Ley de Newton (F = ma)
  </div>

  <!-- Controles -->
  <div style="display:flex; gap:16px; flex-wrap:wrap; margin-bottom:10px; align-items:flex-end;">
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Fuerza (F)</div>
      <input type="range" id="f1_fuerza" min="0.05" max="1.0" step="0.05" value="0.15"
        style="width:110px;"
        oninput="document.getElementById('f1_fVal').innerText=parseFloat(this.value).toFixed(2)">
      <span id="f1_fVal" style="font-size:11px;margin-left:4px;">0.15</span>
    </div>
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Masa (m)</div>
      <input type="range" id="f1_masa" min="0.5" max="5" step="0.5" value="2"
        style="width:110px;"
        oninput="document.getElementById('f1_mVal').innerText=parseFloat(this.value).toFixed(1)">
      <span id="f1_mVal" style="font-size:11px;margin-left:4px;">2.0</span>
    </div>
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Drag (k)</div>
      <input type="range" id="f1_drag" min="0" max="0.08" step="0.005" value="0.01"
        style="width:110px;"
        oninput="document.getElementById('f1_kVal').innerText=parseFloat(this.value).toFixed(3)">
      <span id="f1_kVal" style="font-size:11px;margin-left:4px;">0.010</span>
    </div>
    <button onclick="f1Reset()"
      style="background:#e94560;color:#fff;border:none;padding:6px 14px;
             border-radius:7px;cursor:pointer;font-size:12px;font-weight:bold;">🔄</button>
  </div>

  <div id="f1-canvas"></div>

  <!-- Panel de datos — igual al screenshot -->
  <div id="f1-info"
    style="background:#161b22;border-radius:6px;padding:6px 12px;
           font-family:monospace;font-size:12px;color:#8b949e;margin-top:6px;">
  </div>
</div>

<script>
// ── Vector2D ────────────────────────────────────────────────
function Vec2(x,y){this.x=x;this.y=y;}
Vec2.prototype.add       = function(v){return new Vec2(this.x+v.x,this.y+v.y);};
Vec2.prototype.addScaled = function(v,s){return new Vec2(this.x+v.x*s,this.y+v.y*s);};
Vec2.prototype.multiply  = function(s){return new Vec2(this.x*s,this.y*s);};
Vec2.prototype.length    = function(){return Math.sqrt(this.x*this.x+this.y*this.y);};

// ── Forces ────────────────────────────────
var Forces1 = {};
Forces1.constantGravity = function(m,g){ return new Vec2(0, m*g); };
Forces1.linearDrag      = function(k,v){ return v.multiply(-k); };
Forces1.add             = function(arr){
  var r = new Vec2(0,0);
  for(var i=0;i<arr.length;i++) r=r.add(arr[i]);
  return r;
};

var f1Sketch = function(p){
  var W=560, H=200, R=22;
  var pos, velo, acc, force, fGrav, fDrag;
  var trail=[], paused=false;

  function cfg(){
    return {
      F: parseFloat(document.getElementById('f1_fuerza').value),
      m: parseFloat(document.getElementById('f1_masa').value),
      k: parseFloat(document.getElementById('f1_drag').value)
    };
  }

  function reset(){
    pos   = new Vec2(60, 80);
    velo  = new Vec2(0, 0);
    acc   = new Vec2(0, 0);
    force = new Vec2(0, 0);
    fGrav = new Vec2(0, 0);
    fDrag = new Vec2(0, 0);
    trail = [];
  }
  window.f1Reset = reset;

  // ── Ciclo del libro (p.115/123) ─────────────────────────
  function calcForce(c){
    fGrav = Forces1.constantGravity(c.m, c.F);   // F_g = m*g (usamos F como g)
    fDrag = Forces1.linearDrag(c.k, velo);        // F_d = -k*v
    force = Forces1.add([fGrav, fDrag]);           // resultante
  }
  function updateAccel(c){ acc  = force.multiply(1/c.m); }   // a = F/m
  function updateVelo(){    velo = velo.addScaled(acc, 1); }  // v += a*dt
  function moveObject(){    pos  = pos.addScaled(velo, 1); }  // pos += v*dt

  function handleBounds(){
    if(pos.y > H-R){ pos.y=H-R; velo.y*=-0.7; }
    if(pos.y < R)  { pos.y=R;   velo.y*=-0.7; }
    if(pos.x > W-R){ pos.x=W-R; velo.x*=-0.85; }
    if(pos.x < R)  { pos.x=R;   velo.x*=-0.85; }
  }

  // Dibujar flecha
  function arrow(px,py,vx,vy,col,sc){
    sc=sc||1;
    var ex=px+vx*sc, ey=py+vy*sc;
    var len=Math.sqrt(vx*vx+vy*vy)*sc;
    if(len<2) return;
    p.stroke(col); p.strokeWeight(2.2);
    p.line(px,py,ex,ey);
    var ang=Math.atan2(ey-py,ex-px), hs=8;
    p.fill(col); p.noStroke();
    p.push(); p.translate(ex,ey); p.rotate(ang);
    p.triangle(0,0,-hs,-hs*0.4,-hs,hs*0.4);
    p.pop();
  }

  p.setup=function(){
    p.createCanvas(W,H).parent('f1-canvas');
    p.frameRate(60); reset();
  };

  p.draw=function(){
    p.background(13,17,23);
    // cuadrícula
    p.stroke(255,255,255,12); p.strokeWeight(1);
    for(var gx=0;gx<W;gx+=40) p.line(gx,0,gx,H);
    for(var gy=0;gy<H;gy+=40) p.line(0,gy,W,gy);
    // suelo
    p.stroke(233,69,96,140); p.strokeWeight(2);
    p.line(0,H-R,W,H-R);

    var c=cfg();
    calcForce(c); updateAccel(c); updateVelo(); moveObject(); handleBounds();
    trail.push({x:pos.x,y:pos.y});
    if(trail.length>100) trail.shift();

    // trayectoria
    for(var i=1;i<trail.length;i++){
      var a=p.map(i,0,trail.length,0,180);
      p.stroke(100,160,255,a); p.strokeWeight(1.5);
      p.line(trail[i-1].x,trail[i-1].y,trail[i].x,trail[i].y);
    }

    // vectores
    arrow(pos.x,    pos.y, 0,       fGrav.y, p.color(255,100,100), 60);
    arrow(pos.x,    pos.y, fDrag.x, fDrag.y, p.color(78,205,196),  60);
    arrow(pos.x+22, pos.y, force.x, force.y, p.color(255,230,109), 60);
    arrow(pos.x-22, pos.y, velo.x,  velo.y,  p.color(162,155,254), 18);

    // partícula
    p.noStroke(); p.fill(0,0,0,60);
    p.ellipse(pos.x+3,pos.y+3,R*2,R*2);
    p.stroke(180,210,255,160); p.strokeWeight(2);
    p.fill(100,160,255);
    p.ellipse(pos.x,pos.y,R*2,R*2);
    p.noStroke(); p.fill(255,255,255,100);
    p.ellipse(pos.x-R*0.3,pos.y-R*0.35,R*0.55,R*0.38);

    // info panel — igual al screenshot
    document.getElementById('f1-info').innerHTML =
      'Fuerza: ' + fGrav.y.toFixed(3) +
      ' | Masa: ' + c.m.toFixed(1) +
      ' | Accel: ' + acc.y.toFixed(3) +
      ' | Velocidad: (' + velo.x.toFixed(2) + ', ' + velo.y.toFixed(2) + ')';
  };
};
new p5(f1Sketch);
</script>
""")

```

## Proyecto 4 — Conservación del Momento e Impacto

**Referencia:** Capítulo 5, páginas 123–128 | **Archivo:** `newton_momento_p5js.ipynb`

### Descripción

Notebook con dos simulaciones que sigue la estructura visual de la figura de referencia del libro. La primera muestra F = ma con panel de datos. La segunda demuestra la conservación del momento mediante la colisión entre dos partículas de masas variables.

### Simulación 1 — F = ma

Implementa el ciclo completo con el objeto Forces. El panel muestra en tiempo real los valores de Fuerza, Masa, Aceleración y Velocidad, replicando la figura del libro en la página 123.

### Simulación 2 — Colisión

#### Verificación de la Conservación

El momento total antes y después de la colisión se calcula y muestra en tiempo real. Para cualquier combinación de parámetros, ambos valores son iguales, demostrando que la conservación es independiente del tipo de choque.

#### Parámetros Interactivos

| Control | Rango | Efecto físico |
|---------|-------|--------------|
| Masa roja $m_1$ | 0.5 – 5.0 | Radio visual proporcional; afecta intercambio de velocidades |
| Masa azul $m_2$ | 0.5 – 5.0 | Idem |
| Velocidad inicial $v_1$ | 1.0 – 8.0 | Define el momento inicial total del sistema |
| Elasticidad $e$ | 0.0 – 1.0 | Controla el tipo de colisión |

#### Casos de Prueba Notables

| Configuración | Resultado | Principio demostrado |
|---|---|---|
| $m_1 = m_2$, $e = 1$ | La roja para; la azul sale a velocidad $u_1$ | Intercambio completo de momento |
| $m_1 \gg m_2$, $e = 1$ | La roja apenas cambia; la azul sale a $\approx 2u_1$ | Asimetría de masas |
| $e = 0$ | Ambas se mueven juntas | Colisión perfectamente inelástica |

---

``` # Celda 3 — SIMULACIÓN 2: Conservación del Momento — Colisión (p. 128)
# p = m*v  →  m1*v1 + m2*v2 = constante
from IPython.display import HTML

HTML("""
<div style="font-family:'Segoe UI',sans-serif; background:#0d1117; padding:18px;
            border-radius:12px; display:inline-block; color:#e0e0e0;">

  <!-- Título igual al screenshot -->
  <div style="background:#1c2333; padding:10px 16px; border-radius:8px;
              font-size:15px; font-weight:bold; margin-bottom:12px; color:#c9d1d9;">
    Página 128: Conservación del Momento (Impacto)
  </div>

  <!-- Controles -->
  <div style="display:flex; gap:16px; flex-wrap:wrap; margin-bottom:10px; align-items:flex-end;">
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Masa roja (m1)</div>
      <input type="range" id="p2_m1" min="0.5" max="5" step="0.5" value="2"
        style="width:100px;"
        oninput="document.getElementById('p2_m1Val').innerText=parseFloat(this.value).toFixed(1)">
      <span id="p2_m1Val" style="font-size:11px;margin-left:4px;">2.0</span>
    </div>
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Masa azul (m2)</div>
      <input type="range" id="p2_m2" min="0.5" max="5" step="0.5" value="1.5"
        style="width:100px;"
        oninput="document.getElementById('p2_m2Val').innerText=parseFloat(this.value).toFixed(1)">
      <span id="p2_m2Val" style="font-size:11px;margin-left:4px;">1.5</span>
    </div>
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Vel. roja (v1)</div>
      <input type="range" id="p2_v1" min="1" max="8" step="0.5" value="4"
        style="width:100px;"
        oninput="document.getElementById('p2_v1Val').innerText=parseFloat(this.value).toFixed(1)">
      <span id="p2_v1Val" style="font-size:11px;margin-left:4px;">4.0</span>
    </div>
    <div>
      <div style="font-size:11px;color:#8b949e;margin-bottom:3px;">Elasticidad (e)</div>
      <input type="range" id="p2_e" min="0" max="1" step="0.05" value="1"
        style="width:100px;"
        oninput="document.getElementById('p2_eVal').innerText=parseFloat(this.value).toFixed(2)">
      <span id="p2_eVal" style="font-size:11px;margin-left:4px;">1.00</span>
    </div>
    <button onclick="p2Reset()"
      style="background:#e94560;color:#fff;border:none;padding:6px 14px;
             border-radius:7px;cursor:pointer;font-size:12px;font-weight:bold;">🔄</button>
  </div>

  <div id="p2-canvas"></div>

  <!-- Info panel -->
  <div id="p2-info"
    style="background:#161b22;border-radius:6px;padding:6px 12px;
           font-family:monospace;font-size:12px;color:#8b949e;margin-top:6px;
           line-height:1.7;">
  </div>

  <!-- Instrucción igual al screenshot -->
  <div style="margin-top:8px;font-size:12px;color:#555;">
    Presione <b style="color:#e94560;">CUALQUIER TECLA</b> para resetear todo
  </div>
</div>

<script>
var p2Sketch = function(p){
  var W=560, H=160;
  var b1, b2;          // las dos partículas
  var collided=false;
  var momentoAntes, momentoDespues;

  function cfg(){
    return {
      m1: parseFloat(document.getElementById('p2_m1').value),
      m2: parseFloat(document.getElementById('p2_m2').value),
      v1: parseFloat(document.getElementById('p2_v1').value),
      e:  parseFloat(document.getElementById('p2_e').value)
    };
  }

  function reset(){
    var c = cfg();
    var r1 = 14 + c.m1*4;   // radio proporcional a la masa
    var r2 = 14 + c.m2*4;
    b1 = { x:80,     y:H/2, vx:c.v1, vy:0, m:c.m1, r:r1, trail:[] };
    b2 = { x:W-100,  y:H/2, vx:0,    vy:0, m:c.m2, r:r2, trail:[] };
    collided=false;
    momentoAntes = b1.m*b1.vx + b2.m*b2.vx;
    momentoDespues = momentoAntes;
  }
  window.p2Reset = reset;

  // ── Colisión elástica/inelástica (Conservación del Momento) ──
  // Fórmulas del libro: m1*v1 + m2*v2 = m1*v1' + m2*v2'
  function checkCollision(){
    var dx = b2.x - b1.x;
    var dist = Math.abs(dx);
    if(dist < b1.r + b2.r && !collided){
      collided = true;
      var c  = cfg();
      var e  = c.e;         // coeficiente de restitución (1=elástico, 0=perfectamente inelástico)
      var m1 = b1.m, m2 = b2.m;
      var u1 = b1.vx, u2 = b2.vx;
      // Fórmulas de colisión con coef. de restitución:
      // v1' = ((m1-e*m2)*u1 + (1+e)*m2*u2) / (m1+m2)
      // v2' = ((m2-e*m1)*u2 + (1+e)*m1*u1) / (m1+m2)
      b1.vx = ((m1-e*m2)*u1 + (1+e)*m2*u2) / (m1+m2);
      b2.vx = ((m2-e*m1)*u2 + (1+e)*m1*u1) / (m1+m2);
      // Separar partículas para evitar overlap
      var overlap = (b1.r+b2.r - dist)/2;
      b1.x -= overlap; b2.x += overlap;
      momentoDespues = m1*b1.vx + m2*b2.vx;
    }
    // Rebote en paredes
    if(b1.x-b1.r < 0)  { b1.x=b1.r;   b1.vx=Math.abs(b1.vx); }
    if(b2.x+b2.r > W)  { b2.x=W-b2.r; b2.vx=-Math.abs(b2.vx); }
    if(b1.x+b1.r > b2.x-b2.r && collided){
      // segunda colisión: reiniciar flag
      collided=false;
    }
  }

  // Dibujar rastro
  function drawTrail(b, col){
    for(var i=1;i<b.trail.length;i++){
      var a=p.map(i,0,b.trail.length,0,160);
      p.stroke(p.red(col),p.green(col),p.blue(col),a);
      p.strokeWeight(p.map(i,0,b.trail.length,0.5,2.5));
      p.line(b.trail[i-1].x,H/2,b.trail[i].x,H/2);
    }
  }

  p.setup=function(){
    p.createCanvas(W,H).parent('p2-canvas');
    p.frameRate(60); reset();
  };

  p.keyPressed=function(){ reset(); };

  p.draw=function(){
    p.background(13,17,23);
    // cuadrícula
    p.stroke(255,255,255,10); p.strokeWeight(1);
    for(var gx=0;gx<W;gx+=40) p.line(gx,0,gx,H);
    // línea central
    p.stroke(255,255,255,20); p.strokeWeight(1);
    p.line(0,H/2,W,H/2);

    // mover
    b1.x += b1.vx; b2.x += b2.vx;
    checkCollision();

    // trails
    b1.trail.push({x:b1.x}); if(b1.trail.length>60) b1.trail.shift();
    b2.trail.push({x:b2.x}); if(b2.trail.length>60) b2.trail.shift();
    drawTrail(b1, p.color(255,100,100));
    drawTrail(b2, p.color(100,160,255));

    // ── partícula roja (b1) ──
    p.noStroke(); p.fill(0,0,0,50);
    p.ellipse(b1.x+3,H/2+3,b1.r*2,b1.r*2);
    // gradiente simulado
    p.fill(220,60,80); p.stroke(255,120,120); p.strokeWeight(2);
    p.ellipse(b1.x,H/2,b1.r*2,b1.r*2);
    p.noStroke(); p.fill(255,200,200,100);
    p.ellipse(b1.x-b1.r*0.3,H/2-b1.r*0.35,b1.r*0.55,b1.r*0.38);
    // etiqueta masa
    p.fill(255); p.noStroke(); p.textSize(10); p.textAlign(p.CENTER);
    p.text('m='+b1.m.toFixed(1), b1.x, H/2+b1.r+13);

    // ── partícula azul (b2) ──
    p.noStroke(); p.fill(0,0,0,50);
    p.ellipse(b2.x+3,H/2+3,b2.r*2,b2.r*2);
    p.fill(60,120,220); p.stroke(120,170,255); p.strokeWeight(2);
    p.ellipse(b2.x,H/2,b2.r*2,b2.r*2);
    p.noStroke(); p.fill(180,210,255,100);
    p.ellipse(b2.x-b2.r*0.3,H/2-b2.r*0.35,b2.r*0.55,b2.r*0.38);
    p.fill(255); p.noStroke(); p.textSize(10); p.textAlign(p.CENTER);
    p.text('m='+b2.m.toFixed(1), b2.x, H/2+b2.r+13);

    // flechas de velocidad
    if(Math.abs(b1.vx)>0.1){
      p.stroke(255,100,100); p.strokeWeight(2); p.fill(255,100,100);
      p.line(b1.x,H/2-b1.r-10, b1.x+b1.vx*8, H/2-b1.r-10);
      var d1=b1.vx>0?1:-1;
      p.triangle(b1.x+b1.vx*8,H/2-b1.r-10,
                 b1.x+b1.vx*8-d1*8,H/2-b1.r-14,
                 b1.x+b1.vx*8-d1*8,H/2-b1.r-6);
    }
    if(Math.abs(b2.vx)>0.1){
      p.stroke(100,160,255); p.strokeWeight(2); p.fill(100,160,255);
      p.line(b2.x,H/2-b2.r-10, b2.x+b2.vx*8, H/2-b2.r-10);
      var d2=b2.vx>0?1:-1;
      p.triangle(b2.x+b2.vx*8,H/2-b2.r-10,
                 b2.x+b2.vx*8-d2*8,H/2-b2.r-14,
                 b2.x+b2.vx*8-d2*8,H/2-b2.r-6);
    }

    // ── info panel ──
    var pTot = b1.m*b1.vx + b2.m*b2.vx;
    document.getElementById('p2-info').innerHTML =
      '<span style="color:#ff6b6b;">● Roja</span>:  ' +
      'v = ' + b1.vx.toFixed(2) + '  p = ' + (b1.m*b1.vx).toFixed(2) +
      ' &nbsp;&nbsp; ' +
      '<span style="color:#6eb4ff;">● Azul</span>:  ' +
      'v = ' + b2.vx.toFixed(2) + '  p = ' + (b2.m*b2.vx).toFixed(2) +
      '<br>' +
      'Momento total antes: <b style="color:#ffe66d;">' + momentoAntes.toFixed(3) + '</b>' +
      ' &nbsp;|&nbsp; después: <b style="color:#ffe66d;">' + momentoDespues.toFixed(3) + '</b>' +
      ' &nbsp;|&nbsp; actual: <b style="color:#4ecdc4;">' + pTot.toFixed(3) + '</b>' +
      (collided ? '' : '  conservado');
  };
};
new p5(p2Sketch);
</script>
""")

```

## 5. Conclusiones

### Conclusiones Técnicas

El método de Euler hacia adelante es suficientemente preciso para simulaciones visuales en tiempo real. El patrón del objeto `Forces` es altamente escalable: agregar una nueva fuerza solo requiere un método adicional en el objeto sin modificar el ciclo principal. El uso de p5.js mediante `IPython.display.HTML` en JupyterLab es una solución robusta que no requiere servidores externos ni kernels especiales.

### Conclusiones Físicas

La Segunda Ley $\vec{F} = m\vec{a}$ es suficiente para describir una amplia variedad de movimientos; la complejidad emerge de la combinación de fuerzas simples. La conservación del momento se verifica numéricamente: el momento total permanece constante antes y después de cada colisión, independientemente de masas, velocidades y elasticidad. El coeficiente de restitución $e$ encapsula efectivamente la física del impacto sin necesidad de modelar las fuerzas de contacto.

### Conclusiones Metodológicas

La simulación demuestra ser más versátil que la animación: programar la causa produce el efecto de forma automática y para infinitos escenarios. El proceso de cuatro pasos del libro constituye una metodología aplicable a cualquier problema de simulación, no exclusivamente a los aquí tratados. La visualización de vectores en tiempo real resultó ser la herramienta pedagógica más efectiva al hacer visible lo que normalmente permanece abstracto.

---

## 6. Referencias

1. Dobre, A. & Bhatt, D. (2012). *Physics for JavaScript Games, Animation & Simulations*. Apress.
   - Capítulo 1, pp. 7–14: Introducción a la programación de física. Simulación vs. animación.
   - Capítulo 5, pp. 111–128: Las leyes de Newton. El objeto Forces. Conservación del momento.

2. p5.js Reference. (2024). *p5.js v1.9.0 Documentation*. The Processing Foundation. Recuperado de https://p5js.org/reference/

3. IPython Development Team. (2024). *IPython Documentation — display module*. Recuperado de https://ipython.readthedocs.io/

4. Newton, I. (1687). *Philosophiæ Naturalis Principia Mathematica*. Londres: Royal Society.

---

*Documentación elaborada para la materia de Métodos Numéricos*  
*Universidad Cuauhtémoc · Víctor Dominguez · Jesús Macias · Diego Vazquez · Mario Rangel*