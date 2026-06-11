# FASE 11: Sistema Bilingüe Español / Inglés (i18n)

## Contexto

StudyBuddy se usa en escuelas y colegios bilingües. Los estudiantes manejan materias en español (Español, Estudios Sociales) y en inglés (Science, Math). Necesitamos un sistema que permita alternar TODA la experiencia entre español e inglés: la interfaz de usuario, los prompts a Gemini y el contenido generado.

La app es un único archivo `index.html` (~1870 líneas) con CSS y JS embebidos. No usamos frameworks ni dependencias externas. El sistema de i18n debe ser igualmente ligero y autocontenido.

## Documentos de referencia

- **index.html** → El archivo principal que será modificado
- **docs/instrucciones/FASE-1.md** → Referencia de la estructura HTML y componentes CSS
- **docs/instrucciones/FASE-3.md** → Referencia de la función `generateContent()` y el prompt de Gemini

## Skill a usar

Lee y aplica la skill en `.agents/skills/frontend-design/SKILL.md`. El tono visual debe ser **PLAYFUL / TOY-LIKE**: alegre, colorido, amigable. El switch de idioma debe sentirse como un juguete divertido, no como un control corporativo.

Después de implementar, aplica la skill `.agents/skills/simplify/SKILL.md` para refinar el código resultante.

## Archivo a modificar

| Archivo | Qué cambia |
|---------|------------|
| `index.html` | Se agregan: diccionario i18n, switch en header, función `setLanguage()`, atributos `data-i18n` en HTML, idioma en perfiles, prompt bilingüe en `generateContent()` |

**NO se crean archivos nuevos.** Todo se implementa dentro de `index.html`.

---

## Parte A: Diccionario de traducciones

### A1. Definir el objeto `i18n`

En el bloque `<script>`, **ANTES** de la declaración de `const state = {`, agrega el diccionario completo de traducciones:

```javascript
// ── i18n: DICCIONARIO DE TRADUCCIONES ──
const i18n = {
  es: {
    // ── Header ──
    headerBadge: '✨ Con IA',
    
    // ── Tabs ──
    tabInicio: 'Inicio',
    tabGenerar: 'Generar',
    tabPracticar: 'Practicar',
    tabHistorial: 'Historial',
    
    // ── Inicio: Bienvenida ──
    welcomeEmoji: '👋',
    welcomeTitle: '¡Hola, estudioso!',
    welcomeDesc: 'Toma una foto de tu libro o cuaderno y yo genero resúmenes y prácticas automáticamente. ¡Estudiar nunca fue tan fácil!',
    welcomeCta: 'Empezar ahora →',
    
    // ── Inicio: PWA Install Card ──
    pwaTitle: '¡Lleva StudyBuddy en tu celular!',
    pwaDesc: 'Instala la aplicación para abrirla al instante y estudiar a pantalla completa.',
    pwaBtnInstall: 'Instalar App',
    pwaBtnClose: 'Cerrar',
    pwaIosInstructions: 'Para instalar en tu iPhone/iPad:<br>1. Presiona el botón de <strong>Compartir 📤</strong> en Safari.<br>2. Desliza hacia abajo y selecciona <strong>"Agregar a inicio" 📲</strong>.',
    pwaAndroidInstructions: 'Instala StudyBuddy en tu celular para tener un acceso rápido y usarla a pantalla completa.',
    pwaManualInstructions: 'Para instalarla, abre el menú del navegador (los tres puntos <strong>⋮</strong> arriba o abajo) y selecciona <strong>"Instalar aplicación"</strong> o <strong>"Agregar a la pantalla de inicio"</strong>.',
    
    // ── Inicio: Perfiles ──
    profilesTitle: '👤 Gestión de Estudiantes',
    profilesAdd: '➕ Agregar Estudiante',
    profilesEmpty: 'No hay estudiantes registrados. Agrega uno para empezar.',
    profileFormTitleAdd: 'Agregar Estudiante',
    profileFormTitleEdit: 'Editar Estudiante',
    profileNameLabel: 'Nombre del estudiante',
    profileNamePlaceholder: 'Ej: Sofía',
    profileGradeLabel: 'Grado / Nivel',
    profileGradeLow: '1° - 3° Primaria (6-9 años)',
    profileGradeMid: '4° - 6° Primaria (9-12 años)',
    profileGradeHigh: 'Secundaria (12-15 años)',
    profileGradeLowShort: '1°-3° Primaria',
    profileGradeMidShort: '4°-6° Primaria',
    profileGradeHighShort: 'Secundaria',
    profileGradeLowAge: '6-9 años',
    profileGradeMidAge: '9-12 años',
    profileGradeHighAge: '12-15 años',
    profileCancel: 'Cancelar',
    profileSave: '💾 Guardar Estudiante',
    profileLangLabel: 'Idioma del material',
    
    // ── Inicio: API Key ──
    apiKeyTitle: '🔑 API Key Familiar (Google AI Studio)',
    apiKeyPlaceholder: 'AIza...',
    apiKeyLink: '🔑 Obtener API Key gratis en Google AI Studio →',
    apiKeyInfo: '💡 Tu API Key se guarda solo en tu navegador y es compartida por todos los perfiles de estudiantes de la familia.',
    apiKeySave: '💾 Guardar API Key',
    
    // ── Inicio: Cómo funciona ──
    howTitle: '📋 ¿Cómo funciona?',
    howStep1Title: '1. Fotografía el material',
    howStep1Desc: 'Toma foto(s) de las páginas del libro o cuaderno con tu celular.',
    howStep2Title: '2. La IA analiza el contenido',
    howStep2Desc: 'Gemini lee el texto de la imagen y genera un resumen adaptado al nivel del niño.',
    howStep3Title: '3. Practica con el quiz',
    howStep3Desc: 'Responde preguntas generadas automáticamente para reforzar lo aprendido.',
    
    // ── Generar ──
    genTitle: '✨ Generar material',
    genSub: 'Sube fotos del libro o cuaderno y elige qué generar.',
    genImagesTitle: '📸 Imágenes del material',
    genBtnCamera: '📷 Usar cámara',
    genBtnUpload: '🖼️ Subir imágenes',
    genDropTitle: 'Arrastra las fotos aquí',
    genDropSub: 'o haz clic para seleccionar • JPG, PNG, WEBP',
    genOptionsTitle: '🎛️ Opciones de generación',
    genSubjectLabel: 'Materia',
    genSubjectPlaceholder: 'Ej: Ciencias Naturales',
    genTopicLabel: 'Tema específico',
    genTopicPlaceholder: 'Ej: El sistema solar',
    genTypeLabel: '¿Qué quieres generar?',
    genTypeResumen: '📝 Resumen',
    genTypeQuiz: '❓ Quiz (10 preguntas)',
    genTypeFlash: '🗂️ Flashcards',
    genTypeMapa: '🧠 Mapa conceptual',
    genNotesLabel: 'Notas adicionales (opcional)',
    genNotesPlaceholder: 'Ej: El examen es mañana, enfócate en los conceptos clave...',
    genBtn: '✨ Generar con IA',
    genBtnLoading: 'Generando...',
    genStatusReading: 'Leyendo las imágenes...',
    genStatusSending: 'Enviando imágenes a la IA...',
    genStatusConnecting: 'Conectando con la IA...',
    genStatusRetry: 'La IA está muy solicitada. Reintentando en breve...',
    genStatusProcessing: 'Procesando respuesta...',
    
    // ── Resultados ──
    resultResumen: '📝 Resumen',
    resultQuiz: '❓ Quiz',
    resultFlashcards: '🗂️ Flashcards',
    resultMapa: '🧠 Mapa conceptual',
    resultCopy: '📋 Copiar',
    resultPrint: '🖨️ Imprimir',
    resultStartQuiz: '🎯 Iniciar Quiz',
    resultQuizReady: 'El quiz tiene {n} preguntas de opción múltiple. ¡Pon a prueba lo que aprendiste!',
    resultFlashHint: '👆 Haz clic en cada tarjeta para ver la respuesta',
    
    // ── Quiz ──
    quizTitle: '🎯 Practicar',
    quizSub: 'Pon a prueba lo que aprendiste.',
    quizEmpty: 'Aún no hay quiz disponible',
    quizEmptySub: 'Genera contenido desde la pestaña <strong>Generar</strong> con la opción Quiz activada.',
    quizEmptyCta: 'Ir a Generar →',
    quizQuestion: 'Pregunta {current} de {total}',
    quizCorrectCount: '⭐ {n} correctas',
    quizCorrect: '✅ ¡Correcto! ',
    quizWrong: '❌ Incorrecto. ',
    quizWrongAnswer: 'La respuesta correcta es: {answer}',
    quizNext: 'Siguiente pregunta →',
    quizSeeResults: 'Ver resultados 🎉',
    quizRetry: '🔄 Volver a intentar',
    quizSeeMaterial: '📚 Ver material',
    quizScoreExcellent: '¡Excelente trabajo!',
    quizScoreGreat: '¡Muy bien hecho!',
    quizScoreGood: '¡Buen intento!',
    quizScoreKeepGoing: '¡Sigue practicando!',
    quizScorePct: '{pct}% de respuestas correctas',
    quizScoreOf: 'de {total}',
    
    // ── Historial ──
    histTitle: '📂 Historial',
    histSub: 'Sesiones guardadas de estudio.',
    histEmpty: 'Sin historial aún',
    histEmptySub: 'Las sesiones generadas aparecerán aquí.',
    
    // ── Toasts ──
    toastConfigSaved: '✅ Configuración guardada',
    toastApiKeySaved: '✅ API Key guardada',
    toastProfileSaved: '💾 Estudiante guardado',
    toastProfileDeleted: '🗑️ Perfil eliminado',
    toastProfileActive: 'Estudiante activo: {name}',
    toastNoName: '⚠️ Ingresa el nombre del estudiante',
    toastNoProfile: '⚠️ Primero agrega un Estudiante en Configuración',
    toastNoApiKey: '⚠️ Primero configura tu API Key en la sección de Configuración',
    toastNoImages: '⚠️ Sube al menos una imagen',
    toastPhotoTaken: '📸 Foto tomada',
    toastCameraError: '⚠️ No se pudo acceder a la cámara. Usa "Subir imágenes".',
    toastCopied: '📋 ¡Copiado!',
    toastCopyFail: 'No se pudo copiar',
    toastNoQuiz: 'No hay quiz disponible aún',
    toastError: '❌ Error: ',
    toastParseError: 'La IA devolvió una respuesta inesperada. Intenta de nuevo.',
    toastPwaInstalled: '🎉 ¡StudyBuddy instalada! Búscala en tu pantalla de inicio.',
    
    // ── Confirmar borrar perfil ──
    confirmDeleteProfile: '¿Estás seguro de que quieres eliminar el perfil de {name}? Se perderá todo su historial.',
    
    // ── Camera ──
    cameraCaptureTitle: 'Tomar foto',
    cameraClose: '✕ Cerrar',
    
    // ── Preview ──
    previewPage: 'Pág. {n}',
    previewRemoveTitle: 'Eliminar',
    
    // ── PWA Banner ──
    pwaBannerTitle: 'Instalar StudyBuddy',
    pwaBannerSub: 'Agrégala a tu pantalla de inicio para acceso rápido',
    pwaBannerBtn: 'Instalar',
    
    // ── Gemini Prompt Fragments ──
    promptRole: 'Eres un tutor educativo experto.',
    promptInstruction: 'Analiza las imágenes de material escolar (libro/cuaderno) y genera el siguiente contenido para {name}, {gradeLabel}.',
    promptSubject: 'Materia: {subject}. Tema: {topic}.',
    promptSubjectDefault: 'General',
    promptTopicDefault: 'el contenido de las imágenes',
    promptNotes: 'Notas del padre/tutor: {notes}',
    promptHandwriting: 'Ten en cuenta que el material escolar puede incluir texto manuscrito (escrito a mano). Si encuentras caligrafía difícil de leer o trazos ambiguos, usa el contexto educativo del tema para deducir las palabras correctas de forma lógica y coherente.',
    promptTone: 'Usa un lenguaje claro, amigable y apropiado para la edad. Responde SIEMPRE en español.',
    promptJsonInstruction: 'Devuelve la respuesta en JSON con esta estructura exacta (sin markdown, solo JSON puro):',
    promptQuizInstruction: 'Genera exactamente 10 preguntas de opción múltiple variadas. respuesta_correcta es el índice (0-3) de la opción correcta.',
    promptFlashInstruction: 'Genera al menos 8 flashcards con los conceptos clave.',
    promptGradeLabels: {
      primaria_baja: 'niño de 6 a 9 años, 1°-3° primaria',
      primaria_alta: 'niño de 9 a 12 años, 4°-6° primaria',
      secundaria: 'adolescente de 12 a 15 años, secundaria'
    },
    
    // ── Tipos para el historial ──
    typeLabels: { resumen: 'Resumen', quiz: 'Quiz', flashcards: 'Flashcards', mapa: 'Mapa' },
    
    // ── Print ──
    printTitle: 'StudyBuddy - Material de estudio',
    
    // ── Retrying ──
    retryMsg: 'Reintentando con {model} (intento {attempt}/{max})...',
    retryAllFailed: 'No se pudo conectar con los servidores de la IA después de varios intentos.',
    
    // ── Date locale ──
    dateLocale: 'es-CR'
  },
  
  en: {
    // ── Header ──
    headerBadge: '✨ AI Powered',
    
    // ── Tabs ──
    tabInicio: 'Home',
    tabGenerar: 'Generate',
    tabPracticar: 'Practice',
    tabHistorial: 'History',
    
    // ── Inicio: Bienvenida ──
    welcomeEmoji: '👋',
    welcomeTitle: 'Hello, study buddy!',
    welcomeDesc: 'Take a photo of your textbook or notebook and I\'ll automatically generate summaries and practice exercises. Studying has never been so easy!',
    welcomeCta: 'Get started →',
    
    // ── Inicio: PWA Install Card ──
    pwaTitle: 'Take StudyBuddy with you!',
    pwaDesc: 'Install the app to open it instantly and study in full screen.',
    pwaBtnInstall: 'Install App',
    pwaBtnClose: 'Close',
    pwaIosInstructions: 'To install on your iPhone/iPad:<br>1. Tap the <strong>Share 📤</strong> button in Safari.<br>2. Scroll down and select <strong>"Add to Home Screen" 📲</strong>.',
    pwaAndroidInstructions: 'Install StudyBuddy on your phone for quick access and full screen mode.',
    pwaManualInstructions: 'To install, open the browser menu (the three dots <strong>⋮</strong>) and select <strong>"Install app"</strong> or <strong>"Add to Home Screen"</strong>.',
    
    // ── Inicio: Perfiles ──
    profilesTitle: '👤 Student Management',
    profilesAdd: '➕ Add Student',
    profilesEmpty: 'No students registered. Add one to get started.',
    profileFormTitleAdd: 'Add Student',
    profileFormTitleEdit: 'Edit Student',
    profileNameLabel: 'Student name',
    profileNamePlaceholder: 'E.g., Sofia',
    profileGradeLabel: 'Grade / Level',
    profileGradeLow: '1st - 3rd Grade (ages 6-9)',
    profileGradeMid: '4th - 6th Grade (ages 9-12)',
    profileGradeHigh: 'Middle School (ages 12-15)',
    profileGradeLowShort: '1st-3rd Grade',
    profileGradeMidShort: '4th-6th Grade',
    profileGradeHighShort: 'Middle School',
    profileGradeLowAge: 'Ages 6-9',
    profileGradeMidAge: 'Ages 9-12',
    profileGradeHighAge: 'Ages 12-15',
    profileCancel: 'Cancel',
    profileSave: '💾 Save Student',
    profileLangLabel: 'Content language',
    
    // ── Inicio: API Key ──
    apiKeyTitle: '🔑 Family API Key (Google AI Studio)',
    apiKeyPlaceholder: 'AIza...',
    apiKeyLink: '🔑 Get a free API Key from Google AI Studio →',
    apiKeyInfo: '💡 Your API Key is stored only in your browser and is shared across all student profiles in your family.',
    apiKeySave: '💾 Save API Key',
    
    // ── Inicio: Cómo funciona ──
    howTitle: '📋 How does it work?',
    howStep1Title: '1. Photograph the material',
    howStep1Desc: 'Take photo(s) of the textbook or notebook pages with your phone.',
    howStep2Title: '2. AI analyzes the content',
    howStep2Desc: 'Gemini reads the text from the image and generates a summary adapted to your child\'s level.',
    howStep3Title: '3. Practice with the quiz',
    howStep3Desc: 'Answer automatically generated questions to reinforce what you learned.',
    
    // ── Generar ──
    genTitle: '✨ Generate material',
    genSub: 'Upload photos of the textbook or notebook and choose what to generate.',
    genImagesTitle: '📸 Material images',
    genBtnCamera: '📷 Use camera',
    genBtnUpload: '🖼️ Upload images',
    genDropTitle: 'Drag photos here',
    genDropSub: 'or click to browse • JPG, PNG, WEBP',
    genOptionsTitle: '🎛️ Generation options',
    genSubjectLabel: 'Subject',
    genSubjectPlaceholder: 'E.g., Natural Sciences',
    genTopicLabel: 'Specific topic',
    genTopicPlaceholder: 'E.g., The solar system',
    genTypeLabel: 'What do you want to generate?',
    genTypeResumen: '📝 Summary',
    genTypeQuiz: '❓ Quiz (10 questions)',
    genTypeFlash: '🗂️ Flashcards',
    genTypeMapa: '🧠 Concept map',
    genNotesLabel: 'Additional notes (optional)',
    genNotesPlaceholder: 'E.g., The test is tomorrow, focus on key concepts...',
    genBtn: '✨ Generate with AI',
    genBtnLoading: 'Generating...',
    genStatusReading: 'Reading the images...',
    genStatusSending: 'Sending images to AI...',
    genStatusConnecting: 'Connecting to AI...',
    genStatusRetry: 'AI is busy. Retrying shortly...',
    genStatusProcessing: 'Processing response...',
    
    // ── Resultados ──
    resultResumen: '📝 Summary',
    resultQuiz: '❓ Quiz',
    resultFlashcards: '🗂️ Flashcards',
    resultMapa: '🧠 Concept map',
    resultCopy: '📋 Copy',
    resultPrint: '🖨️ Print',
    resultStartQuiz: '🎯 Start Quiz',
    resultQuizReady: 'The quiz has {n} multiple-choice questions. Test what you learned!',
    resultFlashHint: '👆 Click each card to see the answer',
    
    // ── Quiz ──
    quizTitle: '🎯 Practice',
    quizSub: 'Test what you learned.',
    quizEmpty: 'No quiz available yet',
    quizEmptySub: 'Generate content from the <strong>Generate</strong> tab with the Quiz option enabled.',
    quizEmptyCta: 'Go to Generate →',
    quizQuestion: 'Question {current} of {total}',
    quizCorrectCount: '⭐ {n} correct',
    quizCorrect: '✅ Correct! ',
    quizWrong: '❌ Incorrect. ',
    quizWrongAnswer: 'The correct answer is: {answer}',
    quizNext: 'Next question →',
    quizSeeResults: 'See results 🎉',
    quizRetry: '🔄 Try again',
    quizSeeMaterial: '📚 View material',
    quizScoreExcellent: 'Excellent work!',
    quizScoreGreat: 'Great job!',
    quizScoreGood: 'Good try!',
    quizScoreKeepGoing: 'Keep practicing!',
    quizScorePct: '{pct}% correct answers',
    quizScoreOf: 'of {total}',
    
    // ── Historial ──
    histTitle: '📂 History',
    histSub: 'Saved study sessions.',
    histEmpty: 'No history yet',
    histEmptySub: 'Generated sessions will appear here.',
    
    // ── Toasts ──
    toastConfigSaved: '✅ Settings saved',
    toastApiKeySaved: '✅ API Key saved',
    toastProfileSaved: '💾 Student saved',
    toastProfileDeleted: '🗑️ Profile deleted',
    toastProfileActive: 'Active student: {name}',
    toastNoName: '⚠️ Enter the student\'s name',
    toastNoProfile: '⚠️ First add a Student in Settings',
    toastNoApiKey: '⚠️ First set up your API Key in the Settings section',
    toastNoImages: '⚠️ Upload at least one image',
    toastPhotoTaken: '📸 Photo taken',
    toastCameraError: '⚠️ Could not access camera. Use "Upload images".',
    toastCopied: '📋 Copied!',
    toastCopyFail: 'Could not copy',
    toastNoQuiz: 'No quiz available yet',
    toastError: '❌ Error: ',
    toastParseError: 'The AI returned an unexpected response. Try again.',
    toastPwaInstalled: '🎉 StudyBuddy installed! Find it on your home screen.',
    
    // ── Confirmar borrar perfil ──
    confirmDeleteProfile: 'Are you sure you want to delete {name}\'s profile? All their history will be lost.',
    
    // ── Camera ──
    cameraCaptureTitle: 'Take photo',
    cameraClose: '✕ Close',
    
    // ── Preview ──
    previewPage: 'Pg. {n}',
    previewRemoveTitle: 'Remove',
    
    // ── PWA Banner ──
    pwaBannerTitle: 'Install StudyBuddy',
    pwaBannerSub: 'Add it to your home screen for quick access',
    pwaBannerBtn: 'Install',
    
    // ── Gemini Prompt Fragments ──
    promptRole: 'You are an expert educational tutor.',
    promptInstruction: 'Analyze the images of school material (textbook/notebook) and generate the following content for {name}, {gradeLabel}.',
    promptSubject: 'Subject: {subject}. Topic: {topic}.',
    promptSubjectDefault: 'General',
    promptTopicDefault: 'the content of the images',
    promptNotes: 'Parent/tutor notes: {notes}',
    promptHandwriting: 'Keep in mind that the school material may include handwritten text. If you find difficult-to-read handwriting or ambiguous strokes, use the educational context of the topic to logically and coherently deduce the correct words.',
    promptTone: 'Use clear, friendly, and age-appropriate language. ALWAYS respond in English.',
    promptJsonInstruction: 'Return the response in JSON with this exact structure (no markdown, only pure JSON):',
    promptQuizInstruction: 'Generate exactly 10 varied multiple-choice questions. respuesta_correcta is the index (0-3) of the correct option.',
    promptFlashInstruction: 'Generate at least 8 flashcards with the key concepts.',
    promptGradeLabels: {
      primaria_baja: 'child aged 6 to 9, 1st-3rd grade elementary',
      primaria_alta: 'child aged 9 to 12, 4th-6th grade elementary',
      secundaria: 'teenager aged 12 to 15, middle school'
    },
    
    // ── Tipos para el historial ──
    typeLabels: { resumen: 'Summary', quiz: 'Quiz', flashcards: 'Flashcards', mapa: 'Concept map' },
    
    // ── Print ──
    printTitle: 'StudyBuddy - Study material',
    
    // ── Retrying ──
    retryMsg: 'Retrying with {model} (attempt {attempt}/{max})...',
    retryAllFailed: 'Could not connect to AI servers after multiple attempts.',
    
    // ── Date locale ──
    dateLocale: 'en-US'
  }
};
```

### NOTAS CRÍTICAS SOBRE EL DICCIONARIO:
1. Las claves del JSON que Gemini devuelve (`resumen`, `quiz`, `flashcards`, `mapa_conceptual`, `titulo`, `pregunta`, `opciones`, `respuesta_correcta`, `explicacion`, `frente`, `reverso`) **NUNCA cambian de idioma**. Son nombres internos del código. Solo el CONTENIDO que Gemini escribe dentro de esos campos cambia de idioma.
2. Las plantillas con `{variable}` se reemplazan en runtime con la función helper `t()`.

---

## Parte B: Función helper `t()` y `setLanguage()`

### B1. Agregar la función `t()` (traducir)

**Inmediatamente DESPUÉS** del cierre del objeto `i18n`, agrega:

```javascript
// ── i18n: FUNCIONES DE TRADUCCIÓN ──
function t(key, replacements = {}) {
  const lang = state.lang || 'es';
  let text = i18n[lang]?.[key] ?? i18n['es'][key] ?? key;
  for (const [k, v] of Object.entries(replacements)) {
    text = text.replace(new RegExp(`\\{${k}\\}`, 'g'), v);
  }
  return text;
}

function tNested(key, subkey) {
  const lang = state.lang || 'es';
  return i18n[lang]?.[key]?.[subkey] ?? i18n['es']?.[key]?.[subkey] ?? subkey;
}
```

### B2. Agregar `lang` al estado global

Modifica la declaración de `const state = { ... }` para agregar la propiedad `lang`:

**Busca:**
```javascript
const state = {
  images: [],       // { file, dataUrl, base64, mimeType }
  apiKey: '',       // API Key global de la familia
  profiles: [],     // Array de perfiles: { id, name, grade, history }
  activeProfileId: '', // ID del perfil activo
  currentQuiz: null,
  quizIndex: 0,
  quizScore: 0,
  quizAnswered: false,
  history: [],
  genTypes: new Set(['resumen','quiz','flashcards'])
};
```

**Reemplaza con:**
```javascript
const state = {
  images: [],       // { file, dataUrl, base64, mimeType }
  apiKey: '',       // API Key global de la familia
  lang: 'es',      // Idioma activo: 'es' o 'en'
  profiles: [],     // Array de perfiles: { id, name, grade, history, lang }
  activeProfileId: '', // ID del perfil activo
  currentQuiz: null,
  quizIndex: 0,
  quizScore: 0,
  quizAnswered: false,
  history: [],
  genTypes: new Set(['resumen','quiz','flashcards'])
};
```

### B3. Agregar la función `setLanguage()`

**Después** de la función `t()` y `tNested()`, agrega:

```javascript
function setLanguage(lang) {
  state.lang = lang;
  localStorage.setItem('sb_lang', lang);
  document.documentElement.lang = lang;

  // Actualizar todos los elementos con data-i18n
  document.querySelectorAll('[data-i18n]').forEach(el => {
    const key = el.dataset.i18n;
    const attr = el.dataset.i18nAttr; // Para atributos como placeholder, title
    if (attr) {
      el.setAttribute(attr, t(key));
    } else {
      el.innerHTML = t(key);
    }
  });

  // Actualizar placeholders con data-i18n-ph
  document.querySelectorAll('[data-i18n-ph]').forEach(el => {
    el.placeholder = t(el.dataset.i18nPh);
  });

  // Actualizar el switch visual
  updateLangSwitch();

  // Actualizar las opciones del select de grado
  const gradeSelect = document.getElementById('studentGrade');
  if (gradeSelect) {
    gradeSelect.options[0].text = t('profileGradeLow');
    gradeSelect.options[1].text = t('profileGradeMid');
    gradeSelect.options[2].text = t('profileGradeHigh');
  }

  // Re-renderizar componentes dinámicos
  renderProfilesList();
  updateActiveProfileHeader();
  renderHistory();
}

function updateLangSwitch() {
  const switchEl = document.getElementById('langSwitch');
  if (!switchEl) return;
  const knob = switchEl.querySelector('.lang-knob');
  const labelEs = switchEl.querySelector('.lang-label-es');
  const labelEn = switchEl.querySelector('.lang-label-en');
  if (state.lang === 'en') {
    knob.style.transform = 'translateX(100%)';
    labelEs.style.opacity = '0.5';
    labelEn.style.opacity = '1';
  } else {
    knob.style.transform = 'translateX(0)';
    labelEs.style.opacity = '1';
    labelEn.style.opacity = '0.5';
  }
}
```

---

## Parte C: CSS del Language Switch

### C1. Agregar estilos del switch

Busca el comentario `/* ── SISTEMA DE PERFILES (PREMIUM) ── */` en el bloque `<style>`.

**ANTES** de ese comentario, agrega este bloque CSS:

```css
  /* ── LANGUAGE SWITCH ── */
  .lang-switch {
    display: flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    user-select: none;
    -webkit-tap-highlight-color: transparent;
  }
  .lang-switch-track {
    position: relative;
    width: 56px;
    height: 28px;
    background: var(--primary-light);
    border-radius: 14px;
    border: 1.5px solid var(--border);
    display: flex;
    align-items: center;
    padding: 0 3px;
    transition: background 0.25s ease;
  }
  .lang-switch:hover .lang-switch-track {
    border-color: var(--primary);
    box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
  }
  .lang-knob {
    width: 22px;
    height: 22px;
    border-radius: 50%;
    background: var(--primary);
    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    box-shadow: 0 2px 6px rgba(79, 70, 229, 0.35);
    position: absolute;
    left: 3px;
  }
  .lang-label {
    font-size: 12px;
    font-weight: 800;
    font-family: 'Sora', sans-serif;
    transition: opacity 0.2s ease;
    letter-spacing: 0.02em;
  }
  .lang-label-es { color: var(--primary); }
  .lang-label-en { color: var(--primary); }
```

---

## Parte D: HTML del Language Switch en el Header

### D1. Agregar el switch al header

Busca esta línea en el HTML:

```html
  <div style="display: flex; align-items: center; gap: 8px;">
    <div id="activeProfileHeader" class="active-profile-header-pill" style="display: none;" onclick="showTab('inicio')"></div>
    <span class="header-badge">✨ Con IA</span>
  </div>
```

**Reemplaza con:**

```html
  <div style="display: flex; align-items: center; gap: 10px;">
    <div id="activeProfileHeader" class="active-profile-header-pill" style="display: none;" onclick="showTab('inicio')"></div>
    <div id="langSwitch" class="lang-switch" onclick="toggleLanguage()" title="Cambiar idioma / Switch language">
      <span class="lang-label lang-label-es">ES</span>
      <div class="lang-switch-track">
        <div class="lang-knob">🌐</div>
      </div>
      <span class="lang-label lang-label-en" style="opacity: 0.5;">EN</span>
    </div>
    <span class="header-badge" data-i18n="headerBadge">✨ Con IA</span>
  </div>
```

### D2. Agregar la función toggleLanguage()

**Después** de la función `setLanguage()`, agrega:

```javascript
function toggleLanguage() {
  const newLang = state.lang === 'es' ? 'en' : 'es';
  setLanguage(newLang);

  // Si hay un perfil activo, actualizar su idioma también
  const activeProf = state.profiles.find(p => p.id === state.activeProfileId);
  if (activeProf) {
    activeProf.lang = newLang;
    localStorage.setItem('sb_profiles', JSON.stringify(state.profiles));
  }
}
```

---

## Parte E: Agregar atributos `data-i18n` al HTML estático

### REGLA IMPORTANTE:
Solo se agregan atributos `data-i18n` a elementos cuyo contenido sea texto fijo (no dinámico). Los textos que se generan con JavaScript en runtime se traducen usando `t('clave')` directamente en el código JS.

### E1. Tabs

Busca los 4 botones de tab:

```html
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
```

**Reemplaza con:**

```html
    <button class="tab active" onclick="showTab('inicio')" id="tab-inicio">
      <span class="tab-emoji">🏠</span> <span data-i18n="tabInicio">Inicio</span>
    </button>
    <button class="tab" onclick="showTab('generar')" id="tab-generar">
      <span class="tab-emoji">✨</span> <span data-i18n="tabGenerar">Generar</span>
    </button>
    <button class="tab" onclick="showTab('quiz')" id="tab-quiz">
      <span class="tab-emoji">🎯</span> <span data-i18n="tabPracticar">Practicar</span>
    </button>
    <button class="tab" onclick="showTab('historial')" id="tab-historial">
      <span class="tab-emoji">📂</span> <span data-i18n="tabHistorial">Historial</span>
    </button>
```

### E2. Card de Bienvenida

Busca la card de bienvenida con el gradiente morado en `section-inicio`. Agrega `data-i18n` a los textos estáticos internos:

**Reemplaza la card de bienvenida completa:**

```html
    <div class="card animate-in" style="background: linear-gradient(135deg, #4F46E5 0%, #7C3AED 100%); border: none; color: white;">
      <div style="font-size: 2.5rem; margin-bottom: 0.75rem;">👋</div>
      <div style="font-family: 'Sora', sans-serif; font-size: 1.4rem; font-weight: 700; margin-bottom: 0.5rem;" data-i18n="welcomeTitle">¡Hola, estudioso!</div>
      <p style="opacity: 0.85; font-size: 15px; line-height: 1.6; margin-bottom: 1.25rem;" data-i18n="welcomeDesc">
        Toma una foto de tu libro o cuaderno y yo genero resúmenes y prácticas automáticamente. ¡Estudiar nunca fue tan fácil!
      </p>
      <button class="btn" style="background: white; color: var(--primary); font-weight: 700;" onclick="showTab('generar')" data-i18n="welcomeCta">
        Empezar ahora →
      </button>
    </div>
```

### E3. Card de Perfiles

Busca el título de la card de perfiles:

```html
        <span>👤 Gestión de Estudiantes</span>
```

**Reemplaza con:**

```html
        <span data-i18n="profilesTitle">👤 Gestión de Estudiantes</span>
```

Busca el botón "Agregar Estudiante":

```html
        <button class="btn btn-primary" onclick="openProfileForm()" id="btnAddProfile" style="padding: 6px 12px; font-size: 13px; font-weight: 600; width: auto; margin-top: 0;">➕ Agregar Estudiante</button>
```

**Reemplaza con:**

```html
        <button class="btn btn-primary" onclick="openProfileForm()" id="btnAddProfile" style="padding: 6px 12px; font-size: 13px; font-weight: 600; width: auto; margin-top: 0;" data-i18n="profilesAdd">➕ Agregar Estudiante</button>
```

### E4. Formulario del perfil — Agregar campo de idioma

Busca el form-row dentro de `profileFormWrapper`:

```html
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">Nombre del estudiante</label>
            <input class="form-control" id="studentName" type="text" placeholder="Ej: Sofía" value="">
          </div>
          <div class="form-group">
            <label class="form-label">Grado / Nivel</label>
            <select class="form-control" id="studentGrade">
              <option value="primaria_baja">1° - 3° Primaria (6-9 años)</option>
              <option value="primaria_alta" selected>4° - 6° Primaria (9-12 años)</option>
              <option value="secundaria">Secundaria (12-15 años)</option>
            </select>
          </div>
        </div>
```

**Reemplaza con:**

```html
        <div class="form-row">
          <div class="form-group">
            <label class="form-label" data-i18n="profileNameLabel">Nombre del estudiante</label>
            <input class="form-control" id="studentName" type="text" placeholder="Ej: Sofía" data-i18n-ph="profileNamePlaceholder" value="">
          </div>
          <div class="form-group">
            <label class="form-label" data-i18n="profileGradeLabel">Grado / Nivel</label>
            <select class="form-control" id="studentGrade">
              <option value="primaria_baja">1° - 3° Primaria (6-9 años)</option>
              <option value="primaria_alta" selected>4° - 6° Primaria (9-12 años)</option>
              <option value="secundaria">Secundaria (12-15 años)</option>
            </select>
          </div>
        </div>
```

### E5. Todas las demás secciones estáticas

Aplica el mismo patrón `data-i18n="clave"` a estos elementos:

| Buscar texto actual | Atributo a agregar | Clave |
|---|---|---|
| `🔑 API Key Familiar (Google AI Studio)` | `data-i18n="apiKeyTitle"` | apiKeyTitle |
| Input API Key placeholder `AIza...` | `data-i18n-ph="apiKeyPlaceholder"` | apiKeyPlaceholder |
| `💡 Tu API Key se guarda solo...` (dentro del alert) | `data-i18n="apiKeyInfo"` | apiKeyInfo |
| `💾 Guardar API Key` (botón) | `data-i18n="apiKeySave"` | apiKeySave |
| `📋 ¿Cómo funciona?` | `data-i18n="howTitle"` | howTitle |
| `1. Fotografía el material` | `data-i18n="howStep1Title"` | howStep1Title |
| (y su descripción) | `data-i18n="howStep1Desc"` | howStep1Desc |
| `2. La IA analiza el contenido` | `data-i18n="howStep2Title"` | howStep2Title |
| (y su descripción) | `data-i18n="howStep2Desc"` | howStep2Desc |
| `3. Practica con el quiz` | `data-i18n="howStep3Title"` | howStep3Title |
| (y su descripción) | `data-i18n="howStep3Desc"` | howStep3Desc |
| `✨ Generar material` (section-title) | `data-i18n="genTitle"` | genTitle |
| `Sube fotos del libro...` (section-sub) | `data-i18n="genSub"` | genSub |
| `📸 Imágenes del material` | `data-i18n="genImagesTitle"` | genImagesTitle |
| `📷 Usar cámara` (botón) | `data-i18n="genBtnCamera"` | genBtnCamera |
| `🖼️ Subir imágenes` (botón) | `data-i18n="genBtnUpload"` | genBtnUpload |
| `Arrastra las fotos aquí` | `data-i18n="genDropTitle"` | genDropTitle |
| `o haz clic para seleccionar...` | `data-i18n="genDropSub"` | genDropSub |
| `🎛️ Opciones de generación` | `data-i18n="genOptionsTitle"` | genOptionsTitle |
| `Materia` (label) | `data-i18n="genSubjectLabel"` | genSubjectLabel |
| Input materia placeholder | `data-i18n-ph="genSubjectPlaceholder"` | genSubjectPlaceholder |
| `Tema específico` (label) | `data-i18n="genTopicLabel"` | genTopicLabel |
| Input topic placeholder | `data-i18n-ph="genTopicPlaceholder"` | genTopicPlaceholder |
| `¿Qué quieres generar?` (label) | `data-i18n="genTypeLabel"` | genTypeLabel |
| `📝 Resumen` (pill) | `data-i18n="genTypeResumen"` | genTypeResumen |
| `❓ Quiz (10 preguntas)` (pill) | `data-i18n="genTypeQuiz"` | genTypeQuiz |
| `🗂️ Flashcards` (pill) | `data-i18n="genTypeFlash"` | genTypeFlash |
| `🧠 Mapa conceptual` (pill) | `data-i18n="genTypeMapa"` | genTypeMapa |
| `Notas adicionales (opcional)` (label) | `data-i18n="genNotesLabel"` | genNotesLabel |
| Textarea notes placeholder | `data-i18n-ph="genNotesPlaceholder"` | genNotesPlaceholder |
| `✨ Generar con IA` (botón) | `data-i18n="genBtn"` | genBtn |
| `🎯 Practicar` (section-title) | `data-i18n="quizTitle"` | quizTitle |
| `Pon a prueba...` (section-sub) | `data-i18n="quizSub"` | quizSub |
| `Aún no hay quiz disponible` | `data-i18n="quizEmpty"` | quizEmpty |
| `📂 Historial` (section-title) | `data-i18n="histTitle"` | histTitle |
| `Sesiones guardadas de estudio.` | `data-i18n="histSub"` | histSub |
| `Sin historial aún` | `data-i18n="histEmpty"` | histEmpty |
| `Las sesiones generadas aparecerán aquí.` | `data-i18n="histEmptySub"` | histEmptySub |

---

## Parte F: Traducir TODO el contenido dinámico (funciones JS)

Esta es la parte más crítica. Cada string hardcodeado en las funciones JS debe reemplazarse con llamadas a `t()`.

### F1. Función `renderProfilesList()`

**Reemplaza** todos los strings literales dentro de esta función con llamadas a `t()`. Los cambios clave:

- `'No hay estudiantes registrados. Agrega uno para empezar.'` → `t('profilesEmpty')`
- `'1°-3° Primaria'` → `t('profileGradeLowShort')`
- `'4°-6° Primaria'` → `t('profileGradeMidShort')`
- `'Secundaria'` → `t('profileGradeHighShort')`

### F2. Función `updateActiveProfileHeader()`

- `'6-9 años'` → `t('profileGradeLowAge')`
- `'9-12 años'` → `t('profileGradeMidAge')`
- `'12-15 años'` → `t('profileGradeHighAge')`

### F3. Función `selectProfile()`

- `'Estudiante activo: ...'` → `t('toastProfileActive', { name: ... })`

Además, al seleccionar un perfil que tiene un idioma guardado (`prof.lang`), actualizar el idioma:

```javascript
function selectProfile(id) {
  state.activeProfileId = id;
  localStorage.setItem('sb_active_profile_id', id);

  const activeProf = state.profiles.find(p => p.id === id);
  state.history = activeProf ? (activeProf.history || []) : [];

  // Aplicar el idioma del perfil si tiene uno guardado
  if (activeProf && activeProf.lang) {
    setLanguage(activeProf.lang);
  }

  renderProfilesList();
  updateActiveProfileHeader();
  renderHistory();
  showToast(t('toastProfileActive', { name: activeProf ? activeProf.name : '' }));
}
```

### F4. Función `openProfileForm()`

- `'Editar Estudiante'` → `t('profileFormTitleEdit')`
- `'Agregar Estudiante'` → `t('profileFormTitleAdd')`

### F5. Función `saveProfile()`

- `'⚠️ Ingresa el nombre del estudiante'` → `t('toastNoName')`
- `'💾 Estudiante guardado'` → `t('toastProfileSaved')`

Además, al guardar un nuevo perfil, guardar el idioma actual:

```javascript
// Al crear un nuevo perfil:
const newProf = {
  id: newId,
  name: name,
  grade: grade,
  lang: state.lang,  // ← AGREGAR ESTA LÍNEA
  history: []
};
```

### F6. Función `confirmDeleteProfile()`

- Template del confirm → `t('confirmDeleteProfile', { name: prof.name })`
- `'🗑️ Perfil eliminado'` → `t('toastProfileDeleted')`

### F7. Función `saveApiKey()`

- `'✅ API Key guardada'` → `t('toastApiKeySaved')`

### F8. Función `capturePhoto()`

- `'📸 Foto tomada'` → `t('toastPhotoTaken')`

### F9. Función `openCamera()` (catch block)

- `'⚠️ No se pudo acceder a la cámara...'` → `t('toastCameraError')`

### F10. Función `renderPreviews()`

- `'Página ${i+1}'` (alt) → puede dejarse como está (accesibilidad básica)
- `'Pág. ${i+1}'` → `t('previewPage', { n: i + 1 })`
- `'Eliminar'` (title) → `t('previewRemoveTitle')`

### F11. Función `generateContent()` — LA MÁS IMPORTANTE

**IMPORTANTE**: Los nombres de los campos JSON que Gemini devuelve (`resumen`, `quiz`, `flashcards`, `mapa_conceptual`, `titulo`, `pregunta`, `opciones`, `respuesta_correcta`, `explicacion`, `frente`, `reverso`) **NO se traducen**. Solo cambia el idioma del contenido generado y del prompt.

Los cambios clave:

1. **Toasts de validación**: Reemplazar strings con `t('toastNoProfile')`, `t('toastNoApiKey')`, `t('toastNoImages')`
2. **`gradeMap`**: Reemplazar con `t('promptGradeLabels')` accediendo al sub-objeto
3. **`instructions` (el prompt completo)**: Reconstruir usando las claves `promptRole`, `promptInstruction`, `promptSubject`, etc. del diccionario

```javascript
  const gradeLabel = tNested('promptGradeLabels', grade) || tNested('promptGradeLabels', 'primaria_alta');

  let instructions = `${t('promptRole')} ${t('promptInstruction', { name, gradeLabel })}
${t('promptSubject', { subject: subject || t('promptSubjectDefault'), topic: topic || t('promptTopicDefault') })}
${notes ? t('promptNotes', { notes }) : ''}
${t('promptHandwriting')}
${t('promptTone')}
${t('promptJsonInstruction')}
{
  ${wantResumen ? '"resumen": "string HTML con el resumen (usa <h2>, <h3>, <p>, <ul>, <strong>)",' : ''}
  ${wantQuiz ? '"quiz": [{"pregunta": "...", "opciones": ["A)...", "B)...", "C)...", "D)..."], "respuesta_correcta": 0, "explicacion": "..."}],' : ''}
  ${wantFlash ? '"flashcards": [{"frente": "Término o pregunta corta", "reverso": "Definición o respuesta"}],' : ''}
  ${wantMapa ? '"mapa_conceptual": "string HTML con el mapa (usa divs anidados con estilos inline)",' : ''}
  "titulo": "Título del tema detectado"
}
${wantQuiz ? t('promptQuizInstruction') : ''}
${wantFlash ? t('promptFlashInstruction') : ''}`;
```

4. **Status messages**: `t('genStatusSending')`, `t('genStatusConnecting')`, `t('genStatusRetry')`, `t('genStatusProcessing')`
5. **Retry messages**: `t('retryMsg', { model, attempt, max: maxRetriesPerModel })`
6. **Error de parseo**: `t('toastParseError')`
7. **Error general**: `t('toastError') + err.message`
8. **Fecha del historial**: usar `t('dateLocale')` como locale:

```javascript
date: new Date().toLocaleDateString(t('dateLocale'), { day: '2-digit', month: 'short', year: 'numeric' }),
```

9. **Guardar el idioma en la entrada de historial**:

```javascript
const entry = {
  id: Date.now(),
  titulo: parsed.titulo || `${subject} - ${topic}` || 'Sesión de estudio',
  date: new Date().toLocaleDateString(t('dateLocale'), { day: '2-digit', month: 'short', year: 'numeric' }),
  types,
  data: parsed,
  lang: state.lang  // ← AGREGAR ESTA LÍNEA
};
```

### F12. Función `setGenLoading()`

- `'Generando...'` → `t('genBtnLoading')`
- `'✨ Generar con IA'` → `t('genBtn')`

```javascript
function setGenLoading(loading, msg) {
  const btn = document.getElementById('genBtn');
  const prog = document.getElementById('genProgress');
  if (loading) {
    btn.disabled = true;
    btn.innerHTML = `<span class="spinner"></span> ${t('genBtnLoading')}`;
    prog.style.display = 'block';
    setGenStatusMsg(msg || '');
  } else {
    btn.disabled = false;
    btn.innerHTML = t('genBtn');
    prog.style.display = 'none';
  }
}
```

### F13. Función `renderResults()`

Reemplazar strings literales con llamadas a `t()`:
- `'📝 Resumen'` → `t('resultResumen')`
- `'❓ Quiz'` → `t('resultQuiz')`
- `'🗂️ Flashcards'` → `t('resultFlashcards')`
- `'🧠 Mapa conceptual'` → `t('resultMapa')`
- `'🖨️ Imprimir'` → `t('resultPrint')`
- `'🎯 Iniciar Quiz'` → `t('resultStartQuiz')`
- `'📋 Copiar'` → `t('resultCopy')`
- `'👆 Haz clic en cada tarjeta...'` → `t('resultFlashHint')`
- El map de tipos en la cabecera de resultados debe usar `tNested('typeLabels', tipo)` en vez del objeto hardcodeado.

### F14. Función `renderQuizQuestion()`

- `'Pregunta ${...} de ${...}'` → `t('quizQuestion', { current: state.quizIndex + 1, total })`
- `'⭐ ${...} correctas'` → `t('quizCorrectCount', { n: state.quizScore })`
- `'Siguiente pregunta →'` / `'Ver resultados 🎉'` → `t('quizNext')` / `t('quizSeeResults')`

### F15. Función `answerQuiz()`

- `'✅ ¡Correcto! '` → `t('quizCorrect')`
- `'❌ Incorrecto. '` → `t('quizWrong')`
- `'La respuesta correcta es: ...'` → `t('quizWrongAnswer', { answer: ... })`

### F16. Función `showQuizResults()`

- Mensajes de puntuación según porcentaje: usar `t('quizScoreExcellent')`, etc.
- `'de ${total}'` → `t('quizScoreOf', { total })`
- `'${pct}% de respuestas correctas'` → `t('quizScorePct', { pct })`
- `'🔄 Volver a intentar'` → `t('quizRetry')`
- `'📚 Ver material'` → `t('quizSeeMaterial')`

### F17. Función `renderHistory()`

- `'Sin historial aún'` → `t('histEmpty')`
- `'Las sesiones generadas aparecerán aquí.'` → `t('histEmptySub')`
- El mapeo de tipos `{resumen:'Resumen',...}` → usar `tNested('typeLabels', tipo)`

### F18. Función `copyText()`

- `'📋 ¡Copiado!'` → `t('toastCopied')`
- `'No se pudo copiar'` → `t('toastCopyFail')`

### F19. Función `printContent()`

- `'StudyBuddy - Material de estudio'` → `t('printTitle')`

### F20. Funciones PWA (`showInstallBanner`, `installApp`, etc.)

- Strings del banner de instalación: `t('pwaBannerTitle')`, `t('pwaBannerSub')`, `t('pwaBannerBtn')`
- `'🎉 ¡StudyBuddy instalada!...'` → `t('toastPwaInstalled')`
- Instrucciones de iOS/Android: `t('pwaIosInstructions')`, `t('pwaAndroidInstructions')`, `t('pwaManualInstructions')`

### F21. Sección de quiz vacía (en el HTML)

Busca el empty state del quiz:

```html
      <div class="empty-state-title">Aún no hay quiz disponible</div>
      <div class="empty-state-sub">Genera contenido desde la pestaña <strong>Generar</strong> con la opción Quiz activada.</div>
```

Agrega `data-i18n`:

```html
      <div class="empty-state-title" data-i18n="quizEmpty">Aún no hay quiz disponible</div>
      <div class="empty-state-sub" data-i18n="quizEmptySub">Genera contenido desde la pestaña <strong>Generar</strong> con la opción Quiz activada.</div>
```

---

## Parte G: Inicialización del idioma

### G1. Cargar idioma en la IIFE `init()`

Busca dentro de la IIFE init, justo después de cargar la API Key (el bloque `try { state.apiKey = ... } catch {}`), y **ANTES** de cargar perfiles, agrega:

```javascript
  // 1.5. Cargar idioma
  try {
    state.lang = localStorage.getItem('sb_lang') || 'es';
    document.documentElement.lang = state.lang;
  } catch {}
```

### G2. Aplicar idioma después de cargar perfiles

Al final de la IIFE `init()`, justo **ANTES** del cierre `})();`, agrega:

```javascript
  // Aplicar idioma del perfil activo si existe
  const activeProfile = state.profiles.find(p => p.id === state.activeProfileId);
  if (activeProfile && activeProfile.lang) {
    state.lang = activeProfile.lang;
    localStorage.setItem('sb_lang', activeProfile.lang);
  }
  // Aplicar traducciones a la interfaz
  setLanguage(state.lang);
```

---

## Parte H: Actualizar el Service Worker cache

Busca en `sw.js` la constante `CACHE_NAME`:

```javascript
const CACHE_NAME = 'studybuddy-v1';
```

**Reemplaza con:**

```javascript
const CACHE_NAME = 'studybuddy-v2';
```

Esto forzará la actualización del caché para los usuarios existentes.

---

## Verificación

Al terminar, el archivo `index.html` debe:

1. [ ] Abrirse sin errores en la consola del navegador
2. [ ] Mostrar el switch de idioma ES/EN en el header, al lado del badge "Con IA"
3. [ ] Al hacer clic en el switch, TODA la interfaz debe cambiar de idioma instantáneamente:
   - Tabs (Inicio→Home, Generar→Generate, etc.)
   - Todas las cards y sus textos
   - Placeholders de los inputs
   - Opciones del select de grado
   - Mensajes de empty state
   - Botones
4. [ ] La preferencia de idioma debe persistir al recargar la página (guardada en localStorage)
5. [ ] Al cambiar el idioma y luego generar contenido:
   - Si está en inglés, el prompt se envía en inglés y Gemini responde en inglés
   - Si está en español, funciona como antes
   - Los campos JSON internos (`resumen`, `quiz`, etc.) se mantienen iguales en ambos idiomas
6. [ ] El quiz funciona correctamente en ambos idiomas (los strings de "Correcto", "Incorrecto", etc. se traducen)
7. [ ] El historial muestra los tipos traducidos al idioma actual
8. [ ] Al seleccionar un perfil de estudiante que tenía un idioma guardado, el switch cambia automáticamente a ese idioma
9. [ ] Al crear un nuevo perfil, se guarda el idioma actual como preferencia del perfil
10. [ ] Los toasts de feedback se muestran en el idioma activo
11. [ ] La tarjeta de instalación PWA se traduce correctamente
12. [ ] El banner de instalación PWA se traduce correctamente
13. [ ] La app sigue siendo responsive en pantallas menores a 600px
14. [ ] El switch de idioma tiene una animación suave al cambiar (el knob se desliza)
15. [ ] El `<html lang="...">` se actualiza dinámicamente al cambiar idioma

### Prueba rápida de ida y vuelta

1. Abrir la app → debe cargar en español (o el último idioma usado)
2. Hacer clic en el switch → toda la interfaz cambia a inglés
3. Hacer clic de nuevo → vuelve a español
4. Recargar la página → mantiene el último idioma seleccionado
5. Agregar un estudiante con idioma EN → al seleccionarlo, la app cambia a inglés
6. Generar contenido en inglés → el resumen y quiz salen en inglés
7. Cambiar a español → los botones y la interfaz cambian, pero el contenido generado previamente en inglés permanece en inglés (es correcto, se generó en ese idioma)

---

## Resumen de cambios

| Archivo | Tipo de cambio |
|---------|---------------|
| `index.html` | **CSS**: ~30 líneas nuevas para `.lang-switch` |
| `index.html` | **HTML**: Switch en header + atributos `data-i18n` en ~40 elementos |
| `index.html` | **JS**: Diccionario `i18n` (~300 líneas), funciones `t()`, `tNested()`, `setLanguage()`, `toggleLanguage()`, `updateLangSwitch()` (~60 líneas), + modificación de ~20 funciones existentes para usar `t()` |
| `sw.js` | **JS**: 1 línea — incrementar versión de caché |

**Total estimado**: ~400 líneas nuevas de JS (mayormente el diccionario), ~30 CSS, ~40 atributos HTML.
