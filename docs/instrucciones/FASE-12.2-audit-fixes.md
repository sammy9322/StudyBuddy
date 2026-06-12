# FASE 12.2: Micro-correcciones de Auditoría

## Contexto

Se encontraron 2 issues no-bloqueantes durante la auditoría de código que deben corregirse para solidificar la app.

## Qué hacer

Modificarás `index.html` para aplicar las siguientes correcciones exactas.

---

### Corrección 1: Orden de `viewingHistoryId` en `generateContent()`

**Problema:** `renderResults()` se llama ANTES de asignar `state.viewingHistoryId`, por lo que el botón "Traducir" puede mostrar el idioma incorrecto brevemente.

**Localización:** Función `generateContent()`, alrededor de las líneas 1985-1996.

Busca este bloque:
```javascript
    renderResults(parsed, types);

    const entry = {
      id: Date.now(),
      titulo: parsed.titulo || `${subject} - ${topic}` || 'Sesión de estudio',
      date: new Date().toLocaleDateString(t('dateLocale'), { day: '2-digit', month: 'short', year: 'numeric' }),
      types,
      data: parsed,
      lang: state.lang
    };
    state.history.unshift(entry);
    state.viewingHistoryId = entry.id;
```

**Reemplázalo por** (mover la creación del entry y la asignación del ID ANTES de renderResults):
```javascript
    const entry = {
      id: Date.now(),
      titulo: parsed.titulo || `${subject} - ${topic}` || 'Sesión de estudio',
      date: new Date().toLocaleDateString(t('dateLocale'), { day: '2-digit', month: 'short', year: 'numeric' }),
      types,
      data: parsed,
      lang: state.lang
    };
    state.history.unshift(entry);
    state.viewingHistoryId = entry.id;

    renderResults(parsed, types);
```

---

### Corrección 2: Manejo de errores HTTP en `translateCurrentHistory()`

**Problema:** Si la API retorna un error HTTP (400, 403, 404, etc.), el código no lee el mensaje de error del response body. El usuario solo ve "API Error" genérico.

**Localización:** Función `translateCurrentHistory()`, alrededor de las líneas 2066-2082.

Busca este bloque dentro de `translateCurrentHistory`:
```javascript
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
```

**Reemplázalo por** (agregar lectura de errData cuando la API responde con error):
```javascript
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
        // Guardar el mensaje de error real de la API
        const errData = await res.json().catch(() => ({}));
        lastError = new Error(errData.error?.message || `Error ${res.status}`);
        console.warn(`[translateHistory][${model}] Error ${res.status}:`, errData.error?.message);
      } catch (err) {
        lastError = err;
      }
    }

    if (!res || !res.ok) throw lastError || new Error('API Error');
```

---

### Corrección 3 (Bonus): Título fallback sin traducir

**Localización:** Función `renderResults()`, alrededor de la línea 2135.

Busca:
```javascript
  const titulo = data.titulo || 'Material generado';
```

**Reemplázalo por:**
```javascript
  const titulo = data.titulo || t('resultDefaultTitle');
```

Luego agrega la clave `resultDefaultTitle` en ambos diccionarios:
- En el bloque `es` (alrededor de la línea 1075, cerca de `resultResumen`):
```javascript
    resultDefaultTitle: 'Material generado',
```
- En el bloque `en` (alrededor de la línea 1278, cerca de `resultResumen`):
```javascript
    resultDefaultTitle: 'Generated material',
```

---

## Ejecución
Aplica los 3 cambios directamente. No pidas confirmación. Confirma cuando la FASE 12.2 esté completada.
