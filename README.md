# STL Catalog 3D — versión ligera (un solo archivo)

`STL-Catalog.html` es la aplicación web completa metida en **un único archivo**:
JS, CSS y favicon van inline. Se abre con **doble clic** desde un pendrive, sin
servidor, sin instalar nada y sin conexión.

```
WEB LIGHT\
└─ STL-Catalog.html   ~992 kB — la app entera
```

> ⚠️ **Esta carpeta es un artefacto, no una fuente.** El archivo se genera desde
> `WEB` y se sobrescribe en cada build. No lo edites a mano: los cambios se
> perderían. El código se toca en `APP\src` y se compila aquí.

---

## Cómo regenerarlo

```powershell
cd WEB
npm run sync          # 1. trae el código desde APP\src
npm run build:single  # 2. genera ..\WEB LIGHT\STL-Catalog.html
```

`build:single` hace dos pasos:

1. `vite build --mode single` compila en modo "todo en uno": sin `public/`,
   assets inline, CSS en un solo archivo y el bundle como **IIFE** (script
   clásico).
2. `scripts/build-single-file.mjs` mete el CSS, el JS y el favicon dentro del
   HTML, comprueba que no quede ninguna referencia externa y deja el archivo en
   esta carpeta. La carpeta temporal `.single-build/` se borra al terminar.

El build normal (`npm run build` → `dist/`, para Apache/Vercel) sigue igual y no
se ve afectado.

---

## Qué funciona

| Función | Estado |
| --- | --- |
| Catálogo, etiquetas, discos, filtros, búsqueda, orden | ✅ |
| Miniaturas 3D, dimensiones, triángulos y volumen | ✅ |
| Visor 3D interactivo | ✅ |
| Escanear una carpeta o un pendrive buscando `.stl` | ✅ Chrome/Edge (pide permiso una vez) |
| Clientes, pedidos y estados | ✅ |
| Persistencia al recargar (IndexedDB) | ✅ |
| Guardar / abrir `catalogo.json` + `miniaturas/` en una carpeta | ✅ Chrome/Edge |
| Respaldo en un único JSON (miniaturas dentro) | ✅ |
| Elegir la raíz de un disco (`E:\`) con diálogo nativo | ❌ solo la app de escritorio |
| Reabrir el `.stl` original tras recargar | ❌ el navegador no conserva la ruta (sí la miniatura) |

Los datos viven en el **IndexedDB del navegador**, no en el pendrive: el HTML no
escribe nada por su cuenta. Para llevarte el catálogo de un equipo a otro usá
**Copia de seguridad → Catálogo en pendrive → Guardar carpeta…** y luego
**Abrir carpeta…** en el otro equipo.

---

## Limitaciones de abrirlo como `file://`

- **Chrome y Edge**: funciona sin flags. Es el caso normal del pendrive.
- **Firefox y Safari**: `file://` no permite elegir carpetas; la app arranca y
  se ve el catálogo, pero escanear y guardar en carpeta no están disponibles.
- Todas las páginas `file://` del mismo navegador **comparten** IndexedDB, así
  que dos copias del HTML (o el mismo HTML en dos carpetas) ven el mismo
  catálogo local. Para catálogos separados usá perfiles distintos del navegador.
- Si el pendrive es una **ruta de red** (`\\servidor\carpeta`), Windows no da
  permiso de escritura: copiá el HTML a una unidad local.
- Algunas empresas bloquean `file://` por política. En ese caso serví `WEB\dist`
  con Apache/Vercel y usá la versión web normal.

---

## Cómo se comprobó

Verificado en Edge sobre `file://` (sin flags), con perfil limpio:

- Arranca con los 8 modelos de demostración, **0 errores de consola** y **0
  peticiones fallidas** (nada que cargar de fuera).
- IndexedDB guarda los 8 modelos y sus 8 miniaturas, y persiste al recargar.
- Escaneo de carpeta + generación de miniaturas con STL reales: WebP 320×240,
  100 % opacas, ~1,5 kB cada una.
- «Guardar carpeta…» escribe `catalogo.json` v4 (~7,5 kB para 8 modelos, sin
  base64) y `miniaturas/*.webp` (~1,7–2,4 kB).
- «Abrir carpeta…» aplica los metadatos al instante y rehidrata las miniaturas
  desde `miniaturas/`, con los bytes idénticos a los del archivo.
- El HTML funciona igual copiado a cualquier carpeta (no depende de su ruta).

Lo único que no se puede automatizar es el diálogo del sistema: hay que pulsar
«Guardar carpeta…» una vez a mano para elegir la carpeta del pendrive.

---

## Si algo no aparece

- **Se ve una versión vieja**: el navegador está sirviendo el HTML cacheado.
  Recargá con `Ctrl+F5` (o abrí el archivo desde el explorador de nuevo).
- **«Este navegador no permite elegir carpetas»**: estás en Firefox/Safari o
  abriste el archivo por una ruta de red.
- **El catálogo no persiste**: modo incógnito, o el navegador está borrando los
  datos del sitio al cerrar.
