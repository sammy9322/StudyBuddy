# FASE 7: Revisión de Código y Simplificación

## Contexto

El archivo `index.html` ya tiene TODA la funcionalidad implementada (Fases 1-6). Ahora debes usar las skills de revisión de código y simplificación para asegurar la calidad del resultado final.

## Skills a usar

1. **Lee y aplica** `.agents/skills/code-reviewer/SKILL.md`
2. **Lee y aplica** `.agents/skills/simplify/SKILL.md`

---

## Parte A: Code Review

Revisa el archivo `index.html` completo usando el flujo de la skill code-reviewer:

### 1. Context
- La app es una SPA educativa single-file (HTML+CSS+JS) para niños de 6-15 años
- Usa Google Gemini API directamente desde el frontend
- Almacena datos en localStorage
- No tiene dependencias externas

### 2. Checklist de revisión

**Seguridad:**
- [ ] ¿La función `escHtml()` se usa en TODOS los lugares donde se renderiza texto del usuario o de la API? Busca todos los usos de `${...}` dentro de innerHTML y template literals
- [ ] ¿Hay riesgo de XSS en el resumen o mapa conceptual? (Nota: estos se renderizan como HTML crudo intencionalmente para mostrar formato)
- [ ] ¿La API Key se maneja correctamente? (localStorage, no se envía a servidores propios)

**Bugs potenciales:**
- [ ] ¿Qué pasa si `data.candidates` es undefined o vacío?
- [ ] ¿Qué pasa si el usuario hace clic rápido en "Generar" dos veces?
- [ ] ¿Qué pasa si `state.history` en localStorage está corrupto?
- [ ] ¿Los event listeners de las pills se acumulan si se re-renderizan?

**Rendimiento:**
- [ ] ¿Imágenes grandes en base64 podrían causar problemas de memoria?
- [ ] ¿El historial podría crecer demasiado en localStorage?

**UX:**
- [ ] ¿Todos los estados de error tienen feedback visual?
- [ ] ¿El usuario siempre sabe qué está pasando (loading states)?

### 3. Genera el reporte
Usa el formato de la skill:
1. Summary
2. Critical issues (bugs, seguridad)
3. Major issues (rendimiento, diseño)
4. Minor issues (naming, legibilidad)
5. Positive feedback
6. Verdict

---

## Parte B: Simplificación

Después del review, aplica la skill simplify al código:

1. **NO cambies funcionalidad** — solo mejora cómo está escrito
2. Busca y corrige:
   - Código duplicado que se pueda consolidar
   - Variables con nombres poco claros
   - Nesting excesivo que se pueda aplanar
   - Template literals muy largos que se puedan hacer más legibles
   - Funciones que hagan demasiadas cosas y se puedan separar
3. **NO elimines comentarios útiles** que expliquen el "por qué"
4. **NO cambies la estructura de las funciones públicas** (las que se llaman desde onclick en el HTML)

---

## Parte C: Aplica las correcciones

Aplica directamente al archivo `index.html`:
- Todas las correcciones de bugs encontradas en el review
- Las simplificaciones que mejoren legibilidad
- Cualquier escHtml() faltante en renderizados dinámicos

---

## Verificación

Al terminar:
1. La app debe funcionar exactamente igual que antes
2. No deben haber aparecido errores nuevos en consola
3. El código debe ser más limpio y legible
4. Los issues de seguridad identificados deben estar documentados o corregidos
