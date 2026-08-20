# Archivos para Shopify

Estos `.liquid` NO son parte del sitio de GitHub Pages: son la version de la home
para el tema de Shopify, generada por `build-shopify.js`.

Viven acá solo para tenerlos a mano desde cualquier maquina.

**Por eso el repo tiene un `.nojekyll` en la raiz:** sin el, GitHub Pages corre
Jekyll, Jekyll intenta parsear estos archivos, no reconoce `{% schema %}` (que es
de Shopify) y falla el build de TODO el sitio.
