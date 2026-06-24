---
title: "Capítulo 33: Control de Flujo — Bucles con for"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 33"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-469-474.pdf"
updated: "2026-06-23"
---

# Control de Flujo: Bucles con `for`

El comando `for` es otra construcción de bucle del shell. A diferencia de `while` y `until`, `for` provee un medio conveniente de procesar **secuencias** durante un bucle, haciendo que sea especialmente útil en bash.

Bash ofrece dos formas del comando `for`: la forma tradicional del shell y la forma similar a C.

---

## `for` — Forma Tradicional del Shell

**Sintaxis:**
```bash
for variable [in words]; do
    commands
done
```

- **`variable`**: nombre de la variable que se asignará en cada iteración
- **`[in words]`**: lista opcional de items (si se omite, usa `$@` — parámetros posicionales)
- **`commands`**: comandos ejecutados en cada iteración del bucle
- **`done`**: cierra el bucle

### Ejemplos Básicos

**Ejemplo 1: Lista explícita**

```bash
[me@linuxbox ~]$ for i in A B C D; do echo $i; done
A
B
C
D
```

En cada iteración, `i` toma uno de los valores de la lista (A, B, C, D) y se ejecuta `echo $i`.

**Ejemplo 2: Brace Expansion**

Una forma poderosa de crear la lista es usar **brace expansion**:

```bash
[me@linuxbox ~]$ for i in {A..D}; do echo $i; done
A
B
C
D
```

O con números:

```bash
for i in {1..5}; do echo $i; done
# Outputea: 1 2 3 4 5
```

**Ejemplo 3: Pathname Expansion**

Se puede usar expansión de pathnames (wildcards) para procesar archivos:

```bash
[me@linuxbox ~]$ for i in distros*.txt; do echo $i; done
distros-by-date.txt
distros-dates.txt
distros-key-names.txt
distros-key-vernums.txt
distros-names.txt
distros.txt
distros-vernums.txt
distros-versions.txt
```

**Precaución:** Si la expansión no coincide con nada, los wildcards se devuelven literalmente. Para evitar esto:

```bash
for i in distros*.txt; do
    if [[ -e "$i" ]]; then
        echo "$i"
    fi
done
```

**Ejemplo 4: Command Substitution**

Se puede usar sustitución de comandos para generar la lista:

```bash
#!/bin/bash
# longest-word: encontrar la cadena más larga en un archivo

while [[ -n "$1" ]]; do
    if [[ -r "$1" ]]; then
        max_word=
        max_len=0
        for i in $(strings "$i"); do
            len="$(echo -n "$i" | wc -c)"
            if (( len > max_len )); then
                max_len="$len"
                max_word="$i"
            fi
        done
        echo "$i: '$max_word' ($max_len characters)"
    fi
    shift
done
```

En este ejemplo, `$(strings "$i")` genera una lista de palabras legibles en el archivo, que el bucle `for` procesa una por una.

### Omitiendo `in words` — Usar Parámetros Posicionales

Si se omite la parte `in words`, el bucle por defecto procesa los parámetros posicionales (`$1`, `$2`, etc.):

```bash
#!/bin/bash
# longest-word2: encontrar la cadena más larga en un archivo (versión con parámetros)

for i; do
    if [[ -r "$i" ]]; then
        max_word=
        max_len=0
        for j in $(strings "$i"); do
            len="$(echo -n "$j" | wc -c)"
            if (( len > max_len )); then
                max_len="$len"
                max_word="$j"
            fi
        done
        echo "$i: '$max_word' ($max_len characters)"
    fi
done
```

Aquí, el bucle itera sobre `$@` implícitamente. También nota que se cambió la variable interna a `j` para evitar conflictos.

### ¿Por Qué `i`?

La variable `i` es la más común en bucles `for` por razones históricas. En **Fortran**, las variables que comenzaban con las letras I, J, K, L, M eran automáticamente tipadas como enteros, mientras que las que comenzaban con cualquier otra letra eran reales (números decimales). Esto llevó a que los programadores usaran I, J, K para variables de bucle. Cuando bash fue diseñado, siguió esta convención, de ahí el dicho: **"GOD is real, unless declared integer"**.

---

## `for` — Forma Similar a C

Las versiones recientes de bash han agregado una segunda forma del comando `for` que se parece a la sintaxis del lenguaje C:

**Sintaxis:**
```bash
for (( expression1; expression2; expression3 )); do
    commands
done
```

- **`expression1`**: se usa para inicializar condiciones del bucle
- **`expression2`**: se usa para determinar cuándo termina el bucle
- **`expression3`**: se ejecuta al final de cada iteración del bucle

Esta forma es equivalente a:

```bash
(( expression1 ))
while (( expression2 )); do
    commands
    (( expression3 ))
done
```

### Ejemplo: Contador Simple

```bash
#!/bin/bash
# simple_counter: demo de forma C style para comando for

for (( i=0; i<5; i=i+1 )); do
    echo $i
done
```

Ejecución:

```
[me@linuxbox ~]$ simple_counter
0
1
2
3
4
```

En este ejemplo:
- **`i=0`** inicializa `i` a cero
- **`i<5`** es la condición de continuación (mientras `i` sea menor que 5)
- **`i=i+1`** incrementa `i` en 1 en cada iteración

**Ventaja:** La forma C-style es útil cuando se necesita una **secuencia numérica**. Sin embargo, cualquier expresión aritmética es válida, no solo contadores simples.

---

## Aplicación: Mejora de `sys_info_page`

### Función Mejorada: `report_home_space()`

Con el conocimiento del bucle `for`, ahora podemos mejorar significativamente el script `sys_info_page`. Aquí está la versión mejorada de la función `report_home_space()`:

```bash
report_home_space () {
    local format="%s%10s%10s\n"
    local i dir_list total_files total_dirs total_size user_name
    
    if [[ "$(id -u)" -eq 0 ]]; then
        dir_list="/home/*"
        user_name="All Users"
    else
        dir_list="$HOME"
        user_name="$USER"
    fi

    echo "<h2>Home Space Utilization ($user_name)</h2>"
    
    for i in $dir_list; do
        
        total_files="$(find "$i" -type f | wc -l)"
        total_dirs="$(find "$i" -type d | wc -l)"
        total_size="$(du -sh "$i" | cut -f 1)"
        
        echo "<h3>$i</h3>"
        echo "<pre>"
        printf "$format" "Dirs" "Files" "Size"
        printf "$format" "---" "-----" "----"
        printf "$format" "$total_dirs" "$total_files" "$total_size"
        echo "</pre>"
    done
    
    return
}
```

**Características clave:**

1. **Variables locales:** `local format`, `local i dir_list ...` — separan la lógica de la función
2. **`format` variable:** Define el formato `printf` para salida consistente
3. **Bucle `for i in $dir_list`:** Itera sobre cada directorio (raíz procesa todos; usuario normal solo su `$HOME`)
4. **`find` para contar archivos/directorios:**
   - `find "$i" -type f | wc -l` — total de archivos
   - `find "$i" -type d | wc -l` — total de directorios
5. **`du -sh | cut -f 1`** — obtiene tamaño en formato legible
6. **`printf "$format"`** — formatea la salida con alineación consistente

---

## Resumen: Métodos de Bucle

| Construcción | Mejor para | Ejemplo |
|--------------|-----------|---------|
| `while` | Condiciones complejas, bucles indefinidos | Menús, lectura de archivos |
| `until` | Opuesto lógico de `while` | Menos común que `while` |
| `for (palabras)` | Procesar secuencias, listas | Iterar sobre archivos, parámetros |
| `for ((...))`  | Contadores numéricos, secuencias | Bucles indexados, aritmética |

El script `sys_info_page` ha progresado significativamente desde el Capítulo 25, incorporando:
- Parámetros posicionales (Cap. 32) para opciones de línea de comandos
- Bucles `while` (Cap. 29) para menús interactivos
- Bucles `for` (Cap. 33) para iterar sobre directorios y formatear salida
- Control de flujo con `if` (Cap. 27) y `case` (Cap. 31)

---

## Ver También

- [[wiki/linux/29-control-flujo-while-until.md|Capítulo 29: Bucles while/until]] — otras construcciones de bucle
- [[wiki/linux/32-parametros-posicionales.md|Capítulo 32: Parámetros Posicionales]] — cómo omitir `in words` para usar `$@`
- [[wiki/linux/07-expansiones.md|Capítulo 7: Expansiones]] — brace expansion y pathname expansion para generar listas
