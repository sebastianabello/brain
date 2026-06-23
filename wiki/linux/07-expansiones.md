---
title: "Capítulo 7: Expansiones en Linux"
Sources: raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-99-109.pdf
Raw: Capítulo 7 - SEEING THE WORLD AS THE SHELL SEES IT (págs. 65-75)
Updated: 2026-06-22
---

# Expansiones: Cómo el Shell Ve el Mundo

## Concepto Fundamental

Cada vez que escribimos un comando y presionamos ENTER, **bash realiza varias sustituciones sobre el texto** antes de ejecutar el comando. Este proceso se llama **expansion** (expansión).

**El shell expande:**
- Caracteres comodín (wildcards)
- Variables
- Expresiones aritméticas
- Comandos (para obtener su salida)
- Y más...

Todo esto ocurre antes de que el comando se ejecute. Para demostrar, usamos el comando `echo`, que simplemente imprime sus argumentos:

```bash
echo this is a test
this is a test
```

---

## Expansión de Nombres de Ruta (Pathname Expansion)

El mecanismo por el cual funcionan los comodines se llama **pathname expansion**. 

**Ejemplo:**
```bash
echo Desktop Documents Music
Desktop Documents Music
```

¿Qué pasó aquí? El shell expandió el `*` a los nombres de archivos que corresponden al patrón. Cuando presionamos ENTER, el shell expande automáticamente cualquier carácter comodín (`*`) antes de que se ejecute el comando.

**Comportamiento importante:** El shell expande el comodín ANTES de ejecutar `echo`. El comando `echo` nunca ve el carácter `*`, solo ve los nombres expandidos.

```bash
echo *
Desktop Documents Music
```

### Limitaciones de Pathname Expansion

**Los archivos ocultos (que comienzan con `.`) NO se incluyen por defecto:**

```bash
echo *
# No incluye .bashrc, .hidden, etc.
```

⚠️ **Nota especial sobre archivos ocultos:** 
- `echo *` no incluye archivos ocultos
- `echo .*` incluye puntos de referencia (`.` y `..`) que pueden causar problemas
- Usar un patrón específico es mejor: `echo [.]*` para archivos con un punto seguido de cualquier carácter

Para listado seguro de archivos ocultos:
```bash
ls -A
# Muestra todos excepto . y ..
```

---

## Expansión de Tilde (~)

El carácter `~` (tilde) se expande al nombre del directorio home del usuario actual.

```bash
echo ~
/home/me
```

Si un usuario llamado `bob` tiene una cuenta:
```bash
echo ~bob
/home/bob
```

**Uso práctico:** Navegar rápidamente al home:
```bash
cd ~
```

---

## Expansión Aritmética

El shell permite realizar operaciones aritméticas mediante expansión. Esto permite usar el shell como calculadora.

**Sintaxis:**
```bash
$((expresión))
```

**Ejemplo:**
```bash
echo $((2 + 2))
4
```

### Operadores Aritméticos Soportados

| Operador | Descripción |
|----------|-------------|
| `+` | Adición |
| `-` | Sustracción |
| `*` | Multiplicación |
| `/` | División (solo enteros, no decimales) |
| `%` | Módulo (resto) |
| `**` | Exponenciación |

**Características importantes:**
- Solo soporta enteros (números completos), no decimales
- Los espacios NO son significativos en expresiones aritméticas
- Las expresiones pueden anidarse

**Ejemplos:**
```bash
# Multiplicación
echo $((5 * 3))
15

# División entera
echo $((5 / 2))
2

# Módulo (resto)
echo $((5 % 2))
1

# Exponenciación
echo $((2 ** 3))
8

# Expresiones complejas
echo $(((5**2) * 3))
75

# Paréntesis para agrupar subexpresiones
echo $((5 * 3 + 2))
17
```

💡 **Uso práctico:** Crear directorios con números secuenciales:
```bash
mkdir Photos
cd Photos
mkdir {2007..2009}-{01..12}
# Crea: 2007-01, 2007-02, ..., 2009-12
```

---

## Expansión de Llaves (Brace Expansion)

Quizás la expansión más extraña, **brace expansion** crea múltiples cadenas de texto a partir de un patrón con llaves.

**Sintaxis:**
```bash
{prefijo,suffijo1,suffijo2,...}
{inicio..fin}
```

### Ejemplo Básico

```bash
echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back
```

### Rangos de Números

```bash
echo Number_{1..5}
Number_1 Number_2 Number_3 Number_4 Number_5
```

### Relleno de Ceros (Zero-Padding)

En bash 4.0+, los números pueden rellenarse con ceros:

```bash
echo {01..15}
01 02 03 04 05 06 07 08 09 10 11 12 13 14 15

echo {001..015}
001 002 003 004 005 006 007 008 009 010 011 012 013 014 015
```

### Rangos de Letras (incluso invertidos)

```bash
# Rango normal
echo {A..C}
A B C

# Rango invertido
echo {Z..A}
Z Y X W V U T S R Q P O N M L K J I H G F E D C B A
```

### Expansiones Anidadas

```bash
echo a{A{1,2},B{3,4}}b
aA1b aA2b aB3b aB4b
```

### Aplicación Práctica: Crear Estructura de Directorios

```bash
mkdir Photos
cd Photos
mkdir {2007..2009}-{01..12}
ls
# 2007-01  2007-02  2007-03  ...  2008-12  2009-12
```

Este es uno de los usos más comunes de brace expansion: crear automáticamente una estructura de directorios para organizar fotos por año y mes.

---

## Expansión de Parámetros

El shell almacena pequeños fragmentos de datos en **variables**. Una variable especial importante es `USER`, que contiene el nombre del usuario actual.

```bash
echo $USER
me

echo $LOGNAME
me
```

Para ver todas las variables disponibles:
```bash
printenv | less
```

⚠️ **Comportamiento importante:** Si escribes el nombre de una variable que NO existe, la expansión resulta en una cadena vacía:

```bash
echo $SUSER
# (ninguna salida - cadena vacía)
```

---

## Sustitución de Comandos

La **command substitution** permite usar la salida de un comando como una expansión.

**Sintaxis moderna:**
```bash
$(comando)
```

**Sintaxis antigua (aún soportada):**
```bash
`comando`
# (backticks)
```

### Ejemplos

```bash
# Ver qué archivos hay en /usr/bin
echo $(ls /usr/bin)
Desktop Documents ls-output.txt Music ...

# Combinar con otros comandos
ls -l $(which cp)
-rwxr-xr-x 1 root root 71516 2025-12-05 08:58 /bin/cp

# Caso práctico: listar archivos que contienen "zip"
file $(ls -d /usr/bin/* | grep zip)
/usr/bin/bunzip2: ELF 32-bit LSB executable, Intel 80386...
/usr/bin/bzip2: ELF 32-bit LSB executable, Intel 80386...
/usr/bin/funzip: ELF 32-bit LSB executable, Intel 80386...
```

**Uso avanzado:** La salida del comando se convierte en argumentos para otro comando:

```bash
# Encontrar dónde está un comando
ls -l $(which cp)

# El comando which produce: /bin/cp
# Luego ls -l recibe /bin/cp como argumento
```

💡 **Ventaja:** No necesitas saber la ruta completa del archivo, el comando la encuentra automáticamente.

---

## Quoting (Entrecomillado)

Ahora que hemos visto cómo el shell realiza expansiones, es hora de aprender a **controlarlas**.

El shell proporciona mecanismos de **quoting** para suprimir selectivamente expansiones no deseadas.

### Double Quotes (Comillas Dobles)

Si colocas texto dentro de comillas dobles, la mayoría de caracteres especiales pierden su significado y se tratan como ordinarios. **Las excepciones son:**
- `$` (expansión de variables)
- `\` (backslash - escape)
- `` ` `` (backtick - sustitución de comandos)
- `"` (comilla doble)

**Esto significa:** Word-splitting se suprime, expansión de nombres de ruta se suprime, pero parámetros, aritmética y sustitución de comandos SÍ ocurren.

```bash
# Sin comillas: word-splitting separa argumentos
echo this is a test
this
is
a
test

# Con comillas: un solo argumento
echo "this is a test"
this is a test
```

### Ejemplo Práctico: Archivos con Espacios

Sin comillas, un archivo de dos palabras se trata como dos argumentos:

```bash
ls -l two words.txt
ls: cannot access 'two': No such file or directory
ls: cannot access 'words.txt': No such file or directory
```

Con comillas, se trata como un solo argumento:

```bash
ls -l "two words.txt"
-rw-r--r-- 1 me me 18 2016-02-20 13:03 two words.txt
```

### Expansiones Dentro de Comillas Dobles

Las expansiones clave aún funcionan:

```bash
# Expansión de parámetros
echo "$USER $(date +%j) $((2+2)) $(echo foo)"
me 4 4 foo

# Expansión de variables con if/else condicional
echo "The balance for user $USER is: \$5.00"
The balance for user me is: $5.00
```

Nota: `\$` escapa el `$`, evitando la expansión.

### Single Quotes (Comillas Simples)

Si necesitas suprimir **TODAS las expansiones**, usa comillas simples. Con comillas simples, NADA se expande:

```bash
# Comparación de unquoted, double quoted, single quoted:
echo text ~/.*  $(echo foo) $((2+2)) $USER
text /home/me/ls-output.txt a b foo 4 me

echo "text ~/.*  $(echo foo) $((2+2)) $USER"
text ~/.* foo 4 me

echo 'text ~/.*  $(echo foo) $((2+2)) $USER'
text ~/.*  $(echo foo) $((2+2)) $USER
```

**Resumen:** Con cada nivel de quoting, más expansiones se suprimen:
- **Unquoted:** Todas las expansiones ocurren
- **Double quoted:** Word-splitting y pathname expansion suprimidas; parámetros, aritmética, command substitution ocurren
- **Single quoted:** TODAS las expansiones suprimidas

### Escaping Characters (Caracteres de Escape)

A veces queremos eliminar el significado especial de solo un carácter. Para esto, precede el carácter con una barra invertida (backslash), llamada **escape character**.

```bash
# Sin escape
echo $USER
me

# Con escape
echo \$USER
$USER

# Usar en nombres de archivo
mv bad\filename good_filename
```

**Uso común:** Incluir caracteres especiales en nombres de archivo:

```bash
# Incluir un signo de dólar en un nombre
mv "bad$filename" "good_filename"

# O con escaping
mv bad\$filename good_filename
```

### Backslash Escape Sequences

Además de su rol como escape character, backslash es parte de secuencias especiales que representan caracteres de control. Estos son:

| Secuencia | Significado |
|-----------|-------------|
| `\a` | Bell (alerta que hace beep) |
| `\b` | Backspace |
| `\n` | Newline (salto de línea) |
| `\r` | Carriage return (retorno de carro) |
| `\t` | Tab |

**Uso:** Echo con opción `-e` interpreta estas secuencias:

```bash
sleep 10; echo -e "Time's up\a"
# Espera 10 segundos, luego imprime "Time's up" y hace beep

sleep 10; echo "Time's up $'\a'"
# Alternativa usando $'...' syntax
```

---

## Resumen de Tipos de Expansión

| Tipo | Símbolo | Ejemplo | Resultado |
|------|---------|---------|-----------|
| Pathname | `*` | `ls D*` | Expande a archivos que comienzan con D |
| Tilde | `~` | `cd ~` | Va al home directory |
| Aritmética | `$((...))` | `echo $((2+2))` | `4` |
| Brace | `{...}` | `echo a{1,2,3}` | `a1 a2 a3` |
| Parámetro | `$var` | `echo $USER` | `me` |
| Command | `$(...)`  | `echo $(date)` | Salida de `date` |

---

## Resumen de Quoting

| Tipo | Suprime | Permite | Ejemplo |
|------|---------|---------|---------|
| **Unquoted** | Nada | Todo | `echo $var *.txt $(cmd)` |
| **Double quotes** | Word-split, pathname | Parámetro, aritmética, command | `echo "$var" *.txt` |
| **Single quotes** | TODO | Nada | `echo '$var'` → `$var` |
| **Escape `\`** | Un carácter | N/A | `echo \$var` → `$var` |

---

## Filosofía de Expansión

La verdadera potencia del shell viene de su capacidad para expandir y transformar texto. Sin una comprensión profunda de cómo funcionan las expansiones, el shell siempre será una fuente de misterio y confusión. Pero una vez que las entiendes, ves que la mayoría de lo que hace el shell es lógico y predecible.

**El shell es un lenguaje de procesamiento de texto fundamentalmente**, y las expansiones son su característica central. A medida que avances con Linux, verás que las expansiones se usan cada vez más frecuentemente, especialmente en scripts de shell más avanzados (Capítulo 34).

> "The magic that happens on the command line when we press ENTER is really just the shell performing expansions on the text before it carries out our command. Understanding this process is key to mastering the shell."

---

*Fuente: The Linux Command Line - A Complete Introduction (William E. Shotts, Jr.)*
