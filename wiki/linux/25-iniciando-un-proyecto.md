---
title: "Capítulo 25: Iniciando un Proyecto"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 25"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-375-384.pdf"
updated: "2026-06-23"
---

# Iniciando un Proyecto

## Visión General del Proyecto

Comenzamos aquí a construir un programa práctico. El objetivo es ver cómo se utilizan diversas características del shell para crear programas, y más importante aún, **crear buenos programas**.

El programa que escribiremos es un **generador de reportes** (report generator). Presentará varias estadísticas sobre el sistema y su estado, y producirá este reporte en **formato HTML**, de modo que podamos verlo con un navegador web como Firefox o Chrome.

Los programas generalmente se construyen en una serie de **etapas progresivas**, donde cada etapa agrega características y capacidades. La primera etapa de nuestro programa producirá un documento HTML mínimo sin información del sistema. Eso vendrá después.

---

## Primera Etapa: Documento HTML Mínimo

### Estructura de un Documento HTML Bien Formado

Lo primero que necesitamos saber es el formato de un documento HTML correctamente estructurado:

```html
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    Page body.
  </body>
</html>
```

Si ingresamos esto en un editor de texto y lo guardamos como `foo.html`, podemos usar la siguiente URL en Firefox para ver el archivo:

```
file:///home/<usuario>/foo.html
```

### Primer Script: Salida Múltiple de Echo

La primera etapa de nuestro programa será capaz de outputear este archivo HTML a standard output (salida estándar). Podemos escribir un programa para hacer esto bastante fácilmente.

Creemos un archivo nuevo llamado `~/bin/sys_info_page`:

```bash
#!/bin/bash
# Program to output a system information page

echo "<html>"
echo "  <head>"
echo "    <title>Page Title</title>"
echo "  </head>"
echo "  <body>"
echo "    Page body."
echo "  </body>"
echo "</html>"
```

Nuestro primer intento contiene:
- Un **shebang** (`#!/bin/bash`) para indicar el intérprete
- Un **comentario** (siempre una buena idea)
- Una secuencia de comandos `echo`, uno para cada línea de output

Después de guardar el archivo, lo hacemos ejecutable y lo ejecutamos:

```bash
[me@linuxbox ~]$ chmod 755 ~/bin/sys_info_page
[me@linuxbox ~]$ sys_info_page
```

Cuando el programa se ejecuta, deberíamos ver el texto del documento HTML mostrado en pantalla, ya que los comandos `echo` en el script envían su salida a standard output.

Ahora ejecutamos el programa nuevamente pero **redirigimos la salida** al archivo `sys_info_page.html` para poder verlo en un navegador:

```bash
[me@linuxbox ~]$ sys_info_page > sys_info_page.html
[me@linuxbox ~]$ firefox sys_info_page.html
```

### Simplificación: Strings Multilinea

Cuando escribimos programas, siempre es buena idea esforzarse por la **simplicidad y claridad**. El mantenimiento es más fácil cuando un programa es fácil de leer y entender. Nuestra versión actual funciona bien, pero podría ser más simple.

Podríamos combinar todos los comandos `echo` en uno solo, lo que ciertamente haría más fácil agregar más líneas a la salida del programa:

```bash
#!/bin/bash
# Program to output a system information page

echo "<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    Page body.
  </body>
</html>"
```

Una string entrecomillada puede incluir saltos de línea y por lo tanto contener múltiples líneas de texto. El shell seguirá leyendo el texto hasta que encuentre la comilla de cierre. Esto también funciona en la línea de comandos:

```bash
[me@linuxbox ~]$ echo "<html>
> <head>
> <title>Page Title</title>
> </head>
> <body>
> Page body.
> </body>
> </html>"
```

El carácter `>` al principio es el **prompt PS2** (contenido en la variable de shell PS2). Aparece siempre que escribimos una instrucción multilinea en el shell. Esta característica es un poco oscura ahora, pero más adelante, cuando cubramos instrucciones de programación multilinea, resultará ser bastante útil.

---

## Segunda Etapa: Agregando un Poco de Datos

Ahora que nuestro programa puede generar un documento mínimo, agreguemos algunos datos al reporte:

```bash
#!/bin/bash
# Program to output a system information page

echo "<html>
  <head>
    <title>System Information Report</title>
  </head>
  <body>
    <h1>System Information Report</h1>
  </body>
</html>"
```

Agregamos un título de página y un encabezado al cuerpo del reporte.

---

## Variables y Constantes

### El Problema de la Repetición

Hay un problema con nuestro script, sin embargo. Fijarse cómo la string `System Information Report` se repite. Con nuestro pequeño script no es un problema, pero imaginemos que nuestro script fuera realmente largo y tuviéramos múltiples instancias de esta string. Si quisiéramos cambiar el título a algo más, tendríamos que cambiarlo en múltiples lugares, lo que podría ser mucho trabajo.

¿Y si pudiéramos organizar el script de modo que la string apareciera solo una vez y no múltiples veces? Eso haría que el mantenimiento futuro del script fuera mucho más fácil.

### Introducción a Variables

Aquí es cómo podríamos hacer eso:

```bash
#!/bin/bash
# Program to output a system information page

title="System Information Report"

echo "<html>
  <head>
    <title>$title</title>
  </head>
  <body>
    <h1>$title</h1>
  </body>
</html>"
```

Al crear una variable llamada `title` y asignarle el valor `System Information Report`, podemos aprovechar la **expansión de parámetros** y colocar la string en múltiples ubicaciones.

### Cómo Crear una Variable

¿Cómo creamos una variable? Simple: simplemente la usamos. Cuando el shell encuentra una variable, la crea automáticamente. Esto difiere de muchos lenguajes de programación en los que las variables deben ser **declaradas o definidas explícitamente** antes de usar.

El shell es muy permisivo (lax) sobre esto, lo que puede llevar a algunos problemas. Por ejemplo, considera este escenario:

```bash
[me@linuxbox ~]$ foo="yes"
[me@linuxbox ~]$ echo $foo
yes
[me@linuxbox ~]$ echo $fool
[me@linuxbox ~]$
```

Primero asignamos el valor `yes` a la variable `foo`, y luego mostramos su valor con `echo`. Después, mostramos el valor de la variable con el nombre mal escrito `fool` y obtenemos un resultado en blanco.

Esto es porque el shell alegremente creó la variable `fool` cuando la encontró y le dio el valor por defecto de nada, o vacío. De esto aprendemos que **debemos prestar atención a nuestra ortografía**.

#### Expansión de Parámetros: El Mecanismo

Es importante entender qué pasó realmente en este ejemplo. Del control anterior sobre cómo el shell realiza expansiones, sabemos que el comando:

```bash
[me@linuxbox ~]$ echo $foo
```

Se somete a **expansión de parámetros** y resulta en:

```bash
[me@linuxbox ~]$ echo yes
```

Por el contrario, el comando:

```bash
[me@linuxbox ~]$ echo $fool
```

Se expande a:

```bash
[me@linuxbox ~]$ echo
```

¡La variable vacía se expande a nada! Esto puede causar problemas graves con comandos que requieren argumentos.

Aquí hay un ejemplo:

```bash
[me@linuxbox ~]$ foo=foo.txt
[me@linuxbox ~]$ foo1=foo1.txt
[me@linuxbox ~]$ cp $foo $fool
cp: missing destination file operand after `foo.txt'
Try `cp --help' for more information.
```

Asignamos valores a dos variables, `foo` y `foo1`. Luego realizamos un `cp` pero escribimos mal el nombre del segundo argumento (usando una L minúscula en lugar del numeral 1). Después de la expansión, el comando `cp` se envía solo con un argumento, aunque requiere dos.

### Reglas para Nombres de Variables

Hay algunas reglas sobre nombres de variables:

- Los nombres de variable pueden consistir en caracteres alfanuméricos (letras y números) y caracteres de subrayado.
- El primer carácter de un nombre de variable debe ser una letra o un subrayado.
- Los espacios y símbolos de puntuación no están permitidos.

### Variables vs Constantes

La palabra "variable" implica un valor que cambia, y en muchas aplicaciones, las variables se usan de esta manera. Sin embargo, la variable en nuestra aplicación, `title`, se utiliza como una **constante**. Una constante es como una variable en que tiene un nombre y contiene un valor. La diferencia es que el valor de una constante **no cambia**.

En una aplicación que realiza cálculos geométricos, podríamos definir PI como una constante y asignarle el valor 3.1415, en lugar de usar el número literalmente en todo nuestro programa.

El shell no hace distinción entre variables y constantes; son principalmente para la comodidad del programador. Una **convención común** es usar **letras mayúsculas** para designar constantes y **letras minúsculas** para verdaderas variables.

Modificaremos nuestro script para cumplir con esta convención:

```bash
#!/bin/bash
# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"

echo "<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
  </body>
</html>"
```

También aprovechamos para "adornar" nuestro título agregando el valor de la variable de shell `HOSTNAME`. Esta es el nombre de red de la máquina.

> **Nota**: El shell realmente proporciona una manera de hacer cumplir la inmutabilidad de constantes, a través del uso del comando integrado `declare` con la opción `-r` (read-only). Si hubiéramos asignado `TITLE` de esta manera:
> ```bash
> declare -r TITLE="Page Title"
> ```
> El shell impediría cualquier asignación posterior a `TITLE`. Esta característica se usa raramente, pero existe para scripts muy formales.

---

## Asignando Valores a Variables y Constantes

### Sintaxis Básica de Asignación

Aquí es donde nuestro conocimiento de expansión comienza a rentabilizarse. Como hemos visto, las variables se asignan valores de esta manera, donde `variable` es el nombre de la variable y `value` es una string:

```
variable=value
```

A diferencia de algunos otros lenguajes de programación, el shell no se preocupa por el tipo de datos asignado a una variable; los trata todos como strings. Puedes forzar al shell a restringir la asignación a enteros usando el comando `declare` con la opción `-i`, pero, como establecer variables como read-only, esto se hace raramente.

Nota que en una asignación, no debe haber espacios entre el nombre de la variable, el signo igual y el valor. ¿Entonces qué puede consistir el valor? Puede ser cualquier cosa que podamos expandir en una string.

```bash
a=z                          # Assign the string "z" to variable a.
b="a string"                 # Embedded spaces must be within quotes.
c="a string and $b"          # Other expansions such as variables can be
                             # expanded into the assignment.
d="$(ls -l foo.txt)"         # Results of a command
E=$((5 * 7))                 # Arithmetic expansion
f="\t\ta string\n"           # Escape sequences such as tabs and newlines.
```

Las asignaciones de múltiples variables pueden realizarse en una sola línea:

```bash
a=5 b="a string"
```

### Braces para Desambiguación

Durante la expansión, los nombres de variable pueden estar rodeados por **llaves opcionales** `{}`. Esto es útil en casos donde un nombre de variable se vuelve ambiguo debido a su contexto circundante.

Aquí, intentamos cambiar el nombre de un archivo de `myfile` a `myfile1`, usando una variable:

```bash
[me@linuxbox ~]$ filename="myfile"
[me@linuxbox ~]$ touch "$filename"
[me@linuxbox ~]$ mv "$filename" "$filename1"
mv: missing destination file operand after `myfile'
Try `mv --help' for more information.
```

Este intento falla porque el shell interpreta el segundo argumento del comando `mv` como una nueva variable (y vacía). El problema puede superarse de esta manera:

```bash
[me@linuxbox ~]$ mv "$filename" "${filename}1"
```

Al agregar las llaves circundantes, el shell ya no interpreta el "1" final como parte del nombre de la variable.

> **Nota**: Es una buena práctica encerrar variables y sustituciones de comandos entre comillas dobles para limitar los efectos de word-splitting por el shell. Las comillas son especialmente importantes cuando una variable podría contener un nombre de archivo.

### Ejemplo Práctico: Agregando Datos de Fecha y Usuario

Tomaremos esta oportunidad para agregar algunos datos a nuestro reporte, es decir, la fecha y hora en que se creó el reporte y el nombre de usuario del creador:

```bash
#!/bin/bash
# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

echo "<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
  </body>
</html>"
```

Aquí vemos:
- **CURRENT_TIME**: Usa sustitución de comandos `$(date +...)` para obtener fecha/hora actual
- **TIMESTAMP**: Combina la hora actual con el nombre de usuario (`$USER`)
- El HTML usa `<p>` para agregar un párrafo con la marca de tiempo

---

## Documentos Aquí (Here Documents)

### El Problema con Echo Multilinea

Hemos visto dos métodos diferentes de outputear nuestro texto, ambos usando el comando `echo`. Hay una **tercera forma** llamada **here document** (documento aquí) o **here script**.

Un here document es una forma adicional de **redirección de I/O** en la cual embebemos un cuerpo de texto en nuestro script y lo alimentamos a standard input de un comando. Funciona así:

```
command << token
text
token
```

En este ejemplo, `command` es el nombre del comando que acepta standard input, y `token` es una string usada para indicar el final del texto embebido.

### Implementación con Here Documents

Aquí modificaremos nuestro script para usar un here document:

```bash
#!/bin/bash
# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
  </body>
</html>
_EOF_
```

En lugar de usar `echo`, nuestro script ahora usa `cat` y un here document. La string `_EOF_` (significando "end-of-file", una convención común) fue seleccionada como el token y marca el final del texto embebido. 

Nota que **el token debe aparecer solo** y que **no debe haber espacios finales en la línea**.

### Ventajas de Here Documents

¿Cuál es la ventaja de usar un here document? Es principalmente la misma que usar echo, excepto que, **por defecto**, las comillas simples y dobles dentro de here documents pierden su significado especial para el shell.

Aquí hay un ejemplo de línea de comandos:

```bash
[me@linuxbox ~]$ foo="some text"
[me@linuxbox ~]$ cat << _EOF_
> $foo
> "$foo"
> '$foo'
> \$foo
> _EOF_
some text
"some text"
'some text'
$foo
```

Como podemos ver, el shell no presta atención a las marcas de entrecomillado. Las trata como caracteres ordinarios. Esto nos permite embeber comillas libremente dentro de un here document. Esto podría resultar ser práctico para nuestro programa de reporte.

### Texto dentro de Here Documents y Expansiones

El texto dentro de un here document se somete a **expansión de parámetros**, **sustitución de comandos** y **expansión aritmética**, y los caracteres literales `$` y `\` deben ser escapados con `\`.

Sin embargo, cuando encerramos el token de inicio entre comillas:

```bash
cat << '_EOF_'
```

Las comillas serán **removidas del token** y **no se realizarán expansiones** en el texto.

```bash
[me@linuxbox ~]$ foo="some text"
[me@linuxbox ~]$ cat << '_EOF_'
> $foo
> "$foo"
> '$foo'
> \$foo
> _EOF_
$foo
"$foo"
'$foo'
\$foo
```

### Here Documents con Comandos Reales

Los here documents pueden ser usados con cualquier comando que acepte standard input. En este ejemplo, usamos un here document para pasar una serie de comandos al programa `ftp` para recuperar un archivo de un servidor FTP remoto:

```bash
#!/bin/bash
# Script to retrieve a file via FTP

FTP_SERVER="ftp.nl.debian.org"
FTP_PATH="/debian/dists/bookworm/main/installer-amd64/current/images/cdrom"
REMOTE_FILE="debian-cd_info.tar.gz"

ftp -n << _EOF_
open $FTP_SERVER
user anonymous me@linuxbox
cd $FTP_PATH
hash
get $REMOTE_FILE
bye
_EOF_

ls -l "$REMOTE_FILE"
```

### Indentación de Here Documents

Si cambiamos el operador de redirección de `<<` a `<<-`, el shell ignorará los **caracteres de tabulación líder** (pero no espacios) en el here document. Esto permite que un here document sea indentado, lo que puede mejorar la legibilidad:

```bash
#!/bin/bash
# Script to retrieve a file via FTP

FTP_SERVER="ftp.nl.debian.org"
FTP_PATH="/debian/dists/bookworm/main/installer-amd64/current/images/cdrom"
REMOTE_FILE="debian-cd_info.tar.gz"

ftp -n <<- _EOF_
  open $FTP_SERVER
  user anonymous me@linuxbox
  cd $FTP_PATH
  hash
  get $REMOTE_FILE
  bye
  _EOF_

ls -l "$REMOTE_FILE"
```

> **Advertencia**: Esta característica puede ser algo problemática, porque muchos editores de texto (y los programadores mismos) preferirán usar espacios en lugar de tabulaciones para lograr indentación en sus scripts.

---

## Resumen

En este capítulo, comenzamos un proyecto que nos llevará a través del proceso de construir un script exitoso. Introdujimos el concepto de **variables y constantes** y cómo pueden ser empleadas. Son la primera de muchas aplicaciones que encontraremos para **expansión de parámetros**.

También vimos cómo producir output de nuestro script y **varios métodos para embeber bloques de texto**:

1. **Echo multilinea**: Strings con múltiples líneas
2. **Here documents**: Redirección de entrada para embeber texto complejo

En los capítulos siguientes, continuaremos expandiendo este proyecto, agregando:
- Información del sistema (usando comandos `date`, `df`, `free`, etc.)
- Control de flujo (if/then/else)
- Loops (for, while)
- Funciones
- Mejores prácticas de scripting

---

## Ver También

- [[wiki/linux/24-primer-script.md|Capítulo 24: Escribiendo tu Primer Script]] — Conceptos fundamentales de shell scripts
- [[wiki/linux/07-expansiones.md|Capítulo 7: Expansiones]] — Expansión de parámetros y sustitución de comandos
- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] — Fundamentos de entrada/salida
