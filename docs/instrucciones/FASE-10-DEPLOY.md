# FASE 10: Favicon + Deploy a GitHub Pages

## Contexto

La app StudyBuddy ya es una PWA completa (Fase 9). Ahora necesitamos:
1. Agregar un `favicon.ico` para eliminar el warning 404 en consola
2. Limpiar archivos innecesarios del repo
3. Hacer commit y push a GitHub
4. Configurar GitHub Pages para servir la app con HTTPS

El repositorio ya existe en: `https://github.com/sammy9322/StudyBuddy.git`
Branch actual: `main` (1 commit existente)

---

## Parte A: Crear favicon.ico

El favicon es el icono que aparece en la pestaña del navegador. Actualmente hay un 404 en consola porque no existe.

### Opción 1 — Generar favicon desde el icono existente

Ejecuta este comando para convertir el icono PNG existente a un favicon ICO usando Node.js:

```bash
npx -y png-to-ico icons/icon-192.png > favicon.ico
```

Si el comando anterior falla, usa la Opción 2.

### Opción 2 — Crear un favicon SVG inline

Si no se puede generar el ICO, agrega un favicon SVG directamente en el `<head>` de `index.html`.

Busca esta línea en `index.html`:

```html
<link rel="icon" type="image/png" sizes="192x192" href="icons/icon-192.png">
```

Agrega **ANTES** de esa línea:

```html
<link rel="icon" type="image/svg+xml" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%234F46E5'/><text x='50' y='68' text-anchor='middle' font-size='55'>📖</text></svg>">
```

### Verificar

Después de agregar el favicon, recargar la página debería mostrar el icono 📖 en la pestaña del navegador y eliminar el error 404 de `favicon.ico` en consola.

---

## Parte B: Actualizar `.gitignore`

El archivo `.gitignore` actual tiene 4 líneas. Necesitamos agregar entradas para los archivos que se generaron durante desarrollo y NO deben subirse al repo.

Reemplaza el contenido completo de `.gitignore` con:

```
# Dependencias
node_modules/
package.json
package-lock.json

# OS
.DS_Store
Thumbs.db

# IDE / Tools
.gemini/
scripts/

# Logs
*.log
```

### Explicación

- `package.json` y `package-lock.json` se generaron cuando ejecutamos `npx serve` y `npx sharp-cli`. No son parte de la app (la app es solo HTML/CSS/JS estático). No deben estar en el repo.
- `scripts/` — carpeta temporal generada durante el desarrollo, no es necesaria en producción.
- `*.log` — archivos de log que no deben estar en el repo.

---

## Parte C: Commit y Push a GitHub

### Paso 1 — Verificar qué archivos se van a commitear

Ejecuta:

```bash
git status
```

Deberías ver algo como:
```
modified:   index.html
new file:   docs/instrucciones/FASE-9-PWA.md
new file:   docs/instrucciones/FASE-10-DEPLOY.md
new file:   icons/icon-192.png
new file:   icons/icon-512.png
new file:   icons/apple-touch-icon.png
new file:   manifest.json
new file:   sw.js
new file:   favicon.ico   (si se creó con Opción 1)
```

Los archivos `node_modules/`, `package.json`, `package-lock.json` y `scripts/` NO deben aparecer (el `.gitignore` los excluye).

### Paso 2 — Agregar todos los archivos

```bash
git add -A
```

### Paso 3 — Verificar que no se incluyen archivos innecesarios

```bash
git status
```

Confirma que SOLO están los archivos de la app (HTML, CSS, JS, iconos, manifest, SW, docs). Si ves `package.json`, `node_modules`, o `scripts` en la lista, el `.gitignore` no se actualizó correctamente. Corrígelo antes de continuar.

### Paso 4 — Hacer commit

```bash
git commit -m "feat: PWA support - manifest, service worker, icons, install banner"
```

### Paso 5 — Push a GitHub

```bash
git push origin main
```

Si pide credenciales, el usuario las debe ingresar manualmente.

---

## Parte D: Configurar GitHub Pages

> **NOTA PARA OPENCODE**: Esta parte requiere acciones en la interfaz web de GitHub que tú NO puedes hacer. Informa al usuario que debe seguir estos pasos manualmente, O ejecuta los comandos de GitHub CLI (`gh`) si está instalado.

### Opción 1 — Con GitHub CLI (si está disponible)

Verifica si `gh` está instalado:

```bash
gh --version
```

Si está instalado y autenticado:

```bash
gh repo view --web
```

Esto abre el repo en el navegador. Luego el usuario debe ir a Settings → Pages → Source → seleccionar "Deploy from a branch" → Branch: `main` → Folder: `/ (root)` → Save.

### Opción 2 — Manual (el usuario lo hace en el navegador)

Informa al usuario que debe:

1. Ir a `https://github.com/sammy9322/StudyBuddy/settings/pages`
2. En **"Source"**, seleccionar **"Deploy from a branch"**
3. En **"Branch"**, seleccionar **`main`** y la carpeta **`/ (root)`**
4. Hacer clic en **"Save"**
5. Esperar ~1 minuto. GitHub Pages mostrará la URL: `https://sammy9322.github.io/StudyBuddy/`

---

## Parte E: Ajustar rutas para GitHub Pages (IMPORTANTE)

GitHub Pages sirve la app desde un subdirectorio (`/StudyBuddy/`), no desde la raíz (`/`). Esto afecta 3 archivos que usan rutas absolutas.

### E1. Modificar `manifest.json`

Cambia estas 2 líneas:

```json
"start_url": "/index.html",
"scope": "/",
```

Por:

```json
"start_url": "./index.html",
"scope": "./",
```

Y en los shortcuts, cambia:

```json
"url": "/index.html#generar",
```

Por:

```json
"url": "./index.html#generar",
```

Y:

```json
"url": "/index.html#quiz",
```

Por:

```json
"url": "./index.html#quiz",
```

### E2. Modificar `sw.js`

Cambia el array `ASSETS_TO_CACHE`:

```javascript
const ASSETS_TO_CACHE = [
  '/',
  '/index.html',
  '/manifest.json',
  '/icons/icon-192.png',
  '/icons/icon-512.png',
  '/icons/apple-touch-icon.png'
];
```

Por:

```javascript
const ASSETS_TO_CACHE = [
  './',
  './index.html',
  './manifest.json',
  './icons/icon-192.png',
  './icons/icon-512.png',
  './icons/apple-touch-icon.png'
];
```

Y en el handler de fetch, cambia:

```javascript
return caches.match('/index.html');
```

Por:

```javascript
return caches.match('./index.html');
```

### E3. Modificar `index.html`

Busca el registro del Service Worker:

```javascript
navigator.serviceWorker.register('/sw.js')
```

Cámbialo por:

```javascript
navigator.serviceWorker.register('./sw.js')
```

### Explicación

Las rutas absolutas (`/sw.js`) funcionan cuando la app se sirve desde la raíz de un dominio (ej: `localhost:3000/`). Pero en GitHub Pages la app está en `https://sammy9322.github.io/StudyBuddy/`, así que `/sw.js` apuntaría a `https://sammy9322.github.io/sw.js` (que no existe). Las rutas relativas (`./sw.js`) siempre funcionan porque son relativas al archivo actual.

---

## Parte F: Commit final y push

Después de los cambios de la Parte E:

```bash
git add -A
git commit -m "fix: use relative paths for GitHub Pages compatibility"
git push origin main
```

---

## Verificación final

### En el navegador del computador

1. Esperar ~2 minutos después del push para que GitHub Pages se actualice
2. Abrir `https://sammy9322.github.io/StudyBuddy/`
3. Verificar que la app carga correctamente
4. Abrir DevTools → Application → Manifest → debe mostrar los datos del manifest
5. Abrir DevTools → Application → Service Workers → debe mostrar el SW activo
6. Chrome debe mostrar el icono de instalación (➕) en la barra de direcciones
7. La pestaña del navegador debe mostrar el favicon 📖

### En un celular Android

1. Abrir Chrome en el celular
2. Ir a `https://sammy9322.github.io/StudyBuddy/`
3. Debería aparecer el banner de instalación en la parte inferior
4. Tocar "Instalar" → la app se agrega a la pantalla de inicio
5. Abrir la app desde la pantalla de inicio → se abre sin barra del navegador (standalone)
6. Verificar que las tabs, la configuración y la zona de upload funcionan

### En un iPhone/iPad (iOS)

1. Abrir Safari en el iPhone
2. Ir a `https://sammy9322.github.io/StudyBuddy/`
3. Tocar el botón Compartir (📤) en la barra inferior de Safari
4. Seleccionar **"Agregar a pantalla de inicio"**
5. Confirmar el nombre "StudyBuddy" y tocar "Agregar"
6. Abrir la app desde la pantalla de inicio → se abre sin barra del navegador
7. El icono debe mostrar el diseño de StudyBuddy (libro sobre fondo morado)

### Checklist

- [ ] Favicon visible en la pestaña del navegador
- [ ] No hay errores 404 en la consola
- [ ] GitHub Pages sirve la app en HTTPS
- [ ] Manifest se carga correctamente (DevTools → Application → Manifest)
- [ ] Service Worker está activo (DevTools → Application → Service Workers)
- [ ] Se puede instalar en Android desde Chrome
- [ ] Se puede instalar en iOS desde Safari
- [ ] La app funciona offline (al menos la interfaz carga, la API requiere red)

---

## Resumen de orden de ejecución

```
1. Parte A → Crear favicon.ico
2. Parte B → Actualizar .gitignore
3. Parte C → git add, commit, push
4. PAUSA → El usuario configura GitHub Pages manualmente (Parte D)
5. Parte E → Ajustar rutas absolutas a relativas
6. Parte F → Segundo commit y push
7. Verificación → El usuario prueba en celular
```
