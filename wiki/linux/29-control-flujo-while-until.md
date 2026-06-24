---
title: "Capítulo 29: Control de Flujo – Bucles con while/until"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 29"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-423-431.pdf"
updated: "2026-06-23"
---

# Control de Flujo: Bucles con while/until

El programa de menú del capítulo anterior (Ch. 28) tenía un problema de usabilidad: ejecutaba una sola acción y terminaba. Si el usuario ingresaba una opción inválida, el programa terminaba con error sin darle la oportunidad de intentar de nuevo.

La solución es el concepto de **looping** (bucle): hacer que porciones del programa se repitan. Bash provee tres comandos compuestos para bucles; en este capítulo se cubren `while` y `until`. El tercero (`for`) se cubre en un capítulo posterior.

## Concepto de Looping

La vida cotidiana está llena de actividades repetidas. Ejemplo en pseudocódigo (rebanar una zanahoria):

```
1. Tomar tabla de cortar.
2. Tomar cuchillo.
3. Colocar zanahoria en la tabla.
4. Levantar cuchillo.       ← inicio del bucle
5. Avanzar zanahoria.
6. Cortar zanahoria.
7. Si toda la zanahoria fue cortada, terminar; sino ir al paso 4.
```

Los pasos 4 a 7 forman un **loop**: las acciones se repiten hasta que se cumple la condición de salida.

## `while` — Bucle Mientras

Bash expresa esta idea con el comando `while`. Sintaxis:

```bash
while commands; do
    commands
done
```

Como `if`, `while` evalúa el **exit status** de una lista de comandos. Mientras el exit status sea cero (éxito), ejecuta los comandos dentro del bucle.

### Ejemplo básico — contar del 1 al 5

```bash
#!/bin/bash

# while-count: display a series of numbers

count=1

while [[ "$count" -le 5 ]]; do
    echo "$count"
    count=$((count + 1))
done
echo "Finished."
```

```
[me@linuxbox ~]$ while-count
1
2
3
4
5
Finished.
```

**Cómo funciona:**
1. `count` se inicializa en 1.
2. `while` evalúa `[[ "$count" -le 5 ]]` — mientras count ≤ 5, ejecuta el cuerpo.
3. Cada iteración imprime `count` y lo incrementa con `count=$((count + 1))`.
4. Cuando `count` llega a 6, `[[ ]]` retorna exit status 1 y el bucle termina.
5. La ejecución continúa con el `echo "Finished."` que sigue al `done`.

### Mejorando el menú con `while`

Aplicando `while` al programa de menú del capítulo anterior, podemos hacer que el menú se repita tras cada selección:

```bash
#!/bin/bash

# while-menu: a menu driven system information program

DELAY=3  # Number of seconds to display results

while [[ "$REPLY" != 0 ]]; do
    clear
    cat << _EOF_
        Please Select:

        1. Display System Information
        2. Display Disk Space
        3. Display Home Space Utilization
        0. Quit
_EOF_
    read -r -p "Enter selection [0-3] > "

    if [[ "$REPLY" =~ ^[0-3]$ ]]; then
        if [[ "$REPLY" == 1 ]]; then
            echo "Hostname: $HOSTNAME"
            uptime
            sleep "$DELAY"
        fi
        if [[ "$REPLY" == 2 ]]; then
            df -h
            sleep "$DELAY"
        fi
        if [[ "$REPLY" == 3 ]]; then
            if [[ "$(id -u)" -eq 0 ]]; then
                echo "Home Space Utilization (All Users)"
                du -sh /home/*
            else
                echo "Home Space Utilization ($USER)"
                du -sh "$HOME"
            fi
            sleep "$DELAY"
        fi
    else
        echo "Invalid entry."
        sleep "$DELAY"
    fi
done
echo "Program terminated."
```

**Mejoras sobre la versión anterior:**
- El menú se **redisplaya automáticamente** después de cada acción.
- Tras mostrar los resultados, `sleep "$DELAY"` pausa 3 segundos para que el usuario los vea antes de limpiar la pantalla.
- La condición `[[ "$REPLY" != 0 ]]` mantiene el loop activo hasta que el usuario elige "Quit" (0).

## `break` y `continue`

Bash provee dos builtins para controlar el flujo **dentro** de los bucles:

| Comando | Efecto |
|---------|--------|
| `break` | Termina el bucle **inmediatamente** y continúa con la instrucción que sigue a `done` |
| `continue` | Salta el **resto de la iteración actual** y reanuda con la siguiente iteración del bucle |

### Bucle infinito con `while true`

Una técnica muy común es crear un **endless loop** (bucle infinito) usando `true` como condición:

```bash
while true; do
    commands
done
```

`true` siempre retorna exit status 0, por lo que el bucle nunca termina por sí solo. Es responsabilidad del programador proveer una salida mediante `break`.

### Versión mejorada con `break` y `continue`

```bash
#!/bin/bash

# while-menu2: a menu driven system information program

DELAY=3

while true; do
    clear
    cat << _EOF_
        Please Select:

        1. Display System Information
        2. Display Disk Space
        3. Display Home Space Utilization
        0. Quit
_EOF_
    read -p "Enter selection [0-3] > "

    if [[ "$REPLY" =~ ^[0-3]$ ]]; then
        if [[ "$REPLY" == 1 ]]; then
            echo "Hostname: $HOSTNAME"
            uptime
            sleep "$DELAY"
            continue
        fi
        if [[ "$REPLY" == 2 ]]; then
            df -h
            sleep "$DELAY"
            continue
        fi
        if [[ "$REPLY" == 3 ]]; then
            if [[ "$(id -u)" -eq 0 ]]; then
                echo "Home Space Utilization (All Users)"
                du -sh /home/*
            else
                echo "Home Space Utilization ($USER)"
                du -sh "$HOME"
            fi
            sleep "$DELAY"
            continue
        fi
        if [[ "$REPLY" == 0 ]]; then
            break
        fi
    else
        echo "Invalid entry."
        sleep "$DELAY"
    fi
done
echo "Program terminated."
```

**Por qué `continue` mejora la eficiencia:** Una vez que se identifica y ejecuta la opción 1, el `continue` hace que el script salte directamente a la siguiente iteración, sin necesidad de evaluar las condiciones 2 y 3. Esto es más eficiente y hace la lógica más clara.

**`break` para salida limpia:** Cuando el usuario elige 0, `break` sale del bucle infinito y la ejecución continúa con `echo "Program terminated."`.

## `select` — Menús en un Bucle

`select` es un builtin de bash diseñado específicamente para crear menús interactivos en bucle. Sintaxis:

```bash
select var in [string...]; do
    commands
done
```

**Cómo funciona `select`:**
1. Muestra los `string`s numerados, usando el contenido de la variable **`PS3`** como prompt.
2. El usuario ingresa un número.
3. `select` asigna a `REPLY` lo que el usuario escribió, y a `var` el string correspondiente.
4. Ejecuta los `commands`.
5. Vuelve a mostrar el menú automáticamente.
6. Continúa hasta encontrar `break` o recibir Ctrl-D (EOF).

### Variables de `select`

| Variable | Contenido |
|----------|-----------|
| `REPLY` | Lo que el usuario escribió literalmente |
| `var` (el nombre que elijas) | El string de la lista correspondiente al número elegido |
| `PS3` | Prompt del menú `select` (por defecto: `#?`) |

### Demostración básica

```bash
#!/bin/bash

# select-demo: select builtin demo

PS3="Your choice: "

select my_choice in First Second Third Fourth Quit; do
    echo "REPLY=$REPLY  my_choice=$my_choice"
    [[ "$my_choice" == "Quit" ]] && break
done
```

```
[me@linuxbox ~]$ select-demo
1) First
2) Second
3) Third
4) Fourth
5) Quit
Your choice: 1
REPLY=1  my_choice=First

Your choice: 2
REPLY=2  my_choice=Second

Your choice: 6
REPLY=6  my_choice=

Your choice: abc
REPLY=abc  my_choice=

Your choice: 5
REPLY=5  my_choice=Quit
[me@linuxbox ~]$
```

**Comportamientos importantes:**
- Input **inválido** (número fuera de rango, texto): `var` queda vacía, `REPLY` contiene lo escrito.
- Presionar **ENTER** sin input: redisplaya el menú.
- El bucle termina con `break` o **Ctrl-D**.

> **Característica especial:** `select` usa **stderr** para mostrar el menú y el prompt, no stdout. Esto significa que si redirigimos stdout, el menú sigue siendo visible pero el output de los comandos internos va al archivo/pipe:

```bash
[me@linuxbox ~]$ select-demo > choices.txt
1) First
...
Your choice:
```

### Programa de menú con `select`

```bash
#!/bin/bash

# select-menu: a menu driven system information program

DELAY=3
PS3="Enter selection [1-4] > "

select str in \
    "Display System Information" \
    "Display Disk Space" \
    "Display Home Space Utilization" \
    "Quit"; do
    if [[ "$REPLY" == "1" ]]; then
        echo "Hostname: $HOSTNAME"
        uptime
        sleep "$DELAY"
        continue
    fi
    if [[ "$REPLY" == "2" ]]; then
        df -h
        sleep "$DELAY"
        continue
    fi
    if [[ "$REPLY" == "3" ]]; then
        if [[ "$(id -u)" -eq 0 ]]; then
            echo "Home Space Utilization (All Users)"
            du -sh /home/* 2>/dev/null
        else
            echo "Home Space Utilization ($USER)"
            du -sh "$HOME" 2>/dev/null
        fi
        sleep "$DELAY"
        continue
    fi
    if [[ "$REPLY" == "4" ]]; then
        break
    fi
    if [[ -z "$str" ]]; then
        echo "Invalid entry."
        sleep "$DELAY"
    fi
done
echo "Program terminated."
```

> **`select` vs `while` para menús:** `select` es interesante, pero además de usar stderr para el menú, no ahorra mucho esfuerzo de codificación y limita bastante el diseño visual del menú. Para menus con diseño personalizado, `while` con `read` suele ser más flexible.

## `until` — Bucle Hasta

`until` es el opuesto lógico de `while`. Continúa el bucle mientras el exit status sea **no-cero** (fallo), y termina cuando recibe un exit status de **cero** (éxito):

```bash
until commands; do
    commands
done
```

| Comando | Continúa cuando | Termina cuando |
|---------|----------------|----------------|
| `while` | exit status = 0 (verdadero) | exit status ≠ 0 (falso) |
| `until` | exit status ≠ 0 (falso) | exit status = 0 (verdadero) |

### Ejemplo — contar del 1 al 5 con `until`

```bash
#!/bin/bash

# until-count: display a series of numbers

count=1

until [[ "$count" -gt 5 ]]; do
    echo "$count"
    count=$((count + 1))
done
echo "Finished."
```

La condición `$count -gt 5` (mayor que 5) es la **inversa lógica** de la condición `while` (`-le 5`). El resultado es idéntico.

> **¿Cuándo usar `while` vs `until`?** Normalmente es cuestión de cuál permite escribir la prueba más clara. Elige la que exprese mejor la intención del código.

## Leyendo Archivos con Bucles

`while` y `until` pueden procesar **standard input**, lo que permite leer archivos línea por línea con un bucle.

### Redirección de archivo al bucle

```bash
#!/bin/bash

# while-read: read lines from a file

while read -r distro version release; do
    printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
        "$distro" \
        "$version" \
        "$release"
done < distros.txt
```

**Clave:** El operador de redirección `< distros.txt` se coloca **después del `done`**, no del `while`. Esto redirige el archivo al stdin del bucle completo.

En cada iteración:
- `read` lee una línea del archivo y la divide en campos (`distro`, `version`, `release`).
- Cuando `read` llega al final del archivo retorna exit status no-cero, terminando el bucle.

### Piping al bucle

También se puede enviar stdout de otro comando al bucle:

```bash
#!/bin/bash

# while-read2: read lines from a file

sort -k 1,1 -k 2n distros.txt | while read -r distro version release; do
    printf "Distro: %s\tVersion: %s\tReleased: %s\n" \
        "$distro" \
        "$version" \
        "$release"
done
```

> **⚠️ Advertencia de subshell:** Cuando se usa un pipe (`|`), el bucle `while` se ejecuta en un **subshell**. Cualquier variable creada o modificada dentro del bucle se **perderá** cuando el bucle termine. Si necesitas preservar variables del bucle, usa redirección de archivo (`done < file`) en lugar de pipes.

## Resumen

| Comando | Descripción |
|---------|-------------|
| `while commands; do ... done` | Repite mientras el exit status sea 0 (verdadero) |
| `until commands; do ... done` | Repite mientras el exit status sea ≠ 0 (falso) |
| `while true; do ... done` | Bucle infinito (requiere `break` interno para salir) |
| `break` | Sale del bucle inmediatamente |
| `continue` | Salta al inicio de la siguiente iteración |
| `select var in strings; do ... done` | Menú interactivo en bucle, usa PS3 como prompt |
| `done < file` | Redirige un archivo como stdin del bucle completo |
| `cmd \| while read ...; do` | Piping al bucle (¡crea subshell, variables no persisten!) |
| `sleep N` | Pausa N segundos (útil en menús para mostrar resultados) |

Con `while`, `until` y los comandos de ramificación del capítulo anterior (`if`), se cubren los tipos principales de control de flujo en programación bash.

---

## Ver También

- [[27-control-flujo-if.md|Capítulo 27: Control de Flujo con if]] — Ramificación, `[[ ]]`, exit status
- [[28-lectura-teclado.md|Capítulo 28: Leyendo Input del Teclado]] — `read`, menús interactivos, subshells y pipes
- [[06-redirection.md|Capítulo 6: Redirección]] — Contexto de stdin, redirección `<`, pipelines y subshells
