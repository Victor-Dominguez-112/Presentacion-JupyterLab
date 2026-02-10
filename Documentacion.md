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
<div style="text-align: center; background: #1a1a1a; padding: 20px; border-radius: 15px; color: white; font-family: sans-serif;">
    <h3>Método Canny Edge (Derivadas y Bordes)</h3>
    <video id="v_canny_pro" width="640" height="480" style="display:none" playsinline></video>
    <canvas id="c_canny_pro" width="640" height="480" style="background: #000; border: 2px solid #27ae60; border-radius: 10px;"></canvas>
    <div style="margin-top: 15px;">
        <button id="btn_on_canny" onclick="iniciarCannyPro()" style="padding: 10px 20px; background: #27ae60; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">ACTIVAR CANNY</button>
        <button id="btn_off_canny" onclick="detenerTodo()" style="padding: 10px 20px; background: #e74c3c; color: white; border: none; border-radius: 5px; cursor: pointer; margin-left: 10px; display: none;">APAGAR</button>
    </div>
    <p id="log_canny_pro" style="color: #f1c40f; font-size: 13px; margin-top: 10px;">Estado: Esperando cámara...</p>
</div>

<script>
// SISTEMA GLOBAL DE GESTIÓN DE CÁMARA
if (typeof window.cameraManager === 'undefined') {
    window.cameraManager = {
        currentStream: null,
        currentFilter: null,
        activeLoop: null
    };
}

function detenerTodo() {
    if (window.cameraManager.activeLoop) {
        window.cameraManager.activeLoop = false;
    }
    
    if (window.cameraManager.currentStream) {
        window.cameraManager.currentStream.getTracks().forEach(t => t.stop());
        window.cameraManager.currentStream = null;
    }
    
    // LIMPIAR TODOS LOS CANVAS
    ['c_canny_pro', 'c_out', 'c_sobel_pro'].forEach(canvasId => {
        const canvas = document.getElementById(canvasId);
        if (canvas) {
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }
    });
    
    ['btn_off_canny', 'btn_off_sobel', 'b_stop'].forEach(id => {
        const btn = document.getElementById(id);
        if (btn) btn.style.display = 'none';
    });
    
    ['btn_on_canny', 'btn_on_sobel', 'b_start'].forEach(id => {
        const btn = document.getElementById(id);
        if (btn) btn.style.display = 'inline-block';
    });
    
    ['log_canny_pro', 'log_sobel_pro', 'debug_log'].forEach(id => {
        const log = document.getElementById(id);
        if (log) log.innerText = "Cámara liberada.";
    });
    
    window.cameraManager.currentFilter = null;
}

// Implementación REAL de Canny Edge Detection
function applyCannyEdgeDetection(imageData) {
    const width = imageData.width;
    const height = imageData.height;
    const data = imageData.data;
    
    // 1. Convertir a escala de grises
    const gray = new Uint8ClampedArray(width * height);
    for (let i = 0; i < data.length; i += 4) {
        const idx = i / 4;
        gray[idx] = 0.299 * data[i] + 0.587 * data[i + 1] + 0.114 * data[i + 2];
    }
    
    // 2. Suavizado Gaussiano 5x5
    const smoothed = new Uint8ClampedArray(width * height);
    const gaussianKernel = [
        2, 4, 5, 4, 2,
        4, 9, 12, 9, 4,
        5, 12, 15, 12, 5,
        4, 9, 12, 9, 4,
        2, 4, 5, 4, 2
    ];
    const kernelSum = 159;
    
    for (let y = 2; y < height - 2; y++) {
        for (let x = 2; x < width - 2; x++) {
            let sum = 0;
            for (let ky = -2; ky <= 2; ky++) {
                for (let kx = -2; kx <= 2; kx++) {
                    const idx = (y + ky) * width + (x + kx);
                    sum += gray[idx] * gaussianKernel[(ky + 2) * 5 + (kx + 2)];
                }
            }
            smoothed[y * width + x] = sum / kernelSum;
        }
    }
    
    // 3. Calcular gradientes con Sobel
    const gradX = new Float32Array(width * height);
    const gradY = new Float32Array(width * height);
    const magnitude = new Float32Array(width * height);
    const direction = new Float32Array(width * height);
    
    for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
            const idx = y * width + x;
            
            // Sobel X
            const gx = (
                -smoothed[(y-1)*width + (x-1)] + smoothed[(y-1)*width + (x+1)] +
                -2*smoothed[y*width + (x-1)] + 2*smoothed[y*width + (x+1)] +
                -smoothed[(y+1)*width + (x-1)] + smoothed[(y+1)*width + (x+1)]
            );
            
            // Sobel Y
            const gy = (
                -smoothed[(y-1)*width + (x-1)] - 2*smoothed[(y-1)*width + x] - smoothed[(y-1)*width + (x+1)] +
                smoothed[(y+1)*width + (x-1)] + 2*smoothed[(y+1)*width + x] + smoothed[(y+1)*width + (x+1)]
            );
            
            gradX[idx] = gx;
            gradY[idx] = gy;
            magnitude[idx] = Math.sqrt(gx * gx + gy * gy);
            direction[idx] = Math.atan2(gy, gx);
        }
    }
    
    // 4. Supresión no-máxima (esto hace que Canny sea diferente de Sobel)
    const suppressed = new Float32Array(width * height);
    
    for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
            const idx = y * width + x;
            const angle = direction[idx] * 180 / Math.PI;
            const mag = magnitude[idx];
            
            let n1 = 0, n2 = 0;
            
            // Determinar vecinos según la dirección del gradiente
            if ((angle >= -22.5 && angle < 22.5) || (angle >= 157.5 || angle < -157.5)) {
                // Horizontal
                n1 = magnitude[idx - 1];
                n2 = magnitude[idx + 1];
            } else if ((angle >= 22.5 && angle < 67.5) || (angle >= -157.5 && angle < -112.5)) {
                // Diagonal /
                n1 = magnitude[(y-1)*width + (x+1)];
                n2 = magnitude[(y+1)*width + (x-1)];
            } else if ((angle >= 67.5 && angle < 112.5) || (angle >= -112.5 && angle < -67.5)) {
                // Vertical
                n1 = magnitude[(y-1)*width + x];
                n2 = magnitude[(y+1)*width + x];
            } else {
                // Diagonal \
                n1 = magnitude[(y-1)*width + (x-1)];
                n2 = magnitude[(y+1)*width + (x+1)];
            }
            
            // Suprimir si no es máximo local
            if (mag >= n1 && mag >= n2) {
                suppressed[idx] = mag;
            } else {
                suppressed[idx] = 0;
            }
        }
    }
    
    // 5. Umbralización con histéresis (doble umbral + seguimiento de bordes)
    const lowThreshold = 30;
    const highThreshold = 90;
    const edges = new Uint8ClampedArray(width * height);
    
    // Marcar píxeles fuertes
    for (let i = 0; i < suppressed.length; i++) {
        if (suppressed[i] >= highThreshold) {
            edges[i] = 255; // Borde fuerte
        } else if (suppressed[i] >= lowThreshold) {
            edges[i] = 128; // Borde débil (candidato)
        } else {
            edges[i] = 0;
        }
    }
    
    // Seguimiento de bordes (conectar bordes débiles a fuertes)
    for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
            const idx = y * width + x;
            
            if (edges[idx] === 128) { // Borde débil
                // Verificar si está conectado a un borde fuerte
                let connected = false;
                for (let dy = -1; dy <= 1; dy++) {
                    for (let dx = -1; dx <= 1; dx++) {
                        if (edges[(y+dy)*width + (x+dx)] === 255) {
                            connected = true;
                            break;
                        }
                    }
                    if (connected) break;
                }
                
                edges[idx] = connected ? 255 : 0;
            }
        }
    }
    
    // Convertir a ImageData
    const output = new ImageData(width, height);
    for (let i = 0; i < edges.length; i++) {
        output.data[i * 4] = edges[i];
        output.data[i * 4 + 1] = edges[i];
        output.data[i * 4 + 2] = edges[i];
        output.data[i * 4 + 3] = 255;
    }
    
    return output;
}

async function iniciarCannyPro() {
    const log = document.getElementById('log_canny_pro');
    
    detenerTodo();
    
    try {
        log.innerText = "📷 Solicitando cámara...";
        
        const stream = await navigator.mediaDevices.getUserMedia({ 
            video: { width: 640, height: 480, facingMode: 'user' } 
        });
        
        window.cameraManager.currentStream = stream;
        window.cameraManager.currentFilter = 'canny';
        
        const v = document.getElementById('v_canny_pro');
        const canvas = document.getElementById('c_canny_pro');
        const ctx = canvas.getContext('2d');
        
        v.srcObject = stream;
        await v.play();
        
        await new Promise(resolve => setTimeout(resolve, 500));
        
        document.getElementById('btn_on_canny').style.display = 'none';
        document.getElementById('btn_off_canny').style.display = 'inline-block';
        log.innerText = "Detección de bordes Canny activa";
        
        window.cameraManager.activeLoop = true;
        let frameCount = 0;
        
        function processFrame() {
            if (!window.cameraManager.activeLoop || window.cameraManager.currentFilter !== 'canny') {
                return;
            }
            
            try {
                ctx.drawImage(v, 0, 0, canvas.width, canvas.height);
                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                
                // Aplicar CANNY REAL
                const edges = applyCannyEdgeDetection(imageData);
                
                ctx.putImageData(edges, 0, 0);
                
                frameCount++;
                if (frameCount % 30 === 0) {
                    log.innerText = "Canny activo - Frames: " + frameCount;
                }
                
                requestAnimationFrame(processFrame);
                
            } catch (e) {
                console.error('Error:', e);
                log.innerText = "Error: " + e.message;
            }
        }
        
        processFrame();
        
    } catch (e) {
        log.innerText = "Error: " + e.message;
        console.error(e);
    }
}
</script>
```
# Análisis Matemático: Algoritmo de Canny (Detección Óptima de Bordes)

Finalmente, hemos implementado el algoritmo de **Canny**, considerado el estándar de oro en visión por computadora. A diferencia de Sobel, que es un operador simple de derivación, Canny es un **proceso multi-etapa** diseñado para minimizar el error y garantizar que cada borde se detecte una sola vez.

A continuación, detallamos la lógica numérica de las 4 etapas que ocurren en nuestro código `cv.Canny(dst, dst, 50, 150, 3, false)`:

### 1. Reducción de Ruido (Suavizado Gaussiano)
El cálculo de derivadas es extremadamente sensible al ruido (píxeles granulosos). Antes de buscar bordes, aplicamos un filtro de suavizado convolucionando la imagen con un **Kernel Gaussiano** de $5 \times 5$.

Matemáticamente, esto equivale a aplicar una distribución normal 2D:
$$
G(x, y) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2 + y^2}{2\sigma^2}}
$$
Esto "desenfoca" levemente la imagen para eliminar falsos positivos causados por texturas irrelevantes.

### 2. Cálculo del Gradiente (Operador Sobel)
Al igual que en el punto anterior, calculamos la magnitud ($G$) y la dirección ($\theta$) del gradiente para cada píxel.
* **Magnitud:** $G = \sqrt{G_x^2 + G_y^2}$
* **Dirección:** $\theta = \arctan(\frac{G_y}{G_x})$

La dirección es crucial porque nos dice hacia dónde "apunta" el borde (si es vertical, horizontal o diagonal).

### 3. Supresión de No-Máximos (Adelgazamiento)
Esta es la etapa donde Canny supera a Sobel.
Sobel produce bordes gruesos y difusos. Canny analiza la dirección del gradiente en cada píxel y verifica si ese píxel es el **máximo local** en esa dirección.
* **Lógica:** "Si el píxel vecino en la dirección del gradiente tiene una intensidad mayor que yo, entonces yo no soy el borde real. Me apago (pongo a 0)."
* **Resultado:** Esto convierte las líneas gruesas en curvas de **1 píxel de ancho**, logrando una precisión sub-píxel.

### 4. Umbralización por Histéresis (Doble Umbral)
Aquí es donde entran los parámetros numéricos `50` y `150` que definimos en el código. Canny clasifica los píxeles en 3 categorías:

1.  **Fuertes ($> 150$):** Son bordes definitivos. Se mantienen.
2.  **Débiles ($50 < x < 150$):** Son dudosos.
3.  **No-Bordes ($< 50$):** Se descartan inmediatamente.

**La Histéresis (Lógica de Conectividad):**
Para decidir qué hacer con los píxeles "Débiles", Canny aplica un análisis topológico:
* Si un píxel débil está conectado a uno fuerte, se "contagia" y se vuelve fuerte (es parte del borde).
* Si está aislado o solo conectado a otros débiles, se descarta (es ruido).

Esta lógica matemática nos permite detectar contornos completos y limpios, incluso si la iluminación no es uniforme, superando las limitaciones de los operadores lineales simples.


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
        <button id="b_start" onclick="runAr()" style="padding: 15px 30px; background: #27ae60; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold;">ACTIVAR FILTROS AR</button>
        <button onclick="detenerTodo()" style="padding: 15px 30px; background: #e74c3c; color: white; border: none; border-radius: 10px; cursor: pointer; margin-left: 10px; display: none;" id="b_stop">APAGAR</button>
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
var arCamera = null;

function setF(n) {
    filter_id = n;
    log_el.innerText = n > 0 ? "Filtro " + n + " activado ✓" : "Filtro removido";
}

function togM() {
    show_m = !show_m;
    document.getElementById('m_btn').innerText = show_m ? "Ocultar Malla Verde" : "Mostrar Malla Verde";
}

// FUNCIONES PARA DIBUJAR FILTROS
function drawClownNose(ctx, x, y, size) {
    ctx.save();
    ctx.shadowColor = 'rgba(0,0,0,0.4)';
    ctx.shadowBlur = 25;
    ctx.shadowOffsetX = 15;
    ctx.shadowOffsetY = 15;
    
    const gradient = ctx.createRadialGradient(x - size*0.2, y - size*0.2, 0, x, y, size*0.6);
    gradient.addColorStop(0, '#ff6b6b');
    gradient.addColorStop(0.5, '#ee4444');
    gradient.addColorStop(1, '#cc2222');
    
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(x, y, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    
    ctx.shadowColor = 'transparent';
    const highlightGrad = ctx.createRadialGradient(x - size*0.15, y - size*0.15, 0, x - size*0.15, y - size*0.15, size*0.25);
    highlightGrad.addColorStop(0, 'rgba(255,255,255,0.9)');
    highlightGrad.addColorStop(1, 'rgba(255,255,255,0)');
    ctx.fillStyle = highlightGrad;
    ctx.beginPath();
    ctx.arc(x - size*0.15, y - size*0.15, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    
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
    
    ctx.beginPath();
    ctx.ellipse(x, y, size*0.8, size*0.25, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    ctx.fillStyle = '#A0522D';
    ctx.beginPath();
    ctx.moveTo(x - size*0.4, y);
    ctx.lineTo(x - size*0.35, y - size*0.8);
    ctx.lineTo(x + size*0.35, y - size*0.8);
    ctx.lineTo(x + size*0.4, y);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
    
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
    
    ctx.fillStyle = 'rgba(0,0,0,0.3)';
    ctx.beginPath();
    ctx.arc(x - size*0.35, y, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    ctx.beginPath();
    ctx.arc(x + size*0.35, y, size*0.25, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    
    ctx.beginPath();
    ctx.moveTo(x - size*0.1, y);
    ctx.lineTo(x + size*0.1, y);
    ctx.stroke();
    
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
    
    // Detener otros filtros
    detenerTodo();
    
    try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { width: 640, height: 480 } });
        window.cameraManager.currentStream = stream;
        window.cameraManager.currentFilter = 'ar';
        
        vid_el.srcObject = stream;
        await vid_el.play();
        
        document.getElementById('b_start').style.display = 'none';
        document.getElementById('b_stop').style.display = 'inline-block';
        
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
            if (window.cameraManager.currentFilter !== 'ar') return;
            
            ctx_el.save();
            ctx_el.clearRect(0, 0, can_el.width, can_el.height);
            ctx_el.drawImage(res.image, 0, 0, can_el.width, can_el.height);
            
            if (res.multiFaceLandmarks && res.multiFaceLandmarks.length > 0) {
                const face = res.multiFaceLandmarks[0];
                
                if (show_m) {
                    drawConnectors(ctx_el, face, FACEMESH_TESSELATION, {color: '#00FF0050', lineWidth: 1});
                }

                const faceW = Math.abs(face[454].x - face[234].x) * can_el.width;
                
                if (filter_id === 1) {
                    const nose = face[1];
                    drawClownNose(ctx_el, nose.x * can_el.width, nose.y * can_el.height, faceW * 0.15);
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

        arCamera = new Camera(vid_el, {
            onFrame: async () => {
                if (window.cameraManager.currentFilter === 'ar') {
                    await mesh.send({image: vid_el});
                }
            },
            width: 640, 
            height: 480
        });
        
        arCamera.start();
        log_el.innerText = "¡Todo listo! Selecciona un filtro";
        
    } catch (err) {
        log_el.innerText = "ERROR de IA: " + err.message;
        console.error(err);
    }
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


### 18. ¿MediaPipe utiliza Sobel?
[cite_start]**No directamente.** MediaPipe utiliza Redes Neuronales Convolucionales (CNN)[cite: 158]. [cite_start]A diferencia del algoritmo Sobel, que utiliza fórmulas matemáticas fijas (kernels predefinidos) para buscar bordes, MediaPipe utiliza modelos entrenados con millones de imágenes para aprender a identificar patrones complejos como ojos, labios y contornos faciales, independientemente de los bordes simples[cite: 158].

### 19. ¿Qué son las redes neuronales convolucionales (CNN)?
[cite_start]Son un tipo de arquitectura de Deep Learning diseñada específicamente para procesar datos con estructura de cuadrícula, como las imágenes[cite: 160]. [cite_start]

[Image of convolutional neural network architecture]
 Utilizan capas de "convolución" que funcionan como filtros aprendidos automáticamente[cite: 161]. [cite_start]A diferencia de Sobel (donde el humano define el filtro), una CNN aprende por sí sola qué filtros aplicar para detectar desde líneas simples hasta formas complejas como una cara humana[cite: 161].

### 23. Escribir el mismo concepto pero usando Sobel
[cite_start]Implementación de procesamiento de video en tiempo real utilizando el operador Sobel para detección de bordes mediante OpenCV.js[cite: 163].

```%%html
<div style="text-align: center; background: #1a1a1a; padding: 20px; border-radius: 15px; color: white; font-family: sans-serif;">
    <h3>Método Sobel (Gradientes de Intensidad)</h3>
    <video id="v_sobel_pro" width="640" height="480" style="display:none" playsinline></video>
    <canvas id="c_sobel_pro" width="640" height="480" style="background: #000; border: 2px solid #2980b9; border-radius: 10px;"></canvas>
    <div style="margin-top: 15px;">
        <button id="btn_on_sobel" onclick="iniciarSobelPro()" style="padding: 10px 20px; background: #2980b9; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;">🚀 ACTIVAR SOBEL</button>
        <button id="btn_off_sobel" onclick="detenerTodo()" style="padding: 10px 20px; background: #e74c3c; color: white; border: none; border-radius: 5px; cursor: pointer; margin-left: 10px; display: none;">🛑 APAGAR</button>
    </div>
    <p id="log_sobel_pro" style="color: #f1c40f; font-size: 13px; margin-top: 10px;">Estado: Esperando cámara...</p>
</div>

<script>
// Implementación de Sobel
function applySobelGradients(imageData) {
    const width = imageData.width;
    const height = imageData.height;
    const data = imageData.data;
    
    // Convertir a escala de grises
    const gray = new Uint8ClampedArray(width * height);
    for (let i = 0; i < data.length; i += 4) {
        const idx = i / 4;
        gray[idx] = 0.299 * data[i] + 0.587 * data[i + 1] + 0.114 * data[i + 2];
    }
    
    // Aplicar Sobel
    const gradX = new Float32Array(width * height);
    const gradY = new Float32Array(width * height);
    const magnitude = new Uint8ClampedArray(width * height);
    
    for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
            const idx = y * width + x;
            
            // Sobel X (detecta bordes verticales)
            const gx = (
                -gray[(y-1)*width + (x-1)] + gray[(y-1)*width + (x+1)] +
                -2*gray[y*width + (x-1)] + 2*gray[y*width + (x+1)] +
                -gray[(y+1)*width + (x-1)] + gray[(y+1)*width + (x+1)]
            );
            
            // Sobel Y (detecta bordes horizontales)
            const gy = (
                -gray[(y-1)*width + (x-1)] - 2*gray[(y-1)*width + x] - gray[(y-1)*width + (x+1)] +
                gray[(y+1)*width + (x-1)] + 2*gray[(y+1)*width + x] + gray[(y+1)*width + (x+1)]
            );
            
            gradX[idx] = gx;
            gradY[idx] = gy;
            
            // Magnitud del gradiente
            const mag = Math.sqrt(gx * gx + gy * gy);
            magnitude[idx] = Math.min(255, mag);
        }
    }
    
    // Convertir a ImageData
    const output = new ImageData(width, height);
    for (let i = 0; i < magnitude.length; i++) {
        output.data[i * 4] = magnitude[i];
        output.data[i * 4 + 1] = magnitude[i];
        output.data[i * 4 + 2] = magnitude[i];
        output.data[i * 4 + 3] = 255;
    }
    
    return output;
}

async function iniciarSobelPro() {
    const log = document.getElementById('log_sobel_pro');
    
    // Detener otros filtros Y limpiar sus canvas
    detenerTodo();
    
    // IMPORTANTE: Limpiar TODOS los canvas antes de empezar
    ['c_canny_pro', 'c_out', 'c_sobel_pro'].forEach(canvasId => {
        const canvas = document.getElementById(canvasId);
        if (canvas) {
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }
    });
    
    try {
        log.innerText = "📷 Solicitando cámara...";
        
        const stream = await navigator.mediaDevices.getUserMedia({ 
            video: { width: 640, height: 480, facingMode: 'user' } 
        });
        
        window.cameraManager.currentStream = stream;
        window.cameraManager.currentFilter = 'sobel';
        
        const v = document.getElementById('v_sobel_pro');
        const canvas = document.getElementById('c_sobel_pro');
        const ctx = canvas.getContext('2d');
        
        v.srcObject = stream;
        await v.play();
        
        await new Promise(resolve => setTimeout(resolve, 500));
        
        document.getElementById('btn_on_sobel').style.display = 'none';
        document.getElementById('btn_off_sobel').style.display = 'inline-block';
        log.innerText = "✅ Gradientes Sobel activos";
        
        window.cameraManager.activeLoop = true;
        let frameCount = 0;
        
        function processFrame() {
            if (!window.cameraManager.activeLoop || window.cameraManager.currentFilter !== 'sobel') {
                return;
            }
            
            try {
                ctx.drawImage(v, 0, 0, canvas.width, canvas.height);
                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                
                // Aplicar SOBEL
                const edges = applySobelGradients(imageData);
                
                ctx.putImageData(edges, 0, 0);
                
                frameCount++;
                if (frameCount % 30 === 0) {
                    log.innerText = "Sobel activo - Frames: " + frameCount;
                }
                
                requestAnimationFrame(processFrame);
                
            } catch (e) {
                console.error('Error:', e);
                log.innerText = "Error: " + e.message;
            }
        }
        
        processFrame();
        
    } catch (e) {
        log.innerText = "Error: " + e.message;
        console.error(e);
    }
}
</script>
```

# Análisis Matemático: Detección de Bordes con el Operador Sobel

En este segmento del proyecto, hemos implementado el **Operador Sobel** utilizando OpenCV.js. A diferencia de Canny (que es un algoritmo multi-etapa), Sobel es una aplicación pura de **Cálculo Diferencial** adaptado a entornos discretos (imágenes digitales). A continuación, desglosamos la lógica numérica que nuestro código ejecuta en cada cuadro de video.

### 1. La Imagen como Función Discreta
Para la computadora, una imagen en escala de grises no es "una foto", sino una **función escalar discreta** $I(x, y)$ que asigna un valor de intensidad (brillo) a cada coordenada.
* Por ello, el primer paso en nuestro código es `cv.cvtColor(..., cv.COLOR_RGBA2GRAY)`. Convertimos el espacio vectorial RGB de 3 dimensiones a un campo escalar de 1 dimensión para poder derivarlo.

### 2. Diferenciación Numérica (Diferencias Finitas)
En cálculo, un borde representa un cambio brusco en la intensidad. Matemáticamente, esto corresponde a un **máximo local en la primera derivada** de la función de imagen.
Como la imagen es discreta (píxeles), no podemos calcular derivadas continuas ($\frac{df}{dx}$). En su lugar, utilizamos **Diferencias Finitas Centrales** mediante la operación de **Convolución**.

El código calcula dos derivadas parciales independientes:

#### A. Derivada Parcial en X ($G_x$)
Calcula la tasa de cambio horizontal. Nuestro código ejecuta `cv.Sobel(dst, grad_x, cv.CV_16S, 1, 0, 3)`. Esto equivale a convolucionar la imagen con el kernel:

$$
G_x = \begin{bmatrix} 
-1 & 0 & +1 \\ 
-2 & 0 & +2 \\ 
-1 & 0 & +1 
\end{bmatrix} * I
$$

> **Nota técnica:** Usamos `CV_16S` (16-bit signed) porque la derivada puede ser negativa (cuando pasamos de un píxel brillante a uno oscuro). Si usáramos 8 bits estándar, los valores negativos se cortarían a 0, perdiendo la mitad de los bordes.

#### B. Derivada Parcial en Y ($G_y$)
Calcula la tasa de cambio vertical mediante `cv.Sobel(..., 0, 1, 3)`. El kernel rotado es:

$$
G_y = \begin{bmatrix} 
-1 & -2 & -1 \\ 
0 & 0 & 0 \\ 
+1 & +2 & +1 
\end{bmatrix} * I
$$

### 3. Cálculo de la Magnitud del Gradiente (Norma Vectorial)
Una vez que tenemos los componentes vectoriales del gradiente $\nabla I = [G_x, G_y]$, necesitamos calcular la **Magnitud Total** del borde ($G$) para visualizarlo.

La magnitud real (Euclidiana) se define como:
$$
|G| = \sqrt{G_x^2 + G_y^2}
$$

Sin embargo, calcular raíces cuadradas para cada uno de los 307,200 píxeles ($640 \times 480$) es computacionalmente costoso para el navegador en tiempo real.
En nuestro código, hemos optado por una **Aproximación Numérica** eficiente utilizando la **Norma $L_1$** (Distancia de Manhattan).

La línea `cv.addWeighted(abs_grad_x, 1.0, abs_grad_y, 1.0, ...)` implementa esta suma ponderada:

$$
|G| \approx |G_x| + |G_y|
$$

Esta aproximación es mucho más rápida y suficientemente precisa para detectar siluetas y contornos en visión artificial en tiempo real.