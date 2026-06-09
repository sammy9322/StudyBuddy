# FASE 9: Conversión a Progressive Web App (PWA)

## Contexto

El archivo `index.html` ya tiene toda la funcionalidad completa (Fases 1-8). Ahora vamos a convertir StudyBuddy en una **Progressive Web App (PWA)** para que los usuarios puedan instalarla en sus dispositivos iOS y Android directamente desde el navegador, sin necesidad de tiendas de aplicaciones.

Una PWA requiere 3 cosas:
1. Un **Web App Manifest** (`manifest.json`) — define nombre, iconos y comportamiento de la app
2. Un **Service Worker** (`sw.js`) — permite caché offline y la instalación
3. **Meta tags especiales** en el HTML — para compatibilidad con iOS y configuración visual

## Documentos de referencia

- **index.html** → El archivo principal que debe modificarse (solo el `<head>`)
- **docs/PLAN_DE_IMPLEMENTACION.html** → Referencia de la arquitectura general

## Archivos a crear

| Archivo | Ubicación | Propósito |
|---------|-----------|-----------|
| `manifest.json` | Raíz del proyecto (`c:\Documentos\samuelss\Mis documentos\app\manifest.json`) | Manifiesto de la PWA |
| `sw.js` | Raíz del proyecto (`c:\Documentos\samuelss\Mis documentos\app\sw.js`) | Service Worker |
| `icons/icon-192.png` | Carpeta icons en raíz | Icono Android/Chrome 192×192 |
| `icons/icon-512.png` | Carpeta icons en raíz | Icono Android/Chrome 512×512 |
| `icons/apple-touch-icon.png` | Carpeta icons en raíz | Icono iOS 180×180 |

## Archivo a modificar

| Archivo | Qué cambia |
|---------|------------|
| `index.html` | Agregar meta tags PWA en el `<head>` + registro del Service Worker al final del `<script>` |

---

## Parte A: Crear `manifest.json`

Crea el archivo `manifest.json` en la **raíz del proyecto** (al lado de `index.html`) con este contenido exacto:

```json
{
  "name": "StudyBuddy - Tu asistente de estudio con IA",
  "short_name": "StudyBuddy",
  "description": "Fotografía tu material escolar y genera resúmenes, quizzes y flashcards con inteligencia artificial. Para niños de 6 a 15 años.",
  "start_url": "/index.html",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#F8F7FF",
  "theme_color": "#4F46E5",
  "lang": "es",
  "dir": "ltr",
  "categories": ["education", "productivity"],
  "icons": [
    {
      "src": "icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icons/apple-touch-icon.png",
      "sizes": "180x180",
      "type": "image/png"
    }
  ],
  "screenshots": [],
  "shortcuts": [
    {
      "name": "Generar material",
      "short_name": "Generar",
      "url": "/index.html#generar",
      "icons": [
        {
          "src": "icons/icon-192.png",
          "sizes": "192x192"
        }
      ]
    },
    {
      "name": "Practicar Quiz",
      "short_name": "Quiz",
      "url": "/index.html#quiz",
      "icons": [
        {
          "src": "icons/icon-192.png",
          "sizes": "192x192"
        }
      ]
    }
  ]
}
```

### Explicación de cada campo:
- `name`: Nombre completo que aparece en la pantalla de instalación
- `short_name`: Nombre corto que aparece debajo del icono en la pantalla de inicio (máx ~12 caracteres)
- `display: "standalone"`: La app se abre SIN la barra de direcciones del navegador, como una app nativa
- `orientation: "portrait"`: Fuerza orientación vertical (ideal para el público infantil en celular)
- `background_color: "#F8F7FF"`: Color de fondo del splash screen (coincide con `--surface2` de la app)
- `theme_color: "#4F46E5"`: Color de la barra de estado del sistema (coincide con `--primary`)
- `purpose: "any maskable"`: Los iconos se adaptan a la forma del ícono del OS (círculo, cuadrado redondeado, etc.)
- `shortcuts`: Accesos directos que aparecen al mantener presionado el icono de la app (Android)

---

## Parte B: Crear `sw.js` (Service Worker)

Crea el archivo `sw.js` en la **raíz del proyecto** (al lado de `index.html`) con este contenido exacto:

```javascript
// ─────────────────────────────────────────────
// StudyBuddy — Service Worker v1.0
// ─────────────────────────────────────────────

const CACHE_NAME = 'studybuddy-v1';

// Archivos que se guardan en caché al instalar
const ASSETS_TO_CACHE = [
  '/',
  '/index.html',
  '/manifest.json',
  '/icons/icon-192.png',
  '/icons/icon-512.png',
  '/icons/apple-touch-icon.png'
];

// ── INSTALL ──
// Se ejecuta cuando el Service Worker se instala por primera vez.
// Descarga y almacena en caché todos los archivos esenciales.
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('[SW] Cacheando archivos esenciales');
        return cache.addAll(ASSETS_TO_CACHE);
      })
      .then(() => self.skipWaiting()) // Activa inmediatamente sin esperar
  );
});

// ── ACTIVATE ──
// Se ejecuta cuando una nueva versión del SW toma control.
// Limpia cachés antiguas para no desperdiciar espacio.
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== CACHE_NAME)
          .map((name) => {
            console.log('[SW] Eliminando caché antigua:', name);
            return caches.delete(name);
          })
      );
    }).then(() => self.clients.claim()) // Toma control de todas las pestañas abiertas
  );
});

// ── FETCH ──
// Intercepta todas las peticiones de red.
// Estrategia: "Network First, Cache Fallback"
//   - Primero intenta la red (para tener siempre contenido fresco)
//   - Si la red falla (offline), sirve desde la caché
//   - EXCEPCIÓN: Las llamadas a la API de Gemini NUNCA se cachean
self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);

  // NO interceptar llamadas a la API de Gemini (requieren red)
  if (url.hostname === 'generativelanguage.googleapis.com') {
    return; // Deja que el navegador las maneje normalmente
  }

  // NO interceptar peticiones a Google Fonts (se cachean por el navegador)
  if (url.hostname.includes('googleapis.com') || url.hostname.includes('gstatic.com')) {
    return;
  }

  // Para todo lo demás: Network First
  event.respondWith(
    fetch(event.request)
      .then((networkResponse) => {
        // Si la respuesta de red es válida, la guardamos en caché
        if (networkResponse && networkResponse.status === 200) {
          const responseClone = networkResponse.clone();
          caches.open(CACHE_NAME).then((cache) => {
            cache.put(event.request, responseClone);
          });
        }
        return networkResponse;
      })
      .catch(() => {
        // Si la red falla, intentamos servir desde la caché
        return caches.match(event.request).then((cachedResponse) => {
          if (cachedResponse) {
            return cachedResponse;
          }
          // Si tampoco está en caché y es una navegación, servir index.html
          if (event.request.mode === 'navigate') {
            return caches.match('/index.html');
          }
          // Para otros recursos, no hay nada que hacer
          return new Response('Offline', {
            status: 503,
            statusText: 'Service Unavailable'
          });
        });
      })
  );
});
```

### Explicación del Service Worker:

- **Evento `install`**: Se dispara la primera vez que el usuario visita la app. Descarga y guarda en caché el HTML, los iconos y el manifiesto. `skipWaiting()` asegura que la nueva versión se active de inmediato.

- **Evento `activate`**: Se dispara cuando una nueva versión del SW reemplaza la anterior. Borra las cachés viejas para liberar espacio.

- **Evento `fetch`**: Intercepta cada petición HTTP. Usa la estrategia "Network First":
  - Si hay red → usa la respuesta fresca y actualiza la caché.
  - Si NO hay red → sirve la copia almacenada en caché.
  - Las llamadas a Gemini API y Google Fonts se excluyen porque necesitan red en tiempo real o ya se gestionan por el navegador.

- **Para actualizar la app**: Cuando modifiques `index.html` en producción, cambia `CACHE_NAME` de `'studybuddy-v1'` a `'studybuddy-v2'` (etc.) y el SW viejo será reemplazado automáticamente.

---

## Parte C: Crear los iconos

Crea la carpeta `icons/` en la raíz del proyecto y genera 3 iconos PNG.

### Diseño de los iconos

Todos los iconos deben seguir este diseño:
- **Fondo**: Gradiente de `#4F46E5` (indigo/primary) a `#7C3AED` (purple), en dirección 135°
- **Elemento central**: El emoji 📖 (libro abierto) grande y centrado, en color blanco
- **Forma**: Cuadrado con esquinas redondeadas (el SO se encarga del masking)
- **Estilo**: Limpio, infantil, amigable — coherente con la identidad visual de StudyBuddy

### Tamaños requeridos

| Archivo | Tamaño | Uso |
|---------|--------|-----|
| `icons/icon-192.png` | 192 × 192 px | Icono en Android, pantalla de instalación Chrome |
| `icons/icon-512.png` | 512 × 512 px | Splash screen en Android, Play Store si se usa TWA |
| `icons/apple-touch-icon.png` | 180 × 180 px | Icono en iOS (pantalla de inicio del iPhone/iPad) |

### IMPORTANTE sobre safe zone para `maskable`

Los iconos de 192 y 512 tienen `purpose: "any maskable"` en el manifiesto. Esto significa que el OS puede recortar el icono en diferentes formas. El contenido importante (el emoji del libro) debe estar dentro del **80% central** del icono. Deja un 10% de padding seguro en cada borde.

---

## Parte D: Modificar `index.html`

### D1. Agregar meta tags PWA en el `<head>`

Busca esta línea en `index.html`:

```html
<link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900&family=Sora:wght@400;600;700&display=swap" rel="stylesheet">
```

**Inmediatamente DESPUÉS** de esa línea (antes de `<style>`), agrega este bloque completo:

```html
<!-- ── PWA: Manifest ── -->
<link rel="manifest" href="manifest.json">

<!-- ── PWA: Theme color (barra de estado en Android) ── -->
<meta name="theme-color" content="#4F46E5">

<!-- ── PWA: iOS Support ── -->
<!-- iOS no lee el manifest.json para todo, necesita estas meta tags -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="StudyBuddy">
<link rel="apple-touch-icon" href="icons/apple-touch-icon.png">

<!-- ── PWA: Iconos generales ── -->
<link rel="icon" type="image/png" sizes="192x192" href="icons/icon-192.png">
<link rel="icon" type="image/png" sizes="512x512" href="icons/icon-512.png">

<!-- ── SEO / App Description ── -->
<meta name="description" content="Fotografía tu material escolar y genera resúmenes, quizzes y flashcards con IA. Para niños de 6 a 15 años.">
```

### Explicación de cada meta tag:

| Meta tag | Propósito |
|----------|-----------|
| `<link rel="manifest">` | Conecta el archivo manifest.json con la página |
| `<meta name="theme-color">` | Colorea la barra de estado del sistema operativo en Android |
| `apple-mobile-web-app-capable` | Le dice a Safari que esta web puede funcionar como app standalone |
| `apple-mobile-web-app-status-bar-style` | `black-translucent` hace que la barra de estado de iOS sea semi-transparente y el contenido se extienda debajo |
| `apple-mobile-web-app-title` | Nombre que aparece bajo el icono en iOS |
| `<link rel="apple-touch-icon">` | Icono específico para iOS (Safari no lee los iconos del manifest) |
| `<link rel="icon">` | Favicon/icono para navegadores de escritorio |
| `<meta name="description">` | SEO y descripción que aparece al compartir el link |

### D2. Registrar el Service Worker

Busca la línea de cierre del script, justo antes de `</script>`:

```javascript
  toastTimeout = setTimeout(() => { toast.style.opacity = '0'; }, 2800);
}
```

**Inmediatamente DESPUÉS** de esa función `showToast`, agrega este bloque:

```javascript
// ── PWA: REGISTRO DEL SERVICE WORKER ──
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((reg) => {
        console.log('[PWA] Service Worker registrado:', reg.scope);
      })
      .catch((err) => {
        console.log('[PWA] Error al registrar SW:', err);
      });
  });
}

// ── PWA: PROMPT DE INSTALACIÓN ──
let deferredPrompt = null;

window.addEventListener('beforeinstallprompt', (e) => {
  // Prevenir que el navegador muestre su propio banner de instalación
  e.preventDefault();
  // Guardar el evento para usarlo después
  deferredPrompt = e;
  // Mostrar nuestro propio botón/banner de instalación
  showInstallBanner();
});

window.addEventListener('appinstalled', () => {
  console.log('[PWA] App instalada exitosamente');
  deferredPrompt = null;
  hideInstallBanner();
  showToast('🎉 ¡StudyBuddy instalada! Búscala en tu pantalla de inicio.');
});

function showInstallBanner() {
  // Solo mostrar si no existe ya
  if (document.getElementById('installBanner')) return;

  const banner = document.createElement('div');
  banner.id = 'installBanner';
  banner.style.cssText = `
    position: fixed;
    bottom: 24px;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(135deg, #4F46E5 0%, #7C3AED 100%);
    color: white;
    padding: 14px 20px;
    border-radius: 16px;
    box-shadow: 0 8px 32px rgba(79,70,229,0.35);
    display: flex;
    align-items: center;
    gap: 14px;
    z-index: 9998;
    font-family: 'Nunito', sans-serif;
    max-width: 420px;
    width: calc(100% - 32px);
    animation: fadeUp 0.35s ease both;
  `;
  banner.innerHTML = `
    <div style="font-size: 2rem;">📲</div>
    <div style="flex: 1;">
      <div style="font-weight: 800; font-size: 14px; margin-bottom: 2px;">Instalar StudyBuddy</div>
      <div style="font-size: 12px; opacity: 0.85;">Agrégala a tu pantalla de inicio para acceso rápido</div>
    </div>
    <button onclick="installApp()" style="
      background: white;
      color: #4F46E5;
      border: none;
      padding: 8px 16px;
      border-radius: 10px;
      font-family: 'Nunito', sans-serif;
      font-weight: 800;
      font-size: 13px;
      cursor: pointer;
      white-space: nowrap;
    ">Instalar</button>
    <button onclick="hideInstallBanner()" style="
      background: none;
      border: none;
      color: rgba(255,255,255,0.7);
      font-size: 18px;
      cursor: pointer;
      padding: 4px;
    ">✕</button>
  `;
  document.body.appendChild(banner);
}

function hideInstallBanner() {
  const banner = document.getElementById('installBanner');
  if (banner) banner.remove();
}

async function installApp() {
  if (!deferredPrompt) return;
  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;
  console.log('[PWA] Resultado de instalación:', outcome);
  deferredPrompt = null;
  hideInstallBanner();
}
```

### Explicación del código JS:

1. **Registro del SW**: Verifica que el navegador soporte Service Workers y lo registra al cargar la página. Si falla, solo hace `console.log` (no afecta la funcionalidad).

2. **`beforeinstallprompt`**: Este evento se dispara automáticamente en Chrome/Edge Android cuando se cumplen los criterios de instalación. Lo interceptamos para:
   - Guardar la referencia (`deferredPrompt`)
   - Mostrar nuestro propio banner personalizado en lugar del genérico del navegador

3. **Banner de instalación**: Un banner flotante en la parte inferior con gradiente morado (coherente con el diseño de StudyBuddy) que ofrece instalar la app. Tiene botón "Instalar" y botón "✕" para cerrar.

4. **`appinstalled`**: Se dispara cuando el usuario completa la instalación. Limpia el estado y muestra un toast de celebración.

5. **Nota sobre iOS**: Safari en iOS NO dispara `beforeinstallprompt`. Los usuarios de iPhone deben instalar manualmente usando "Compartir → Agregar a pantalla de inicio". Esto no lo podemos controlar, pero los meta tags de Apple que agregamos en el `<head>` aseguran que la experiencia de instalación sea correcta.

---

## Parte E: Manejo de navegación por hash (para shortcuts)

El `manifest.json` define shortcuts con URLs como `/index.html#generar`. Para que estos funcionen, debemos agregar un listener de hash.

Busca la IIFE `init()` en el `<script>`, y **al final de la IIFE** (justo antes del `})();` de cierre), agrega:

```javascript
  // ── PWA: Manejar navegación por hash (para shortcuts del manifest) ──
  const hash = window.location.hash.replace('#', '');
  if (hash && document.getElementById('section-' + hash)) {
    showTab(hash);
  }
```

Esto permite que cuando el usuario toca un shortcut desde la pantalla de inicio, la app abra directamente en la pestaña correcta.

---

## Verificación

### Requisitos técnicos para que Chrome muestre el prompt de instalación

Chrome requiere que se cumplan TODOS estos criterios:
1. ✅ La app se sirve por HTTPS (o localhost para desarrollo)
2. ✅ Tiene un `manifest.json` válido con `name`, `icons` (mín 192px), `start_url`, `display: "standalone"`
3. ✅ Tiene un Service Worker registrado con un handler de `fetch`
4. ✅ El usuario ha interactuado con la página

### Checklist funcional

Al terminar, verifica:

1. [ ] `manifest.json` existe en la raíz y es JSON válido
2. [ ] `sw.js` existe en la raíz y no tiene errores de sintaxis
3. [ ] La carpeta `icons/` contiene los 3 PNG con los tamaños correctos
4. [ ] `index.html` tiene `<link rel="manifest">` en el `<head>`
5. [ ] `index.html` tiene las meta tags de Apple en el `<head>`
6. [ ] `index.html` tiene `<meta name="theme-color">` en el `<head>`
7. [ ] `index.html` registra el SW al final del `<script>`
8. [ ] `index.html` tiene el código del banner de instalación
9. [ ] Abrir en Chrome DevTools → Application → Manifest muestra los datos correctos
10. [ ] Abrir en Chrome DevTools → Application → Service Workers muestra el SW activo
11. [ ] Los hash shortcuts funcionan: acceder a `/index.html#generar` abre la pestaña Generar
12. [ ] No hay errores en la consola del navegador
13. [ ] La app sigue funcionando normalmente (tabs, configuración, upload, etc.)

### Prueba de instalación en localhost

Para probar la instalación durante desarrollo:
1. Servir la app con `npx serve` (ya configurado)
2. Abrir `http://localhost:3000` en Chrome
3. Ir a DevTools → Application → Manifest → verificar que todo carga
4. Ir a DevTools → Application → Service Workers → verificar que está "activated and running"
5. Chrome debería mostrar el icono de instalación (➕) en la barra de direcciones

### Prueba en dispositivo móvil

Para probar en un celular real (en la misma red WiFi):
1. Encontrar la IP local del computador: `ipconfig` en Windows
2. Acceder desde el celular a `http://<IP_LOCAL>:3000`
3. **Android (Chrome)**: Debería aparecer el banner de instalación o el menú "Instalar app"
4. **iOS (Safari)**: Tocar el botón Compartir (📤) → "Agregar a pantalla de inicio"

---

## Nota sobre despliegue a producción

Para que la PWA funcione en producción (no localhost), necesitas un servidor HTTPS. Las opciones gratuitas más rápidas son:

1. **GitHub Pages**: Subir el proyecto a un repo de GitHub y activar GitHub Pages. URL: `https://tuusuario.github.io/studybuddy/`
2. **Netlify**: Conectar el repo de GitHub a Netlify. Deploy automático en cada push. URL personalizada gratis.
3. **Vercel**: Similar a Netlify. Excelente para proyectos estáticos.
4. **Firebase Hosting**: Gratis, rápido, con CDN global.

Cualquiera de estas opciones genera HTTPS automáticamente, que es requisito para que el Service Worker funcione fuera de localhost.

---

## Resumen de archivos

Al completar esta fase, la estructura del proyecto debe ser:

```
app/
├── index.html          ← Modificado (meta tags + registro SW + banner instalación)
├── manifest.json       ← NUEVO
├── sw.js               ← NUEVO
├── icons/
│   ├── icon-192.png    ← NUEVO (192×192)
│   ├── icon-512.png    ← NUEVO (512×512)
│   └── apple-touch-icon.png  ← NUEVO (180×180)
├── docs/
│   ├── instrucciones/
│   │   ├── FASE-1.md ... FASE-9-PWA.md
│   ├── studybuddy.html
│   └── PLAN_DE_IMPLEMENTACION.html
└── .gitignore
```
