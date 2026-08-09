# SHOCK Studio — contexto del proyecto

Landing de una sola página (`index.html`) para SHOCK, estudio que produce tours
cinematográficos/inmersivos para hospedajes de lujo en Costa Rica, a partir de
material (fotos/video) que envía el cliente + dirección de arte + IA de video.
Sitio en vivo en **shockstudio.cc**, hosteado en GitHub Pages
(`github.com/shockcrc/shockstudio`, rama `main`).

## Antes de tocar nada

- **Revisá `git status` y `git diff` al empezar.** El usuario a veces edita
  `index.html` / `demo-detalle.html` en paralelo con otra herramienta
  (IDE tipo Kiro/Cursor). Si aparece un diff grande que no generaste vos,
  preguntale antes de asumir que es ruido — puede ser trabajo suyo en curso.
- El motor de scroll (GSAP + ScrollTrigger + Lenis, ver más abajo) es
  **delicado**. No lo reinventes; si hay que tocarlo, entendé primero cómo
  funciona `distancia()`, `posicionDe()` y el patrón de "un solo seek por vez"
  (`pedirCuadro` en `demo-detalle.html`) antes de modificar nada ahí.
- Después de cualquier edit a `index.html`, verificá que el HTML y el CSS no
  quedaron rotos:
  ```bash
  python3 -c "
  import re
  content = open('index.html', encoding='utf-8').read()
  body = content[content.find('<body>'):content.find('<script', content.find('<body>'))]
  print('div opens:', len(re.findall(r'<div\b', body)), 'closes:', len(re.findall(r'</div>', body)))
  style = content[content.find('<style>'):content.find('</style>')]
  print('braces:', style.count('{'), style.count('}'))
  "
  ```

## Estructura del sitio

Un solo `<section class="riel-wrap">` con 7 "estaciones" (`data-est-panel
data-num="01"`..`"07"`) que se recorren con scroll horizontal (desktop) o
vertical libre con Lenis (mobile, sin scroll-snap nativo). Orden actual:

1. Portada (hero)
2. Nuestro valor (parte 1/2, con intro + demos)
3. Nuestro valor (parte 2/2)
4. Lo que incluye
5. Cómo se produce (línea de tiempo horizontal, no acordeón)
6. Preguntas (FAQ, acordeón)
7. Planes (precios)

Si agregás/quitás una estación: renumerar `data-num` en orden, y ajustar la
cantidff de `<span data-est>` en `#marcas` (deben coincidir 1 a 1).

## Voz / tono de la copy

- **Todo el sitio en "vos"** (informal, CR) — EXCEPTO **"Preguntas" (FAQ)** y
  **"Cómo se produce"**, que van en **tercera persona formal** ("el cliente
  selecciona su plan...") a pedido explícito del usuario.
- Nunca mencionar **"Casa Bóveda"** por nombre en texto visible — es la
  propiedad de referencia usada en los demos, pero el copy debe hablar del
  "resultado del servicio", no de esa propiedad puntual.
- Nada de frases genéricas de marketing ("comunica el nivel real...").
  El usuario pide texto que diga el mecanismo/por qué concreto, no la
  descripción bonita del producto.
- Evitar sonar "a IA": frases largas con "se confirma/se notifica" en cadena
  se rechazan. Preferir directo y corto.

## Datos de negocio confirmados (no inventar otros)

- Pago: **50% al iniciar, 50% contra entrega**. Varios métodos de pago.
- Comunicación: **WhatsApp**, de principio a fin.
- Entrega: **enlace de descarga (Google Drive, asumido — confirmar con el
  usuario si usa otro)**, con video completo + tomas sueltas + reels.
- Plazo: **72 horas**.
- Planes: Chico $125 / Mediano $250 / Grande $450.

## Tema claro/oscuro

- Oscuro es el default y el **único tema en escritorio** (siempre, sin
  importar localStorage).
- Claro es el default en **mobile únicamente** (`@media(max-width:820px)`),
  salvo que el visitante haya elegido oscuro a mano (`localStorage
  shock-tema === 'dark'`).
- El tema claro **no debe verse "de papel"** (blanco plano) — paleta cálida
  crema/hueso (`#F1EAD9` / `#E4D9C1`) con acentos dorados, no blanco puro.
- Colores en CSS: usar `var(--ink)`/`var(--paper)` o, para rgba con opacidad,
  `rgba(var(--ink-rgb),X)` / `rgba(var(--paper-rgb),X)` — **nunca** hardcodear
  `rgba(242,237,228,X)` ni `rgba(13,11,8,X)` de nuevo, rompe el tema claro.

## Demos (`demo-detalle.html`)

- Las miniaturas "Ver demo" muestran `img/demo-parteN.jpg`, que son
  **fotogramas reales** del video (el punto exacto donde arranca ese tramo,
  no fotos random) — así no hay corte visual al tocar "Ver demo". No
  reemplazar por una imagen genérica sin querer romper esa transición.
- Esas 4 fotos ya tienen un grado de color aplicado (contraste + split-tone
  dorado/oscuro + viñeta) para que no se vean como capturas informales.
- El rótulo dice "SHOCK", no "Casa Bóveda".

## Deploy

- No hay credenciales git guardadas en esta máquina/sesión. Para pushear hace
  falta un Personal Access Token nuevo cada vez (el usuario lo genera en
  `github.com/settings/tokens/new`, scope `repo`, se lo pasa al agente, se
  usa una vez y se le pide que lo revoque después).
- Push: `git -c http.postBuffer=524288000 push https://<TOKEN>@github.com/shockcrc/shockstudio.git main`
  (el buffer grande hace falta por el video de 37MB en el repo).
- Verificación local antes de pushear: servidor Python
  (`python3 -m http.server 8934 --bind 127.0.0.1 --directory .`) + Browser
  pane. El scroll programático (`window.scrollTo`) en desktop suele romper
  Lenis y mostrar pantalla negra en la herramienta de screenshots — no es un
  bug real del sitio, es un problema de la herramienta. Para verificar
  mobile, usar `element.scrollIntoView()` vía `javascript_tool`, que sí
  funciona bien con el scroll libre de Lenis en touch.
