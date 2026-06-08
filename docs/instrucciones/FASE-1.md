# FASE 1: Estructura Base, Diseño CSS y Configuración

## Contexto

Estás construyendo **StudyBuddy**, una app web educativa para niños de 6-15 años. La app permite fotografiar material escolar (libros, cuadernos) y generar resúmenes, quizzes, flashcards y mapas conceptuales usando la API de Google Gemini.

## Documentos de referencia

Antes de escribir código, LEE estos dos archivos completos:

1. **docs/studybuddy.html** → El HTML de referencia con TODOS los estilos CSS y la estructura de la interfaz. Este es tu blueprint visual.
2. **docs/PLAN_DE_IMPLEMENTACION.html** → El plan técnico completo con especificaciones de API, manejo de errores, tokens CSS, etc.

## Skill a usar

Lee y aplica la skill en `.agents/skills/frontend-design/SKILL.md`. El tono debe ser **PLAYFUL / TOY-LIKE**: alegre, colorido, amigable. El público son niños y sus padres.

## Qué crear

Crea el archivo `index.html` en la **raíz del proyecto** (es decir, en `c:\Documentos\samuelss\Mis documentos\app\index.html`).

Este archivo contendrá TODA la aplicación en un solo archivo (HTML + CSS + JS embebidos). No uses frameworks ni dependencias externas.

---

## Parte A: Head del documento

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>StudyBuddy ✏️</title>
  <!-- Google Fonts: Nunito (body) y Sora (titles) -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900&family=Sora:wght@400;600;700&display=swap" rel="stylesheet">
```

---

## Parte B: CSS completo en bloque `<style>`

Copia TODOS los estilos del archivo de referencia `docs/studybuddy.html` (líneas 9 a 545). Esto incluye:

### Variables CSS (`:root`)
```
--primary: #4F46E5       (indigo - acciones principales)
--primary-light: #EEF2FF (fondos hover, badges)
--primary-dark: #3730A3  (hover botones, títulos)
--accent: #F59E0B        (amber - botones secundarios)
--accent-light: #FEF3C7  (highlights, warnings)
--success: #10B981       (respuestas correctas)
--success-light: #D1FAE5
--danger: #EF4444        (respuestas incorrectas)
--danger-light: #FEE2E2
--surface: #FFFFFF       (fondo cards)
--surface2: #F8F7FF      (fondo general app)
--border: #E5E7EB        (bordes)
--text: #1F2937          (texto principal)
--text2: #6B7280         (texto secundario)
--text3: #9CA3AF         (texto terciario)
--radius: 16px           (cards)
--radius-sm: 10px        (inputs, botones)
--shadow: 0 4px 24px rgba(79,70,229,0.10)
--shadow-card: 0 2px 12px rgba(0,0,0,0.07)
```

### Componentes CSS a incluir (todos están en el HTML de referencia)
- Reset: `* { box-sizing: border-box; margin: 0; padding: 0; }`
- `.header` — sticky, flex, height 64px, borde inferior
- `.logo` — flex, font Sora, color primary
- `.logo-icon` — 36x36, fondo primary, border-radius 10px
- `.header-badge` — pill con fondo primary-light
- `.container` — max-width 860px, centrado
- `.tabs` — flex, fondo blanco, borde, border-radius 14px, padding 5px
- `.tab` — flex 1, sin borde, font Nunito 14px bold, cursor pointer, transición
- `.tab.active` — fondo primary, color blanco
- `.section` — display none
- `.section.active` — display block
- `.card` — fondo blanco, borde, border-radius 16px, padding 1.5rem, sombra
- `.card-title` — font Sora, flex con gap 8px
- `.form-row` — grid 2 columnas, gap 1rem
- `.form-group` — flex column, gap 6px
- `.form-label` — 13px, uppercase, letter-spacing
- `.form-control` — padding 10px 14px, borde, border-radius 10px, focus cambia borde
- `.api-key-wrap` — posición relativa para toggle
- `.upload-area` — borde dashed, padding 2.5rem, cursor pointer, hover cambia borde
- `.upload-area.drag-over` — mismos estilos que hover
- `.preview-grid` — grid auto-fill minmax(110px, 1fr)
- `.preview-item` — posición relativa, aspect-ratio 3/4, overflow hidden
- `.preview-remove` — posición absoluta, fondo semitransparente, hover rojo
- `.camera-wrap` — posición relativa, fondo negro, display none
- `.camera-wrap.active` — display block
- `.btn` — inline-flex, padding 12px 22px, font Nunito 15px bold, sin borde
- `.btn-primary`, `.btn-outline`, `.btn-accent`, `.btn-success` — variantes
- `.btn-sm`, `.btn-full`, `.btn-icon` — modificadores
- `.level-pills` — flex, gap 8px
- `.level-pill` — padding 6px 16px, border-radius 20px, borde, cursor pointer
- `.level-pill.active` — fondo primary, color blanco
- `.progress-bar-wrap` — fondo gris, height 8px, border-radius 8px
- `.progress-bar` — gradiente primary→accent, transición width
- `.result-block`, `.result-header`, `.result-body` — bloque de resultados
- `.quiz-progress-info`, `.quiz-question`, `.quiz-options`, `.quiz-option` — quiz
- `.quiz-option.correct` — borde y fondo verde
- `.quiz-option.wrong` — borde y fondo rojo
- `.quiz-feedback` — oculto, con clases `.correct-fb` y `.wrong-fb`
- `.score-display`, `.score-circle`, `.score-number` — resultados quiz
- `.history-item` — flex, borde, cursor pointer, hover con sombra
- `.chip`, `.chip-primary`, `.chip-accent`, `.chip-success`
- `.empty-state` — centrado, padding 3rem
- `.alert`, `.alert-warn`, `.alert-info`
- `.divider` — 1px, fondo border
- `.section-title` — font Sora 1.3rem bold
- `.spinner` — animación spin, 20x20, borde giratorio
- `@media (max-width: 600px)` — header padding reducido, container padding reducido, form-row 1 columna, tab font-size 12px, tab-emoji display none
- `@keyframes fadeUp` — de opacity 0 / translateY 12px a normal
- `.animate-in` — usa fadeUp 0.35s

---

## Parte C: HTML del Body

### C1. Header
```html
<header class="header">
  <div class="logo">
    <div class="logo-icon">📖</div>
    StudyBuddy
  </div>
  <span class="header-badge">✨ Con IA</span>
</header>
```

### C2. Container con Tabs
```html
<div class="container">
  <!-- TABS -->
  <div class="tabs" role="tablist">
    <button class="tab active" onclick="showTab('inicio')" id="tab-inicio">
      <span class="tab-emoji">🏠</span> Inicio
    </button>
    <button class="tab" onclick="showTab('generar')" id="tab-generar">
      <span class="tab-emoji">✨</span> Generar
    </button>
    <button class="tab" onclick="showTab('quiz')" id="tab-quiz">
      <span class="tab-emoji">🎯</span> Practicar
    </button>
    <button class="tab" onclick="showTab('historial')" id="tab-historial">
      <span class="tab-emoji">📂</span> Historial
    </button>
  </div>
```

### C3. Sección INICIO (id="section-inicio", class="section active")

Contiene 3 cards:

**Card 1 — Bienvenida:** Con fondo gradiente `linear-gradient(135deg, #4F46E5 0%, #7C3AED 100%)`, borde none, color blanco. Contenido: emoji 👋 grande, título "¡Hola, estudioso!", párrafo descriptivo, botón CTA blanco "Empezar ahora →" que llama `showTab('generar')`.

**Card 2 — Configuración:** Con card-title "⚙️ Configuración". Contiene:
- Form row con 2 campos:
  - "Nombre del estudiante" → input text, id="studentName", placeholder "Ej: Sofía"
  - "Grado / Nivel" → select id="studentGrade" con 3 options:
    - value="primaria_baja": "1° - 3° Primaria (6-9 años)"
    - value="primaria_alta" selected: "4° - 6° Primaria (9-12 años)"
    - value="secundaria": "Secundaria (12-15 años)"
- Form group "API Key de Gemini":
  - div.api-key-wrap con input password id="apiKey" placeholder "AIza..."
  - Botón toggle 👁 que llama toggleApiKey()
  - Link a https://aistudio.google.com/apikey con texto "🔑 Obtener API Key gratis en Google AI Studio →"
- Alert info: "💡 Tu API Key se guarda solo en tu navegador. Nunca se envía a ningún servidor nuestro."
- Botón "💾 Guardar configuración" que llama saveConfig()

**Card 3 — Cómo funciona:** Con card-title "📋 ¿Cómo funciona?". 3 pasos visuales con iconos en divs de 36x36 con fondos de colores:
1. 📸 fondo primary-light → "Fotografía el material"
2. ✨ fondo accent-light → "La IA analiza el contenido"
3. 🎯 fondo success-light → "Practica con el quiz"

### C4. Sección GENERAR (id="section-generar", class="section")

- section-title "✨ Generar material"
- section-sub "Sube fotos del libro o cuaderno y elige qué generar."

**Card imágenes:** card-title "📸 Imágenes del material"
- btn-row con 2 botones outline: "📷 Usar cámara" (onclick openCamera) y "🖼️ Subir imágenes" (onclick click en fileInput)
- div.camera-wrap#cameraWrap con video#cameraVideo (autoplay playsinline), canvas#cameraCanvas (display none), controles con btn-capture y btn-cam-close
- div.upload-area#uploadArea con handlers ondrop/ondragover/ondragleave. Dentro: input file#fileInput (accept image/*, multiple, onchange handleFiles), icono 📂, título "Arrastra las fotos aquí", sub "o haz clic para seleccionar • JPG, PNG, WEBP"
- div.preview-grid#previewGrid (vacío)

**Card opciones:** card-title "🎛️ Opciones de generación"
- Form row: Materia (input id="subject") y Tema específico (input id="topic")
- Form group "¿Qué quieres generar?" con div.level-pills#genTypePills:
  - pill active data-val="resumen": "📝 Resumen"
  - pill active data-val="quiz": "❓ Quiz (10 preguntas)"
  - pill active data-val="flashcards": "🗂️ Flashcards"
  - pill data-val="mapa": "🧠 Mapa conceptual"
- Form group "Notas adicionales (opcional)": textarea#extraNotes rows=2
- div#genProgress (display none): status-msg#genStatusMsg + progress-bar-wrap con progress-bar#genBar (width 10%)
- Botón primary full "✨ Generar con IA" id="genBtn" onclick generateContent()

- div#genResults (display none) — vacío, se llenará en Fase 4

### C5. Sección QUIZ (id="section-quiz", class="section")
- section-title "🎯 Practicar", section-sub "Pon a prueba lo que aprendiste."
- div#quizContainer con empty-state: icono 🎯, título "Aún no hay quiz disponible", sub con link a Generar, botón "Ir a Generar →"

### C6. Sección HISTORIAL (id="section-historial", class="section")
- section-title "📂 Historial", section-sub "Sesiones guardadas de estudio."
- div#historialContainer con empty-state: icono 📂, título "Sin historial aún", sub "Las sesiones generadas aparecerán aquí."

Cierra container y body.

---

## Parte D: JavaScript en bloque `<script>`

### D1. Estado global
```javascript
const state = {
  images: [],        // { file, dataUrl, base64, mimeType }
  config: {},        // { name, grade, apiKey }
  currentQuiz: null, // array de preguntas
  quizIndex: 0,
  quizScore: 0,
  quizAnswered: false,
  history: [],       // array de sesiones
  genTypes: new Set(['resumen', 'quiz', 'flashcards'])
};
```

### D2. IIFE de inicialización
```javascript
(function init() {
  // Cargar config de localStorage
  const saved = localStorage.getItem('sb_config');
  if (saved) {
    state.config = JSON.parse(saved);
    if (state.config.name) document.getElementById('studentName').value = state.config.name;
    if (state.config.grade) document.getElementById('studentGrade').value = state.config.grade;
    if (state.config.apiKey) document.getElementById('apiKey').value = state.config.apiKey;
  }

  // Cargar historial de localStorage
  const hist = localStorage.getItem('sb_history');
  if (hist) {
    state.history = JSON.parse(hist);
    renderHistory();
  }

  // Configurar pills multi-select
  document.querySelectorAll('#genTypePills .level-pill').forEach(p => {
    const val = p.dataset.val;
    if (state.genTypes.has(val)) p.classList.add('active');
    p.addEventListener('click', () => {
      if (state.genTypes.has(val)) {
        // No permitir deseleccionar si es el último
        if (state.genTypes.size > 1) {
          state.genTypes.delete(val);
          p.classList.remove('active');
        }
      } else {
        state.genTypes.add(val);
        p.classList.add('active');
      }
    });
  });
})();
```

### D3. Navegación
```javascript
function showTab(tab) {
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.getElementById('section-' + tab).classList.add('active');
  document.getElementById('tab-' + tab).classList.add('active');
}
```

### D4. Configuración
```javascript
function saveConfig() {
  state.config = {
    name: document.getElementById('studentName').value.trim(),
    grade: document.getElementById('studentGrade').value,
    apiKey: document.getElementById('apiKey').value.trim()
  };
  localStorage.setItem('sb_config', JSON.stringify(state.config));
  showToast('✅ Configuración guardada');
}

function toggleApiKey() {
  const inp = document.getElementById('apiKey');
  inp.type = inp.type === 'password' ? 'text' : 'password';
}
```

### D5. Utilidades base
```javascript
function escHtml(str) {
  return String(str || '').replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
}

let toastTimeout;
function showToast(msg) {
  clearTimeout(toastTimeout);
  let toast = document.getElementById('sb-toast');
  if (!toast) {
    toast = document.createElement('div');
    toast.id = 'sb-toast';
    toast.style.cssText = 'position:fixed;bottom:24px;left:50%;transform:translateX(-50%);background:#1F2937;color:white;padding:10px 20px;border-radius:20px;font-size:14px;font-weight:700;z-index:9999;opacity:0;transition:opacity 0.2s;pointer-events:none;white-space:nowrap;font-family:Nunito,sans-serif;';
    document.body.appendChild(toast);
  }
  toast.textContent = msg;
  toast.style.opacity = '1';
  toastTimeout = setTimeout(() => { toast.style.opacity = '0'; }, 2800);
}
```

### D6. Placeholders para fases siguientes
```javascript
// ── FILE HANDLING (Fase 2) ──

// ── CAMERA (Fase 2) ──

// ── GENERATE (Fase 3) ──

// ── RENDER RESULTS (Fase 4) ──

// ── QUIZ (Fase 5) ──

// ── HISTORY (Fase 6) ──
function renderHistory() {
  // Placeholder temporal — se implementará en Fase 6
  // Necesario para que init() no falle
  const c = document.getElementById('historialContainer');
  if (!state.history.length) return;
}
```

---

## Verificación

Al terminar, el archivo `index.html` debe:
1. Abrirse directamente en el navegador sin errores en consola
2. Mostrar las 4 pestañas y navegar entre ellas
3. Mostrar la sección Inicio completa con las 3 cards
4. Guardar y cargar configuración desde localStorage
5. Mostrar toast al guardar configuración
6. La sección Generar debe mostrar la zona de upload y las opciones (aunque sin funcionalidad aún)
7. Las pills de tipo de generación deben hacer toggle al hacer clic
8. Ser responsive en pantallas menores a 600px
