# FASE 6: Historial de Sesiones

## Contexto

El archivo `index.html` ya tiene las Fases 1-5. Ahora debes reemplazar la función placeholder de `renderHistory()` con la implementación completa, y agregar la función `loadHistory()`.

## Documentos de referencia

- **docs/studybuddy.html** → líneas 1224-1254: funciones renderHistory() y loadHistory()
- **docs/PLAN_DE_IMPLEMENTACION.html** → sección 2.7 (Historial de Sesiones) y Fase 6 del plan

## Qué hacer

En el archivo `index.html`, busca la función `renderHistory()` que existe como placeholder y reemplázala con la implementación completa. También agrega la función `loadHistory()` justo después.

---

## Función renderHistory() — REEMPLAZAR el placeholder existente

```javascript
function renderHistory() {
  const c = document.getElementById('historialContainer');

  // Estado vacío
  if (!state.history.length) {
    c.innerHTML = `
      <div class="empty-state">
        <div class="empty-state-icon">📂</div>
        <div class="empty-state-title">Sin historial aún</div>
        <div class="empty-state-sub">Las sesiones generadas aparecerán aquí.</div>
      </div>
    `;
    return;
  }

  // Renderizar lista de sesiones
  c.innerHTML = state.history.map((entry, i) => `
    <div class="history-item" onclick="loadHistory(${i})">
      <div class="history-icon" style="background:var(--primary-light);">📚</div>
      <div class="history-meta">
        <div class="history-name">${escHtml(entry.titulo)}</div>
        <div class="history-date">📅 ${entry.date} · ${entry.types.map(t => ({
          resumen: 'Resumen',
          quiz: 'Quiz',
          flashcards: 'Flashcards',
          mapa: 'Mapa'
        }[t] || t)).join(', ')}</div>
      </div>
      <span class="history-arrow">›</span>
    </div>
  `).join('');
}
```

---

## Función loadHistory(i)

```javascript
function loadHistory(i) {
  const entry = state.history[i];
  if (!entry) return;

  // Si la sesión tiene quiz, cargarlo en el estado
  if (entry.data.quiz?.length) {
    state.currentQuiz = entry.data.quiz;
    state.quizIndex = 0;
    state.quizScore = 0;
  }

  // Cambiar a la pestaña Generar y mostrar los resultados
  showTab('generar');
  renderResults(entry.data, entry.types);
  document.getElementById('genResults').scrollIntoView({ behavior: 'smooth' });
}
```

---

## Recordatorio de cómo se conecta

Esta función ya está siendo llamada desde:
1. **init()** (Fase 1): al cargar la página, si hay historial en localStorage
2. **generateContent()** (Fase 3): después de generar contenido nuevo, para actualizar la lista

El historial se guarda en localStorage con la clave `sb_history` y cada entrada tiene la estructura:
```javascript
{
  id: Date.now(),           // timestamp único
  titulo: "...",            // título del tema detectado por la IA
  date: "08 jun 2026",     // fecha formateada en es-CR
  types: ["resumen", ...],  // tipos generados
  data: { ... }            // JSON completo de la respuesta de Gemini
}
```

---

## Verificación

Al terminar:
1. Si hay sesiones en el historial, la pestaña Historial debe mostrar la lista con icono 📚, título, fecha y tipos
2. Si no hay sesiones, debe mostrar el empty state con icono 📂
3. Hacer clic en una sesión debe cambiar a la pestaña Generar y mostrar los resultados de esa sesión
4. Si la sesión tenía quiz, el quiz debe estar disponible para practicar
5. El scroll debe ir suavemente a los resultados
6. Al generar contenido nuevo, la lista de historial debe actualizarse inmediatamente
