---
title: "Capítulo 31: Control de Flujo — Ramificación con case"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 31"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-447-452.pdf"
updated: "2026-06-23"
---

# Control de Flujo: Ramificación con `case`

En el Capítulo 28 se construyeron menús usando una serie de sentencias `if` para identificar cuál de las opciones eligió el usuario. Este patrón de decisiones múltiples es tan frecuente en programación que muchos lenguajes (incluido bash) ofrecen un mecanismo especial para él: el comando `case`.

---

## El Comando `case`

En bash, el comando compuesto para decisiones múltiples se llama `case`. Su sintaxis es:

```bash
case word in
    [pattern [| pattern]...) commands ;;]...
esac
```

- **`word`**: el valor que se evalúa (normalmente una variable)
- **`pattern`**: el patrón contra el que se compara `word`
- **`commands`**: los comandos que se ejecutan si hay coincidencia
- **`;;`**: terminador obligatorio de cada bloque de comandos
- **`esac`**: cierra el bloque `case` (`case` deletreado al revés)

Cuando se encuentra una coincidencia, se ejecutan los comandos asociados. Tras la primera coincidencia, **no se intentan más patrones**.

### Comparación: `if` vs `case`

El programa de menú del Capítulo 28 con `if` requería múltiples bloques anidados:

```bash
#!/bin/bash
# read-menu: menú con if

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

El mismo programa reescrito con `case` es más limpio y legible:

```bash
#!/bin/bash
# case-menu: menú con case

clear
echo "
Please Select:

1. Display System Information
2. Display Disk Space
3. Display Home Space Utilization
0. Quit
"
read -r -p "Enter selection [0-3] > "

case "$REPLY" in
    0)  echo "Program terminated."
        exit
        ;;
    1)  echo "Hostname: $HOSTNAME"
        uptime
        ;;
    2)  df -h
        ;;
    3)  if [[ "$(id -u)" -eq 0 ]]; then
            echo "Home Space Utilization (All Users)"
            du -sh /home/*
        else
            echo "Home Space Utilization ($USER)"
            du -sh "$HOME"
        fi
        ;;
    *)  echo "Invalid entry" >&2
        exit 1
        ;;
esac
```

El patrón `*)` al final captura cualquier valor que no haya coincidido antes. Es buena práctica incluirlo siempre como **último patrón** para manejar entradas inválidas.

---

## Patrones

Los patrones usados por `case` son los mismos que los de la **expansión de pathnames** (wildcards). Cada patrón termina con el carácter `)`.

**Tabla de ejemplos de patrones:**

| Patrón | Descripción |
|--------|-------------|
| `a)` | Coincide si `word` es igual a `a` |
| `[[:alpha:]])` | Coincide si `word` es un único carácter alfabético |
| `???)`  | Coincide si `word` tiene exactamente tres caracteres |
| `*.txt)` | Coincide si `word` termina en `.txt` |
| `*)` | Coincide con cualquier valor de `word` (catch-all) |

Ejemplo práctico con varios tipos de patrones:

```bash
#!/bin/bash

read -r -p "enter word > "

case "$REPLY" in
    [[:alpha:]])    echo "is a single alphabetic character." ;;
    [ABC][0-9])     echo "is A, B, or C followed by a digit." ;;
    ???)            echo "is three characters long." ;;
    *.txt)          echo "is a word ending in '.txt'" ;;
    *)              echo "is something else." ;;
esac
```

### Múltiples Patrones con `|` (OR)

Se pueden combinar múltiples patrones usando `|` como separador, creando una condición **OR**. Muy útil para aceptar tanto mayúsculas como minúsculas:

```bash
#!/bin/bash
# case-menu con letras (mayúsculas y minúsculas)

read -r -p "Enter selection [A, B, C or Q] > "

case "$REPLY" in
    q|Q)    echo "Program terminated."
            exit
            ;;
    a|A)    echo "Hostname: $HOSTNAME"
            uptime
            ;;
    b|B)    df -h
            ;;
    c|C)    if [[ "$(id -u)" -eq 0 ]]; then
                echo "Home Space Utilization (All Users)"
                du -sh /home/*
            else
                echo "Home Space Utilization ($USER)"
                du -sh "$HOME"
            fi
            ;;
    *)      echo "Invalid entry" >&2
            exit 1
            ;;
esac
```

---

## Acciones Múltiples (bash 4.0+)

### El Problema con bash < 4.0

En versiones anteriores a bash 4.0, `case` solo podía ejecutar **una acción** por coincidencia y terminaba. Si un valor coincidía con varios patrones (posible con clases POSIX), solo se ejecutaba el primero.

Script de ejemplo (case4-1) que prueba un carácter contra múltiples clases POSIX:

```bash
#!/bin/bash
# case4-1: probar un carácter

read -r -n 1 -p "Type a character > "
echo

case "$REPLY" in
    [[:upper:]])    echo "'$REPLY' is upper case." ;;
    [[:lower:]])    echo "'$REPLY' is lower case." ;;
    [[:alpha:]])    echo "'$REPLY' is alphabetic." ;;
    [[:digit:]])    echo "'$REPLY' is a digit." ;;
    [[:graph:]])    echo "'$REPLY' is a visible character." ;;
    [[:punct:]])    echo "'$REPLY' is a punctuation symbol." ;;
    [[:space:]])    echo "'$REPLY' is a whitespace character." ;;
    [[:xdigit:]]) echo "'$REPLY' is a hexadecimal digit." ;;
esac
```

Resultado al ingresar `a`:

```
[me@linuxbox ~]$ case4-1
Type a character > a
'a' is lower case.
```

El script falla para caracteres que pertenecen a múltiples clases: `a` es minúscula, alfabética, carácter visible **y** dígito hexadecimal, pero solo reporta la primera coincidencia.

### La Solución: `;;&` (bash 4.0+)

Las versiones modernas de bash agregan la notación **`;;&`** para terminar cada acción y **continuar evaluando el siguiente patrón** (en lugar de terminar como lo hace `;;`):

```bash
#!/bin/bash
# case4-2: probar un carácter (con ;;& para múltiples coincidencias)

read -r -n 1 -p "Type a character > "
echo

case "$REPLY" in
    [[:upper:]])    echo "'$REPLY' is upper case." ;;&
    [[:lower:]])    echo "'$REPLY' is lower case." ;;&
    [[:alpha:]])    echo "'$REPLY' is alphabetic." ;;&
    [[:digit:]])    echo "'$REPLY' is a digit." ;;&
    [[:graph:]])    echo "'$REPLY' is a visible character." ;;&
    [[:punct:]])    echo "'$REPLY' is a punctuation symbol." ;;&
    [[:space:]])    echo "'$REPLY' is a whitespace character." ;;&
    [[:xdigit:]]) echo "'$REPLY' is a hexadecimal digit." ;;&
esac
```

Resultado al ingresar `a`:

```
[me@linuxbox ~]$ case4-2
Type a character > a
'a' is lower case.
'a' is alphabetic.
'a' is a visible character.
'a' is a hexadecimal digit.
```

Ahora se reportan **todas** las categorías a las que pertenece el carácter.

### Comparativa de terminadores

| Terminador | Comportamiento |
|------------|----------------|
| `;;` | Termina tras ejecutar la acción (comportamiento clásico) |
| `;;&` | Ejecuta la acción y **continúa evaluando** los siguientes patrones (bash 4.0+) |

---

## Resumen

El comando `case` es una alternativa más limpia y legible a cadenas de `if/elif` cuando se necesita comparar una variable contra múltiples valores posibles. Puntos clave:

- La sintaxis es `case word in ... esac`
- Los patrones usan las mismas reglas que los wildcards de expansión de pathnames
- `|` entre patrones crea condición OR (útil para mayúsculas/minúsculas)
- `*)` como último patrón captura cualquier valor no coincidido (catch-all)
- `;;` termina cada bloque; `;;&` (bash 4.0+) permite continuar evaluando más patrones

---

## Ver También

- [[wiki/linux/27-control-flujo-if.md|Capítulo 27: Control de Flujo con if]] — alternativa con if/elif/else
- [[wiki/linux/28-lectura-teclado.md|Capítulo 28: Leyendo Input del Teclado]] — read y menús interactivos
- [[wiki/linux/29-control-flujo-while-until.md|Capítulo 29: Bucles while/until]] — combinar case con bucles para menús persistentes
