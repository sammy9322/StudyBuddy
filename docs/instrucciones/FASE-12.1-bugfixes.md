# FASE 12.1: Corrección de Bugs (Modelos y Fallback de Idioma)

## Contexto

Se han detectado dos fallos introducidos en versiones anteriores:
1. Al intentar generar contenido o traducir el historial, la aplicación se queda cargando indefinidamente porque se intenta contactar a modelos de Gemini inexistentes (`gemini-3.5-flash`). Al retornar un error HTTP silencioso (como 404 Not Found), el ciclo de reintentos falla sin mostrar el error al usuario correctamente.
2. Al visualizar una entrada de historial antigua (que no posee la propiedad `lang`), el sistema asume que está en el mismo idioma que la UI (`state.lang`), por lo que si el usuario tiene la UI en inglés, el botón sugiere equivocadamente "Traducir a Español".

## Qué hacer

Modificarás `index.html` para aplicar las siguientes correcciones exactas.

### 1. Arreglar Modelos de IA en toda la aplicación

Existen dos lugares en el código donde se declaran los modelos:
- En la función `generateContent()` (alrededor de la línea 1915).
- En la función `translateCurrentHistory()` (alrededor de la línea 2064).

Busca en ambas funciones la siguiente línea:
```javascript
  const models = ['gemini-3.5-flash', 'gemini-2.5-flash'];
```

**Reemplázala en AMBOS lugares por:**
```javascript
  const models = ['gemini-1.5-flash', 'gemini-1.5-pro'];
```

*(Con esto nos aseguramos de que el fetch sí contacte a un endpoint válido).*

### 2. Arreglar el idioma por defecto del historial

En la función `renderResults(data, types)` (alrededor de la línea 2138), el sistema intenta averiguar el idioma del material actual.

Busca esta línea:
```javascript
  const materialLang = historyEntry ? historyEntry.lang : state.lang;
```

**Reemplázala por:**
```javascript
  // El historial antiguo se generó en Español, por lo que el fallback siempre debe ser 'es'
  const materialLang = (historyEntry && historyEntry.lang) ? historyEntry.lang : 'es';
```

---

## Ejecución
Aplica estos cambios directamente a `index.html`. No pidas confirmación extra, simplemente reescribe las líneas como se ha instruido y confirma cuando la FASE 12.1 esté completada.
