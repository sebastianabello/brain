# Wiki Personal LLM

Manual de trabajo del gestor de conocimiento con IA.

## Responsabilidad de los directorios

- **raw/** — Material original, **inmutable**. La IA solo lee, nunca modifica. Organizado por tema en subdirectorios, se permite anidación múltiple. Los PDF/videos/xlsx/imágenes conservan su nombre de archivo original.
- **wiki/** — Notas procesadas y mantenidas por la IA. Se permiten subdirectorios múltiples. Cada artículo en formato md, nombre libre (descriptivo está bien).
- **assets/** — Recursos gráficos y multimedia.

## Dos archivos especiales (deben mantenerse siempre)

- **wiki/index.md** — Índice global. Una línea por artículo wiki, agrupado por tema, con enlace + resumen de una oración + fecha de actualización.
- **wiki/log.md** — Registro de operaciones append-only. Registra cada ingest / query archivado / lint.

## Tres comportamientos de activación

### Activación 1: Ingest (registrar material)

**Palabras clave:** "agregar a la wiki", "ingest esto", "guarda esto", "añade esto a la wiki", o cuando se proporciona material nuevo con intención de guardarlo.

**Acciones:**
1. Guardar el material en `raw/<tema más apropiado>/`. Si el tema no existe, crearlo. El nombre del archivo se conserva tal cual (PDF/video/imagen) o se le da un nombre descriptivo (notas en md).
2. Compilar en `wiki/<tema correspondiente>/<artículo>.md`:
   - Si es el **mismo argumento central** que un artículo existente → fusionar, actualizar Sources y las secciones afectadas
   - **Concepto completamente nuevo** → crear artículo nuevo, nombrado por concepto, no por el nombre del archivo raw
   - **Abarca múltiples temas** → colocar en el tema más relevante, agregar "Ver también" con referencias cruzadas al final del artículo
3. Verificar conflictos de hechos: si el nuevo material contradice contenido existente, anotar la discrepancia y la fuente en el artículo.
4. **Actualización en cascada**: revisar otros artículos del mismo tema que se vean afectados, actualizar su fecha Updated.
5. **Actualizar wiki/index.md**: agregar o modificar la entrada de cada artículo que haya cambiado.
6. **Agregar al final de wiki/log.md**:
   ```
   ## [AAAA-MM-DD] ingest | <título del artículo principal>
   - Actualizado: <títulos de artículos actualizados en cascada>
   ```

### Activación 2: Query (consulta)

**Palabras clave:** "qué sé sobre X", "hay algo en la wiki sobre Y", "resume Z según mi wiki", "compara A y B".

**Acciones:**
1. Primero leer `wiki/index.md` para localizar los artículos relevantes.
2. Leer los artículos y sintetizar la respuesta. Priorizar el contenido de la wiki; usar el conocimiento propio de la IA solo como complemento.
3. Al citar, usar enlaces markdown: `[Título del artículo](wiki/tema/articulo.md)`.
4. **Por defecto: solo responder en el chat, no escribir archivos.**

**Excepción — si el usuario dice "guárdalo" / "archívalo en la wiki":**
1. Escribir la respuesta como nuevo artículo en `wiki/<tema más relevante>/`.
2. No fusionar con artículos existentes (el contenido archivado es una respuesta sintetizada, no material fuente).
3. Actualizar index.md, agregar el prefijo `[Archivado]` al resumen.
4. Agregar al final de log.md:
   ```
   ## [AAAA-MM-DD] query | Archivado: <título de la página>
   ```

### Activación 3: Lint (revisión de salud)

**Palabras clave:** "lint wiki", "revisión", "chequeo", "qué problemas tiene la wiki".

**Verificaciones deterministas (corregir automáticamente cuando sea posible):**
- Consistencia entre `wiki/index.md` y archivos reales: archivo existe pero falta en el índice → agregar entrada; índice apunta a archivo inexistente → marcar `[FALTANTE]`
- Enlaces markdown internos de la wiki rotos: si el archivo está en otro lugar → corregir ruta; si no se encuentra → reportar
- Enlaces "Ver también": si faltan referencias cruzadas obvias dentro del mismo tema → agregar; si enlazan a archivos eliminados → eliminar

**Verificaciones heurísticas (solo reportar, no corregir automáticamente):**
- Contradicciones de hechos entre artículos
- Material nuevo que deja obsoleto un argumento anterior
- Discrepancias entre fuentes sin anotar
- Páginas huérfanas (sin referencias de ningún otro artículo)
- Conceptos que deberían estar vinculados entre temas pero no lo están
- Conceptos mencionados repetidamente pero sin página propia

**Al finalizar, agregar al final de log.md:**
```
## [AAAA-MM-DD] lint | <N> problemas encontrados, <M> corregidos automáticamente
```

## Formato de entrega (reglas fijas, nunca improvisar)

### Idioma

- **Español por defecto**, independientemente del idioma del material fuente.

### Plantilla de artículo wiki (`wiki/**/*.md`)

Todo artículo generado o actualizado en `wiki/` debe seguir exactamente esta estructura, sin añadir ni quitar bloques:

```markdown
---
title: Título del concepto
tags: [tema, subtema]
updated: AAAA-MM-DD
---

# Título del concepto

<cuerpo libre: Claude decide las secciones según el contenido>
Las citas inline van así: según [^1] o referenciando el texto con superíndice.

## Ver también

- [[wiki/tema/articulo-relacionado.md|Título legible]]

## Fuentes

[^1]: Descripción de la fuente — `raw/<ruta/archivo>`
```

**Reglas de la plantilla:**

- **Frontmatter YAML**: siempre presente, siempre con los tres campos (`title`, `tags`, `updated`). Los `tags` usan el nombre del directorio padre como primer tag; subtemas adicionales son opcionales. Formato de lista YAML `[tag1, tag2]`, en minúsculas, sin espacios (usar guión: `mi-tema`).
- **H1**: igual al campo `title` del frontmatter. Solo un H1 por artículo.
- **Cuerpo**: estructura libre — Claude elige las secciones (`## Concepto`, `## Cómo funciona`, `## Ejemplos`, etc.) según lo que el contenido requiera. No inventar secciones vacías.
- **Ver también**: obligatorio si existen artículos relacionados en la wiki. Usar wikilinks de Obsidian `[[ruta|Título]]` con ruta relativa al vault. Omitir la sección completa si no hay referencias relevantes.
- **Fuentes**: siempre presente. Una entrada por fuente, con formato `[^N]: descripción — \`raw/ruta\``. Si el material no vino de `raw/` (ej. conocimiento propio usado como complemento), no se lista como fuente.
- **Artículo archivado** (Query → "guárdalo"): misma plantilla, pero `tags` incluye `archivado` y el H1 lleva el prefijo `[Archivado]`.

### Respuesta de chat tras un Ingest

Formato fijo, sin variaciones:

```
✓ raw/<ruta/archivo>
✓ wiki/<ruta/articulo.md> [creado | actualizado]
✓ wiki/index.md
✓ wiki/log.md
```

- Una línea por archivo tocado, en ese orden.
- Si hubo cascada, listar también esos archivos con `[cascada]`.
- Nada más. Sin explicaciones, sin resúmenes del contenido, sin encabezados adicionales.

### Respuesta de chat tras un Query

- Prosa directa, sin secciones ni encabezados.
- Las citas van inline usando enlaces markdown: `[Título](wiki/tema/articulo.md)`.
- Si el conocimiento propio de la IA complementa la wiki, integrarlo en la prosa sin distinguirlo visualmente; la wiki siempre es la fuente primaria.
- Si no hay información relevante en la wiki, decirlo en una sola oración antes de responder con conocimiento propio.

### Respuesta de chat tras un Lint

```
Lint AAAA-MM-DD — N problemas, M corregidos

Corregidos automáticamente:
- <descripción breve>

Requieren revisión manual:
- <descripción breve>
```

- Si no hay problemas en alguna categoría, omitir esa sección.
- Sin prosa adicional fuera de esta estructura.

## Convenciones

- Todos los enlaces internos de la wiki usan **rutas relativas al archivo actual**.
- Al citar artículos wiki en el chat, usar **rutas relativas a la raíz del vault** (ej. `wiki/tema/xxx.md`).
- La fecha es la fecha de hoy. Updated refleja cuándo cambió el contenido del artículo, no la marca de tiempo del sistema de archivos.
- **raw nunca se modifica.** Si se detecta un error en raw, anotarlo en el artículo wiki como "la fuente contiene un error", sin tocar raw.

## Restricciones flexibles

- La profundidad de subdirectorios en wiki es **ilimitada**. `wiki/linux/administracion/permisos.md` es válido.
- Los nombres de archivos md son **libres**, no se exige el formato `AAAA-MM-DD-slug.md`.
- Los metadatos del frontmatter (`title`, `tags`, `updated`) son **obligatorios** en todos los artículos wiki. Ver plantilla en "Formato de entrega".
- Los archivos en raw (PDF/video/xlsx/imagen) **nunca se renombran**.
