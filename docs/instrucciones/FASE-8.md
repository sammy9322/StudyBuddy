# FASE 8: Verificación Final

## Contexto

El archivo `index.html` ya tiene toda la funcionalidad implementada y fue revisado/simplificado. Esta fase es la verificación final contra el plan de implementación.

## Documentos de referencia

- **docs/PLAN_DE_IMPLEMENTACION.html** → Secciones 4 (Manejo de Errores), 5 (Seguridad), 7 (Plan de Pruebas)
- **index.html** → El archivo completo a verificar

## Qué hacer

Lee el plan de implementación completo y verifica cada punto contra el código en `index.html`. Corrige cualquier brecha que encuentres.

---

## Checklist de verificación

### 1. Funcionalidad completa
Verifica que TODAS estas funciones existen y están conectadas:

- [ ] `showTab(tab)` — navegación entre pestañas
- [ ] `saveConfig()` — guardar nombre, grado, API Key
- [ ] `toggleApiKey()` — toggle password/text
- [ ] `handleFiles(evt)` — subir archivos
- [ ] `handleDrop(evt)` — drag & drop
- [ ] `handleDragOver(evt)` — feedback visual drag
- [ ] `handleDragLeave()` — quitar feedback drag
- [ ] `addFiles(files)` — procesar archivos con FileReader
- [ ] `renderPreviews()` — grid de previews
- [ ] `removeImage(i)` — eliminar imagen
- [ ] `openCamera()` — acceso a cámara
- [ ] `capturePhoto()` — capturar foto
- [ ] `closeCamera()` — cerrar cámara
- [ ] `generateContent()` — llamada a Gemini API
- [ ] `setGenLoading(loading, msg)` — estado de carga
- [ ] `setGenStatusMsg(msg)` — mensaje de estado
- [ ] `setProgress(pct)` — barra de progreso
- [ ] `renderResults(data, types)` — mostrar resultados
- [ ] `makeResultBlock(title, html, color)` — bloque reutilizable
- [ ] `startQuiz()` — iniciar quiz
- [ ] `renderQuizQuestion()` — mostrar pregunta
- [ ] `answerQuiz(idx)` — responder pregunta
- [ ] `nextQuestion()` — siguiente pregunta
- [ ] `showQuizResults()` — resultados del quiz
- [ ] `renderHistory()` — lista de historial
- [ ] `loadHistory(i)` — cargar sesión
- [ ] `escHtml(str)` — sanitizar HTML
- [ ] `showToast(msg)` — notificación
- [ ] `copyText(text)` — copiar al portapapeles
- [ ] `printContent()` — imprimir

### 2. Tests funcionales (Sección 7.1 del plan)

Verifica que el código soporta estos 15 escenarios:

| # | Test | Qué verificar en el código |
|---|------|---------------------------|
| 1 | Guardar y cargar config | `saveConfig()` guarda en localStorage, `init()` lo lee |
| 2 | Subir imagen desde archivo | `handleFiles` → `addFiles` → `renderPreviews` |
| 3 | Drag & drop | `handleDrop` filtra image/*, agrega clase visual |
| 4 | Capturar foto | `capturePhoto` dibuja en canvas, pushea a state |
| 5 | Eliminar imagen | `removeImage` hace splice y re-render |
| 6 | Generar resumen | `renderResults` verifica `types.includes('resumen')` |
| 7 | Generar quiz | Verifica 10 preguntas con opciones |
| 8 | Flashcards flip | addEventListener click con toggle de dataset.flipped |
| 9 | Completar quiz | `showQuizResults` con emoji y mensaje según % |
| 10 | Historial guarda | `generateContent` hace unshift + localStorage |
| 11 | Cargar historial | `loadHistory` re-renderiza y carga quiz |
| 12 | Sin API Key | Toast "⚠️ Primero configura tu API Key" |
| 13 | Sin imágenes | Toast "⚠️ Sube al menos una imagen" |
| 14 | API Key inválida | Error capturado con err.error?.message |
| 15 | Imprimir | `printContent` abre window con estilos |

### 3. Manejo de errores (Sección 4 del plan)

Verifica que estos escenarios están cubiertos:

- [ ] Sin API Key → toast + return (NO hace fetch)
- [ ] Sin imágenes → toast + return (NO hace fetch)
- [ ] Error de cámara → try/catch en `openCamera` con toast
- [ ] Error de clipboard → .catch en `copyText`
- [ ] HTTP no-OK → extrae `err.error?.message`
- [ ] JSON inválido de la IA → try/catch con mensaje amigable
- [ ] Timeout/red → catch genérico con `err.message`
- [ ] genTypes nunca vacío → validación `size > 1` antes de delete

### 4. Seguridad (Sección 5 del plan)

- [ ] `escHtml()` se usa donde se renderiza input del usuario (títulos, preguntas quiz, opciones, flashcards, historial)
- [ ] API Key solo en localStorage, no en el HTML visible
- [ ] Input de API Key es type="password" por defecto

### 5. Responsive (Media query)

- [ ] Existe `@media (max-width: 600px)` con:
  - Header padding reducido
  - Container padding reducido
  - Form row en 1 columna
  - Tab font-size 12px
  - Tab emoji display none

### 6. localStorage

- [ ] Clave `sb_config`: { name, grade, apiKey }
- [ ] Clave `sb_history`: array de entries con { id, titulo, date, types, data }
- [ ] Límite de 30 entradas en historial

### 7. Prompt a Gemini

- [ ] Incluye nombre del estudiante
- [ ] Incluye descripción del grado (gradeMap)
- [ ] Incluye materia y tema
- [ ] Incluye notas opcionales del tutor
- [ ] Pide JSON puro sin markdown
- [ ] Estructura dinámica según tipos seleccionados
- [ ] "Responde SIEMPRE en español"

---

## Si encuentras brechas

Corrígelas directamente en `index.html`. No dejes nada pendiente.

---

## Reporte final

Al terminar, reporta:
1. ¿Cuántas funciones verificadas? (objetivo: 30/30)
2. ¿Cuántas brechas encontradas y corregidas?
3. ¿Algún issue que no se pueda corregir sin cambiar la arquitectura?
4. Estado final: ✅ Lista para testing manual / ⚠️ Necesita atención
