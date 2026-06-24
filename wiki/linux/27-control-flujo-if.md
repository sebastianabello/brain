---
title: "Capítulo 27: Control de Flujo – Ramificación con if"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 27"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-395-409.pdf"
updated: "2026-06-23"
---

# Control de Flujo: Ramificación con if

El **control de flujo** permite que un script "cambie de dirección" según los resultados de una prueba. En programación esto se llama **branching** (ramificación). El mecanismo principal en bash es el comando `if`.

Ejemplo en pseudocódigo:

```
X = 5
If X = 5, then:
    Say "X equals 5."
Otherwise:
    Say "X does not equal 5."
```

## Sentencias `if`

Sintaxis completa del comando `if`:

```bash
if commands; then
    commands
[elif commands; then
    commands...]
[else
    commands]
fi
```

Ejemplo básico en script:

```bash
x=5

if [ "$x" -eq 5 ]; then
    echo "x equals 5."
else
    echo "x does not equal 5."
fi
```

O comprimido en una línea:

```bash
[me@linuxbox ~]$ x=5
[me@linuxbox ~]$ if [ "$x" -eq 5 ]; then echo "equals 5"; else echo "does not equal 5"; fi
equals 5
[me@linuxbox ~]$ x=0
[me@linuxbox ~]$ if [ "$x" -eq 5 ]; then echo "equals 5"; else echo "does not equal 5"; fi
does not equal 5
```

> **Clave:** `if` evalúa el **éxito o fracaso de comandos**, no expresiones booleanas directamente.

## Exit Status (Estado de Salida)

Todo comando emite un **exit status** al terminar: un entero de 0 a 255.

- **0** = éxito
- **Cualquier otro valor** = fallo (algunos comandos usan distintos valores para distintos errores)

El parámetro especial `$?` contiene el exit status del último comando ejecutado:

```bash
[me@linuxbox ~]$ ls -d /usr/bin
/usr/bin
[me@linuxbox ~]$ echo $?
0

[me@linuxbox ~]$ ls -d /bin/usr
ls: cannot access /bin/usr: No such file or directory
[me@linuxbox ~]$ echo $?
2
```

### Comandos `true` y `false`

Bash provee dos builtins que solo emiten exit status:

```bash
[me@linuxbox ~]$ true
[me@linuxbox ~]$ echo $?
0

[me@linuxbox ~]$ false
[me@linuxbox ~]$ echo $?
1
```

Cuando `if` recibe una **lista** de comandos, evalúa el exit status del **último** de la lista:

```bash
[me@linuxbox ~]$ if false; true; then echo "It's true."; fi
It's true.

[me@linuxbox ~]$ if true; false; then echo "It's true."; fi
[me@linuxbox ~]$
```

## El Comando `test`

El comando más usado junto a `if` es `test`. Realiza verificaciones y comparaciones. Tiene dos formas equivalentes:

```bash
test expression
[ expression ]
```

`test` retorna exit status **0** si la expresión es verdadera, **1** si es falsa.

> **Nota:** `[` es en realidad un comando (builtin en bash, también disponible en `/usr/bin` para otros shells). El `]` es su último argumento obligatorio — se requiere un espacio antes del `]`.

### Expresiones de Archivo (File Expressions)

| Expresión | Verdadera si... |
|-----------|----------------|
| `file1 -ef file2` | file1 y file2 tienen el mismo inode (hard link al mismo archivo) |
| `file1 -nt file2` | file1 es más reciente que file2 |
| `file1 -ot file2` | file1 es más antiguo que file2 |
| `-b file` | file existe y es un archivo especial de bloque (device) |
| `-c file` | file existe y es un archivo especial de caracteres (device) |
| `-d file` | file existe y es un directorio |
| `-e file` | file existe |
| `-f file` | file existe y es un archivo regular |
| `-g file` | file existe y tiene set-group-ID activo |
| `-G file` | file existe y es propiedad del grupo efectivo del usuario |
| `-k file` | file existe y tiene el sticky bit activo |
| `-L file` | file existe y es un enlace simbólico |
| `-O file` | file existe y es propiedad del usuario efectivo |
| `-p file` | file existe y es un named pipe |
| `-r file` | file existe y es legible (por el usuario efectivo) |
| `-s file` | file existe y tiene longitud mayor que cero |
| `-S file` | file existe y es un socket de red |
| `-t fd` | fd es un descriptor de archivo dirigido a/desde la terminal (útil para detectar si stdin/stdout se está redirigiendo) |
| `-u file` | file existe y tiene setuid activo |
| `-w file` | file existe y es escribible (por el usuario efectivo) |
| `-x file` | file existe y es ejecutable/searchable (por el usuario efectivo) |

**Script ejemplo — evaluación de atributos de un archivo:**

```bash
#!/bin/bash

# test-file: Evaluate the status of a file

FILE=~/.bashrc

if [ -e "$FILE" ]; then
    if [ -f "$FILE" ]; then
        echo "$FILE is a regular file."
    fi
    if [ -d "$FILE" ]; then
        echo "$FILE is a directory."
    fi
    if [ -r "$FILE" ]; then
        echo "$FILE is readable."
    fi
    if [ -w "$FILE" ]; then
        echo "$FILE is writable."
    fi
    if [ -x "$FILE" ]; then
        echo "$FILE is executable/searchable."
    fi
else
    echo "$FILE does not exist"
    exit 1
fi

exit
```

> **¿Por qué citar `"$FILE"`?** Si `$FILE` se expande a string vacío o con espacios, los operadores serían interpretados como strings no-nulos en lugar de operadores, causando errores. Las comillas garantizan que siempre haya un string (aunque vacío).

El comando `exit` acepta un argumento opcional que se convierte en el exit status del script. Sin argumento, usa el exit status del último comando ejecutado. Las shell functions usan `return` de forma análoga.

**Versión como shell function** (se reemplaza `exit` por `return`):

```bash
test_file () {
    FILE=~/.bashrc

    if [ -e "$FILE" ]; then
        if [ -f "$FILE" ]; then
            echo "$FILE is a regular file."
        fi
        if [ -d "$FILE" ]; then
            echo "$FILE is a directory."
        fi
        if [ -r "$FILE" ]; then
            echo "$FILE is readable."
        fi
        if [ -w "$FILE" ]; then
            echo "$FILE is writable."
        fi
        if [ -x "$FILE" ]; then
            echo "$FILE is executable/searchable."
        fi
    else
        echo "$FILE does not exist"
        return 1
    fi
}
```

### Expresiones de String (String Expressions)

| Expresión | Verdadera si... |
|-----------|----------------|
| `string` | string no es nulo |
| `-n string` | La longitud de string es mayor que cero |
| `-z string` | La longitud de string es cero |
| `string1 = string2` o `string1 == string2` | string1 y string2 son iguales (`==` preferido en bash; no es POSIX pero generalmente soportado) |
| `string1 != string2` | string1 y string2 no son iguales |
| `string1 > string2` | string1 ordena después de string2 (alfabéticamente) |
| `string1 < string2` | string1 ordena antes de string2 (alfabéticamente) |

> **⚠️ Advertencia:** Los operadores `>` y `<` **deben ir entre comillas** o escapados con `\` cuando se usan con `test`/`[ ]`. Sin esto, el shell los interpreta como operadores de redirección de I/O con resultados potencialmente destructivos (overwrite de archivos).
>
> Nota adicional: el orden de collation de bash puede variar según el locale. ASCII/POSIX se usa en versiones hasta 4.0; corregido en 4.1.

**Script ejemplo — evaluación de strings con `elif`:**

```bash
#!/bin/bash

# test-string: evaluate the value of a string

ANSWER=maybe

if [ -z "$ANSWER" ]; then
    echo "There is no answer." >&2
    exit 1
fi

if [ "$ANSWER" == "yes" ]; then
    echo "The answer is YES."
elif [ "$ANSWER" == "no" ]; then
    echo "The answer is NO."
elif [ "$ANSWER" == "maybe" ]; then
    echo "The answer is MAYBE."
else
    echo "The answer is UNKNOWN."
fi
```

`>&2` redirige el mensaje de error a stderr (la forma correcta de manejar mensajes de error). `elif` es la abreviatura de "else if", permite construir pruebas lógicas más complejas.

### Expresiones de Enteros (Integer Expressions)

| Expresión | Verdadera si... |
|-----------|----------------|
| `integer1 -eq integer2` | integer1 **es igual** a integer2 |
| `integer1 -ne integer2` | integer1 **no es igual** a integer2 |
| `integer1 -le integer2` | integer1 **es menor o igual** a integer2 |
| `integer1 -lt integer2` | integer1 **es menor** que integer2 |
| `integer1 -ge integer2` | integer1 **es mayor o igual** a integer2 |
| `integer1 -gt integer2` | integer1 **es mayor** que integer2 |

**Script ejemplo — evaluación de enteros:**

```bash
#!/bin/bash

# test-integer: evaluate the value of an integer.

INT=-5

if [ -z "$INT" ]; then
    echo "INT is empty." >&2
    exit 1
fi

if [ "$INT" -eq 0 ]; then
    echo "INT is zero."
else
    if [ "$INT" -lt 0 ]; then
        echo "INT is negative."
    else
        echo "INT is positive."
    fi
    if [ $((INT % 2)) -eq 0 ]; then
        echo "INT is even."
    else
        echo "INT is odd."
    fi
fi
```

El operador módulo `%` divide el número entre 2 y retorna el resto, determinando si es par (resto = 0) o impar (resto ≠ 0).

## `[[ ]]` — Versión Moderna de `test`

Las versiones modernas de bash incluyen el comando compuesto `[[ ]]`, reemplazo mejorado de `test`:

```bash
[[ expression ]]
```

Soporta todas las expresiones de `test` y agrega características clave:

### Matching con Expresiones Regulares (`=~`)

```bash
string1 =~ regex
```

Retorna verdadero si `string1` es emparejado por la expresión regular extendida `regex`.

**Ejemplo — validar que INT sea realmente un entero antes de comparar:**

```bash
#!/bin/bash

# test-integer2: evaluate the value of an integer.

INT=-5

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if [ "$INT" -eq 0 ]; then
        echo "INT is zero."
    else
        if [ "$INT" -lt 0 ]; then
            echo "INT is negative."
        else
            echo "INT is positive."
        fi
        if [ $((INT % 2)) -eq 0 ]; then
            echo "INT is even."
        else
            echo "INT is odd."
        fi
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

La regex `^-?[0-9]+$` valida: inicio de string (`^`), signo menos opcional (`-?`), uno o más dígitos (`[0-9]+`), fin de string (`$`). Esto elimina también la posibilidad de valores vacíos.

### Pattern Matching con `==`

`[[ ]]` también soporta pattern matching (como pathname expansion) con `==`:

```bash
[me@linuxbox ~]$ FILE=foo.bar
[me@linuxbox ~]$ if [[ $FILE == foo.* ]]; then
> echo "$FILE matches pattern 'foo.*'"
> fi
foo.bar matches pattern 'foo.*'
```

Muy útil para evaluar nombres de archivo y pathnames.

### `[[ ]]` vs `test` — ¿Cuál usar?

| Característica | `test` / `[ ]` | `[[ ]]` |
|----------------|----------------|---------|
| Portabilidad | POSIX, todos los shells | Específico de bash y algunos shells modernos |
| Expresiones regulares (`=~`) | No | Sí |
| Pattern matching (`==`) | No | Sí |
| Operadores `&&` y `\|\|` | No (usa `-a` y `-o`) | Sí |
| Chars especiales `<` `>` `(` `)` | Deben escaparse | No necesitan escape |
| Recomendación | Scripts portables (sh, startup) | Scripts bash modernos |

> `[[ ]]` es más potente y fácil de codificar. Es la opción preferida para scripts bash modernos.

## `(( ))` — Diseñado para Enteros

Bash también provee el comando compuesto `(( ))` para realizar **arithmetic truth tests** (pruebas de verdad aritméticas). Un resultado **no-cero es verdadero**:

```bash
[me@linuxbox ~]$ if ((1)); then echo "It is true."; fi
It is true.
[me@linuxbox ~]$ if ((0)); then echo "It is true."; fi
[me@linuxbox ~]$
```

Como `(( ))` es parte de la sintaxis del shell (no un comando ordinario), puede:
- Acceder a variables **sin** `$` (aunque `$` también funciona)
- Usar operadores aritméticos naturales: `<`, `>`, `==`, `<=`, `>=`, `!=`

**Versión mejorada del script de enteros usando `(( ))`:**

```bash
#!/bin/bash

# test-integer2a: evaluate the value of an integer.

INT=-5

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if ((INT == 0)); then
        echo "INT is zero."
    else
        if ((INT < 0)); then
            echo "INT is negative."
        else
            echo "INT is positive."
        fi
        if (( (INT % 2) == 0 )); then
            echo "INT is even."
        else
            echo "INT is odd."
        fi
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

`(( ))` se discute en profundidad en el Capítulo 34 junto con la expansión aritmética.

## Combinando Expresiones

Se pueden combinar expresiones con operadores lógicos:

| Operación | `test` / `[ ]` | `[[ ]]` y `(( ))` |
|-----------|:--------------:|:-----------------:|
| AND | `-a` | `&&` |
| OR | `-o` | `\|\|` |
| NOT | `!` | `!` |

**Ejemplo AND — verificar que INT esté dentro de un rango:**

```bash
#!/bin/bash

# test-integer3: determine if an integer is within a specified range.

MIN_VAL=1
MAX_VAL=100
INT=50

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if [[ "$INT" -ge "$MIN_VAL" && "$INT" -le "$MAX_VAL" ]]; then
        echo "$INT is within $MIN_VAL to $MAX_VAL."
    else
        echo "$INT is out of range."
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

Con `test` equivale a usar `-a`:

```bash
if [ "$INT" -ge "$MIN_VAL" -a "$INT" -le "$MAX_VAL" ]; then
```

**Ejemplo NOT — detectar fuera de rango:**

```bash
#!/bin/bash

# test-integer4: determine if an integer is outside a specified range.

MIN_VAL=1
MAX_VAL=100
INT=50

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
    if [[ ! ("$INT" -ge "$MIN_VAL" && "$INT" -le "$MAX_VAL") ]]; then
        echo "$INT is outside $MIN_VAL to $MAX_VAL."
    else
        echo "$INT is in range."
    fi
else
    echo "INT is not an integer." >&2
    exit 1
fi
```

Con `test`, los paréntesis deben escaparse porque son tratados como argumentos de comando:

```bash
if [ ! \( "$INT" -ge "$MIN_VAL" -a "$INT" -le "$MAX_VAL" \) ]; then
```

> **Importante:** Con `test`/`[ ]`, todos los caracteres con significado especial para el shell (`<`, `>`, `(`, `)`) deben ser citados o escapados. Con `[[ ]]` y `(( ))` esto no es necesario.

## Operadores de Control: Otra Forma de Ramificar

Bash tiene dos **operadores de control** que realizan ramificación en línea: `&&` (AND) y `||` (OR).

```bash
command1 && command2
command1 || command2
```

Comportamiento:
- **`&&`**: `command1` siempre se ejecuta; `command2` se ejecuta **solo si** `command1` tiene éxito (exit 0).
- **`||`**: `command1` siempre se ejecuta; `command2` se ejecuta **solo si** `command1` falla (exit ≠ 0).

**Ejemplos prácticos:**

```bash
# Crear directorio y entrar solo si mkdir tiene éxito
mkdir temp && cd temp

# Crear directorio solo si aún no existe
[[ -d temp ]] || mkdir temp

# En un script: salir si el directorio requerido no existe
[ -d temp ] || exit 1
```

**Comandos de grupo con operadores de control:**

```bash
{ true && echo "true"; } && { false || echo "false"; }
```

Los group commands retornan el exit status del **último comando del grupo**.

## Aplicación al Proyecto: `sys_info_page`

Retomando el problema del capítulo anterior: el script `report_home_space` fallaba si el usuario no tenía permisos para leer los directorios home de otros usuarios. Con `if`, podemos detectar si el usuario es superusuario con `id -u` (el superusuario siempre tiene ID = 0):

```bash
report_home_space () {
    if [[ "$(id -u)" -eq 0 ]]; then
        cat << _EOF_
            <h2>Home Space Utilization (All Users)</h2>
            <pre>$(du -sh /home/*)</pre>
_EOF_
    else
        cat << _EOF_
            <h2>Home Space Utilization ($USER)</h2>
            <pre>$(du -sh $HOME)</pre>
_EOF_
    fi
    return
}
```

- Si el usuario es **root** (ID = 0): muestra el espacio de todos los usuarios (`/home/*`).
- Si es un usuario normal: muestra solo su propio espacio (`$HOME`).

## Resumen

| Herramienta | Uso |
|-------------|-----|
| `if / elif / else / fi` | Estructura principal de ramificación |
| `$?` | Exit status del último comando |
| `true` / `false` | Builtins que retornan exit 0 / exit 1 |
| `exit N` | Terminar script con exit status N |
| `return N` | Retornar de una función con exit status N |
| `test` / `[ ]` | Pruebas (POSIX, compatible con todos los shells) |
| `[[ ]]` | Versión moderna de test (bash); soporta `=~` y pattern matching |
| `(( ))` | Pruebas aritméticas; sintaxis natural sin `$`, operadores `<` `>` `==` |
| `&&` / `\|\|` | Operadores de control para ramificación en línea |

---

## Ver También

- [[26-top-down-design.md|Capítulo 26: Diseño Top-Down]] — Shell functions donde se aplica `if`
- [[09-permisos.md|Capítulo 9: Permisos]] — Contexto sobre usuarios y permisos (`id`, sudo)
- [[19-expresiones-regulares.md|Capítulo 19: Expresiones Regulares]] — Regex usada con `=~` en `[[ ]]`
- [[06-redirection.md|Capítulo 6: Redirección]] — Redirección de stderr con `>&2`
