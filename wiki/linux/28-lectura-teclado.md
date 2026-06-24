---
title: "Capítulo 28: Leyendo Input del Teclado"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 28"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-411-421.pdf"
updated: "2026-06-23"
---

# Leyendo Input del Teclado

Los scripts escritos hasta ahora carecen de **interactividad**: la capacidad del programa de interactuar con el usuario. Muchos programas no necesitan ser interactivos, pero algunos se benefician enormemente de poder aceptar input directamente del usuario en tiempo de ejecución — en lugar de requerir editar el script cada vez que cambia un valor.

## `read` — Leer Valores desde Standard Input

El builtin `read` lee una línea de standard input. Puede usarse para leer input del teclado o, cuando se emplea redirección, leer una línea desde un archivo. Sintaxis:

```bash
read [-options] [variable...]
```

- `variable` es el nombre de una o más variables que almacenarán el input.
- Si **no** se especifica ninguna variable, el input completo se asigna a la variable especial **`REPLY`**.

### Ejemplo básico — leer un entero interactivamente

Versión anterior (hardcodeada):
```bash
INT=-5
```

Versión interactiva con `read`:
```bash
#!/bin/bash

# read-integer: evaluate the value of an integer.

echo -n "Please enter an integer -> "
read int

if [[ "$int" =~ ^-?[0-9]+$ ]]; then
    if [ "$int" -eq 0 ]; then
        echo "$int is zero."
    else
        if [ "$int" -lt 0 ]; then
            echo "$int is negative."
        else
            echo "$int is positive."
        fi
        if [ $((int % 2)) -eq 0 ]; then
            echo "$int is even."
        else
            echo "$int is odd."
        fi
    fi
else
    echo "Input value is not an integer." >&2
    exit 1
fi
```

`echo -n` suprime el salto de línea final del prompt, dejando el cursor en la misma línea.

Ejecución:
```
[me@linuxbox ~]$ read-integer
Please enter an integer -> 5
5 is positive.
5 is odd.
```

### Asignación a múltiples variables

`read` puede asignar input a múltiples variables simultáneamente:

```bash
#!/bin/bash

# read-multiple: read multiple values from keyboard

echo -n "Enter one or more values > "
read var1 var2 var3 var4 var5

echo "var1 = '$var1'"
echo "var2 = '$var2'"
echo "var3 = '$var3'"
echo "var4 = '$var4'"
echo "var5 = '$var5'"
```

Comportamiento según la cantidad de palabras ingresadas:

| Input del usuario | Resultado |
|-------------------|-----------|
| `a b c d e` | var1='a', var2='b', var3='c', var4='d', var5='e' |
| `a` | var1='a', var2-var5 vacías |
| `a b c d e f g` | var1='a', var2='b', var3='c', var4='d', var5='**e f g**' |

> Si `read` recibe **menos** palabras de las esperadas, las variables extras quedan vacías. Si recibe **más**, el exceso se asigna completamente a la **última variable**.

### Variable `REPLY` — sin especificar variables

```bash
#!/bin/bash

# read-single: read multiple values into default variable

echo -n "Enter one or more values > "
read

echo "REPLY = '$REPLY'"
```

```
[me@linuxbox ~]$ read-single
Enter one or more values > a b c d
REPLY = 'a b c d'
```

## Opciones de `read`

| Opción | Descripción |
|--------|-------------|
| `-a array` | Asigna el input a un *array* comenzando desde el índice cero (ver Capítulo 35) |
| `-d delimiter` | El primer carácter de *delimiter* indica el fin del input, en lugar de newline |
| `-e` | Usa Readline para el input; permite edición de línea igual que en la línea de comandos |
| `-i string` | Usa *string* como respuesta por defecto si el usuario presiona ENTER (requiere `-e`) |
| `-n num` | Lee solo *num* caracteres de input, no una línea completa |
| `-p prompt` | Muestra *prompt* como mensaje antes de aceptar input |
| `-r` | **Raw mode**: no interpreta los backslashes como caracteres de escape. *Recomendado para seguridad* — por ejemplo, al pedir rutas de Windows, los backslashes se tratan como caracteres literales |
| `-s` | **Silent mode**: no muestra los caracteres mientras se escriben. Útil para contraseñas e información confidencial |
| `-t seconds` | **Timeout**: termina el input después de *seconds* segundos. `read` retorna exit status no-cero si el tiempo expira |
| `-u fd` | Usa input del descriptor de archivo *fd* en lugar de standard input |

### Ejemplos de opciones combinadas

**Prompt integrado con `-p`:**

```bash
read -r -p "Enter one or more values > "
echo "REPLY = '$REPLY'"
```

**Contraseña con timeout (`-r -t -s -p`):**

```bash
#!/bin/bash

# read-secret: input a secret passphrase

if read -r -t 10 -sp "Enter secret passphrase > " secret_pass; then
    echo -e "\nSecret passphrase = '$secret_pass'"
else
    echo -e "\nInput timed out" >&2
    exit 1
fi
```

El script pide una frase secreta y espera 10 segundos. Si no se ingresa a tiempo, sale con error. Los caracteres no se muestran en pantalla gracias a `-s`.

**Valor por defecto con `-e` y `-i`:**

```bash
#!/bin/bash

# read-default: supply a default value if user presses Enter key.

read -e -p "What is your user name? " -i $USER
echo "You answered: '$REPLY'"
```

```
[me@linuxbox ~]$ read-default
What is your user name? me
You answered: 'me'
```

`-i $USER` pre-rellena el campo con el nombre de usuario actual. Si el usuario presiona ENTER sin modificar, se usa ese valor por defecto.

## IFS — Internal Field Separator

Normalmente, el shell realiza **word splitting** sobre el input de `read`: múltiples palabras separadas por uno o más espacios se convierten en ítems separados asignados a distintas variables.

Este comportamiento está controlado por la variable **`IFS`** (*Internal Field Separator*). Su valor por defecto contiene:
- Un espacio
- Un tab
- Un carácter de newline

Cada uno de estos actúa como separador de campos.

### Cambiar IFS para parsear archivos delimitados

El archivo `/etc/passwd` usa `:` como separador. Cambiando `IFS` podemos leer sus campos directamente:

```bash
#!/bin/bash

# read-ifs: read fields from a file

FILE=/etc/passwd

read -r -p "Enter a username > " user_name

file_info="$(grep "^$user_name:" $FILE)"

if [ -n "$file_info" ]; then
    IFS=":" read -r user pw uid gid name home shell <<< "$file_info"
    echo "User =      '$user'"
    echo "UID =       '$uid'"
    echo "GID =       '$gid'"
    echo "Full Name = '$name'"
    echo "Home Dir. = '$home'"
    echo "Shell =     '$shell'"
else
    echo "No such user '$user_name'" >&2
    exit 1
fi
```

La línea clave tiene tres partes:

```bash
IFS=":" read -r user pw uid gid name home shell <<< "$file_info"
```

1. **`IFS=":"`** — asignación de variable **antes del comando**: el shell permite que una o más asignaciones de variables precedan inmediatamente a un comando. Esto altera el environment **solo para ese comando** (efecto temporal). Equivale a:

```bash
OLD_IFS="$IFS"
IFS=":"
read -r user pw uid gid name home shell <<< "$file_info"
IFS="$OLD_IFS"
```

2. **`read -r user pw uid gid name home shell`** — lee 8 campos y los asigna a las variables listadas.

3. **`<<< "$file_info"`** — operador **here string**: como un here document pero de una sola línea/string. Alimenta el contenido de `$file_info` al stdin de `read`.

### ⚠️ No puedes pipear read

```bash
echo "foo" | read    # No funciona como se esperaría
```

Esto parece funcionar, pero `REPLY` siempre estará vacío. **¿Por qué?**

Las **pipelines** en bash (y sh) crean **subshells**: copias del shell y su environment para ejecutar el comando dentro del pipeline. Cuando el proceso termina, la copia del environment se destruye.

> **Un subshell nunca puede alterar el environment de su proceso padre.**

`read` asigna variables al environment del subshell, pero cuando el subshell termina, esa asignación se pierde. El here string (`<<<`) es una solución: alimenta el input sin crear subshell. Otra solución se discute en el Capítulo 36.

## Validando Input

Con interactividad surge el desafío de **validar el input**. La diferencia entre un programa bien escrito y uno malo frecuentemente radica en la capacidad de manejar lo inesperado — especialmente input inválido.

Ejemplo completo de script que valida múltiples tipos de input:

```bash
#!/bin/bash

# read-validate: validate input

invalid_input () {
    echo "Invalid input '$REPLY'" >&2
    exit 1
}

read -r -p "Enter a single item > "

# input is empty (invalid)
[[ -z "$REPLY" ]] && invalid_input

# input is multiple items (invalid)
(( "$(echo "$REPLY" | wc -w)" > 1 )) && invalid_input

# is input a valid filename?
if [[ "$REPLY" =~ ^[-[:alnum:]\._]+$ ]]; then
    echo "'$REPLY' is a valid filename."
    if [[ -e "$REPLY" ]]; then
        echo "And file '$REPLY' exists."
    else
        echo "However, file '$REPLY' does not exist."
    fi

    # is input a floating point number?
    if [[ "$REPLY" =~ ^-?[[:digit:]]*\.[[:digit:]]+$ ]]; then
        echo "'$REPLY' is a floating point number."
    else
        echo "'$REPLY' is not a floating point number."
    fi

    # is input an integer?
    if [[ "$REPLY" =~ ^-?[[:digit:]]+$ ]]; then
        echo "'$REPLY' is an integer."
    else
        echo "'$REPLY' is not an integer."
    fi
else
    echo "The string '$REPLY' is not a valid filename."
fi
```

Análisis de las validaciones:

| Verificación | Técnica usada |
|-------------|---------------|
| Input vacío | `[[ -z "$REPLY" ]]` |
| Múltiples palabras | `(( "$(echo "$REPLY" \| wc -w)" > 1 ))` — cuenta palabras con `wc -w` |
| Nombre de archivo válido | `=~ ^[-[:alnum:]\._]+$` — solo alfanuméricos, guión, punto, underscore |
| Número flotante | `=~ ^-?[[:digit:]]*\.[[:digit:]]+$` — dígitos opcionales, punto obligatorio, más dígitos |
| Entero | `=~ ^-?[[:digit:]]+$` — signo opcional, uno o más dígitos |

Este script combina shell functions, `[[ ]]`, `(( ))`, el operador de control `&&`, `if`, y una dosis sana de expresiones regulares.

## Menús Interactivos

Un tipo común de interactividad son los programas **menu-driven**: se presenta al usuario una lista de opciones numeradas y se pide elegir una.

**Script completo de menú de información del sistema:**

```bash
#!/bin/bash

# read-menu: a menu driven system information program

clear
echo "
Please Select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit
"
read -r -p "Enter selection [0-3] > "

if [[ "$REPLY" =~ ^[0-3]$ ]]; then
    if [[ "$REPLY" == 0 ]]; then
        echo "Program terminated."
        exit
    fi
    if [[ "$REPLY" == 1 ]]; then
        echo "Hostname: $HOSTNAME"
        uptime
        exit
    fi
    if [[ "$REPLY" == 2 ]]; then
        df -h
        exit
    fi
    if [[ "$REPLY" == 3 ]]; then
        if [[ "$(id -u)" -eq 0 ]]; then
            echo "Home Space Utilization (All Users)"
            du -sh /home/*
        else
            echo "Home Space Utilization ($USER)"
            du -sh "$HOME"
        fi
        exit
    fi
else
    echo "Invalid entry." >&2
    exit 1
fi
```

**Estructura del script:**
- **Primera parte**: muestra el menú y captura la respuesta del usuario.
- **Segunda parte**: identifica la respuesta y ejecuta la acción seleccionada.

La validación inicial `[[ "$REPLY" =~ ^[0-3]$ ]]` asegura que solo se acepten los dígitos 0, 1, 2 o 3.

> **Nota sobre múltiples `exit`:** Tener múltiples puntos de salida en un programa es generalmente mala práctica (dificulta entender la lógica), pero funciona en este script sencillo. En el capítulo siguiente se mejorará este diseño.

## Resumen

| Elemento | Descripción |
|----------|-------------|
| `read` | Builtin que lee una línea de stdin |
| `REPLY` | Variable especial: recibe todo el input si no se especifican variables |
| `IFS` | Internal Field Separator; controla cómo se divide el input en campos |
| `<<<` | Here string; alimenta un string como stdin sin crear subshell |
| `-p prompt` | Muestra un prompt antes de leer |
| `-r` | Raw mode; backslashes son literales (recomendado) |
| `-s` | Silent; no muestra caracteres al escribir (para contraseñas) |
| `-t N` | Timeout de N segundos |
| `-e` / `-i string` | Readline + valor por defecto pre-rellenado |
| Asignación temporal | `VAR=value command` — cambia el environment solo para ese comando |
| Subshell | Las pipelines crean subshells; las variables asignadas en subshells **no** persisten en el padre |

---

## Ver También

- [[27-control-flujo-if.md|Capítulo 27: Control de Flujo con if]] — `if`, `[[ ]]`, `(( ))` usados extensamente aquí
- [[19-expresiones-regulares.md|Capítulo 19: Expresiones Regulares]] — Regex usadas para validar input
- [[06-redirection.md|Capítulo 6: Redirección]] — Contexto de stdin/stdout y here documents
- [[25-iniciando-un-proyecto.md|Capítulo 25: Iniciando un Proyecto]] — Here documents (`<<`) que anteceden a here strings (`<<<`)
