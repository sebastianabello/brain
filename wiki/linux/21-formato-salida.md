---
title: "Capítulo 21: Formato de Salida (Formatting Output)"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 21"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-325-341.pdf"
updated: "2026-06-23"
---

# Formato de Salida (Formatting Output)

Continuación del bloque de herramientas de texto, este capítulo se enfoca en **formatear** la salida (no en modificar el texto en sí). Estas herramientas se usan principalmente para preparar texto para impresión o para scripts.

| Comando  | Descripción |
|----------|-------------|
| `nl`     | Numerar líneas |
| `fold`   | Ajustar cada línea a una longitud especificada |
| `fmt`    | Formateador de texto simple |
| `pr`     | Preparar texto para impresión (paginar) |
| `printf` | Formatear e imprimir datos |
| `groff`  | Sistema de formateo de documentos |

---

## Herramientas de Formato Simple

### `nl` — Numerar Líneas

`nl` numera líneas, similar a `cat -n`, pero con muchas más opciones. Acepta múltiples archivos o stdin.

**Característica clave: páginas lógicas**

`nl` soporta el concepto de **logical pages** (páginas lógicas), lo que permite reiniciar la numeración y asignar estilos distintos para tres secciones: header, body y footer. Las secciones se indican con markup especial que `nl` borra del flujo tras procesarlo:

| Markup  | Significado |
|---------|-------------|
| `\:\:\:` | Inicio del header de la página lógica |
| `\:\:`   | Inicio del body de la página lógica |
| `\:`     | Inicio del footer de la página lógica |

> Cada elemento de markup debe estar **solo en su línea**.

**Opciones comunes:**

| Opción | Descripción |
|--------|-------------|
| `-b style` | Estilo de numeración del body: `a`=todas las líneas, `t`=solo no-en-blanco (defecto), `n`=ninguna, `pregexp`=las que coincidan con la regex |
| `-f style` | Estilo para el footer (defecto: `n`, ninguna) |
| `-h style` | Estilo para el header (defecto: `n`, ninguna) |
| `-i number` | Incremento de numeración (defecto: 1) |
| `-n format` | Formato del número: `ln`=izquierda sin ceros, `rn`=derecha sin ceros (defecto), `rz`=derecha con ceros |
| `-p` | No reiniciar la numeración al inicio de cada página lógica |
| `-s string` | Separador después del número (defecto: tab) |
| `-v number` | Número de la primera línea de cada página lógica (defecto: 1) |
| `-w width` | Ancho del campo de número de línea (defecto: 6) |

**Ejemplo — pipeline completo con sort, sed y nl:**

```bash
sort -k 1,1 -k 2n distros.txt | sed -f distros-nl.sed | nl
```

El script `distros-nl.sed` añade el markup de páginas lógicas de `nl` (el header, body y footer) y un encabezado de columnas.

**Variantes de formato:**

```bash
nl -n rz          # números con ceros a la izquierda
nl -w 3 -s ' '    # campo de ancho 3, separador espacio
```

---

### `fold` — Ajustar Ancho de Línea

`fold` **pliega** (rompe) líneas al alcanzar un ancho especificado. Acepta archivos o stdin.

**Comportamiento por defecto:** ancho de 80 caracteres, rompe sin respetar límites de palabras.

```bash
echo "The quick brown fox jumps over the lazy dog." | fold -w 12
```
```
The quick br
own fox jump
s over the l
azy dog.
```

**Con `-s` (split at space):** rompe en el último espacio disponible antes del límite, respetando palabras:

```bash
echo "The quick brown fox jumps over the lazy dog." | fold -w 12 -s
```
```
The quick
brown fox
jumps over
the lazy
dog.
```

| Opción | Descripción |
|--------|-------------|
| `-w width` | Ancho máximo de línea (defecto: 80) |
| `-s` | Rompe en el último espacio antes del límite |

---

### `fmt` — Formateador de Texto Simple

`fmt` hace más que `fold`: además de ajustar el ancho, realiza **formateo de párrafos**, rellenando y uniendo líneas mientras preserva líneas en blanco e indentación.

**Comportamiento por defecto:**
- Preserva: líneas en blanco, espacios entre palabras, indentación
- Líneas consecutivas con **diferente** indentación **no** se unen
- Prefiere romper al final de una oración
- Ancho por defecto: **75 caracteres** (formatea ligeramente más corto para balance de líneas)

**Ejemplo con texto indentado:**

```bash
# Sin opción: preserva la indentación del primer párrafo → resultado raro
fmt -w 50 fmt-info.txt | head

# Con -c (crown margin): alinea las siguientes líneas con la segunda
fmt -cw 50 fmt-info.txt
```

**Opciones:**

| Opción | Descripción |
|--------|-------------|
| `-c` | Modo *crown margin*: preserva la indentación de las primeras dos líneas; las subsiguientes se alinean con la segunda |
| `-p string` | Formatea solo las líneas que comienzan con el prefijo `string`; el prefijo se añade a cada línea reformateada. Útil para comentarios en código fuente |
| `-s` | Modo solo-separación: las líneas largas se dividen pero las cortas **no** se unen. Útil para código |
| `-u` | Espaciado uniforme: un espacio entre palabras, dos entre oraciones; elimina el relleno de justificación |
| `-w width` | Ancho de columna objetivo (defecto: 75) |

**Ejemplo — formatear solo los comentarios de un archivo de código:**

```bash
# Archivo fmt-code.txt contiene líneas de código y comentarios (# ...)
fmt -w 50 -p '# ' fmt-code.txt
```

Las líneas de comentario adyacentes se unen y reformatean; las líneas de código y las líneas en blanco se preservan intactas.

---

### `pr` — Preparar Texto para Impresión

`pr` **pagina** texto: separa la salida en páginas añadiendo márgenes superiores/inferiores y un **header** automático en cada página.

El header por defecto contiene: fecha/hora de modificación del archivo, nombre del archivo y número de página.

```bash
pr -l 15 -w 65 distros.txt
```

```
2025-12-11 18:27                 distros.txt                 Page 1


SUSE          10.2   12/07/2006
Fedora        10     11/25/2008
...


2025-12-11 18:27                 distros.txt                 Page 2
...
```

| Opción | Descripción |
|--------|-------------|
| `-l number` | Longitud de página en líneas |
| `-w number` | Ancho de página en columnas |

> `pr` tiene muchas más opciones para controlar el diseño de página, que se tratan en el Capítulo 22.

---

### `printf` — Formatear e Imprimir Datos

`printf` es **diferente** al resto: no acepta stdin ni se usa en pipelines directamente. Se usa principalmente en **scripts**. En bash es un builtin.

Origen: lenguaje C (de ahí "print formatted"). Implementado en muchos lenguajes.

**Sintaxis:**

```
printf [-v var] "formato" argumentos
```

- `-v var`: guarda el resultado en la variable `var` en lugar de imprimirlo

**El string de formato puede contener:**
- Texto literal
- Secuencias de escape (`\n`, `\t`, `\a`, etc.)
- **Conversion specifications** (empiezan con `%`)

#### Especificadores de tipo de dato

| Especificador | Descripción |
|---------------|-------------|
| `d` | Entero decimal con signo |
| `f` | Número de punto flotante |
| `o` | Entero en octal |
| `s` | Cadena de texto |
| `x` | Hexadecimal (minúsculas a-f) |
| `X` | Hexadecimal (mayúsculas A-F) |
| `%` | Literal `%` (escribir `%%`) |

**Ejemplo básico:**

```bash
printf "%d, %f, %o, %s, %x, %X\n" 380 380 380 380 380 380
# → 380, 380.000000, 574, 380, 17c, 17C
```

#### Sintaxis completa de una conversion specification

```
%[flags][width][.precision]especificador
```

**Componentes opcionales:**

| Componente | Descripción |
|------------|-------------|
| `flags` | Modificadores de formato (ver tabla abajo) |
| `width` | Ancho mínimo del campo (en caracteres) |
| `.precision` | Para flotantes: dígitos decimales. Para strings: máximo de caracteres a imprimir |

**Flags:**

| Flag | Descripción |
|------|-------------|
| `#` | Formato alternativo: octal prefijado con `0`, hex con `0x`/`0X` |
| `0` | Rellenar con ceros a la izquierda (ej. `00380`) |
| `-` | Alinear a la izquierda (por defecto: derecha) |
| `' '` | Espacio inicial para números positivos |
| `+` | Mostrar signo en números positivos (por defecto solo muestra `-` en negativos) |

#### Tabla de ejemplos

| Argumento | Formato | Resultado | Notas |
|-----------|---------|-----------|-------|
| 380 | `%d` | `380` | Entero simple |
| 380 | `%#x` | `0x17c` | Hex con formato alternativo |
| 380 | `%05d` | `00380` | Ceros a la izquierda, ancho 5 |
| 380 | `%05.5f` | `380.00000` | Flotante con 5 decimales |
| 380 | `%010.5f` | `0380.00000` | Ancho 10, relleno con ceros |
| 380 | `%+d` | `+380` | Muestra signo positivo |
| 380 | `%-d` | `380` | Alineado a la izquierda |
| abcdefghijk | `%5s` | `abcdefghijk` | Ancho mínimo (sin efecto si string es más largo) |
| abcdefghijk | `%.5s` | `abcde` | Precisión trunca el string |

**Ejemplos prácticos:**

```bash
# Guardar en variable
printf -v result "I formatted the string: %s\n" foo
echo "$result"

# Separar campos con tabs
printf "%s\t%s\t%s\n" str1 str2 str3
# → str1    str2    str3

# Formateo de tabla
printf "Line: %05d %15.3f Result: %+15d\n" 1071 3.14156295 32589
# → Line: 01071           3.142 Result:          +32589

# Generar HTML
printf "<html>\n\t<head>\n\t\t<title>%s</title>\n\t</head>\n\t<body>\n\t\t<p>%s</p>\n\t</body>\n</html>\n" \
  "Page Title" "Page Content"
```

> `printf` se usa principalmente en scripts para formatear datos tabulares, no en la línea de comandos directamente.

---

## Sistemas de Formateo de Documentos

Para documentos grandes y complejos (publicaciones científicas, libros, etc.) existen dos familias principales:

### Historia

El primer UNIX se desarrolló en una PDP-7. Para justificar el costo de una PDP-11, los desarrolladores propusieron implementar un sistema de formateo de documentos para la división de patentes de AT&T. Ese primer programa fue una reimplementación del `roff` de McIllroy, escrito por J. F. Ossanna.

### Familia roff/troff

- **Origen**: programa `roff` original (*run off*, como "imprimir una copia")
- **`nroff`**: formatea documentos para dispositivos de fuente monoespaciada (terminales, impresoras de teletipo)
- **`troff`**: formatea para **typesetters** (dispositivos de composición tipográfica para impresión comercial)
- **Programas complementarios**: `eqn` (ecuaciones matemáticas), `tbl` (tablas)
- **`groff`**: implementación GNU de troff, incluye scripts para emular nroff y toda la familia roff

### Familia TeX

- Desarrollado por **Donald Knuth** (pronunciado "tek", la "E" mayúscula en el medio es intencional)
- Apareció en forma estable en **1989**, desplazando a troff para salida de typesetter
- No se cubre en este libro por su complejidad y porque no está instalado por defecto
- Para instalar: paquete **texlive**; editor gráfico: **LyX**

---

### `groff` — Sistema de Formateo de Documentos

`groff` es la suite GNU que implementa `troff`. Usa **macro packages** que condensan los comandos de bajo nivel en comandos de alto nivel más fáciles de usar.

**Conexión con las páginas `man`:** Los man pages se renderizan con `groff` usando el macro package `mandoc`. Están almacenados en `/usr/share/man/` como archivos de texto comprimidos con gzip.

**Ver el markup raw de un man page:**

```bash
zcat /usr/share/man/man1/ls.1.gz | head
```
```
.TH LS "1" "January 2025" "GNU coreutils 8.28" "User Commands"
.SH NAME
ls \- list directory contents
.SH SYNOPSIS
.B ls
[\fI\,OPTION\/\fR]... [\fI\,FILE\/\fR]...
```

**Simular el comando `man`:**

```bash
# Salida ASCII (equivale a lo que muestra man)
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc -T ascii | head

# Salida PostScript (formato por defecto de groff)
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc | head
```

**Generar un archivo PostScript y convertir a PDF:**

```bash
# Guardar como PostScript
zcat /usr/share/man/man1/ls.1.gz | groff -mandoc > ~/Desktop/ls.ps

# Convertir a PDF (ps2pdf es parte del paquete ghostscript)
ps2pdf ~/Desktop/ls.ps ~/Desktop/ls.pdf
```

> **Convención de nombres**: Linux incluye muchos programas de conversión de formatos con el patrón `formato2formato`. Para encontrarlos:
> ```bash
> ls /usr/bin/*[[:alpha:]]2[[:alpha:]]]*
> ```

#### `tbl` — Formateo de Tablas con groff

`tbl` es el programa de la familia roff para formatear tablas. Se integra con `groff` mediante la opción `-t`.

El markup de tbl usa las instrucciones `.TS` (table start) y `.TE` (table end). Las líneas entre ellas definen las propiedades globales de la tabla (alineación, bordes) y el layout de cada fila.

**Pipeline completo para generar un informe con tabla:**

```bash
# Salida ASCII en terminal
sort -k 1,1 -k 2n distros.txt | sed -f distros-tbl.sed | groff -t -T ascii

# Salida PostScript (mejor calidad tipográfica)
sort -k 1,1 -k 2n distros.txt | sed -f distros-tbl.sed | groff -t > ~/Desktop/foo.ps
```

- `-t`: indica a groff que procese el stream con `tbl`
- `-T ascii`: salida en ASCII en lugar de PostScript (defecto)

---

## Resumen

Este capítulo cubre las herramientas de **formateo de salida** de Linux:

- **`nl`**: numeración de líneas con soporte para páginas lógicas (header/body/footer), estilos y formatos de número
- **`fold`**: ajuste de ancho de línea, con o sin respeto de límites de palabras (`-s`)
- **`fmt`**: formateo de párrafos con modos crown margin (`-c`), prefijo selectivo (`-p`), solo-split (`-s`) y espaciado uniforme (`-u`)
- **`pr`**: paginación con headers automáticos para preparar texto para impresión
- **`printf`**: formateo preciso de datos con especificadores de tipo (`d`, `f`, `s`, `x`, `o`), flags, ancho y precisión; principalmente para scripts
- **`groff`**: suite de formateo de documentos GNU (implementación de troff); renderiza man pages con el macro package `mandoc`; produce PostScript por defecto; se complementa con `tbl` para tablas y `ps2pdf` para convertir a PDF

Las herramientas simples (`fmt`, `pr`) son útiles en scripts para documentos cortos; `groff` y sus compañeros se usan para producir libros y publicaciones técnicas completas.

---

## Ver También

- [[wiki/linux/20-procesamiento-texto.md|Capítulo 20: Procesamiento de Texto]] — `sed`, `sort`, `cut` y otros usados junto con estas herramientas de formato
- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] — Pipelines que conectan estas herramientas
