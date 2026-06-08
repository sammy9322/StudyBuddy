# FASE 4: Renderizado de Contenido Generado

## Contexto

El archivo `index.html` ya tiene la estructura (Fase 1), imágenes (Fase 2) y la llamada a la API (Fase 3). Ahora debes implementar las funciones que toman la respuesta JSON de Gemini y la convierten en HTML visual dentro de la app.

## Documentos de referencia

- **docs/studybuddy.html** → líneas 1027-1126: funciones renderResults(), makeResultBlock(), copyText(), printContent()
- **docs/PLAN_DE_IMPLEMENTACION.html** → Fase 4 del plan paso a paso

## Qué hacer

En el archivo `index.html`, busca el comentario `// ── RENDER RESULTS (Fase 4) ──` o la función temporal `renderResults()` que se creó como placeholder, y reemplázalo con la implementación completa.

---

## Función renderResults(data, types)

Esta función toma el JSON parseado de la respuesta de Gemini y genera todo el HTML visual.

### Paso 1: Preparar container
```javascript
const container = document.getElementById('genResults');
container.style.display = 'block';
container.innerHTML = '';
const titulo = data.titulo || 'Material generado';
```

### Paso 2: Header de resultados
Crear un div.card.animate-in con display flex, align-items center, justify-content space-between, flex-wrap, gap 10px. Contenido:

- Lado izquierdo:
  - Título del tema con font Sora, bold, 1.1rem: `📚 ${escHtml(titulo)}`
  - Chips de tipos: mapear cada tipo a su emoji+nombre (`resumen→'📝 Resumen'`, `quiz→'❓ Quiz'`, `flashcards→'🗂️ Flashcards'`, `mapa→'🧠 Mapa'`), unir con ` · `
- Lado derecho (btn-row):
  - Si hay quiz: botón accent sm "🎯 Iniciar Quiz" con onclick `startQuiz()`
  - Botón outline sm "🖨️ Imprimir" con onclick `printContent()`

### Paso 3: Resumen
Si `types.includes('resumen')` Y `data.resumen` existe:
```javascript
const block = makeResultBlock('📝 Resumen', data.resumen, '#4F46E5');
container.appendChild(block);
```

### Paso 4: Flashcards
Si `types.includes('flashcards')` Y `data.flashcards?.length`:

1. Crear div.result-block.animate-in
2. Header con result-title: `🗂️ Flashcards (${data.flashcards.length})`
3. Body como div con display grid, `grid-template-columns: repeat(auto-fill, minmax(200px, 1fr))`, gap 10px
4. Para cada flashcard en `data.flashcards`:
   - Crear div con estilos: borde 1.5px solid var(--border), border-radius 12px, padding 14px, cursor pointer, transición, min-height 90px, flex column, justify-content center
   - Guardar `dataset.flipped = 'false'`
   - innerHTML con dos divs:
     - `.fc-front`: font-weight 700, font-size 14px, color primary → `escHtml(fc.frente)`
     - `.fc-back`: display none, font-size 13px, color text2 → `escHtml(fc.reverso)`
   - addEventListener click:
     ```javascript
     card.addEventListener('click', () => {
       const flipped = card.dataset.flipped === 'true';
       card.querySelector('.fc-front').style.display = flipped ? '' : 'none';
       card.querySelector('.fc-back').style.display = flipped ? 'none' : '';
       card.style.background = flipped ? '' : 'var(--primary-light)';
       card.dataset.flipped = flipped ? 'false' : 'true';
     });
     ```
5. Después del body, agregar hint: "👆 Haz clic en cada tarjeta para ver la respuesta" (font-size 12px, color text3, border-top)

### Paso 5: Mapa conceptual
Si `types.includes('mapa')` Y `data.mapa_conceptual`:
```javascript
const block = makeResultBlock('🧠 Mapa conceptual', data.mapa_conceptual, '#059669');
container.appendChild(block);
```

### Paso 6: Quiz preview
Si `types.includes('quiz')` Y `data.quiz?.length`:

Crear div.result-block.animate-in con:
- Header: result-title "❓ Quiz listo (${data.quiz.length} preguntas)" + botón accent sm "Iniciar →" con onclick `startQuiz()`
- Body: texto "El quiz tiene N preguntas de opción múltiple. ¡Pon a prueba lo que aprendiste!"

---

## Función makeResultBlock(title, html, color)

Crea un bloque reutilizable para mostrar contenido (resumen, mapa):

```javascript
function makeResultBlock(title, html, color) {
  const b = document.createElement('div');
  b.className = 'result-block animate-in';
  b.innerHTML = `
    <div class="result-header">
      <div class="result-title"><span style="color:${color}">${title}</span></div>
      <button class="btn btn-outline btn-sm" onclick="copyText(this.closest('.result-block').querySelector('.result-body').innerText)">📋 Copiar</button>
    </div>
    <div class="result-body">${html}</div>
  `;
  return b;
}
```

---

## Función copyText(text)

```javascript
function copyText(text) {
  navigator.clipboard.writeText(text)
    .then(() => showToast('📋 ¡Copiado!'))
    .catch(() => showToast('No se pudo copiar'));
}
```

---

## Función printContent()

```javascript
function printContent() {
  const results = document.getElementById('genResults');
  if (!results) return;
  const w = window.open('', '_blank');
  w.document.write(`<html><head><title>StudyBuddy - Material de estudio</title>
    <style>body{font-family:sans-serif;max-width:800px;margin:0 auto;padding:2rem;} h2{color:#4F46E5;} ul{padding-left:1.5rem;}</style>
    </head><body>${results.innerHTML}</body></html>`);
  w.document.close();
  w.print();
}
```

---

## Verificación

Al terminar, si ya implementaste la Fase 3 y tienes una API Key válida:
1. Generar contenido con imágenes debe mostrar los resultados renderizados
2. El resumen debe mostrar HTML formateado (títulos, párrafos, listas)
3. Las flashcards deben hacer flip al hacer clic (alternar frente/reverso)
4. El botón "📋 Copiar" debe copiar el texto al portapapeles y mostrar toast
5. El botón "🖨️ Imprimir" debe abrir la ventana de impresión
6. El quiz preview debe mostrar el número de preguntas y el botón "Iniciar"
