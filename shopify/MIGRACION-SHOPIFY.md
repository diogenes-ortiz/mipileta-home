# Migración de la home de Mi Pileta a Shopify

Pasar la página que hoy vive en `https://diogenes-ortiz.github.io/mipileta-home/` a una
página del ecommerce (`mipileta.com.ar/pages/…`), sin tocar nada de lo que ya está armado.

---

## 1. Qué hay en esta carpeta

```
mipileta-shopify/
├── sections/mipileta-home.liquid        ← toda la página (CSS + HTML + JS + schema)
├── templates/page.mipileta-home.liquid  ← el molde de la página (layout none)
├── assets/  (9 imágenes)                ← las fotos que usa
├── _preview.html                        ← cómo va a quedar (abrir en el navegador)
└── MIGRACION-SHOPIFY.md                 ← esto
```

**Nada de esto se edita a mano.** Los dos `.liquid` los genera `../build-shopify.js` a
partir de `../mipileta-home.html`, que sigue siendo la única fuente de verdad. Al final
del documento está el circuito para cuando hagamos cambios de diseño.

---

## 2. Decisiones que ya están tomadas

| Tema | Decisión | Por qué |
|---|---|---|
| Marco | **Pantalla completa** (`{% layout none %}`) | La página se ve exactamente como la de GitHub. Al no cargar el CSS del tema, no hay ni una chance de que la tienda pise los estilos. La contra: en esa página no está el header ni el carrito de la tienda; se sale por los links (Tienda online, colecciones, etc.). |
| Edición | **Sección con 105 campos** en el editor de temas | Fotos, títulos, bajadas, chips y links se cambian desde *Personalizar*, sin tocar código. Las grillas, los iconos y las animaciones quedan fijas. |
| Contraseña | **No viaja** | El gate de GitHub era JS. En Shopify, mientras esté en revisión: dejá la página fuera del menú y descomentá el `noindex` que está marcado en el template. |
| Fotos | Suben a `assets/` del tema **y** se pueden reemplazar desde el editor | Si las cargás desde el editor, las sirve el CDN de Shopify ya redimensionadas y en WebP: la página pesa bastante menos. |

---

## 3. Ruta A — Editor de código (sin instalar nada)

> Antes de empezar: **Tienda online → Temas → ⋯ → Duplicar**. Trabajá sobre el duplicado
> y publicalo recién cuando esté aprobado.

**1. La sección**
- Temas → ⋯ → **Editar código**
- Panel izquierdo → *Secciones* → **Agregar una sección nueva**
- Nombre: `mipileta-home` · Tipo: **liquid**
- Borrá todo el contenido de ejemplo y pegá `sections/mipileta-home.liquid` completo → Guardar

**2. La plantilla**
- *Plantillas* → **Agregar una plantilla nueva**
- Plantilla para: **página** · Tipo: **liquid** ← ojo, no JSON · Nombre: `mipileta-home`
- Borrá el contenido de ejemplo y pegá `templates/page.mipileta-home.liquid` → Guardar
- Queda como `templates/page.mipileta-home.liquid`

**3. Las imágenes**
- *Recursos* (Assets) → **Agregar un recurso nuevo** → subí las 9 de `assets/`
- **No les cambies el nombre**, el código las busca así:
  `home-hero-cocina.jpg` · `zingara-hero-cobre.png` · `logo-blanco.png` ·
  `familia-essentia-3.jpg` · `zingara-hero.png` · `ess-lifestyle.jpg` ·
  `home-marca.jpg` · `home-post-cocina.jpg` · `home-post-tabla.jpg`

**4. La página**
- Admin → **Páginas** → *Agregar página*
- Título: el que quieras (define la URL: "Mi Pileta" → `/pages/mi-pileta`)
- Abajo a la derecha, **Plantilla de tema**: `page.mipileta-home`
- Guardar y abrir la URL

**5. Editar textos y fotos**
- Temas → **Personalizar** → arriba al centro, el selector de páginas → *Páginas* → tu página
- Aparece la sección **Home Mi Pileta** con todos los campos agrupados por bloque

---

## 4. Ruta B — Shopify CLI (para iterar cómodo)

```bash
npm install -g @shopify/cli
shopify theme pull --store TU-TIENDA.myshopify.com
```

Copiás los archivos dentro del tema descargado (misma estructura de carpetas que esta):

```bash
cp mipileta-shopify/sections/mipileta-home.liquid   <tema>/sections/
cp mipileta-shopify/templates/page.mipileta-home.liquid <tema>/templates/
cp mipileta-shopify/assets/*                        <tema>/assets/
```

Y lo probás sin tocar el tema en vivo:

```bash
shopify theme dev --store TU-TIENDA.myshopify.com     # preview local con hot reload
shopify theme push --unpublished --theme "Home nueva" # sube una copia para revisar
```

La página igual hay que crearla una vez a mano en Admin → Páginas (paso 4 de la Ruta A):
la plantilla la pone el tema, pero la página es un registro del admin.

---

## 5. Checklist después de subir

- [ ] La página abre y **no** muestra el header ni el footer de la tienda (es lo esperado).
- [ ] Cargan las 9 fotos. Si alguna sale rota → falta ese archivo en *Recursos*.
- [ ] El menú de la página (Líneas, La marca, …) baja a cada sección.
- [ ] "Tienda online" va a `/collections/todos-los-productos`.
- [ ] Los botones de WhatsApp abren el chat.
- [ ] En celular: hero partido en dos, hamburguesa, sin scroll horizontal.
- [ ] En *Personalizar* aparece la sección "Home Mi Pileta" y los cambios se guardan.
- [ ] Mientras esté en revisión: página fuera del menú + `noindex` descomentado en el template.

---

## 6. Cómo seguimos con los cambios de diseño

El HTML manda. El circuito completo es:

```bash
# 1. editar el diseño
#    Downloads/Cloud/mipileta-home.html

# 2. regenerar los archivos de Shopify (valida solo: si algo no matchea, corta)
node build-shopify.js
node check-shopify.js

# 3. publicar la versión de GitHub
cp mipileta-home.html mipileta-web/index.html
cd mipileta-web && git add -A && git commit -m "..." && git push
```

Y en Shopify se vuelve a pegar `sections/mipileta-home.liquid` (o `shopify theme push`).
El script **corta con error** si un texto que estaba mapeado a un campo del editor
desapareció del HTML, así nunca se sube un Liquid a medias.

⚠️ Los textos que se editen desde el editor de temas **no vuelven** al HTML. Si el cliente
cambia una bajada en Shopify y después regeneramos, el valor del editor sigue ganando
(el HTML solo define el valor por defecto). Para cambios de contenido finos, editar en Shopify;
para cambios de diseño, editar el HTML.

---

## 7. Brief para pegar en Cowork

> Estoy migrando la home de marca de Mi Pileta a Shopify.
> Los archivos ya están generados en `Downloads/Cloud/mipileta-shopify/`:
> `sections/mipileta-home.liquid`, `templates/page.mipileta-home.liquid` y `assets/` (9 imágenes).
> El instructivo completo está en `mipileta-shopify/MIGRACION-SHOPIFY.md`.
> La fuente de verdad es `Downloads/Cloud/mipileta-home.html`; los `.liquid` se regeneran
> con `node build-shopify.js` y se validan con `node check-shopify.js`.
> Tengo plan con editor de código en Shopify. Necesito que me acompañes a subirlo por la
> Ruta A del instructivo y después revisemos el checklist del punto 5.

Ojo con esto en Cowork: **la sesión no puede loguearse a Shopify ni cargar credenciales**.
El login y los clicks de "Guardar" en el admin los tenés que hacer vos; Cowork prepara los
archivos, te dice exactamente qué pegar dónde y revisa el resultado.

---

## 8. Pendientes conocidos

- **Peso**: las fotos suman ~9 MB. Cargándolas desde el editor de temas (image_picker) las
  sirve el CDN de Shopify redimensionadas; si quedan como assets del tema, conviene
  achicarlas antes (`familia-essentia-3.jpg` sola son 3 MB).
- **Links a las líneas**: hoy apuntan a las GitHub Pages de Essentia y Zíngara. Cuando esas
  también migren, se cambian desde el editor (campos *Línea 1 / Línea 2 → Link*).
- **Datos de negocio**: la franja "La marca" usa datos del material, no cifras de la empresa.
  Si aparecen los números reales (años, distribuidores, modelos), van ahí.
