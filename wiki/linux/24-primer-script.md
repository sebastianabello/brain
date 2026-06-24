---
title: "Capítulo 24: Escribiendo tu Primer Script"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 24"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-367-373.pdf"
updated: "2026-06-23"
---

# Escribiendo tu Primer Script

## Introducción

En los capítulos anteriores, hemos ensamblado un arsenal de herramientas de línea de comandos. Aunque estas herramientas pueden resolver muchos tipos de problemas computacionales, estamos limitados a usarlas una por una en la línea de comandos. ¿No sería genial si pudiéramos pedirle al shell que haga más del trabajo?

Podemos. **Uniendo nuestras herramientas en programas de nuestro propio diseño**, el shell puede llevar a cabo secuencias complejas de tareas, todo por sí mismo. Podemos habilitar esto escribiendo **shell scripts**.

---

## ¿Qué son los Shell Scripts?

En los términos más simples, un **shell script** es un archivo que contiene una serie de comandos. El shell lee este archivo y lleva a cabo los comandos como si hubieran sido ingresados directamente en la línea de comandos.

**Característica única del shell:**
El shell es algo único en que es tanto una **interfaz de línea de comandos potente** como un **intérprete de lenguaje de scripting**. Como veremos, la mayoría de las cosas que se pueden hacer en la línea de comandos se pueden hacer en scripts, y la mayoría de las cosas que se pueden hacer en scripts se pueden hacer en la línea de comandos.

**Características enfocadas en scripting:**
Hemos cubierto muchas características del shell, pero nos hemos enfocado en las características más frecuentemente usadas directamente en la línea de comandos. El shell también proporciona un conjunto de características usualmente (pero no siempre) utilizadas al escribir programas.

---

## Cómo Escribir un Shell Script

Para crear y ejecutar exitosamente un shell script, necesitamos hacer tres cosas:

### 1. Escribir un script

Los shell scripts son **archivos de texto ordinarios**. Por lo tanto, necesitamos un editor de texto para escribirlos. Los mejores editores de texto proporcionarán **syntax highlighting** (resaltado de sintaxis), permitiendo vernos una vista codificada por colores de los elementos del script.

**Recomendaciones de editores:**
El resaltado de sintaxis ayudará a identificar ciertos tipos de errores comunes. `vim`, `gedit`, `kate` y muchos otros editores son buenos candidatos para escribir scripts.

### 2. Hacer el script ejecutable

El sistema es bastante exigente sobre no permitir que ningún archivo de texto sea tratado como un programa, y por una buena razón. Necesitamos establecer los permisos del archivo script para permitir ejecución.

### 3. Colocar el script en algún lugar donde el shell pueda encontrarlo

El shell busca automáticamente ciertos directorios para archivos ejecutables cuando no se especifica una ruta explícita. Para máxima conveniencia, colocaremos nuestros scripts en estos directorios.

---

## Formato del Archivo Script

Siguiendo la tradición de programación, crearemos un programa "Hello World!" para demostrar un script extremadamente simple. Abrimos nuestro editor de texto e ingresamos el siguiente script:

```bash
#!/bin/bash

# This is our first script.

echo 'Hello World!'
```

### El Shebang

La primera línea de nuestro script es un poco misteriosa. Se ve como si debería ser un comentario ya que comienza con `#`, pero se ve demasiado propósito para serlo simplemente. El carácter secuencia `#!`, de hecho, es una construcción especial llamada **shebang**. El shebang es usado para decirle al kernel el nombre del intérprete que debería ser usado para ejecutar el script que sigue.

**Cada shell script debe incluir esto como su primera línea.** Usemos `/bin/bash`.

### Comentarios

La segunda línea de nuestro script es también familiar. Se parece a un comentario que hemos visto en muchos de los archivos de configuración que hemos examinado y editado. Una cosa sobre comentarios en shell scripts es que **pueden también aparecer al final de líneas**, siempre que sean precedidos con al menos un carácter de espacio en blanco, como así:

```bash
echo 'Hello World!' # This is a comment too
```

**Todo desde el símbolo `#` en adelante en la línea es ignorado**, como muchas cosas, esto trabaja en la línea de comandos también.

```bash
[me@linuxbox ~]$ echo 'Hello World!' # This is a comment too
Hello World!
```

Aunque comentarios son de poco uso en la línea de comandos, funcionarán.

### Estructura Básica

La última línea de nuestro script es simplemente un comando `echo` con un argumento string. El `echo` es el comando para mostrar texto en la salida estándar, así que es una elección natural como el primer comando en un script.

---

## Permisos Ejecutables

El siguiente paso que tenemos que hacer es hacer que nuestro script sea ejecutable. Esto se hace fácilmente usando `chmod`.

```bash
[me@linuxbox ~]$ ls -l hello_world
-rw-r--r-- 1 me       me       63 2025-03-07 10:10 hello_world
[me@linuxbox ~]$ chmod 755 hello_world
[me@linuxbox ~]$ ls -l hello_world
-rwxr-xr-x 1 me       me       63 2025-03-07 10:10 hello_world
```

### Configuración de Permisos Común

Hay dos configuraciones de permisos comunes para scripts:

| Permiso | Descripción |
|---------|-------------|
| `755` | Para scripts que todos pueden ejecutar |
| `700` | Para scripts que solo el propietario puede ejecutar |

Nota que los scripts deben ser legibles para ser ejecutados.

---

## Ubicación del Script

Con los permisos establecidos, podemos ahora ejecutar nuestro script.

```bash
[me@linuxbox ~]$ ./hello_world
Hello World!
```

Para que el script se ejecute, debemos preceder el nombre del script con una ruta explícita. Si no lo hacemos, obtenemos esto:

```bash
[me@linuxbox ~]$ hello_world
bash: hello_world: command not found
```

¿Por qué ocurre esto? ¿Qué hace que nuestro script sea diferente de otros programas? Como resulta, nada. Su ubicación es el problema.

### Búsqueda de Programas

En el Capítulo 11, discutimos la variable **PATH** de entorno y su efecto en cómo el sistema busca programas ejecutables. Para recapitular, el sistema busca una lista de directorios cada vez que necesita encontrar un programa ejecutable, si no se especifica una ruta explícita. Este es cómo el sistema sabe ejecutar `/bin/ls` cuando escribimos `ls` en la línea de comandos.

El directorio `/bin` es uno de los directorios en los que el sistema busca automáticamente. La lista de directorios se mantiene en una variable de entorno llamada `PATH`. El variable `PATH` contiene una lista separada por dos puntos de directorios a ser buscados.

Podemos ver los contenidos de `PATH`:

```bash
[me@linuxbox ~]$ echo $PATH
/home/me/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

Aquí vemos nuestra lista de directorios. Si nuestro script se ubicara en cualquiera de los directorios en la lista, nuestro problema sería resuelto.

**Solución: crear directorio `~/bin`**

Nota el primer directorio en la lista. `/home/me/bin`. La mayoría de distribuciones Linux configuran la variable `PATH` para contener un directorio `bin` en el directorio home del usuario para permitir que usuarios ejecuten sus propios programas. Así que, si creamos el directorio `bin` y colocamos nuestro script dentro de él, debería comenzar a funcionar como otros programas.

```bash
[me@linuxbox ~]$ mkdir bin
[me@linuxbox ~]$ mv hello_world bin
[me@linuxbox ~]$ hello_world
Hello World!
```

¡Y así funciona!

### Agregar `bin` a PATH Manualmente

Si la variable `PATH` no contiene el directorio, podemos fácilmente agregarlo incluyendo esta línea en nuestro archivo `.bashrc`:

```bash
export PATH=~/bin:"$PATH"
```

Después de este cambio es hecho, tomará efecto en cada nueva sesión de terminal. Para aplicar el cambio a la sesión de terminal actual, debemos tener el shell re-leer el archivo `.bashrc`. Esto se puede hacer "sourcing" it.

```bash
[me@linuxbox ~]$ . .bashrc
```

El comando `.` (dot) es un sinónimo para el comando `source`, un shell builtin que lee un archivo especificado de comandos shell y los trata como si fueran entrada del teclado.

> **Nota:** Ubuntu (y la mayoría de otras distribuciones basadas en Debian) automáticamente agregan el directorio `~/bin` a la variable `PATH` si el directorio `~/bin` existe cuando se ejecuta el archivo `.bashrc` del usuario. Así que, en sistemas Ubuntu, si creas el directorio `~/bin` y luego cierras sesión y vuelves a conectar, todo funciona.

---

## Ubicaciones Buenas para Scripts

El directorio `~/bin` es un buen lugar para poner scripts destinados para uso personal. Si escribimos un script que todos en un sistema se permite usar, la ubicación tradicional es `/usr/local/bin`.

**Convención de directorios:**

| Ubicación | Descripción |
|-----------|-------------|
| `~/bin` | Para scripts destinados para uso personal |
| `/usr/local/bin` | Para software compilado localmente (scripts u otros programas compilados) |
| `/usr/local/sbin` | Para scripts del administrador del sistema |

Estos directorios se especifican por el **Linux Filesystem Hierarchy Standard** para contener solo archivos suministrados y mantenidos por el distribuidor de Linux.

---

## Trucos de Formato Adicionales

Uno de los objetivos clave de la escritura seria de scripts es **facilidad de mantenimiento**, es decir, la facilidad con la cual un script puede ser modificado por su autor u otros para adaptarlo a necesidades cambiantes.

### Nombres de Opciones Largos

Muchos de los comandos que hemos estudiado presentan tanto nombres de opciones cortos como largos. Por ejemplo, el comando `ls` tiene muchas opciones que pueden ser expresadas en forma corta o larga. Por ejemplo, los siguientes:

```bash
[me@linuxbox ~]$ ls -ad
```

es equivalente a esto:

```bash
[me@linuxbox ~]$ ls -all --directory
```

**Ventaja en scripts:**
En los intereses de tipeo reducido, las opciones cortas son preferidas cuando se ingresan opciones en la línea de comandos, pero cuando se escriben scripts, las opciones largas pueden proporcionar legibilidad mejorada.

### Indentación y Continuación de Línea

Cuando se emplean comandos largos, la legibilidad puede ser mejorada esparciendo el comando sobre varias líneas. En el Capítulo 17, miramos un ejemplo particular largo del comando `find`.

```bash
[me@linuxbox ~]$ find playground \( -type f -not -perm 0600 -exec chmod 0600 '{}' ';' \) -or \( -type d -not -perm 0700 -exec chmod 0700 '{}' ';' \)
```

Obviamente, este comando es un poco difícil de descifrar a primera vista. En un script, este comando podría ser más fácil de entender si fuera escrito de esta manera:

```bash
find playground \
  \( \
    -type f \
    -not -perm 0600 \
    -exec chmod 0600 '{}' ';' \
  \) \
  -or \
  \( \
    -type d \
    -not -perm 0700 \
    -exec chmod 0700 '{}' ';' \
  \)
```

Al usar continuaciones de línea (secuencias backslash-linefeed) e indentación, la lógica de este comando complejo es más claramente descrita al lector. Esta técnica funciona en la línea de comandos también, aunque es raramente utilizada, ya que es incómodo escribir y editar. Una diferencia entre un script y una línea de comandos es que el script puede emplear caracteres tab para lograr indentación, mientras que la línea de comandos no puede ya que tabs son usados para activación de completado.

---

## Configuración de Vim para Escritura de Scripts

El editor de texto vim tiene muchas, muchas opciones de configuración. Varias opciones comunes pueden facilitar la escritura de scripts.

### Activar Resaltado de Sintaxis

La siguiente opción activa el resaltado de sintaxis:

```vim
:syntax on
```

Con esta opción, diferentes elementos de la sintaxis del shell serán mostrados en diferentes colores cuando se vea un script. Esto es útil para identificar ciertos tipos de errores de programación. Nota que para que esta característica funcione, debes tener una versión completa de vim instalada, y el archivo que estás editando debe tener un shebang indicando el archivo es un shell script. Si tienes dificultad con el comando anterior, intenta `:set syntax=sh` en su lugar.

### Configuración de Búsqueda de Palabra Clave

La siguiente opción activa el highlighting de resultados de búsqueda:

```vim
:set hlsearch
```

Di que buscamos la palabra `echo`. Con esta opción activada, cada instancia de la palabra será resaltada.

### Configuración de Tabulaciones

La siguiente establece el número de columnas ocupadas por un carácter de tabulación:

```vim
:set tabstop=4
```

El predeterminado es ocho columnas. Establecer el valor a 4 (que es una práctica común) permite que líneas largas quepan más fácilmente en la pantalla.

### Activar Auto-indentación

La siguiente opción activa la característica de "auto indent":

```vim
:set autoindent
```

Esto causa que vim indente una nueva línea la misma cantidad que la línea que acaba de escribirse. Esto acelera la escritura en muchos tipos de construcciones de programación. Para detener la indentación automática, presiona CTRL-D.

### Hacer Cambios Permanentes

Estos cambios pueden ser hechos permanentemente agregando estos comandos (sin los dos puntos iniciales) en tu archivo `~/.vimrc`.

---

## Resumen

En este primer capítulo de scripting, miramos cómo los scripts se escriben y se hacen fáciles para ejecutar en nuestro sistema. También vimos cómo podemos usar varias técnicas de formateo para mejorar la legibilidad (y así la mantenibilidad) de nuestros scripts.

En futuros capítulos, la facilidad de mantenimiento vendrá nuevamente y nuevamente como un principio central en la buena escritura de scripts.

---

## Ver También

- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment (Entorno) de Linux]] — Variables de entorno, PATH, startup files
- [[wiki/linux/12-vi-editor.md|Capítulo 12: Una Introducción Suave a VI(M)]] — Editor vi/vim y sus características
