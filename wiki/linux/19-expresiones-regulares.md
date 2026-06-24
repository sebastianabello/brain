---
title: "Capítulo 19: Expresiones Regulares"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 19"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-269-287.pdf"
updated: "2026-06-23"
---

# Expresiones Regulares

Las expresiones regulares son una herramienta fundamental en Linux para manipular y buscar texto. Aunque su curva de aprendizaje puede ser pronunciada, dominarlas permite realizar búsquedas y transformaciones muy sofisticadas. Este capítulo introduce los conceptos esenciales y su aplicación práctica.

## ¿Qué son las Expresiones Regulares?

Las **expresiones regulares** (regex) son notaciones simbólicas utilizadas para identificar patrones en texto. Funcionan de forma similar a los comodines del shell (wildcards), pero operan a una escala mucho más granular.

Las expresiones regulares son soportadas por:
- Muchas herramientas de línea de comandos
- Lenguajes de programación
- El estándar **POSIX** (que cubriremos aquí, usado en la mayoría de herramientas Linux)

> **Nota importante**: Existen variaciones en la sintaxis entre herramientas y lenguajes. Este capítulo se enfoca en las expresiones regulares POSIX como se usan en las herramientas de línea de comandos de Linux.

## El Comando `grep`

El programa principal para trabajar con expresiones regulares es **grep** (global regular expression print). Su nombre procede de la frase "global regular expression print", lo que significa que grep busca archivos de texto en busca de coincidencias con una expresión regular especificada y muestra cualquier línea que contenga una coincidencia en la salida estándar.

### Sintaxis básica

```bash
grep [opciones] regex [archivo...]
```

### Ejemplo inicial

Para buscar un patrón en archivos:

```bash
[me@linuxbox ~]$ grep bzip dirlist*.txt
dirlist-bin.txt:bzip2
dirlist-bin.txt:bzip2recover
```

### Opciones comunes de `grep`

| Opción | Opción larga | Descripción |
|--------|---|---|
| `-i` | `--ignore-case` | Ignora mayúsculas/minúsculas |
| `-v` | `--invert-match` | Invierte la coincidencia. Muestra líneas que NO contienen el patrón |
| `-c` | `--count` | Imprime el número de coincidencias en lugar de las líneas |
| `-l` | `--files-with-matches` | Imprime solo los nombres de archivos que contienen coincidencias |
| `-L` | `--files-without-match` | Imprime solo los nombres de archivos que NO contienen coincidencias |
| `-n` | `--line-number` | Prefija cada línea coincidente con su número de línea |
| `-h` | `--no-filename` | Para búsquedas multarchivo, suprime la salida de nombres de archivo |
| `-q` | `--quiet, --silent` | Modo silencioso. Útil en scripting para verificar si se encontró una coincidencia |

## Metacaracteres y Literales

### Tipos de caracteres

En una expresión regular, los caracteres se dividen en:
- **Literales**: Caracteres normales que coinciden exactamente consigo mismos
- **Metacaracteres**: Caracteres especiales con significado especial en regex

### Metacaracteres básicos

```
^ $ . [ ] ( ) - ? * + { } | \
```

> **Nota**: Cuando pasamos expresiones regulares con metacaracteres en la línea de comandos, es vital encerrarlas entre comillas para evitar que el shell intente expandirlas.

## El Carácter Comodín (.)

El primer metacarácter es el **punto (.)**, que representa "cualquier carácter" en esa posición.

```bash
[me@linuxbox ~]$ grep -h '.zip' dirlist*.txt
bunzip2
bzip2
bzip2recover
gzip
...
```

Este ejemplo busca cualquier línea que contenga exactamente 4 caracteres terminados en "zip" (siendo el primer carácter cualquiera).

> **Nota importante**: Si quisiéramos buscar un punto literal, necesitaríamos escaparlo con una barra invertida: `\.`

## Anclajes

Los anclajes determinan dónde en la línea debe ocurrir la coincidencia:

### Circunflejo (^) - Inicio de línea

Especifica que la coincidencia debe ocurrir al **inicio de la línea**:

```bash
[me@linuxbox ~]$ grep -h '^zip' dirlist*.txt
zip
zipcloak
zipgrep
zipinfo
zipnote
zipsplit
```

### Dólar ($) - Fin de línea

Especifica que la coincidencia debe ocurrir al **final de la línea**:

```bash
[me@linuxbox ~]$ grep -h 'zip$' dirlist*.txt
zip
```

### Combinación: Inicio y final

Para buscar líneas que contienen SOLO la cadena "zip":

```bash
[me@linuxbox ~]$ grep -h '^zip$' dirlist*.txt
zip
```

> **Nota**: Una línea en blanco se representa como `^$` (inicio inmediatamente seguido de fin).

## Expresiones de Corchetes y Clases de Caracteres

### Definición de conjuntos

Las **expresiones de corchetes** permiten especificar un conjunto de caracteres de los que debe coincidir exactamente uno en esa posición.

```bash
[me@linuxbox ~]$ grep -h '[bg]zip' dirlist*.txt
bzip2
bzip2recover
gzip
```

### Negación

Si el primer carácter en una expresión de corchetes es un **circunflejo (^)**, el significado se invierte: coincide con cualquier carácter que NO esté en el conjunto.

```bash
[me@linuxbox ~]$ grep -h '[^bg]zip' dirlist*.txt
funzip
prezip
```

> **Importante**: El circunflejo solo invierte cuando es el PRIMER carácter. Si aparece en otro lugar, pierde su significado especial.

## Rangos de Caracteres Tradicionales

Para evitar escribir todos los caracteres, podemos usar **rangos** con el carácter de guión (-).

### Rango de mayúsculas

```bash
[me@linuxbox ~]$ grep -h '^[A-Z]' dirlist*.txt
MAKEDEV
GET
HEAD
POST
X
X11
...
```

### Rangos múltiples

Podemos combinar múltiples rangos en una expresión:

```bash
[me@linuxbox ~]$ grep -h '^[A-Za-z0-9]' dirlist*.txt
```

Este patrón coincide con cualquier archivo que comience con letras o números.

### Inclusión del guión

Para incluir un guión literal en un rango, colócalo como el **primer carácter** de la expresión:

```bash
[me@linuxbox ~]$ grep -h '[-A-Z]' dirlist*.txt
```

## Clases de Caracteres POSIX

Los rangos de caracteres tradicionales tienen limitaciones, especialmente con caracteres acentuados y configuraciones de locale diferentes.

### ¿Por qué surgen las clases POSIX?

En los años 80, cuando Unix era un sistema comercial popular, cada fabricante agregaba cambios propietarios. Esto provocó incompatibilidades. El estándar **POSIX** (Portable Operating System Interface, IEEE 1003) fue desarrollado por el IEEE para estandarizar cómo deberían comportarse los sistemas Unix.

Las clases de caracteres POSIX proporciona rangos útiles que funcionan consistentemente:

| Clase | Descripción |
|-------|---|
| `[:alnum:]` | Caracteres alfanuméricos; equivalente a `[A-Za-z0-9]` |
| `[:word:]` | Igual que `[:alnum:]`, con adición del guión bajo (_) |
| `[:alpha:]` | Caracteres alfabéticos; equivalente a `[A-Za-z]` |
| `[:blank:]` | Espacios y caracteres de tabulación |
| `[:cntrl:]` | Códigos de control ASCII (0-31 y 127) |
| `[:digit:]` | Números (0-9) |
| `[:graph:]` | Caracteres visibles (ASCII 33-126) |
| `[:lower:]` | Letras minúsculas |
| `[:print:]` | Caracteres imprimibles (todos en `[:graph:]` más espacio) |
| `[:punct:]` | Caracteres de puntuación |
| `[:space:]` | Espacios en blanco (espacio, tabulación, salto de línea, etc.) |
| `[:upper:]` | Letras mayúsculas |
| `[:xdigit:]` | Caracteres para números hexadecimales (0-9a-fA-f) |

### Ejemplo de uso

```bash
[me@linuxbox ~]$ ls /usr/sbin/[[:upper:]]*
/usr/sbin/MAKEFLOPPIES
/usr/sbin/NetworkManagerDispatcher
/usr/sbin/NetworkManager
```

### Locale y orden de colación

La variable de entorno **LANG** determina el idioma y conjunto de caracteres usado en el sistema:

```bash
[me@linuxbox ~]$ echo $LANG
en_US.UTF-8
```

Para cambiar a orden tradicional POSIX (ASCII):

```bash
[me@linuxbox ~]$ export LANG=POSIX
```

Esto es especialmente útil cuando trabajas con rangos de caracteres y quieres asegurar comportamiento predecible.

## Expresiones Regulares Básicas vs Extendidas (BRE vs ERE)

POSIX divide las expresiones regulares en dos tipos:

### Expresiones Regulares Básicas (BRE - Basic Regular Expressions)

Los metacaracteres reconocidos son: `^ $ . [ ] *`

Todos los demás caracteres se consideran literales.

### Expresiones Regulares Extendidas (ERE - Extended Regular Expressions)

Agrega metacaracteres adicionales: `( ) { } ? + |`

Con ERE, estos caracteres ganan significado especial sin necesidad de escape.

Para usar ERE con `grep`, necesitas la opción `-E`. Alternativamente, puedes usar `egrep` (GNU grep también soporta `-E`).

## Alternancia

La **alternancia** permite que una coincidencia ocurra a partir de un conjunto de expresiones. Se especifica con el metacarácter **tubo vertical (|)**.

### Ejemplo simple

```bash
[me@linuxbox ~]$ echo "AAA" | grep -E 'AAA|BBB'
AAA
[me@linuxbox ~]$ echo "BBB" | grep -E 'AAA|BBB'
BBB
[me@linuxbox ~]$ echo "CCC" | grep -E 'AAA|BBB'
[me@linuxbox ~]$
```

### Agrupación con paréntesis

Puedes combinar alternancia con otras características usando paréntesis:

```bash
[me@linuxbox ~]$ grep -Eh '^(bz|gz|zip)' dirlist*.txt
```

Este patrón busca líneas que comiencen con `bz`, `gz`, o `zip`.

## Cuantificadores

Los cuantificadores especifican cuántas veces debe coincidir un elemento:

### `?` - Coincidencia cero o una vez

Hace el elemento anterior opcional:

```bash
[me@linuxbox ~]$ echo "(555) 123-4567" | grep -E '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$'
(555) 123-4567
```

### `*` - Coincidencia cero o más veces

El elemento anterior puede aparecer cualquier número de veces (incluyendo cero):

```bash
[me@linuxbox ~]$ echo "This works." | grep -E '^[[:upper:]][[:upper:][:lower:] ]*\.'
This works.
```

### `+` - Coincidencia una o más veces

Requiere al menos una aparición del elemento anterior:

```bash
[me@linuxbox ~]$ echo "This that" | grep -E '^([[:alpha:]]+) ([[:alpha:]]+)$'
This that
```

### `{}` - Coincidencia un número específico de veces

Permite especificar exactamente cuántas veces coincidir:

| Especificador | Significado |
|-------|---|
| `{n}` | Coincide exactamente n veces |
| `{n,m}` | Coincide al menos n veces pero no más de m |
| `{n,}` | Coincide n o más veces |
| `{,m}` | Coincide no más de m veces |

### Validación de números de teléfono

Un ejemplo práctico usando cuantificadores:

```bash
[me@linuxbox ~]$ echo "(555) 123-4567" | grep -E '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$'
(555) 123-4567
```

Este patrón coincide con números telefónicos en el formato `(NNN) NNN-NNNN` donde N es un dígito.

## Aplicaciones Prácticas

### Validar una lista de números telefónicos

Escenario: Queremos validar que los números en una lista tienen el formato correcto y encontrar los malformados.

```bash
[me@linuxbox ~]$ grep -Ev '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$' phonelist.txt
(292) 108-518
(129) 44-1379
```

Aquí usamos `-v` para mostrar las líneas que NO coinciden con el patrón de número válido.

### Buscar archivos con `find` y expresiones regulares

El comando `find` soporta pruebas de expresiones regulares:

```bash
[me@linuxbox ~]$ find . -regex '.*[0-9a-zA-Z]*\.'
```

Este busca archivos con caracteres alfanuméricos en sus nombres.

### Buscar en archivos comprimidos con `locate`

El comando `locate` soporta tanto expresiones regulares básicas (`--regex`) como extendidas (`--regex`):

```bash
[me@linuxbox ~]$ locate --regex 'bin/(bz|gz|zip)'
/bin/bzcat
/bin/bzcmp
...
```

### Buscar texto en editores

Tanto `less` como `vim` permiten búsquedas con expresiones regulares usando el prefijo `/`:

En `less`:

```bash
[me@linuxbox ~]$ less phonelist.txt
~
/^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$
```

En `vim`, la sintaxis es ligeramente diferente (BRE por defecto), sin escape de paréntesis:

```
:1search /\([0-9]\{3\}\) [0-9]\{3\}-[0-9]\{4\}
```

### Búsqueda en páginas de manual

```bash
[me@linuxbox ~]$ cd /usr/share/man/man1
[me@linuxbox man1]$ zgrep -l 'regex|regular expression' *.gz
```

El programa `zgrep` proporciona una interfaz frontal para grep que permite leer archivos comprimidos.

## Resumen

Las expresiones regulares son una herramienta poderosa para:
- Búsquedas de patrones sofisticadas
- Validación de datos
- Procesamiento de texto
- Manipulación de archivos

**Conceptos clave**:
- Metacaracteres vs literales
- Anclajes (`^` y `$`)
- Rangos y clases de caracteres
- Clases POSIX para portabilidad
- BRE vs ERE
- Cuantificadores para especificar frecuencias
- Alternancia para múltiples patrones

Aunque dominar las expresiones regulares requiere práctica, la inversión de tiempo es ampliamente recompensada con la capacidad de realizar búsquedas y transformaciones muy sofisticadas.

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]]
- [[wiki/linux/02-navegacion-sistema-archivos.md|Capítulo 2: Navegación del Sistema de Archivos]]
