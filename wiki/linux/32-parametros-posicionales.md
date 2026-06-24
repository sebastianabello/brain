---
title: "Capítulo 32: Parámetros Posicionales"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 32"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-453-467.pdf"
updated: "2026-06-23"
---

# Parámetros Posicionales

Hasta ahora, los scripts no podían aceptar ni procesar opciones ni argumentos de la línea de comandos. Este capítulo cubre las características del shell que permiten a los programas acceder al contenido de la línea de comandos.

---

## Accediendo a la Línea de Comandos

El shell provee un conjunto de variables llamadas **parámetros posicionales** (*positional parameters*) que contienen las palabras individuales de la línea de comandos. Se nombran del `0` al `9`.

Script de demostración:

```bash
#!/bin/bash
# posit-param: mostrar parámetros de la línea de comandos

echo "
\$0 = $0
\$1 = $1
\$2 = $2
\$3 = $3
\$4 = $4
\$5 = $5
\$6 = $6
\$7 = $7
\$8 = $8
\$9 = $9
"
```

Sin argumentos, `$0` siempre contiene el pathname del programa:

```
$0 = /home/me/bin/posit-param
$1 =
$2 =
...
```

Con argumentos (`posit-param a b c d`):

```
$0 = /home/me/bin/posit-param
$1 = a
$2 = b
$3 = c
$4 = d
$5 =
...
```

> **Nota:** Para acceder a parámetros más allá del noveno, usar llaves: `${10}`, `${55}`, `${211}`, etc.

---

## Variables Especiales de la Línea de Comandos

| Variable | Descripción |
|----------|-------------|
| `$0` | Nombre/pathname del programa en ejecución |
| `$1`–`$9` | Argumentos posicionales 1 al 9 |
| `${N}` | Argumento posicional N (para N > 9, requiere llaves) |
| `$#` | Número total de argumentos en la línea de comandos |
| `$*` | Todos los parámetros posicionales como lista |
| `$@` | Todos los parámetros posicionales (forma preferida) |
| `$FUNCNAME` | Nombre de la función shell que se está ejecutando actualmente |
| `$OPTARG` | Valor del argumento para la opción procesada por `getopts` |
| `OPTIND` | Índice del próximo parámetro posicional a procesar por `getopts` |

### Contando Argumentos con `$#`

```bash
echo "Number of arguments: $#"
```

Al ejecutar `posit-param a b c d`:
```
Number of arguments: 4
$0 = /home/me/bin/posit-param
$1 = a  $2 = b  $3 = c  $4 = d
```

---

## `shift` — Procesar Muchos Argumentos

¿Qué pasa cuando hay más de 9 argumentos (o docenas)? El comando **`shift`** hace que todos los parámetros "bajen uno": `$2` pasa a ser `$1`, `$3` pasa a `$2`, etc. `$#` también se reduce en uno. `$0` nunca cambia.

Con `shift`, basta con leer siempre `$1` en un bucle:

```bash
#!/bin/bash
# posit-param2: mostrar todos los argumentos

count=1

while [[ $# -gt 0 ]]; do
    echo "Argument $count = $1"
    count=$((count + 1))
    shift
done
```

Ejecución: `posit-param2 a b c d`

```
Argument 1 = a
Argument 2 = b
Argument 3 = c
Argument 4 = d
```

---

## Aplicaciones Simples con Parámetros Posicionales

### `basename` y `$PROGNAME`

El comando **`basename`** elimina la parte inicial de un pathname, dejando solo el nombre del archivo:

```bash
PROGNAME="$(basename "$0")"
```

Esto hace que los mensajes de uso siempre muestren el nombre correcto del programa, aunque el script sea renombrado.

### Ejemplo: `file-info`

```bash
#!/bin/bash
# file-info: programa de información de archivos

PROGNAME="$(basename "$0")"

if [[ -e "$1" ]]; then
    echo -e "\nFile Type:"
    file "$1"
    echo -e "\nFile Status:"
    stat "$1"
else
    echo "$PROGNAME: usage: $PROGNAME file" >&2
    exit 1
fi
```

- Usa `$1` para el nombre del archivo a examinar
- `file "$1"` — determina el tipo de archivo
- `stat "$1"` — muestra el estado completo del archivo
- Si no se provee argumento, muestra mensaje de uso en stderr y sale con error

### Parámetros Posicionales en Shell Functions

Los parámetros posicionales funcionan igual dentro de las shell functions: `$1` recibe el primer argumento pasado a la función, `$2` el segundo, etc.

Versión como función:

```bash
file_info () {
    # file_info: función para mostrar información de archivo

    if [[ -e "$1" ]]; then
        echo -e "\nFile Type:"
        file "$1"
        echo -e "\nFile Status:"
        stat "$1"
    else
        echo "$FUNCNAME: usage: $FUNCNAME file" >&2
        return 1
    fi
}
```

> **Importante:** Dentro de una función, `$0` sigue conteniendo el pathname del script (no el nombre de la función). Usar **`$FUNCNAME`** para obtener el nombre de la función actual. El shell actualiza esta variable automáticamente.

---

## Manejo Masivo de Parámetros: `$*` vs `"$@"`

Cuando se quiere pasar todos los argumentos de golpe (por ejemplo, al crear un *wrapper* de otro programa), el shell provee dos parámetros especiales:

| Parámetro | Sin comillas | Con comillas dobles |
|-----------|-------------|---------------------|
| `$*` | Se expande en la lista de parámetros (igual que `$@`) | Se expande en **un solo string** con todos los parámetros separados por el primer carácter de `$IFS` (espacio por defecto) |
| `$@` | Se expande en la lista de parámetros (igual que `$*`) | **Cada parámetro se expande como una palabra separada**, como si cada uno estuviera entre comillas dobles |

### Demostración de la Diferencia

Script de prueba con dos argumentos: `"word"` y `"words with spaces"`:

```bash
pass_params () {
    echo -e "\n" '$* :';      print_params $*
    echo -e "\n" '"$*" :';    print_params "$*"
    echo -e "\n" '$@ :';      print_params $@
    echo -e "\n" '"$@" :';    print_params "$@"
}
pass_params "word" "words with spaces"
```

Resultados:

| Forma | Resultado | Descripción |
|-------|-----------|-------------|
| `$*` (sin comillas) | 4 palabras: `word words with spaces` | Se divide en palabras sueltas |
| `"$*"` (con comillas) | 1 palabra: `"word words with spaces"` | Todo combinado en un string |
| `$@` (sin comillas) | 4 palabras: `word words with spaces` | Igual que `$*` sin comillas |
| `"$@"` (con comillas) | **2 palabras**: `"word"` `"words with spaces"` | ✅ Preserva cada argumento original |

**`"$@"` es la forma correcta para la mayoría de situaciones** porque preserva la integridad de cada argumento original. Debe usarse siempre, a menos que haya una razón específica para no hacerlo.

---

## Procesamiento de Opciones de Línea de Comandos

### Patrón: `while` + `case` + `shift`

La técnica estándar para procesar opciones de línea de comandos combina un bucle `while`, un `case` para identificar la opción, y `shift` para avanzar:

```bash
usage () {
    echo "$PROGNAME: usage: $PROGNAME [-f file | -i]"
    return
}

# Variables de estado
interactive=
filename=

# Bucle de procesamiento de opciones
while [[ -n "$1" ]]; do
    case "$1" in
        -f | --file)        shift
                            filename="$1"
                            ;;
        -i | --interactive) interactive=1
                            ;;
        -h | --help)        usage
                            exit
                            ;;
        *)                  usage >&2
                            exit 1
                            ;;
    esac
    shift   # avanzar al siguiente argumento
done
```

**Cómo funciona:**
- El bucle continúa mientras `$1` no esté vacío
- `case` evalúa la opción actual en `$1`
- Soporta tanto formas cortas (`-f`) como largas (`--file`) con `|`
- La opción `-f` requiere un argumento: se llama `shift` **extra** dentro del case para mover el filename a `$1`, se guarda, y luego el `shift` final del bucle avanza más allá del filename
- `*)` captura opciones desconocidas y muestra el mensaje de uso

### La Opción `getopts` (Builtin)

El builtin **`getopts`** ofrece una alternativa más compacta para opciones de un solo carácter:

**Sintaxis:**
```bash
getopts optstring var [arg ...]
```

- **`optstring`**: string con las letras de opción válidas. Si una letra va seguida de `:`, esa opción requiere un argumento.
- Si `optstring` empieza con `:`, se silencian los mensajes de error propios de `getopts`
- **`var`**: variable que recibirá la opción actual en cada iteración
- `getopts` retorna exit status exitoso hasta que se agotan los argumentos
- La opción actual se almacena en `var`; su argumento (si aplica) en `$OPTARG`
- `OPTIND` se incrementa para apuntar al siguiente parámetro posicional
- `getopts` retorna `?` ante opción inválida y `:` ante argumento faltante; en ambos casos `$OPTARG` contiene la letra de la opción problemática

Ejemplo:

```bash
#!/bin/bash
# getopts-test: procesar opciones con getopts

PROGNAME="$(basename "$0")"
interactive=
filename=

usage () {
    echo "$PROGNAME: usage: $PROGNAME [-f file | -i]"
    return
}

while getopts :f:ih opt; do
    case "$opt" in
        f)  filename="$OPTARG" ;;
        i)  interactive=1 ;;
        h)  usage ;;
        \?) echo "option '$OPTARG' invalid" ;;
        :)  echo "option '$OPTARG' missing argument";;
    esac
done

echo "interactive = '$interactive' filename = '$filename'"
```

Ejecución de prueba:

```
$ getopts-test              → interactive = '' filename = ''
$ getopts-test -i           → interactive = '1' filename = ''
$ getopts-test -f foo.html  → interactive = '' filename = 'foo.html'
$ getopts-test -if foo.html → interactive = '1' filename = 'foo.html'
$ getopts-test -i -f foo.html → interactive = '1' filename = 'foo.html'
$ getopts-test -a           → option 'a' invalid
$ getopts-test -f           → option 'f' missing argument
```

**Ventaja de `getopts`:** soporta la sintaxis de opciones combinadas sin espacio (`-if foo.html`), igual que comandos estándar como `ls -la`.

### Comparación: `while/shift/case` vs `getopts`

| Característica | `while`/`shift`/`case` | `getopts` |
|----------------|------------------------|-----------|
| Opciones largas (`--file`) | ✅ Sí | ❌ No (fácilmente) |
| Opciones combinadas (`-if`) | ❌ No | ✅ Sí |
| Control total del flujo | ✅ Máximo | Limitado |
| Cantidad de código | Más | Menos |

---

## Aplicación Completa: `sys_info_page`

Integrando todo lo anterior, el programa `sys_info_page` (iniciado en el Cap. 25) se extiende con tres opciones de línea de comandos:

| Opción | Descripción |
|--------|-------------|
| `-f file` / `--file file` | Nombre del archivo de salida |
| `-i` / `--interactive` | Modo interactivo: solicita el nombre del archivo y confirma sobreescritura |
| `-h` / `--help` | Muestra mensaje de uso |

### Modo Interactivo

```bash
# Modo interactivo
if [[ -n "$interactive" ]]; then
    while true; do
        read -r -p "Enter name of output file: " filename
        if [[ -e "$filename" ]]; then
            read -r -p "'$filename' exists. Overwrite? [y/n/q] > "
            case "$REPLY" in
                Y|y)    break ;;
                Q|q)    echo "Program terminated."
                        exit ;;
                *)      continue ;;
            esac
        elif [[ -z "$filename" ]]; then
            continue
        else
            break
        fi
    done
fi
```

- Si el archivo ya existe: pregunta si sobreescribir (`Y/y` = aceptar, `Q/q` = terminar, cualquier otra cosa = pedir de nuevo)
- Si el nombre está vacío: volver a pedir
- Si el nombre es nuevo: salir del bucle y continuar

### Salida a Archivo

```bash
write_html_page () {
    cat << _EOF_
<html>
    <head><title>$TITLE</title></head>
    <body>
        <h1>$TITLE</h1>
        <p>$TIMESTAMP</p>
        $(report_uptime)
        $(report_disk_space)
        $(report_home_space)
    </body>
</html>
_EOF_
    return
}

# Lógica de salida
if [[ -n "$filename" ]]; then
    if touch "$filename" && [[ -f "$filename" ]]; then
        write_html_page > "$filename"
    else
        echo "$PROGNAME: Cannot write file '$filename'" >&2
        exit 1
    fi
else
    write_html_page
fi
```

- Se usa `touch` + `-f` para verificar que el archivo sea escribible y sea un archivo regular (no un directorio ni un dispositivo)
- Si `filename` está vacío, la salida va a stdout; si tiene valor, se redirige al archivo

---

## Resumen

| Concepto | Detalle |
|----------|---------|
| `$0` | Nombre del programa (usar `basename "$0"` para el nombre sin ruta) |
| `$1`–`$9` | Argumentos; más allá del 9 usar `${N}` |
| `$#` | Número de argumentos |
| `shift` | Desplaza parámetros: `$2→$1`, `$3→$2`… reduce `$#` en 1 |
| `"$@"` | Forma correcta de expandir todos los argumentos preservando cada uno |
| `"$*"` | Todos los argumentos en un solo string |
| `$FUNCNAME` | Nombre de la función en ejecución |
| `getopts` | Builtin para opciones de un carácter; soporta `-if` combinado |
| `$OPTARG` | Argumento de la opción actual (getopts) |
| `OPTIND` | Índice del siguiente parámetro a procesar (getopts) |

---

## Ver También

- [[wiki/linux/26-top-down-design.md|Capítulo 26: Top-Down Design]] — shell functions y parámetros
- [[wiki/linux/28-lectura-teclado.md|Capítulo 28: Leyendo Input del Teclado]] — `read` e input interactivo
- [[wiki/linux/29-control-flujo-while-until.md|Capítulo 29: Bucles while/until]] — patrón while + shift para procesar opciones
- [[wiki/linux/31-control-flujo-case.md|Capítulo 31: Ramificación con case]] — `case` usado en el procesamiento de opciones
