# PRD — Editor de artículos con versiones, ramas y revisión

**Estado:** Prototipo funcional (research preview)
**Última actualización:** 24 jun 2026
**Autor:** Guille
**Artefacto asociado:** `editor-versiones.html` (prototipo de una sola página, autónomo)

> Este documento sirve como contexto para retomar el trabajo (por una persona o por Claude) y para alinear al equipo. Describe **qué hace** el producto, **cómo funciona** y, sobre todo, **por qué** está diseñado así.

---

## 1. Resumen ejecutivo

Un editor de artículos donde el contenido evoluciona de forma controlada: cada cambio queda registrado, se puede comparar visualmente con versiones anteriores, y los cambios importantes pasan por un flujo de revisión y aprobación antes de publicarse —igual que un repositorio de código con ramas y *pull requests*, pero pensado para texto editorial y para personas no técnicas.

El prototipo demuestra cuatro pilares:

1. **Historial de versiones** navegable (por cronología y por ramas).
2. **Visor de diferencias** (diff) legible para texto, no para código.
3. **Editor enriquecido** con barra de formato.
4. **Flujo editorial con roles**: propuesta en rama → revisión con comentarios → aprobación = publicación.

---

## 2. Problema y motivación (el "por qué")

Las herramientas habituales fuerzan a elegir entre dos extremos:

- **Google Docs / Notion**: excelentes para editar, pero el control de versiones es plano (una sola línea temporal) y la "aprobación" es informal. No existe el concepto de *proponer* un cambio aislado que un responsable revisa y publica.
- **Git / GitHub**: control de versiones y revisión potentísimos, pero pensados para código y para perfiles técnicos. Su diff es línea a línea (inútil para párrafos que se reajustan) y su curva de aprendizaje es alta.

**Hipótesis del producto:** los equipos editoriales necesitan el *modelo mental* de Git (ramas, revisión, merge, historial) con la *experiencia* de un editor de texto moderno, y un diff diseñado para la **legibilidad** del artículo, no para la precisión de la línea de código.

---

## 3. Objetivos y no-objetivos

### Objetivos
- Que cualquier cambio sea **trazable**: quién, cuándo y qué cambió.
- Que comparar dos versiones sea **inmediato y legible**.
- Que la publicación esté **gobernada por permisos** (no cualquiera publica).
- Que la revisión sea **conversacional**: comentar sobre el texto y asignar a alguien.

### No-objetivos (de momento)
- Edición colaborativa simultánea en tiempo real (varios cursores).
- Resolución automática de conflictos de *merge* a 3 vías.
- Backend real, autenticación y base de datos compartida.
- Soporte de imágenes/medios incrustados en el cuerpo del artículo (más allá de adjuntos en comentarios).

---

## 4. Usuarios y roles

El sistema define **dos roles** (decisión deliberada de simplicidad; ver §9):

| Rol | Puede editar | Puede proponer rama | Puede aprobar/publicar |
|-----|:---:|:---:|:---:|
| **Editor** | ✅ | ✅ | ❌ |
| **Admin** | ✅ | ✅ | ✅ |

- **Editor**: redacta y propone cambios, pero **no** puede publicar en "principal". Asigna su propuesta a un Admin.
- **Admin**: además de editar y proponer, **aprueba y publica**. Puede aprobar su propia propuesta directamente ("Publicar ahora").

**Por qué solo dos roles:** se evaluó un tercer rol intermedio ("Manager" que aprueba pero no publica, separando *aprobar* de *publicar* en dos pasos). Se descartó para el prototipo porque añade fricción sin valor demostrable a esta escala. **Aprobar = publicar** en un único paso.

> En el prototipo la identidad se simula con un selector de usuario (Guille/Admin, Marta/Editor, Lucía/Admin, Diego/Editor). En producción iría contra el sistema de autenticación real.

---

## 5. Conceptos del modelo

- **Principal (main):** la línea publicada del artículo. Es la "verdad" visible.
- **Versión:** un estado publicado de "principal" (v1, v2, …), con autor, fecha y resumen.
- **Rama (branch):** una propuesta de cambio que parte de una versión concreta de principal y vive aparte hasta que se aprueba.
- **Estados de una rama:** `abierta` → `fusionada` (publicada) o `rechazada`.
- **Asignación:** cada rama abierta se asigna a un Admin que debe revisarla.
- **Comentario de revisión:** una nota anclada a un fragmento de texto del diff, opcionalmente asignada a alguien y con adjuntos.

---

## 6. Requisitos funcionales

### 6.1 Historial de versiones (sidebar)
El panel lateral muestra el historial con **dos vistas conmutables**:

- **Cronología:** todos los eventos (versiones de principal y aperturas de rama) en orden temporal descendente.
- **Ramas:** "Principal" como columna vertical con sus versiones, y cada rama **colgando de la versión de la que partió**; debajo, sección de "Ramas abiertas".

**Por qué dos vistas:** la cronología responde a "¿qué ha pasado y cuándo?"; la vista de ramas responde a "¿qué propuestas hay vivas y de dónde salen?". Son dos preguntas distintas que merecen dos lecturas.

### 6.2 Visor de diferencias (diff)
- **Dos modos conmutables:**
  - *Unificada* (una columna): texto eliminado tachado en rojo con `−`, añadido resaltado en verde con `+`, párrafos reescritos marcados con `✎` y resaltado palabra a palabra.
  - *Dividida* (dos columnas): "antes" a la izquierda, "después" a la derecha (estilo Wikipedia/GitHub).
- **Granularidad por bloque** (párrafo, título, cita, lista) **con resaltado interno de palabra** en los bloques modificados.
- **Cambio de tipo de bloque** señalizado (p. ej. un párrafo que pasa a cita muestra una etiqueta `⇄ Cita`).
- **Resumen de cambios:** contadores de añadidos / eliminados / modificados.

**Por qué por bloque y no por línea:** un artículo no tiene "líneas" estables; los párrafos se reajustan. Comparar por bloques semánticos (y resaltar la palabra exacta dentro) es lo que hace el diff legible.

**Accesibilidad (deliberada):** los cambios **no dependen solo del color**. Lo eliminado va tachado y con `−`; lo añadido subrayado y con `+`. Fondos pastel suaves con texto oscuro. Esto cubre a personas con daltonismo (deuteranopía).

### 6.3 Editor enriquecido
- Editor WYSIWYG con **barra de formato**: Párrafo / Título (H2) / Subtítulo (H3), **Negrita**, *Cursiva*, Enlace (⌘K), lista con viñetas, lista numerada, Cita, y quitar formato.
- Los botones reflejan el **estado activo** según la posición del cursor.
- Al guardar, el HTML se **sanea** a un subconjunto seguro de etiquetas (se descartan `script`, `style`, estilos en línea y envoltorios desconocidos).

**Por qué importa el saneado:** el contenido pegado o generado por el navegador trae basura (spans con estilos, etiquetas raras). Normalizarlo mantiene el diff limpio y el almacenamiento seguro.

### 6.4 Flujo de ramas, roles y publicación
- **Editar** parte siempre de la última versión de principal y produce una **propuesta en rama** (no toca principal directamente).
- Al proponer: se asigna un **nombre** a la rama y un **Admin revisor**.
- Un **Admin** revisando una rama abierta puede **Aprobar y publicar** (crea una nueva versión de principal y marca la rama como fusionada) o **Rechazar**.
- Un **Admin** editando puede usar **Publicar ahora** (crea la propuesta y la publica en un solo gesto).
- Un **Editor** ve la propuesta pero con la acción de aprobar **bloqueada** (🔒), indicando a qué Admin está asignada.
- **Aviso de divergencia:** si "principal" avanzó mientras la rama estaba abierta, se muestra una advertencia ("principal avanzó a vX: revisa al publicar").

**Por qué la edición siempre crea una rama:** garantiza que "principal" solo cambie por una acción explícita y permisada. La trazabilidad y el control de publicación son el punto del producto.

### 6.5 Comentarios de revisión
- En el modo revisión (viendo una rama), el usuario **selecciona texto** del diff → aparece un **bocadillo "💬 Comentar"**.
- Al pulsarlo se abre un **popup** para: escribir el comentario, **asignarlo a una persona**, y **adjuntar un enlace o una imagen**.
- Al guardar, el comentario aparece en el panel **"Notas del revisor"** y el fragmento citado queda **subrayado** en el texto. Pasar el ratón por una nota resalta su fragmento.
- Cada nota se puede **resolver / reabrir** o **eliminar**; las resueltas se agrupan aparte.

**Por qué anclado al texto:** comentar "en abstracto" pierde contexto. Anclar la nota a la frase exacta (estilo Google Docs/sugerencias) hace la revisión accionable.

### 6.6 Persistencia
- Todo el estado (versiones, ramas, comentarios, usuario y vista activos) se guarda en **localStorage**.
- Botón **"Reiniciar demo"** para restaurar los datos de ejemplo.

---

## 7. Matriz de permisos (resumen)

| Acción | Editor | Admin |
|---|:---:|:---:|
| Ver historial y diffs | ✅ | ✅ |
| Editar y proponer rama | ✅ | ✅ |
| Asignar revisor | ✅ | ✅ |
| Comentar / asignar comentario | ✅ | ✅ |
| Aprobar y publicar | ❌ | ✅ |
| Rechazar propuesta | ❌ | ✅ |
| Publicar directamente ("Publicar ahora") | ❌ | ✅ |

---

## 8. Notas técnicas (estado actual del prototipo)

- **Un único archivo HTML autónomo**, sin dependencias externas ni CDN: funciona offline con doble clic.
- **Motor de diff propio** basado en LCS (Longest Common Subsequence), en tres niveles: bloques → palabras → agrupado para resaltado continuo. Empareja bloques eliminados/añadidos por **similitud** (Jaccard de palabras) para detectar "modificados" en lugar de mostrar borrado+añadido.
- **Editor** sobre `contenteditable` + `document.execCommand`.
- **Almacenamiento** en `localStorage` (serialización JSON del estado completo).

---

## 9. Limitaciones conocidas y deuda (prototipo → producción)

1. **Sin backend ni datos compartidos.** `localStorage` es local a cada navegador: distintas personas que abran la misma URL **no** ven las ramas/comentarios de las demás. Para colaboración real hace falta base de datos y API.
2. **Identidad simulada.** El selector de usuario sustituye a la autenticación/SSO real.
3. **`execCommand` está obsoleto.** Funciona en navegadores actuales, pero para producción conviene un editor moderno (TipTap/ProseMirror/Lexical).
4. **Sin merge a 3 vías.** Se avisa de la divergencia, pero no se resuelven conflictos cuando dos ramas tocan lo mismo; el merge sustituye por el texto de la rama.
5. **Anclaje de comentarios por texto.** Se resalta la primera coincidencia del fragmento citado; no es un anclaje de rango robusto ante reescrituras.
6. **Dos roles fijos.** No hay paso separado de "aprobar" vs "publicar" ni roles configurables.

---

## 10. Roadmap propuesto

- **Visual:** *app shell* tipo Abstract (estructura, secciones, estado) + grafo de raíl de color tipo Git en la pestaña de ramas (topología principal/rama/merge).
- **Colaboración real:** backend + base de datos + auth (datos compartidos entre usuarios).
- **Editor de producción:** migrar a TipTap/ProseMirror; soportar medios en el cuerpo.
- **Revisión avanzada:** hilos de respuesta en comentarios, notificaciones al asignado, sugerencias de cambio aceptables/rechazables (estilo Google Docs).
- **Permisos:** roles configurables y opción de separar aprobar/publicar.
- **Componente reutilizable:** empaquetar como componente React/Vue para integrarlo en la plataforma.

---

## 11. Glosario

- **Principal (main):** línea publicada del artículo.
- **Rama (branch):** propuesta de cambio aislada que parte de una versión.
- **Merge / fusionar:** integrar una rama aprobada en principal, creando una nueva versión.
- **Diff:** comparación visual entre dos textos.
- **Unificada / Dividida:** los dos modos de visualizar el diff (una columna / dos columnas).
- **Nota del revisor:** comentario anclado a un fragmento, con asignación y adjuntos.
