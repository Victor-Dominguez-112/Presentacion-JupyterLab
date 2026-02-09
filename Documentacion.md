# Trabajo #1: Análisis de Face Tracking y Visión por Computadora
**Proyecto de Métodos Numéricos**

### 1. ¿Qué es MindAR?
MindAR es una biblioteca de software de código abierto y ligera diseñada para desarrollar experiencias de Realidad Aumentada (AR) en la web. Permite el reconocimiento de imágenes y seguimiento facial directamente en el navegador utilizando tecnologías estándar como WebGL y WebAssembly, eliminando la necesidad de instalar aplicaciones externas.

### 2. ¿Qué es OpenCV?
OpenCV (Open Source Computer Vision Library) es la biblioteca de visión artificial de código abierto más utilizada a nivel mundial. Provee una infraestructura común para aplicaciones de visión por computadora y contiene más de 2500 algoritmos optimizados para tareas como detección de rostros, identificación de objetos, clasificación de acciones en video, rastreo de movimientos y procesamiento de imágenes (filtros, bordes, transformaciones).

### 3. ¿De manera interna MindAR usa OpenCV?
**No.** Aunque ambas herramientas procesan imágenes, MindAR no depende de OpenCV.
* **MindAR** está construida sobre TensorFlow.js y utiliza modelos de aprendizaje profundo (Deep Learning) propietarios y ligeros para realizar la detección y seguimiento de características faciales o imágenes planas.
* **OpenCV** se basa en algoritmos clásicos de procesamiento de matrices de píxeles, mientras que MindAR se basa en inferencia de redes neuronales.

### 4. ¿Se puede utilizar OpenCV en JavaScript?
**Sí.** Existe una versión oficial llamada OpenCV.js. Mediante la tecnología WebAssembly (Wasm), el código original de C++ de OpenCV es compilado para que pueda ser ejecutado directamente por el navegador web (lado del cliente) con un rendimiento cercano al nativo, permitiendo realizar procesamiento de imágenes complejo en tiempo real dentro de páginas web.

### 5. ¿Para qué sirve el algoritmo de Canny Edge Detection?
El algoritmo de Canny es una técnica de procesamiento de imágenes utilizada para detectar bordes de manera robusta. Es considerado el algoritmo estándar óptimo para esta tarea porque cumple tres criterios clave:
1.  **Detección:** Baja tasa de error (encuentra todos los bordes reales).
2.  **Localización:** Los puntos detectados deben estar lo más cerca posible del borde real.
3.  **Respuesta única:** Debe marcar una sola línea por cada borde (evita bordes gruesos o múltiples respuestas al mismo contorno).

### 6. Ejemplo del algoritmo de Canny Edge Detection en JavaScript

```%%html
<div id="canny_final_container" style="text-align: center; background: #1a1a1a; padding: 25px; border-radius: 20px; color: white; font-family: sans-serif;">
    <h2 style="color: #3498db;">Canny Edge: Detección de Bordes</h2>
    <p style="font-size: 13px; color: #aaa;">Usa derivadas y diferencias finitas para encontrar contornos.</p>
    
    <div style="position: relative; display: inline-block; margin-bottom: 20px;">
        <video id="v_canny_raw" width="640" height="480" style="display:none" playsinline></video>
        <canvas id="c_canny_final" width="640" height="480" style="background: #000; border: 3px solid #333; border-radius: 15px;"></canvas>
    </div>
    
    <div style="margin-top: 10px;">
        <button id="btn_on_canny" onclick="forzarCannyAbsoluto()" style="padding: 15px 30px; background: #27ae60; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; font-size: 16px;">
            🚀 ACTIVAR CANNY AQUÍ
        </button>
        <button id="btn_off_canny" onclick="apagarCannyAbsoluto()" style="padding: 15px 30px; background: #e74c3c; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; font-size: 16px; margin-left: 10px; display: none;">
            🛑 APAGAR
        </button>
    </div>
    <p id="msg_canny_final" style="color: #f1c40f; font-size: 14px; margin-top: 15px;">Estado: Esperando acción del usuario.</p>
</div>

<script async src="https://docs.opencv.org/4.8.0/opencv.js" type="text/javascript"></script>

<script>
// Referencias únicas para evitar choques entre celdas
var vidCanny = document.getElementById('v_canny_raw');
var canCanny = document.getElementById('c_canny_final');
var txtStatus = document.getElementById('msg_canny_final');
var btnOn = document.getElementById('btn_on_canny');
var btnOff = document.getElementById('btn_off_canny');

var cannyRunning = false;
var cannyStream = null;

async function forzarCannyAbsoluto() {
    txtStatus.innerText = "Limpiando conexiones previas...";
    
    // 1. LIMPIEZA TOTAL: Matamos cualquier cámara abierta en la ventana
    try {
        if (window.currentStream) {
            window.currentStream.getTracks().forEach(t => t.stop());
            window.currentStream = null;
        }
    } catch(e) { console.log("Limpio."); }

    // 2. FORZAR INICIO
    try {
        cannyStream = await navigator.mediaDevices.getUserMedia({ video: true });
        window.currentStream = cannyStream; // Compartimos el stream para que otros códigos puedan cerrarlo
        vidCanny.srcObject = cannyStream;
        vidCanny.play();
        
        btnOn.style.display = 'none';
        btnOff.style.display = 'inline-block';
        txtStatus.innerText = "Cámara forzada con éxito. Procesando...";
        
        cannyRunning = true;
        iniciarIA_Canny();
    } catch (err) {
        txtStatus.innerText = "Error: No se pudo forzar la cámara. Revisa permisos.";
        console.error(err);
    }
}

function apagarCannyAbsoluto() {
    cannyRunning = false;
    if (cannyStream) {
        cannyStream.getTracks().forEach(track => track.stop());
    }
    vidCanny.pause();
    vidCanny.srcObject = null;
    btnOn.style.display = 'inline-block';
    btnOff.style.display = 'none';
    txtStatus.innerText = "Cámara liberada y apagada.";
}

function iniciarIA_Canny() {
    if (!cannyRunning) return;

    // Inicializamos matrices OpenCV (C++ compilado a WebAssembly)
    let src = new cv.Mat(vidCanny.height, vidCanny.width, cv.CV_8UC4);
    let dst = new cv.Mat(vidCanny.height, vidCanny.width, cv.CV_8UC1);
    let cap = new cv.VideoCapture(vidCanny);

    function procesarFrame() {
        if (!cannyRunning) {
            src.delete(); dst.delete();
            return;
        }
        
        try {
            cap.read(src); // Leer frame del video
            
            // Paso 1: Escala de Grises (Requisito fundamental de Canny)
            cv.cvtColor(src, dst, cv.COLOR_RGBA2GRAY); 
            
            // Paso 2: Algoritmo Canny (Detecta bordes con derivadas)
            // Umbrales 50 y 150 para conectar bordes
            cv.Canny(dst, dst, 50, 150, 3, false); 
            
            cv.imshow('c_canny_final', dst); // Mostrar resultado
            requestAnimationFrame(procesarFrame);
        } catch (err) {
            console.error("Error en loop:", err);
            cannyRunning = false;
        }
    }
    requestAnimationFrame(procesarFrame);
}
</script>
```

# Análisis Matemático y Lógico de Nuestro Algoritmo AR

En este proyecto, nuestro equipo no ha utilizado imágenes estáticas pre-cargadas para los filtros. En su lugar, hemos desarrollado un motor de renderizado basado en **Geometría Computacional** y **Álgebra Lineal**. A continuación, explicamos la lógica matemática interna que permite que los objetos se adapten al rostro en tiempo real.

### 1. Transformación de Espacios Vectoriales (Mapeo Lineal)
El modelo de Inteligencia Artificial (MediaPipe Face Mesh) nos devuelve una matriz de 468 puntos (landmarks). Sin embargo, estos puntos vienen en un **Espacio Normalizado**, es decir, son valores flotantes entre $0$ y $1$ donde:
* $(0,0)$ representa la esquina superior izquierda.
* $(1,1)$ representa la esquina inferior derecha.

Para dibujar en el canvas, hemos aplicado una **Transformación Lineal de Escala** para mapear estos valores al **Espacio Cartesiano de Píxeles** ($640 \times 480$).

La fórmula que hemos implementado para cada punto $P(x,y)$ es:

$$
P_{pixel} = P_{norm} \cdot \vec{S}
$$

Donde el vector de escala $\vec{S}$ corresponde a las dimensiones del canvas:
$$
x_{pixel} = x_{norm} \cdot Ancho_{canvas}
$$
$$
y_{pixel} = y_{norm} \cdot Alto_{canvas}
$$

> **En nuestro código:** Esto se observa cuando pasamos los argumentos a las funciones de dibujo, por ejemplo: `nose.x * can_el.width`.

---

### 2. Escalamiento Dinámico (Geometría Proyectiva)
Uno de los retos principales que enfrentamos fue lograr que los lentes o el sombrero cambiaran de tamaño si el usuario se acerca o aleja de la cámara (simulación de profundidad $Z$). Como no contamos con un sensor de profundidad real, utilizamos una **referencia métrica relativa**.

Hemos seleccionado dos puntos ancla anatómicamente estables: los pómulos (Landmarks **#454** y **#234**). Calculamos la magnitud del vector diferencia entre ellos en el eje X para determinar el "ancho aparente" de la cara ($W_{cara}$).

$$
W_{cara} = |x_{454} - x_{234}| \cdot Ancho_{pixel}
$$

Este valor $W_{cara}$ se convierte en nuestro escalar base ($k$). Todas las figuras geométricas se dibujan como una función de $k$:
* Radio de la nariz = $0.15 \cdot k$
* Ancho de los lentes = $1.0 \cdot k$

De esta forma, mantenemos la **proporcionalidad** euclidiana sin importar la distancia de la cámara.

---

### 3. Construcción de Primitivas Geométricas
En lugar de pegar imágenes (bitmaps), hemos utilizado ecuaciones geométricas para rasterizar los objetos píxel por píxel.

#### A. La Nariz (Gradientes Radiales)
Para la nariz, no dibujamos un círculo plano. Para simular volumen (3D), implementamos una función de **Gradiente Radial** $G(r)$. Matemáticamente, interpolamos el color desde un centro desplazado $(x - \Delta, y - \Delta)$ hacia el radio exterior $r$.

La ecuación base es la del círculo:
$$
(x - h)^2 + (y - k)^2 = r^2
$$

Al desplazar el foco del gradiente hacia la esquina superior izquierda, simulamos una fuente de luz, creando un efecto de esfericidad (especularidad) mediante el cálculo de la intensidad de color $I$ en función del radio.

#### B. La Corona y el Sombrero (Polígonos y Traslación)
Para estos objetos, definimos polígonos irregulares conectando una secuencia de vértices $V_1, V_2, \dots, V_n$.

El desafío matemático aquí es la **Traslación de Vectores**. Si dibujáramos la corona en las coordenadas de la frente (Landmark #10), quedaría *dentro* de la cabeza. Para corregirlo, aplicamos un vector de desplazamiento negativo en el eje Y (recordando que en computación gráfica, el eje Y positivo va hacia abajo).

$$
P_{objeto} = P_{frente} + \vec{v}_{desplazamiento}
$$

Donde $\vec{v}_{desplazamiento} = (0, -0.6 \cdot W_{cara})$. Esto eleva el objeto proporcionalmente al tamaño de la cabeza detectada.

---

### 4. Topología de Malla (Teoría de Grafos)
Finalmente, la "Malla Verde" que visualizamos es una representación directa de la topología del grafo que utiliza la red neuronal.
* **Vértices ($V$):** Los 468 puntos detectados.
* **Aristas ($E$):** Las conexiones predefinidas (teselación) que unen los puntos para formar triángulos.

Hemos utilizado la función `drawConnectors` que recorre la matriz de adyacencia del grafo y renderiza las líneas que conectan los nodos $V_i$ y $V_j$, permitiendo visualizar la geometría subyacente que la computadora "ve" en el rostro.

### 7. ¿El algoritmo Canny Edge Detection utiliza derivadas?
[cite_start]**Sí.** En el contexto de una imagen, un borde se define matemáticamente como un cambio brusco en la intensidad de los píxeles[cite: 34]. [cite_start]El algoritmo de Canny busca los puntos donde la primera derivada de la función de intensidad de la imagen alcanza un máximo local (es decir, donde el gradiente es más pronunciado)[cite: 35].

### 8. ¿En qué forma el algoritmo Canny Edge utiliza las diferencias finitas en su cálculo?
[cite_start]Dado que una imagen digital es una matriz discreta de píxeles y no una función continua, no es posible calcular derivadas analíticas[cite: 38]. [cite_start]El algoritmo utiliza **Diferencias Finitas** para aproximar el gradiente[cite: 39]. [cite_start]Se aplican núcleos de convolución (como el operador Sobel) que realizan restas ponderadas entre píxeles vecinos (ej. $f(x+1)-f(x-1)$) para estimar la tasa de cambio (derivada) en las direcciones horizontal y vertical[cite: 39].

### 9. Algoritmos para detectar bordes
[cite_start]Existen diversos operadores basados en el cálculo del gradiente a través de diferencias finitas[cite: 41]:
* [cite_start]Operador Sobel [cite: 42]
* [cite_start]Operador Prewitt [cite: 43]
* [cite_start]Operador Roberts [cite: 44]
* [cite_start]Laplaciano de Gaussiana (LOG) [cite: 45]
* [cite_start]Algoritmo de Canny [cite: 46]

### 10. ¿Para qué sirve el algoritmo de Sobel?
[cite_start]El operador Sobel sirve para calcular una aproximación del gradiente de intensidad de una imagen[cite: 48]. [cite_start]Se utiliza para detectar bordes resaltando las regiones de alta frecuencia espacial[cite: 49]. [cite_start]Es computacionalmente eficiente y efectivo para detectar la orientación y magnitud de los bordes simples[cite: 50].

### 11 y 12. ¿Cómo utiliza Sobel las derivadas?
[cite_start]Calcula la **Primera Derivada discreta** usando máscaras de $3\times3$ para $G_{x}$ (horizontal) y $G_{y}$ (vertical)[cite: 52].
La magnitud total se obtiene con:
[cite_start]$$G=\sqrt{G_{x}^{2}+G_{y}^{2}}$$ [cite: 53]

### 13. Relación de Sobel con diferencias finitas
La relación es directa. [cite_start]El núcleo aplica una **Diferencia Central** combinada con un suavizado[cite: 55]. [cite_start]Por ejemplo, el kernel `[-1, 0, +1]` es la definición numérica de la primera diferencia finita[cite: 56].

### 14. ¿Se requiere escala de grises para Sobel?
[cite_start]**Sí.** Los algoritmos de detección de bordes operan sobre cambios de intensidad (luminosidad), no sobre información cromática[cite: 59]. [cite_start]Convertir la imagen a escala de grises simplifica la entrada de 3 canales (RGB) a 1 canal, reduciendo la complejidad computacional y eliminando el ruido que podrían introducir las variaciones de tono[cite: 60].

### 15. ¿MindAR es de fuente abierta?
[cite_start]**Sí.** MindAR se distribuye bajo la licencia MIT[cite: 62]. [cite_start]Esto significa que es software libre y de código abierto, permitiendo su uso, modificación y distribución tanto para proyectos personales como comerciales sin restricciones significativas[cite: 63].

### 16. ¿Qué es MediaPipe Face Mesh de Google?
[cite_start]MediaPipe Face Mesh es una solución de aprendizaje automático (Machine Learning) desarrollada por Google que permite la estimación geométrica de rostros en tiempo real[cite: 65]. [cite_start]Es capaz de detectar **468 puntos de referencia (landmarks)** en 3D sobre el rostro humano, funcionando eficientemente incluso en dispositivos móviles sin hardware dedicado[cite: 66].

### 17, 20, 21 y 22. Implementación Técnica: Face Mesh con Filtros y WebCam
[cite_start]A continuación se presenta el código fuente que integra los requerimientos[cite: 76]:
* [cite_start]**Punto 17:** Carga MediaPipe Face Mesh y dibuja los 468 puntos[cite: 78].
* [cite_start]**Punto 20:** Solicitud de acceso a webcam[cite: 79].
* [cite_start]**Punto 21:** Edición de máscara colocando un objeto[cite: 80].
* [cite_start]**Punto 22:** Filtro en un punto específico (nariz)[cite: 81].

```%%html
<div id="ar_final_box" style="text-align: center; background: #1a1a1a; padding: 20px; border-radius: 15px; color: white; font-family: sans-serif;">
    <h2 style="color: #3498db;">PROYECTO FINAL: Filtros AR 3D</h2>
    
    <div style="margin-bottom: 10px; display: flex; justify-content: center; gap: 5px; flex-wrap: wrap;">
        <button onclick="setF(0)">❌ Quitar</button>
        <button onclick="setF(1)">🔴 Nariz Roja</button>
        <button onclick="setF(2)">🤠 Sombrero</button>
        <button onclick="setF(3)">😎 Lentes</button>
        <button onclick="setF(4)">👑 Corona</button>
    </div>

    <div style="margin-bottom: 15px;">
        <button id="m_btn" onclick="togM()" style="padding: 8px 15px; background: #8e44ad; color: white; border: none; border-radius: 5px; cursor: pointer;">Ocultar Malla Verde</button>
    </div>

    <div style="position: relative; display: inline-block;">
        <video id="v_src" style="display:none" playsinline></video>
        <canvas id="c_out" width="640" height="480" style="background: #000; border: 2px solid #444; border-radius: 10px;"></canvas>
    </div>
    
    <div style="margin-top: 15px;">
        <button id="b_start" onclick="runAr()" style="padding: 15px 30px; background: #27ae60; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold;">🚀 ACTIVAR CÁMARA</button>
        <p id="debug_log" style="color: #f1c40f; font-size: 13px; margin-top: 10px; background: #000; padding: 5px;">Estado: Esperando clic...</p>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>

<script>
var vid_el = document.getElementById('v_src');
var can_el = document.getElementById('c_out');
var ctx_el = can_el.getContext('2d');
var log_el = document.getElementById('debug_log');

var show_m = true;
var filter_id = 0;

function setF(n) {
    filter_id = n;
    log_el.innerText = n > 0 ? "Filtro " + n + " activado ✓" : "Filtro removido";
}

function togM() {
    show_m = !show_m;
    document.getElementById('m_btn').innerText = show_m ? "Ocultar Malla Verde" : "Mostrar Malla Verde";
}

// FUNCIONES PARA DIBUJAR FILTROS (sin imágenes externas)
//1. NARIZ DE PAYASO ROJA con brillo realista
function drawClownNose(ctx, x, y, size) {
    ctx.save();
    
    // Sombra
    ctx.shadowColor = 'rgba(0,0,0,0.4)';
    ctx.shadowBlur = 25;
    ctx.shadowOffsetX = 15;
    ctx.shadowOffsetY = 15;
    
    // Gradiente principal rojo
    const gradient = ctx.createRadialGradient(x - size*0.2, y - size*0.2, 0, x, y, size*0.6);
    gradient.addColorStop(0, '#ff6b6b');
    gradient.addColorStop(0.5, '#ee4444');
    gradient.addColorStop(1, '#cc2222');
    
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(x, y, size*0.25, 0, Math.PI * 8);
    ctx.fill();
    
    // Brillo superior izquierdo (grande)
    ctx.shadowColor = 'transparent';
    const highlightGrad = ctx.createRadialGradient(x - size*0.15, y - size*0.15, 0, x - size*0.15, y - size*0.15, size*0.25);
    highlightGrad.addColorStop(0, 'rgba(255,255,255,0.9)');
    highlightGrad.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.fillStyle = highlightGrad;
    ctx.beginPath();
    ctx.arc(x - size*0.15, y - size*0.15, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    
    // Brillo pequeño extra
    ctx.fillStyle = 'rgba(255,255,255,0.6)';
    ctx.beginPath();
    ctx.arc(x + size*0.1, y + size*0.15, size*0.08, 0, Math.PI * 2);
    ctx.fill();
    
    ctx.restore();
}

function drawHat(ctx, x, y, size) {
    ctx.save();
    ctx.fillStyle = '#8B4513';
    ctx.strokeStyle = '#654321';
    ctx.lineWidth = 3;
    ctx.shadowColor = 'rgba(0,0,0,0.5)';
    ctx.shadowBlur = 15;
    
    // Ala del sombrero
    ctx.beginPath();
    ctx.ellipse(x, y, size*0.8, size*0.25, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    // Copa del sombrero
    ctx.fillStyle = '#A0522D';
    ctx.beginPath();
    ctx.moveTo(x - size*0.4, y);
    ctx.lineTo(x - size*0.35, y - size*0.8);
    ctx.lineTo(x + size*0.35, y - size*0.8);
    ctx.lineTo(x + size*0.4, y);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
    
    // Banda
    ctx.fillStyle = '#FFD700';
    ctx.fillRect(x - size*0.35, y - size*0.3, size*0.7, size*0.15);
    ctx.restore();
}

function drawGlasses(ctx, x, y, size) {
    ctx.save();
    ctx.strokeStyle = '#000000';
    ctx.lineWidth = 4;
    ctx.shadowColor = 'rgba(0,0,0,0.5)';
    ctx.shadowBlur = 10;
    
    // Lente izquierdo
    ctx.fillStyle = 'rgba(0,0,0,0.3)';
    ctx.beginPath();
    ctx.arc(x - size*0.35, y, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    // Lente derecho
    ctx.beginPath();
    ctx.arc(x + size*0.35, y, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    // Puente
    ctx.beginPath();
    ctx.moveTo(x - size*0.1, y);
    ctx.lineTo(x + size*0.1, y);
    ctx.stroke();
    
    // Brillos
    ctx.fillStyle = 'rgba(255,255,255,0.6)';
    ctx.beginPath();
    ctx.arc(x - size*0.42, y - size*0.08, size*0.08, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.arc(x + size*0.28, y - size*0.08, size*0.08, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();
}

function drawCrown(ctx, x, y, size) {
    ctx.save();
    ctx.fillStyle = '#FFD700';
    ctx.strokeStyle = '#FFA500';
    ctx.lineWidth = 3;
    ctx.shadowColor = 'rgba(0,0,0,0.5)';
    ctx.shadowBlur = 15;
    
    ctx.beginPath();
    ctx.moveTo(x - size*0.5, y);
    ctx.lineTo(x - size*0.4, y - size*0.4);
    ctx.lineTo(x - size*0.25, y - size*0.2);
    ctx.lineTo(x, y - size*0.5);
    ctx.lineTo(x + size*0.25, y - size*0.2);
    ctx.lineTo(x + size*0.4, y - size*0.4);
    ctx.lineTo(x + size*0.5, y);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
    
    // Joyas
    const jewels = [
        {x: x - size*0.4, y: y - size*0.4, c: '#ff0000'},
        {x: x - size*0.25, y: y - size*0.2, c: '#00ff00'},
        {x: x, y: y - size*0.5, c: '#ff0000'},
        {x: x + size*0.25, y: y - size*0.2, c: '#0000ff'},
        {x: x + size*0.4, y: y - size*0.4, c: '#ff00ff'}
    ];
    
    jewels.forEach(j => {
        ctx.fillStyle = j.c;
        ctx.beginPath();
        ctx.arc(j.x, j.y, size*0.05, 0, Math.PI * 2);
        ctx.fill();
    });
    ctx.restore();
}

async function runAr() {
    log_el.innerText = "Paso 1: Solicitando cámara...";
    try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { width: 640, height: 480 } });
        vid_el.srcObject = stream;
        await vid_el.play();
        log_el.innerText = "Paso 2: Cámara activa. Cargando IA...";
        setTimeout(startIA, 500);
    } catch (e) {
        log_el.innerText = "ERROR: No se pudo abrir la cámara. ¿Diste permiso?";
        console.error(e);
    }
}

function startIA() {
    try {
        const mesh = new FaceMesh({
            locateFile: (f) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${f}`
        });
        
        mesh.setOptions({ 
            maxNumFaces: 1, 
            refineLandmarks: true, 
            minDetectionConfidence: 0.5,
            minTrackingConfidence: 0.5
        });
        
        mesh.onResults((res) => {
            ctx_el.save();
            ctx_el.clearRect(0, 0, can_el.width, can_el.height);
            ctx_el.drawImage(res.image, 0, 0, can_el.width, can_el.height);
            
            if (res.multiFaceLandmarks && res.multiFaceLandmarks.length > 0) {
                const face = res.multiFaceLandmarks[0];
                
                if (show_m) {
                    drawConnectors(ctx_el, face, FACEMESH_TESSELATION, {color: '#00FF0050', lineWidth: 1});
                }

                // Calcular ancho de cara
                const faceW = Math.abs(face[454].x - face[234].x) * can_el.width;
                
                // Dibujar filtros
                if (filter_id === 1) {
                    const nose = face[1];
                    drawNose(ctx_el, nose.x * can_el.width, nose.y * can_el.height, faceW * 0.15);
                } 
                else if (filter_id === 2) {
                    const top = face[10];
                    drawHat(ctx_el, top.x * can_el.width, (top.y * can_el.height) - faceW*0.6, faceW * 0.8);
                }
                else if (filter_id === 3) {
                    const eyes = face[168];
                    drawGlasses(ctx_el, eyes.x * can_el.width, eyes.y * can_el.height, faceW);
                }
                else if (filter_id === 4) {
                    const top = face[10];
                    drawCrown(ctx_el, top.x * can_el.width, (top.y * can_el.height) - faceW*0.5, faceW * 0.6);
                }
            }
            ctx_el.restore();
        });

        const cam = new Camera(vid_el, {
            onFrame: async () => { 
                await mesh.send({image: vid_el}); 
            },
            width: 640, 
            height: 480
        });
        
        cam.start();
        log_el.innerText = "✅ ¡Todo listo! Selecciona un filtro";
        
    } catch (err) {
        log_el.innerText = "ERROR de IA: " + err.message;
        console.error(err);
    }
}
</script>
```

### 18. ¿MediaPipe utiliza Sobel?
[cite_start]**No directamente.** MediaPipe utiliza Redes Neuronales Convolucionales (CNN)[cite: 158]. [cite_start]A diferencia del algoritmo Sobel, que utiliza fórmulas matemáticas fijas (kernels predefinidos) para buscar bordes, MediaPipe utiliza modelos entrenados con millones de imágenes para aprender a identificar patrones complejos como ojos, labios y contornos faciales, independientemente de los bordes simples[cite: 158].

### 19. ¿Qué son las redes neuronales convolucionales (CNN)?
[cite_start]Son un tipo de arquitectura de Deep Learning diseñada específicamente para procesar datos con estructura de cuadrícula, como las imágenes[cite: 160]. [cite_start]

[Image of convolutional neural network architecture]
 Utilizan capas de "convolución" que funcionan como filtros aprendidos automáticamente[cite: 161]. [cite_start]A diferencia de Sobel (donde el humano define el filtro), una CNN aprende por sí sola qué filtros aplicar para detectar desde líneas simples hasta formas complejas como una cara humana[cite: 161].

### 23. Escribir el mismo concepto pero usando Sobel
[cite_start]Implementación de procesamiento de video en tiempo real utilizando el operador Sobel para detección de bordes mediante OpenCV.js[cite: 163].

```from IPython.display import display, HTML

# Interfaz unificada: Cámara + OpenCV.js + Método de Sobel (Sin Canny)
opencv_realtime_sobel = """
<div style="display: flex; flex-direction: column; align-items: center; background: #1a1a1a; padding: 20px; border-radius: 15px; color: white; font-family: sans-serif; width: 450px; margin: auto;">
    <h3 style="margin-bottom: 10px;">OpenCV Live: Método Sobel</h3>
    <div id="status" style="color: #ffcc00; margin-bottom: 10px;">Cargando OpenCV.js...</div>
    
    <video id="videoInput" width="400" height="300" autoplay playsinline style="display:none;"></video>
    
    <canvas id="canvasOutput" width="400" height="300" style="border: 2px solid #007bff; border-radius: 10px; background: #000;"></canvas>
    
    <div style="margin-top: 15px; font-size: 0.8em; color: #888; text-align: center;">
        <p>Calculando Magnitud del Gradiente en tiempo real:</p>
        <code>G = sqrt(Gx² + Gy²)</code>
    </div>
</div>

<script async src="https://docs.opencv.org/4.5.4/opencv.js" type="text/javascript"></script>

<script>
    const status = document.getElementById('status');
    const video = document.getElementById('videoInput');

    // Función que inicia cuando OpenCV.js está cargado en el navegador
    function onOpenCvReady() {
        status.innerHTML = "OpenCV Listo. Iniciando Cámara...";
        status.style.color = "#28a745";
        startVideo();
    }

    // Verificar si OpenCV ya cargó (por si el evento 'async' falla)
    let checkCv = setInterval(() => {
        if (typeof cv !== 'undefined' && cv.Mat) {
            onOpenCvReady();
            clearInterval(checkCv);
        }
    }, 1000);

    async function startVideo() {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
            video.srcObject = stream;
            video.play();
            processVideo();
        } catch (err) {
            status.innerHTML = " Error: Permiso de cámara denegado";
            status.style.color = "#ff4444";
        }
    }

    function processVideo() {
        // Inicializar matrices de OpenCV
        let src = new cv.Mat(video.height, video.width, cv.CV_8UC4);
        let dst = new cv.Mat(video.height, video.width, cv.CV_8UC1);
        let cap = new cv.VideoCapture(video);

        let grad_x = new cv.Mat();
        let grad_y = new cv.Mat();
        let abs_grad_x = new cv.Mat();
        let abs_grad_y = new cv.Mat();

        function process() {
            try {
                // Leer el cuadro actual de la cámara
                cap.read(src);
                
                // 1. Convertir a Gris (Requisito para derivadas)
                cv.cvtColor(src, dst, cv.COLOR_RGBA2GRAY);
                
                // 2. MÉTODOS NUMÉRICOS: Sobel (Derivadas parciales)
                // Calculamos Gx (Derivada horizontal)
                cv.Sobel(dst, grad_x, cv.CV_16S, 1, 0, 3);
                cv.convertScaleAbs(grad_x, abs_grad_x);

                // Calculamos Gy (Derivada vertical)
                cv.Sobel(dst, grad_y, cv.CV_16S, 0, 1, 3);
                cv.convertScaleAbs(grad_y, abs_grad_y);

                // 3. Combinar gradientes (Aproximación de la magnitud total)
                // Aumentamos el peso a 1.0 para que la silueta sea brillante
                cv.addWeighted(abs_grad_x, 1.0, abs_grad_y, 1.0, 0, dst);

                // Dibujar el resultado en el canvas
                cv.imshow('canvasOutput', dst);

                // Llamar al siguiente cuadro (Tiempo Real)
                requestAnimationFrame(process);
            } catch (err) {
                console.log("Esperando flujo de video...");
                requestAnimationFrame(process);
            }
        }
        requestAnimationFrame(process);
    }
</script>
"""

display(HTML(opencv_realtime_sobel))
```

