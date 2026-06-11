# FASE 12: Gestión del Historial (Traducir y Eliminar)

## Contexto

El usuario necesita poder gestionar su historial. Específicamente quiere:
1. **Eliminar** entradas desde la lista de historial (con una protección de "dos toques" para evitar accidentes).
2. **Traducir** entradas al otro idioma sin tener que volver a escanear las fotos. El botón de traducir debe aparecer en la cabecera cuando el usuario está viendo el material.

## Documentos de referencia

- **index.html** → El archivo principal donde se hacen todos los cambios.
- **docs/instrucciones/FASE-11-i18n.md** → Referencia del diccionario `i18n`.

## Qué hacer

Modificarás `index.html` para agregar el diccionario correspondiente, la lógica de UI y las funciones que hacen llamadas a la API de Gemini.

---

## Parte A: Actualizar Diccionario `i18n`

Busca el objeto `const i18n = {`.

### A1. Claves en Español (`es: { ... }`)
Agrega al final del bloque de historial (alrededor de la línea donde dice `histEmptySub`):
```javascript
    histDeleteAction: '🗑️',
    histDeleteConfirm: '¿Borrar?',
```

Agrega al final del bloque de resultados (alrededor de `resultFlashHint`):
```javascript
    resultTranslateToEn: '🇺🇸 Traducir a Inglés',
    resultTranslateToEs: '🇪🇸 Traducir a Español',
    resultTranslating: 'Traduciendo...',
```

Agrega al final del bloque de toasts (alrededor de `toastPwaInstalled`):
```javascript
    toastHistoryDeleted: '🗑️ Sesión eliminada',
    toastTranslateSuccess: '✅ Traducción completada',
```

Agrega al final de los prompts de Gemini (alrededor de `promptFlashInstruction`):
```javascript
    promptTranslateBase: 'Traduce los valores del siguiente objeto JSON al {targetLang}. IMPORTANTE: Mantén exactamente la misma estructura original, arrays, y exactamente los mismos nombres de llaves (keys). NO cambies ninguna llave (por ejemplo si dice "resumen" o "quiz", déjalo igual). Responde SOLO con el JSON puro resultante, sin markdown. Idioma destino: {targetLangName}. JSON original:',
```

### A2. Claves en Inglés (`en: { ... }`)
Agrega las equivalencias en el bloque `en: { ... }`:
```javascript
    histDeleteAction: '🗑️',
    histDeleteConfirm: 'Delete?',
    
    resultTranslateToEn: '🇺🇸 Translate to English',
    resultTranslateToEs: '🇪🇸 Translate to Spanish',
    resultTranslating: 'Translating...',
    
    toastHistoryDeleted: '🗑️ Session deleted',
    toastTranslateSuccess: '✅ Translation completed',
    
    promptTranslateBase: 'Translate the values of the following JSON object to {targetLang}. IMPORTANT: Keep exactly the original structure, arrays, and exactly the same key names. DO NOT change any key (for example if it says "resumen" or "quiz", leave it exactly like that). Respond ONLY with the resulting pure JSON, no markdown. Target language: {targetLangName}. Original JSON:',
```

---

## Parte B: UI del Historial (Borrar en dos toques)

Busca la función `renderHistory()`. Actualmente tiene este mapa:
```javascript
    <div class="history-item" onclick="loadHistory(${i})">
      <div class="history-icon" style="background:var(--primary-light);">📚</div>
      <div class="history-meta">
        <div class="history-name">${escHtml(entry.titulo)}</div>
        <div class="history-date">📅 ${entry.date} · ${entry.types.map(typeItem => tNested('typeLabels', typeItem) || typeItem).join(', ')}</div>
      </div>
      <span class="history-arrow">›</span>
    </div>
```

**Modifícalo** para incluir el botón de eliminar. IMPORTANTE: Para que el botón de eliminar no active `loadHistory`, debemos manejar los eventos correctamente (`e.stopPropagation()`). 

Reemplaza ese bloque completo por:
```javascript
    <div class="history-item" onclick="loadHistory(${i})" style="position:relative; padding-right:60px;">
      <div class="history-icon" style="background:var(--primary-light);">📚</div>
      <div class="history-meta">
        <div class="history-name">${escHtml(entry.titulo)}</div>
        <div class="history-date">📅 ${entry.date} · ${entry.types.map(typeItem => tNested('typeLabels', typeItem) || typeItem).join(', ')}</div>
      </div>
      <button class="btn-delete-history" data-idx="${i}" onclick="handleDeleteHistoryClick(event, ${i})" style="position:absolute; right:12px; top:50%; transform:translateY(-50%); background:var(--danger-light); color:var(--danger); border:none; padding:6px 10px; border-radius:8px; font-weight:700; font-size:12px; cursor:pointer; font-family:'Nunito',sans-serif;">
        ${t('histDeleteAction')}
      </button>
    </div>
```

Agrega la función para manejar los "dos toques" justo debajo de `renderHistory()`:
```javascript
let deleteConfirmTimeout = null;

function handleDeleteHistoryClick(e, idx) {
  e.stopPropagation(); // Evitar que se abra el historial
  const btn = e.currentTarget;
  
  if (btn.dataset.confirm === 'true') {
    // Segundo toque: Eliminar
    deleteHistoryEntry(idx);
  } else {
    // Primer toque: Pedir confirmación
    clearTimeout(deleteConfirmTimeout);
    
    // Resetear todos los demás botones
    document.querySelectorAll('.btn-delete-history').forEach(b => {
      b.dataset.confirm = 'false';
      b.innerHTML = t('histDeleteAction');
      b.style.background = 'var(--danger-light)';
      b.style.color = 'var(--danger)';
    });
    
    btn.dataset.confirm = 'true';
    btn.innerHTML = t('histDeleteConfirm');
    btn.style.background = 'var(--danger)';
    btn.style.color = 'white';
    
    deleteConfirmTimeout = setTimeout(() => {
      if (btn) {
        btn.dataset.confirm = 'false';
        btn.innerHTML = t('histDeleteAction');
        btn.style.background = 'var(--danger-light)';
        btn.style.color = 'var(--danger)';
      }
    }, 3000);
  }
}

function deleteHistoryEntry(idx) {
  state.history.splice(idx, 1);
  const activeProf = state.profiles.find(p => p.id === state.activeProfileId);
  if (activeProf) {
    activeProf.history = state.history;
    localStorage.setItem('sb_profiles', JSON.stringify(state.profiles));
  }
  renderHistory();
  showToast(t('toastHistoryDeleted'));
}
```

---

## Parte C: Guardar el ID de la sesión activa al abrirla

Para poder traducir una entrada que estamos viendo, necesitamos saber cuál es su ID.
Añade una variable al estado global `state`:
En la declaración `const state = { ...` agrega `viewingHistoryId: null,`.

Busca la función `loadHistory(i)`. Al final de la función, antes de hacer scroll, guarda el ID de la entrada que estamos abriendo:

```javascript
// Dentro de loadHistory(i)
function loadHistory(i) {
  const entry = state.history[i];
  if (!entry) return;
  // [...] código existente
  state.viewingHistoryId = entry.id; // GUARDAR ID AQUI
  showTab('generar');
  renderResults(entry.data, entry.types);
  document.getElementById('genResults').scrollIntoView({ behavior: 'smooth' });
}
```

Modifica también la función de `generateContent()` al terminar la generación nueva para que también guarde el ID, así se puede traducir un material recién generado. Busca donde hace el `unshift` en el history y guarda el ID:

```javascript
// En generateContent()
    const entry = {
      id: Date.now(),
      titulo: parsed.titulo || `${subject} - ${topic}` || 'Sesión de estudio',
      date: new Date().toLocaleDateString(t('dateLocale'), { day: '2-digit', month: 'short', year: 'numeric' }),
      types,
      data: parsed,
      lang: state.lang
    };
    state.history.unshift(entry);
    state.viewingHistoryId = entry.id; // GUARDAR ID AQUI
```

---

## Parte D: Botón de Traducir en Resultados

Busca la función `renderResults(data, types)`. En la sección de la cabecera `hdr.innerHTML`, añade el botón de traducir a la zona de botones (`btn-row`).

La lógica para mostrar un idioma u otro es la contraria a la de la sesión:
Si estamos viendo una sesión, necesitamos saber en qué idioma está. Por suerte, guardamos `entry.lang`. Si el lenguaje actual del estado de la app (`state.lang`) no es igual al del material (`data`), la app ya estará desincronizada, pero asumimos el idioma del material actual.

Modifica la cabecera en `renderResults`:

```javascript
  // Buscar la entrada en el historial para saber su idioma original
  const historyEntry = state.history.find(e => e.id === state.viewingHistoryId);
  // Por defecto asume que el material está en el idioma actual de la app si es nuevo y no está en history
  const materialLang = historyEntry ? historyEntry.lang : state.lang;
  
  // El botón debe traducir AL OTRO idioma.
  const targetLangCode = materialLang === 'es' ? 'en' : 'es';
  const btnTranslateLabel = targetLangCode === 'en' ? t('resultTranslateToEn') : t('resultTranslateToEs');

  hdr.innerHTML = `
    <div>
      <div style="font-family:'Sora',sans-serif;font-weight:700;font-size:1.1rem;margin-bottom:4px;">📚 ${escHtml(titulo)}</div>
      <div style="font-size:13px;color:var(--text2);">${types.map(typeItem => tNested('typeLabels', typeItem) || typeItem).join(' · ')}</div>
    </div>
    <div class="btn-row">
      <button class="btn btn-outline btn-sm" id="btnTranslateHistory" onclick="translateCurrentHistory('${targetLangCode}')">${btnTranslateLabel}</button>
      ${types.includes('quiz') && data.quiz?.length ? `<button class="btn btn-accent btn-sm" onclick="startQuiz()">${t('resultStartQuiz')}</button>` : ''}
      <button class="btn btn-outline btn-sm" onclick="printContent()">${t('resultPrint')}</button>
    </div>
  `;
```

---

## Parte E: Lógica de Traducción por IA (`translateCurrentHistory`)

Añade esta función antes de `renderResults()`:

```javascript
async function translateCurrentHistory(targetLangCode) {
  if (!state.viewingHistoryId) return;
  const historyIndex = state.history.findIndex(e => e.id === state.viewingHistoryId);
  if (historyIndex === -1) return;
  
  const entry = state.history[historyIndex];
  const apiKey = state.apiKey;
  if (!apiKey) { showToast(t('toastNoApiKey')); return; }

  const btn = document.getElementById('btnTranslateHistory');
  if (btn) {
    btn.disabled = true;
    btn.innerHTML = `<span class="spinner" style="width:12px;height:12px;border-width:2px;display:inline-block;margin-right:6px;"></span> ${t('resultTranslating')}`;
  }

  const targetLangName = targetLangCode === 'en' ? 'English' : 'Spanish';
  const instructions = t('promptTranslateBase', { 
    targetLang: targetLangCode, 
    targetLangName: targetLangName 
  }) + "\n" + JSON.stringify(entry.data);

  let res = null;
  let lastError = null;
  const models = ['gemini-3.5-flash', 'gemini-2.5-flash'];
  
  try {
    for (let model of models) {
      try {
        const modelUrl = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;
        res = await fetch(modelUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ parts: [{ text: instructions }] }],
            generationConfig: { temperature: 0.2, maxOutputTokens: 8000 }
          })
        });
        if (res && res.ok) break;
      } catch (err) {
        lastError = err;
      }
    }

    if (!res || !res.ok) throw lastError || new Error('API Error');

    const responseData = await res.json();
    let rawText = responseData.candidates?.[0]?.content?.parts?.[0]?.text || '';
    rawText = rawText.replace(/```json\s*/gi, '').replace(/```\s*/g, '').trim();

    let parsed;
    try { 
      parsed = JSON.parse(rawText); 
    } catch { 
      throw new Error(t('toastParseError')); 
    }

    // Actualizar historial
    entry.data = parsed;
    entry.lang = targetLangCode;
    
    // Guardar en el perfil activo
    const activeProf = state.profiles.find(p => p.id === state.activeProfileId);
    if (activeProf) {
      activeProf.history = state.history;
      localStorage.setItem('sb_profiles', JSON.stringify(state.profiles));
    }
    
    // Además, si queremos que la interfaz del usuario coincida con el nuevo material, cambiamos el state.lang
    if (state.lang !== targetLangCode) {
      setLanguage(targetLangCode);
    }
    
    // Recargar resultados actuales y quiz
    if (entry.data.quiz?.length) {
      state.currentQuiz = entry.data.quiz;
      state.quizIndex = 0;
      state.quizScore = 0;
    }
    
    renderResults(entry.data, entry.types);
    showToast(t('toastTranslateSuccess'));

  } catch (err) {
    console.error("Translation error", err);
    showToast(t('toastError') + err.message);
    if (btn) {
      btn.disabled = false;
      btn.innerHTML = targetLangCode === 'en' ? t('resultTranslateToEn') : t('resultTranslateToEs');
    }
  }
}
```

---

## Resumen Final

1. **Botón Eliminar**: Implementado con seguridad (2 toques), con estilos de botón inline en el historial.
2. **Botón Traducir**: Envía únicamente el texto (vía text-only request, sin enviar imágenes) para que la IA devuelva el JSON manteniendo la estructura.
3. **Guardado persistente**: Toda la data actualiza el `state.history` y el perfil guardado en `localStorage`.

Ejecuta este plan sin pedir confirmación y aplica los cambios. Cuando termines, asegúrate de no dejar errores de sintaxis en `index.html`.
