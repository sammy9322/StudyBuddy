# FASE 5: Sistema de Evaluación (Quiz)

## Contexto

El archivo `index.html` ya tiene las Fases 1-4. Ahora debes implementar el sistema de quiz interactivo: mostrar preguntas una por una, evaluar respuestas, dar feedback y mostrar resultados finales.

## Documentos de referencia

- **docs/studybuddy.html** → líneas 1128-1222: funciones startQuiz(), renderQuizQuestion(), answerQuiz(), nextQuestion(), showQuizResults()
- **docs/PLAN_DE_IMPLEMENTACION.html** → sección 2.6 (Sistema de Evaluación) y Fase 5 del plan

## Qué hacer

En el archivo `index.html`, busca el comentario `// ── QUIZ (Fase 5) ──` y reemplázalo con todas las funciones del quiz.

---

## Función startQuiz()

```javascript
function startQuiz() {
  if (!state.currentQuiz?.length) {
    showToast('No hay quiz disponible aún');
    return;
  }
  state.quizIndex = 0;
  state.quizScore = 0;
  showTab('quiz');
  renderQuizQuestion();
}
```

---

## Función renderQuizQuestion()

1. Obtener `#quizContainer` y la pregunta actual: `state.currentQuiz[state.quizIndex]`
2. Obtener total: `state.currentQuiz.length`
3. Reset: `state.quizAnswered = false`
4. Establecer innerHTML del container con un div.card.animate-in que contenga:

**Progreso:**
```html
<div class="quiz-progress-info">
  <span>Pregunta ${state.quizIndex + 1} de ${total}</span>
  <span>⭐ ${state.quizScore} correctas</span>
</div>
<div class="progress-bar-wrap" style="margin-bottom:1.25rem;">
  <div class="progress-bar" style="width:${((state.quizIndex)/total*100).toFixed(0)}%"></div>
</div>
```

**Pregunta:**
```html
<div class="quiz-question">${escHtml(q.pregunta)}</div>
```

**Opciones (A, B, C, D):**
```html
<div class="quiz-options" id="quizOptions">
  ${q.opciones.map((op, i) => `
    <button class="quiz-option" onclick="answerQuiz(${i})">
      <span class="option-letter">${'ABCD'[i]}</span>
      <span>${escHtml(op)}</span>
    </button>
  `).join('')}
</div>
```

**Feedback (oculto inicialmente):**
```html
<div class="quiz-feedback" id="quizFeedback"></div>
```

**Botón siguiente (oculto inicialmente):**
```html
<div id="quizNextBtn" style="margin-top:1rem;display:none;">
  <button class="btn btn-primary" onclick="nextQuestion()">
    ${state.quizIndex + 1 < total ? 'Siguiente pregunta →' : 'Ver resultados 🎉'}
  </button>
</div>
```

---

## Función answerQuiz(idx)

```javascript
function answerQuiz(idx) {
  // Guard: no permitir responder dos veces
  if (state.quizAnswered) return;
  state.quizAnswered = true;

  const q = state.currentQuiz[state.quizIndex];
  const opts = document.querySelectorAll('.quiz-option');
  const fb = document.getElementById('quizFeedback');

  // Deshabilitar todas las opciones
  opts.forEach(o => o.classList.add('disabled'));

  // Marcar la correcta siempre en verde
  opts[q.respuesta_correcta].classList.add('correct');

  if (idx === q.respuesta_correcta) {
    // CORRECTO
    state.quizScore++;
    fb.className = 'quiz-feedback show correct-fb';
    fb.textContent = '✅ ¡Correcto! ' + (q.explicacion || '');
  } else {
    // INCORRECTO
    opts[idx].classList.add('wrong');
    fb.className = 'quiz-feedback show wrong-fb';
    fb.textContent = '❌ Incorrecto. ' + (q.explicacion || `La respuesta correcta es: ${q.opciones[q.respuesta_correcta]}`);
  }

  // Mostrar botón siguiente
  document.getElementById('quizNextBtn').style.display = 'block';
}
```

---

## Función nextQuestion()

```javascript
function nextQuestion() {
  state.quizIndex++;
  if (state.quizIndex >= state.currentQuiz.length) {
    showQuizResults();
  } else {
    renderQuizQuestion();
  }
}
```

---

## Función showQuizResults()

```javascript
function showQuizResults() {
  const total = state.currentQuiz.length;
  const pct = Math.round(state.quizScore / total * 100);

  // Emoji según porcentaje
  const emoji = pct >= 90 ? '🏆' : pct >= 70 ? '🌟' : pct >= 50 ? '👍' : '💪';

  // Mensaje según porcentaje
  const msg = pct >= 90 ? '¡Excelente trabajo!'
    : pct >= 70 ? '¡Muy bien hecho!'
    : pct >= 50 ? '¡Buen intento!'
    : '¡Sigue practicando!';

  document.getElementById('quizContainer').innerHTML = `
    <div class="card animate-in">
      <div class="score-display">
        <div class="score-circle">
          <span class="score-number">${state.quizScore}</span>
          <span class="score-total">de ${total}</span>
        </div>
        <div class="score-msg">${emoji} ${msg}</div>
        <div class="score-sub">${pct}% de respuestas correctas</div>
        <div class="btn-row" style="justify-content:center;">
          <button class="btn btn-primary" onclick="startQuiz()">🔄 Volver a intentar</button>
          <button class="btn btn-outline" onclick="showTab('generar')">📚 Ver material</button>
        </div>
      </div>
    </div>
  `;
}
```

---

## Verificación

Al terminar (necesitas haber generado un quiz con la API para probar):
1. Hacer clic en "Iniciar Quiz" debe cambiar a la pestaña Practicar y mostrar la primera pregunta
2. Debe mostrar "Pregunta 1 de 10" y la barra de progreso en 0%
3. Al hacer clic en una opción:
   - Si es correcta: se marca en verde, feedback verde con ✅ y explicación
   - Si es incorrecta: la seleccionada se marca en rojo, la correcta en verde, feedback rojo con ❌
4. No se puede hacer clic en más opciones después de responder
5. El botón "Siguiente pregunta" aparece tras responder
6. En la última pregunta, el botón dice "Ver resultados 🎉"
7. La pantalla de resultados muestra el score-circle, emoji apropiado y mensaje
8. "Volver a intentar" reinicia el quiz desde la pregunta 1
9. "Ver material" regresa a la pestaña Generar
