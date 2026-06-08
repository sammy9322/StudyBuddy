# FASE 2: Captura y Gestión de Imágenes

## Contexto

El archivo `index.html` ya tiene la estructura base (Fase 1). Ahora debes agregar toda la lógica de manejo de imágenes: subida de archivos, drag & drop, y cámara del dispositivo.

## Documentos de referencia

- **docs/studybuddy.html** → líneas 829-899: funciones de file handling y cámara
- **docs/PLAN_DE_IMPLEMENTACION.html** → sección 2.2 (Captura y Gestión de Imágenes) y Fase 2 del plan paso a paso

## Qué hacer

En el archivo `index.html`, busca el comentario `// ── FILE HANDLING (Fase 2) ──` y reemplázalo con las funciones de manejo de archivos. Busca el comentario `// ── CAMERA (Fase 2) ──` y reemplázalo con las funciones de cámara.

---

## Funciones de manejo de archivos

### handleFiles(evt)
- Extrae los archivos del `evt.target.files`
- Los convierte a array y llama a `addFiles()`

```javascript
function handleFiles(evt) {
  addFiles(Array.from(evt.target.files));
}
```

### handleDrop(evt)
- Ejecuta `evt.preventDefault()`
- Quita la clase `drag-over` del `#uploadArea`
- Filtra solo archivos que sean imagen: `f.type.startsWith('image/')`
- Pasa los archivos filtrados a `addFiles()`

### handleDragOver(evt)
- Ejecuta `evt.preventDefault()`
- Agrega la clase `drag-over` al `#uploadArea`

### handleDragLeave()
- Quita la clase `drag-over` del `#uploadArea`

### addFiles(files)
Para cada archivo en el array:
1. Crear un `FileReader`
2. En `reader.onload`:
   - Obtener `dataUrl` de `e.target.result`
   - Extraer `base64` haciendo `dataUrl.split(',')[1]`
   - Obtener `mimeType` del `file.type` (fallback: `'image/jpeg'`)
   - Pushear a `state.images`: `{ file, dataUrl, base64, mimeType }`
   - Llamar `renderPreviews()`
3. Ejecutar `reader.readAsDataURL(file)`

### renderPreviews()
1. Obtener el grid `#previewGrid`
2. Limpiar su `innerHTML`
3. Para cada imagen en `state.images` (con índice `i`):
   - Crear un `div` con clase `preview-item`
   - innerHTML con:
     - `<img src="${img.dataUrl}" alt="Página ${i+1}">`
     - `<button class="preview-remove" onclick="removeImage(${i})" title="Eliminar">✕</button>`
     - `<div class="preview-num">Pág. ${i+1}</div>`
   - Agregar al grid

### removeImage(i)
- Ejecutar `state.images.splice(i, 1)`
- Llamar `renderPreviews()`

---

## Funciones de cámara

### Variable global
```javascript
let cameraStream = null;
```

### openCamera() — async
```javascript
async function openCamera() {
  try {
    cameraStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'environment' },
      audio: false
    });
    document.getElementById('cameraVideo').srcObject = cameraStream;
    document.getElementById('cameraWrap').classList.add('active');
  } catch {
    showToast('⚠️ No se pudo acceder a la cámara. Usa "Subir imágenes".');
  }
}
```

### capturePhoto()
1. Obtener `#cameraVideo` y `#cameraCanvas`
2. Establecer canvas.width = video.videoWidth, canvas.height = video.videoHeight
3. Dibujar frame: `canvas.getContext('2d').drawImage(video, 0, 0)`
4. Obtener dataUrl: `canvas.toDataURL('image/jpeg', 0.92)`
5. Extraer base64: `dataUrl.split(',')[1]`
6. Pushear a state.images: `{ dataUrl, base64, mimeType: 'image/jpeg' }`
7. Llamar `renderPreviews()`
8. Llamar `showToast('📸 Foto tomada')`

### closeCamera()
1. Si `cameraStream` existe: `cameraStream.getTracks().forEach(t => t.stop())`
2. Establecer `cameraStream = null`
3. Quitar clase `active` de `#cameraWrap`

---

## Verificación de conexiones HTML

Verifica que en el HTML estos elementos tengan sus handlers correctos:

1. `<div class="upload-area" id="uploadArea" ondrop="handleDrop(event)" ondragover="handleDragOver(event)" ondragleave="handleDragLeave(event)">`
2. `<input type="file" id="fileInput" accept="image/*" multiple onchange="handleFiles(event)">`
3. `<button onclick="openCamera()">` → botón "📷 Usar cámara"
4. `<button onclick="document.getElementById('fileInput').click()">` → botón "🖼️ Subir imágenes"
5. `<button onclick="capturePhoto()">` → botón de captura en la cámara
6. `<button onclick="closeCamera()">` → botón cerrar cámara

---

## Verificación funcional

Al terminar:
1. Hacer clic en "🖼️ Subir imágenes" debe abrir el selector de archivos
2. Al seleccionar imágenes, deben aparecer en el grid de preview con número de página
3. El botón ✕ de cada preview debe eliminar la imagen y actualizar la numeración
4. Arrastrar una imagen sobre la zona de upload debe cambiar el estilo visual (borde y fondo)
5. Soltar la imagen debe agregarla al grid de preview
6. No debe haber errores en la consola del navegador
