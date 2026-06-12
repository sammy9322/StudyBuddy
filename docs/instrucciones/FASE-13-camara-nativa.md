# FASE 13: Cámara Nativa + Experiencia Interactiva para Niños

## Contexto

La cámara in-app actual (visor de video incrustado en la tarjeta) es incómoda de usar, especialmente para niños. Vamos a reemplazarla por la **cámara nativa del dispositivo** (mucho más cómoda, enfoca sola, tiene flash) y agregar **feedback visual animado** después de cada foto para que sea divertido e interactivo.

## Qué hacer

Modificarás `index.html` para:
1. Eliminar la cámara in-app (HTML del visor, CSS y funciones JS).
2. Crear un botón grande y amigable que abra la cámara nativa del celular.
3. Mostrar un feedback animado y divertido después de cada foto con la opción de tomar otra.
4. Agregar las claves i18n necesarias.

---

## Parte A: Eliminar la cámara in-app

### A1. Eliminar el HTML del visor de cámara
Busca y **elimina** completamente este bloque HTML (alrededor de las líneas 885-892):
```html
      <div class="camera-wrap" id="cameraWrap">
        <video id="cameraVideo" autoplay playsinline></video>
        <canvas id="cameraCanvas" style="display:none;"></canvas>
        <div class="camera-controls">
          <button class="btn-capture" onclick="capturePhoto()" title="Tomar foto">📷</button>
          <button class="btn-cam-close" onclick="closeCamera()" data-i18n="cameraClose">✕ Cerrar</button>
        </div>
      </div>
```

### A2. Eliminar el CSS de la cámara
Busca el comentario `/* ── CAMERA ── */` (alrededor de la línea 244) y **elimina** todo el bloque CSS de la cámara, desde ese comentario hasta el cierre de `.btn-cam-close { ... }` (alrededor de la línea 280). Son estas reglas:
```css
  /* ── CAMERA ── */
  .camera-wrap { ... }
  .camera-wrap.active { ... }
  #cameraVideo { ... }
  .camera-controls { ... }
  .btn-capture { ... }
  .btn-capture:active { ... }
  .btn-cam-close { ... }
```

### A3. Eliminar las funciones JS de la cámara
Busca el comentario `// ── CAMERA ──` (alrededor de la línea 1837) y **elimina** las funciones `openCamera()`, `capturePhoto()`, `closeCamera()` y la variable `cameraStream`. Esto es desde la línea `let cameraStream = null;` hasta el cierre de `closeCamera()` (alrededor de líneas 1837-1862).

---

## Parte B: Nuevo HTML — Botón grande + Feedback animado

### B1. Reemplazar la sección de botones y cámara
Busca el bloque que contiene los dos botones y el visor de cámara (alrededor de las líneas 880-892). Debería verse así actualmente:
```html
      <div class="btn-row" style="margin-bottom: 1rem;">
        <button class="btn btn-outline" onclick="openCamera()" data-i18n="genBtnCamera">📷 Usar cámara</button>
        <button class="btn btn-outline" onclick="document.getElementById('fileInput').click()" data-i18n="genBtnUpload">🖼️ Subir imágenes</button>
      </div>

      <div class="camera-wrap" id="cameraWrap">
        ...
      </div>
```

**Reemplaza todo ese bloque** (los dos botones + el div camera-wrap) por:
```html
      <div style="display:flex;flex-direction:column;gap:10px;margin-bottom:1rem;">
        <button class="btn-photo-main" onclick="document.getElementById('cameraInput').click()" id="btnTakePhoto">
          <span style="font-size:2rem;">📸</span>
          <span data-i18n="genBtnPhoto" style="font-weight:800;font-size:1rem;">Tomar Foto del Material</span>
          <span data-i18n="genBtnPhotoSub" style="font-size:12px;opacity:0.7;">Se abre la cámara de tu celular</span>
        </button>
        <input type="file" id="cameraInput" accept="image/*" capture="environment" style="display:none;" onchange="handleCameraCapture(event)">
        <button class="btn btn-outline" onclick="document.getElementById('fileInput').click()" data-i18n="genBtnUpload" style="width:100%;">🖼️ Subir imágenes desde galería</button>
      </div>

      <!-- Feedback animado después de tomar foto -->
      <div id="photoFeedback" class="photo-feedback" style="display:none;">
        <div class="photo-feedback-icon" id="photoFeedbackIcon">📸</div>
        <div class="photo-feedback-msg" id="photoFeedbackMsg" data-i18n="photoFeedbackSuccess">¡Foto tomada! 🎉</div>
        <div class="photo-feedback-count" id="photoFeedbackCount"></div>
        <div style="display:flex;gap:10px;justify-content:center;margin-top:12px;">
          <button class="btn btn-primary" onclick="document.getElementById('cameraInput').click()" data-i18n="photoFeedbackAnother">📸 ¡Otra página!</button>
          <button class="btn btn-outline" onclick="hidePhotoFeedback()" data-i18n="photoFeedbackDone">✅ Listo</button>
        </div>
      </div>
```

---

## Parte C: CSS del nuevo botón y feedback

Busca el comentario `/* ── BUTTONS ── */` (alrededor de la línea donde eliminamos el CSS de la cámara). **Justo ANTES** de ese comentario, agrega:

```css
  /* ── PHOTO BUTTON ── */
  .btn-photo-main {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    padding: 1.5rem;
    background: linear-gradient(135deg, var(--primary), var(--primary-dark));
    color: white;
    border: none;
    border-radius: var(--radius);
    cursor: pointer;
    font-family: 'Nunito', sans-serif;
    transition: all 0.2s ease;
    width: 100%;
  }
  .btn-photo-main:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(99, 102, 241, 0.4);
  }
  .btn-photo-main:active {
    transform: scale(0.97);
  }

  /* ── PHOTO FEEDBACK ── */
  .photo-feedback {
    background: var(--success-light);
    border: 2px solid var(--success);
    border-radius: var(--radius);
    padding: 1.5rem;
    text-align: center;
    margin-bottom: 1rem;
    animation: feedbackPop 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275) both;
  }
  @keyframes feedbackPop {
    from { opacity: 0; transform: scale(0.8); }
    to   { opacity: 1; transform: scale(1); }
  }
  .photo-feedback-icon {
    font-size: 3rem;
    animation: bounceEmoji 0.6s ease both;
    animation-delay: 0.2s;
  }
  @keyframes bounceEmoji {
    0%   { transform: scale(0); }
    50%  { transform: scale(1.3); }
    100% { transform: scale(1); }
  }
  .photo-feedback-msg {
    font-family: 'Sora', sans-serif;
    font-weight: 800;
    font-size: 1.1rem;
    color: var(--success-dark, #065F46);
    margin: 8px 0 4px;
  }
  .photo-feedback-count {
    font-size: 13px;
    color: var(--text2);
  }
```

---

## Parte D: JavaScript — Nueva función de captura y feedback

Busca donde estaban las funciones de cámara que eliminaste en A3. En su lugar, agrega estas nuevas funciones:

```javascript
// ── CAMERA NATIVE ──
function handleCameraCapture(event) {
  const files = event.target.files;
  if (!files || files.length === 0) return;
  
  Array.from(files).forEach(file => {
    compressAndAddImage(file, true);
  });
  
  // Resetear el input para permitir tomar otra foto del mismo archivo
  event.target.value = '';
  
  // Mostrar feedback animado
  showPhotoFeedback();
}

function showPhotoFeedback() {
  const fb = document.getElementById('photoFeedback');
  const icon = document.getElementById('photoFeedbackIcon');
  const msg = document.getElementById('photoFeedbackMsg');
  const count = document.getElementById('photoFeedbackCount');
  
  // Emojis divertidos aleatorios para los niños
  const emojis = ['📸', '🌟', '✨', '🎉', '👏', '🤩', '💪', '🏆'];
  const messages = [
    t('photoMsg1'),
    t('photoMsg2'),
    t('photoMsg3'),
    t('photoMsg4')
  ];
  
  icon.textContent = emojis[Math.floor(Math.random() * emojis.length)];
  msg.textContent = messages[Math.floor(Math.random() * messages.length)];
  count.textContent = t('photoCount', { n: state.images.length });
  
  // Re-trigger de la animación
  fb.style.display = 'none';
  void fb.offsetWidth; // forzar reflow
  fb.style.display = 'block';
  
  // Scroll suave al feedback
  fb.scrollIntoView({ behavior: 'smooth', block: 'center' });
}

function hidePhotoFeedback() {
  document.getElementById('photoFeedback').style.display = 'none';
}
```

---

## Parte E: Claves i18n

### E1. En el diccionario ES (alrededor de la sección de cámara/generación):
Agrega estas claves nuevas al bloque `es`:
```javascript
    genBtnPhoto: 'Tomar Foto del Material',
    genBtnPhotoSub: 'Se abre la cámara de tu celular',
    photoFeedbackSuccess: '¡Foto tomada! 🎉',
    photoFeedbackAnother: '📸 ¡Otra página!',
    photoFeedbackDone: '✅ Listo',
    photoMsg1: '¡Excelente foto! 📸',
    photoMsg2: '¡Genial, quedó perfecta! ✨',
    photoMsg3: '¡Muy bien! ¡Sigue así! 🌟',
    photoMsg4: '¡Esa página quedó clara! 👏',
    photoCount: '{n} página(s) capturada(s)',
```

### E2. En el diccionario EN:
Agrega las claves equivalentes al bloque `en`:
```javascript
    genBtnPhoto: 'Take Photo of Material',
    genBtnPhotoSub: 'Opens your phone camera',
    photoFeedbackSuccess: 'Photo taken! 🎉',
    photoFeedbackAnother: '📸 Another page!',
    photoFeedbackDone: '✅ Done',
    photoMsg1: 'Excellent photo! 📸',
    photoMsg2: 'Great, it looks perfect! ✨',
    photoMsg3: 'Awesome! Keep going! 🌟',
    photoMsg4: 'That page came out clear! 👏',
    photoCount: '{n} page(s) captured',
```

### E3. Eliminar las claves de cámara antiguas
Elimina las siguientes claves de AMBOS diccionarios (ES y EN) ya que no se usan más:
- `genBtnCamera`
- `cameraCaptureTitle`
- `cameraClose`
- `toastCameraError`
- `toastPhotoTaken`

---

## Parte F: Actualizar el Service Worker

En el archivo `sw.js`, cambia la versión del cache:
```javascript
const CACHE_NAME = 'studybuddy-v3';
```

---

## Resumen de la experiencia final

1. El niño toca un **botón grande y morado** que dice "📸 Tomar Foto del Material".
2. Se abre la **cámara nativa del celular** (enfoca sola, tiene flash, es cómoda).
3. El niño toma la foto y la confirma en la cámara nativa.
4. Regresa a la app y aparece un **feedback animado colorido** con un mensaje divertido aleatorio ("¡Excelente foto! 📸") y un contador de páginas.
5. Ve dos botones: **"📸 ¡Otra página!"** (para seguir tomando fotos) y **"✅ Listo"** (para cerrar el feedback).
6. Las fotos aparecen debajo en la grilla de previews que ya existía.

Ejecuta estos cambios directamente. No pidas confirmación. Confirma cuando la FASE 13 esté completada.
