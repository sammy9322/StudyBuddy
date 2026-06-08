# FASE 3: Integración con Google Gemini API

## Contexto

El archivo `index.html` ya tiene la estructura base (Fase 1) y la gestión de imágenes (Fase 2). Ahora debes implementar la función principal que envía las imágenes a la API de Google Gemini y recibe el contenido generado.

## Documentos de referencia

- **docs/studybuddy.html** → líneas 901-1026: función generateContent() y auxiliares
- **docs/PLAN_DE_IMPLEMENTACION.html** → secciones 2.3, 2.4 (Especificación Técnica API) y 2.5 (Prompt Engineering)

## Qué hacer

En el archivo `index.html`, busca el comentario `// ── GENERATE (Fase 3) ──` y reemplázalo con todas las funciones de generación.

---

## Función principal: generateContent() — async

Esta es la función más importante de la app. Se ejecuta cuando el usuario hace clic en "✨ Generar con IA".

### Paso 1: Validaciones
```javascript
const apiKey = document.getElementById('apiKey').value.trim() || state.config.apiKey;
if (!apiKey) {
  showToast('⚠️ Primero configura tu API Key en Inicio');
  return;
}
if (state.images.length === 0) {
  showToast('⚠️ Sube al menos una imagen');
  return;
}
```

### Paso 2: Leer datos del formulario
```javascript
const subject = document.getElementById('subject').value.trim();
const topic = document.getElementById('topic').value.trim();
const notes = document.getElementById('extraNotes').value.trim();
const grade = document.getElementById('studentGrade').value || state.config.grade || 'primaria_alta';
const name = document.getElementById('studentName').value.trim() || state.config.name || 'el estudiante';
```

### Paso 3: Mapeo de grado a descripción natural
```javascript
const gradeMap = {
  primaria_baja: 'niño de 6 a 9 años, 1°-3° primaria',
  primaria_alta: 'niño de 9 a 12 años, 4°-6° primaria',
  secundaria: 'adolescente de 12 a 15 años, secundaria'
};
const gradeLabel = gradeMap[grade] || gradeMap['primaria_alta'];
```

### Paso 4: Determinar tipos seleccionados
```javascript
const types = [...state.genTypes];
const wantResumen = types.includes('resumen');
const wantQuiz = types.includes('quiz');
const wantFlash = types.includes('flashcards');
const wantMapa = types.includes('mapa');
```

### Paso 5: Construir el prompt
Construye un string `instructions` que contenga:
- Rol: "Eres un tutor educativo experto"
- Instrucción: "Analiza las imágenes de material escolar (libro/cuaderno) y genera contenido para {name}, {gradeLabel}"
- Materia y tema
- Notas del tutor si existen
- "Usa un lenguaje claro, amigable y apropiado para la edad"
- "Responde SIEMPRE en español"
- "Devuelve la respuesta en JSON con esta estructura exacta (sin markdown, solo JSON puro)"
- La estructura JSON dinámica según los tipos seleccionados:
  - Si wantResumen: `"resumen": "string HTML con el resumen (usa <h2>, <h3>, <p>, <ul>, <strong>)"`
  - Si wantQuiz: `"quiz": [{"pregunta": "...", "opciones": ["A)...", "B)...", "C)...", "D)..."], "respuesta_correcta": 0, "explicacion": "..."}]`
  - Si wantFlash: `"flashcards": [{"frente": "Término o pregunta corta", "reverso": "Definición o respuesta"}]`
  - Si wantMapa: `"mapa_conceptual": "string HTML con el mapa (usa divs anidados con estilos inline)"`
  - Siempre: `"titulo": "Título del tema detectado"`
- Si wantQuiz: "Genera exactamente 10 preguntas de opción múltiple variadas. respuesta_correcta es el índice (0-3) de la opción correcta."
- Si wantFlash: "Genera al menos 8 flashcards con los conceptos clave."

### Paso 6: Preparar imágenes como inline_data
```javascript
const imageParts = state.images.map(img => ({
  inline_data: { mime_type: img.mimeType, data: img.base64 }
}));
```

### Paso 7: Mostrar loading
```javascript
setGenLoading(true, 'Enviando imágenes a la IA...');
setProgress(20);
```

### Paso 8: Llamada a la API (dentro de try/catch)
```javascript
try {
  const res = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        contents: [{ parts: [...imageParts, { text: instructions }] }],
        generationConfig: { temperature: 0.7, maxOutputTokens: 8000 }
      })
    }
  );

  setProgress(70);
  setGenStatusMsg('Procesando respuesta...');

  if (!res.ok) {
    const err = await res.json();
    throw new Error(err.error?.message || 'Error de API');
  }

  const data = await res.json();
  let rawText = data.candidates?.[0]?.content?.parts?.[0]?.text || '';

  // Limpiar markdown fences que la IA a veces agrega
  rawText = rawText.replace(/```json\s*/gi, '').replace(/```\s*/g, '').trim();

  let parsed;
  try {
    parsed = JSON.parse(rawText);
  } catch {
    throw new Error('La IA devolvió una respuesta inesperada. Intenta de nuevo.');
  }

  setProgress(100);
  setGenLoading(false);

  // Renderizar resultados (Fase 4)
  renderResults(parsed, types);

  // Guardar en historial
  const entry = {
    id: Date.now(),
    titulo: parsed.titulo || `${subject} - ${topic}` || 'Sesión de estudio',
    date: new Date().toLocaleDateString('es-CR', { day: '2-digit', month: 'short', year: 'numeric' }),
    types,
    data: parsed
  };
  state.history.unshift(entry);
  if (state.history.length > 30) state.history.pop();
  localStorage.setItem('sb_history', JSON.stringify(state.history));
  renderHistory();

  // Cargar quiz si fue generado
  if (wantQuiz && parsed.quiz?.length) {
    state.currentQuiz = parsed.quiz;
    state.quizIndex = 0;
    state.quizScore = 0;
  }

  showTab('generar');
  document.getElementById('genResults').scrollIntoView({ behavior: 'smooth' });

} catch (err) {
  setGenLoading(false);
  showToast('❌ Error: ' + err.message);
  console.error(err);
}
```

---

## Funciones auxiliares de loading

### setGenLoading(loading, msg)
```javascript
function setGenLoading(loading, msg) {
  const btn = document.getElementById('genBtn');
  const prog = document.getElementById('genProgress');
  if (loading) {
    btn.disabled = true;
    btn.innerHTML = '<span class="spinner"></span> Generando...';
    prog.style.display = 'block';
    setGenStatusMsg(msg || '');
  } else {
    btn.disabled = false;
    btn.innerHTML = '✨ Generar con IA';
    prog.style.display = 'none';
  }
}
```

### setGenStatusMsg(msg)
```javascript
function setGenStatusMsg(msg) {
  document.getElementById('genStatusMsg').textContent = msg;
}
```

### setProgress(pct)
```javascript
function setProgress(pct) {
  document.getElementById('genBar').style.width = pct + '%';
}
```

---

## NOTA IMPORTANTE

Las funciones `renderResults()` y `renderHistory()` se implementarán en las Fases 4 y 6. Si aún no existen como funciones completas, agrega placeholders temporales para que no haya errores:

```javascript
// Solo si renderResults no existe aún:
function renderResults(data, types) {
  console.log('Resultados recibidos:', data);
  document.getElementById('genResults').style.display = 'block';
  document.getElementById('genResults').innerHTML = '<div class="card"><div class="card-title">✅ Contenido generado exitosamente</div><p>Las funciones de visualización se agregarán en la siguiente fase.</p></div>';
}
```

---

## Verificación

Al terminar:
1. Sin API Key: debe mostrar toast de advertencia y NO hacer la llamada
2. Sin imágenes: debe mostrar toast de advertencia y NO hacer la llamada
3. Con API Key e imágenes: debe mostrar la barra de progreso, hacer la llamada a la API, y manejar la respuesta
4. Si la API devuelve error: debe mostrar toast con el mensaje de error
5. El botón debe mostrar spinner mientras se procesa y volver a la normalidad al terminar
6. No debe haber errores en consola (excepto si la API Key es inválida, eso es esperado)
