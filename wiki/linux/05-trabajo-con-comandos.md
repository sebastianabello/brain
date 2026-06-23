---
title: "Capítulo 5: Trabajo con Comandos"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 5"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-75-84.pdf"
updated: "2026-06-22"
---

# Trabajo con Comandos

## Introducción

Hasta este punto, hemos visto una serie de comandos misteriosos, cada uno con sus propias opciones y argumentos misteriosos. En este capítulo, intentaremos remover algo de ese misterio e incluso crearemos nuestros propios comandos.

Los comandos introducidos en este capítulo son:

- **`type`** — Indica cómo se interpreta un nombre de comando
- **`which`** — Muestra qué programa ejecutable será ejecutado
- **`help`** — Obtén ayuda para shell builtins
- **`man`** — Muestra la página manual de un comando
- **`apropos`** — Muestra una lista de comandos apropiados
- **`info`** — Muestra la entrada de información de un comando
- **`whatis`** — Muestra descripciones de una línea de páginas manuales
- **`alias`** — Crea un alias para un comando

## ¿Qué Exactamente Son los Comandos?

Un comando puede ser una de cuatro cosas diferentes:

### 1. Un Programa Ejecutable

Como todos los archivos que vimos en `/usr/bin`. Dentro de esta categoría, los programas pueden ser **compiled binaries** (programas compilados), como programas escritos en C y C++, o programas escritos en **scripting languages** (lenguajes de scripting), como shell, Perl, Python, Ruby, y demás.

### 2. Un Shell Builtin

El bash soporta un número de comandos internamente llamados **shell builtins** (incluidos del shell). El comando `cd`, por ejemplo, es un shell builtin.

### 3. Una Shell Function

Las Shell functions (funciones del shell) son miniprogramas shell incorporados en el **environment** (entorno). Cubriremos la configuración del entorno y la escritura de funciones del shell en capítulos posteriores, pero por ahora solo tenga en cuenta que existen.

### 4. Un Alias

Los aliases (alias) son comandos que podemos definir nosotros mismos, construidos a partir de otros comandos.

## Identificando Comandos

### type — Mostrar el Tipo de Comando

El comando **`type`** es un shell builtin que muestra el tipo de comando que el shell ejecutará, dado un nombre de comando particular. Funciona así:

```
type comando
```

Aquí hay algunos ejemplos:

```bash
[me@linuxbox ~]$ type type
type is a shell builtin

[me@linuxbox ~]$ type ls
ls is aliased to 'ls --color=auto'

[me@linuxbox ~]$ type cp
cp is /usr/bin/cp
```

Aquí vemos los resultados para tres comandos diferentes. Observa que `ls` (tomado de un sistema Fedora) y cómo el comando `ls` es actualmente un alias para `ls` con la opción `--color=auto` añadida. Ahora sabemos por qué la salida de `ls` se muestra en color.

### which — Mostrar la Ubicación de un Ejecutable

A veces hay más de una versión de un programa ejecutable instalado en un sistema. Aunque esto no es común en sistemas de escritorio, no es inusual en servidores grandes. Para determinar la ubicación exacta de un ejecutable dado, se usa el comando `which`:

```bash
[me@linuxbox ~]$ which ls
/usr/bin/ls
```

El comando `which` funciona solo con programas ejecutables, no con shell builtins o aliases que sean sustitutos de programas ejecutables reales. Cuando intentamos usar `which` en un shell builtin, por ejemplo `cd`, obtenemos o ninguna respuesta o un mensaje de error:

```bash
[me@linuxbox ~]$ which cd
/usr/bin/which: no cd in (/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games)
```

Esta respuesta es una forma elegante de decir "comando no encontrado".

## Obtención de Documentación de un Comando

Con este conocimiento de qué es un comando, ahora podemos buscar la documentación disponible para cada tipo de comando.

### help — Obtén Ayuda para Shell Builtins

El bash tiene una facilidad de ayuda integrada disponible para cada uno de los shell builtins. Para usarla, escribe `help` seguido del nombre del shell builtin. Aquí hay un ejemplo:

```bash
[me@linuxbox ~]$ help cd
cd: cd [-L|-P] [-e|]] [dir]
  Change the shell working directory.

  Change the current directory to DIR. The default DIR is the value of the
  HOME shell variable.
  ...
```

La sintaxis de la ayuda para el comando `cd` incluye convenciones como:

- **Elementos entre corchetes** (`[...]`) son opcionales
- **Una barra vertical** (`|`) indica que los elementos son mutuamente excluyentes

En este caso, el comando puede ser seguido opcionalmente por `-L` o `-P`; si la opción `-P` está especificada, la opción `-e` puede ser incluida seguida por el argumento opcional `dir`.

La salida de `help` para el comando `cd` es concisa y precisa, pero a veces no es muy tutorial, y también parece mencionar muchas cosas que aún no hemos hablado. No te preocupes. Lo veremos.

> **Nota**: Por usar el comando `help` con la opción `-m`, `help` mostrará su salida en un formato alterno, parecido a una página man.

### --help — Mostrar Información de Uso

Muchos programas ejecutables soportan una opción `--help` que muestra una descripción de la sintaxis y opciones soportadas del comando. Por ejemplo:

```bash
[me@linuxbox ~]$ mkdir --help
Usage: mkdir [OPTION] DIRECTORY...
Create the DIRECTORY(ies), if they do not already exist.

Mandatory arguments to long options are mandatory for short options too.
  -m, --mode=MODE             set file mode (as in chmod), not a+rwx - umask
  -p, --parents               no error if existing, make parent directories as
                               needed
  -v, --verbose               print a message for each created directory
      --help                  display this help and exit
      --version               output version information and exit
```

Algunos programas no soportan la opción `--help`, pero inténtalo de todos modos. A menudo resulta en un mensaje de error que revelará la misma información de uso.

### man — Mostrar la Página Manual de un Programa

La mayoría de programas ejecutables destinados para uso desde la línea de comandos proporcionan una pieza formal de documentación llamada **manual** o **man page**. Un programa de paging especial llamado `man` se usa para verlos:

```
man programa
```

Las páginas del manual varían algo en formato pero generalmente contienen:

- **Un título** (el nombre de la página)
- **Una sinopsis** (resumen de la sintaxis del comando)
- **Una descripción** (descripción del propósito del comando)
- **Un listado y descripción** de cada una de las opciones del comando

Las páginas del manual generalmente no incluyen ejemplos, y están destinadas como referencia, no como tutorial.

#### Organización de las Páginas del Manual

El "manual" que `man` muestra está dividido en secciones y cubre no solo comandos de usuario sino también comandos de administración, interfaces de programación, formatos de archivo, y más. La tabla siguiente describe la disposición del manual:

| Sección | Contenidos |
|---------|------------|
| 1 | Comandos de usuario |
| 2 | Interfaces de programación para system calls |
| 3 | Interfaces de programación para la librería C |
| 4 | Archivos especiales como device nodes y drivers |
| 5 | Formatos de archivo |
| 6 | Juegos y entretenimiento como screensavers |
| 7 | Miscelánea |
| 8 | Comandos de administración del sistema |

A veces necesitamos referirnos a una sección específica del manual para encontrar lo que buscamos. Esto es particularmente cierto si buscamos un formato de archivo que también es el nombre de un comando. Sin especificar un número de sección, siempre obtendrás la primera instancia de una coincidencia, probablemente en la sección 1. Para especificar una sección número, usamos `man` así:

```
man sección search_term
```

Aquí hay un ejemplo:

```bash
[me@linuxbox ~]$ man 5 passwd
```

Esto mostrará la página man describiendo el formato de archivo del archivo `/etc/passwd`.

#### Búsqueda en las Páginas del Manual

A veces es difícil encontrar una página de man específica si no sabes su nombre exacto. Afortunadamente, podemos buscar en las páginas del manual por palabra clave. Para hacer esto, usamos el comando `man` con la opción `-k`:

```bash
[me@linuxbox ~]$ man -k partition
addpart (8)                  - simple wrapper around the "add partition"...
all-swaps (7)                - event signalling that all swap partitions...
cfdisk (8)                   - display or manipulate disk partition table
cgdisk (8)                   - Curses-based GUID partition table (GPT)...
cfdisk (8)                   - simple wrapper around the "del partition"...
cfdisk (8)                   - Curses-based GUID partition table (GPT)...
gparted (1)                  - partition an MSDOS hard disk
gdisk (8)                    - Interactive GUID partition table (GPT)...
parted (8)                   - inform the OS of partition table changes
parted (8)                   - tell the Linux kernel about the presence...
resizepart (8)               - simple wrapper around the "resize partition...
sgdisk (8)                   - Command-line GUID partition table (GPT)...
```

El primer campo en cada línea de salida es el nombre de la página man, y el segundo campo muestra la sección. Observa que el comando `man` con la opción `-k` realiza la misma función que `apropos`.

### apropos — Mostrar Comandos Apropiados

Es posible buscar en la lista de páginas man para posibles coincidencias basadas en un término de búsqueda. Es crudo pero a veces útil:

```bash
[me@linuxbox ~]$ apropos partition
addpart (8)                  - simple wrapper around the "add partition"...
all-swaps (7)                - event signalling that all swap partitions...
cfdisk (8)                   - display or manipulate disk partition table
cgdisk (8)                   - Curses-based GUID partition table (GPT)...
cfdisk (8)                   - simple wrapper around the "del partition"...
cfdisk (8)                   - Curses-based GUID partition table (GPT)...
gparted (1)                  - partition an MSDOS hard disk
gdisk (8)                    - Interactive GUID partition table (GPT)...
parted (8)                   - inform the OS of partition table changes
parted (8)                   - tell the Linux kernel about the presence...
resizepart (8)               - simple wrapper around the "resize partition...
sgdisk (8)                   - Command-line GUID partition table (GPT)...
```

El primer campo en cada línea de salida es el nombre de la página man, y el segundo campo muestra la sección. Observa que el comando `man` con la opción `-k` realiza la misma función que `apropos`.

### whatis — Mostrar Descripciones de Una Línea de Páginas Manuales

El programa **`whatis`** muestra el nombre y una descripción de una línea de una página man que coincide con una palabra clave especificada:

```bash
[me@linuxbox ~]$ whatis ls
ls                          (1)  - list directory contents
```

### info — Mostrar la Entrada de Información de un Programa

El Proyecto GNU proporciona una alternativa a las páginas man para sus programas, llamada `info`. Las páginas Info se muestran con un programa lector llamado, apropiadamente, `info`. Las páginas Info son **hyperlinked** (hipervínculadas) como las páginas web.

Aquí hay un ejemplo:

```
File: coreutils.info, Node: ls invocation, Next: dir invocation, Up:
Directory listing
10.1 'ls': List directory contents
==============================
The 'ls' program lists information about files (of any type, including
directories). Options and file arguments can be intermixed arbitrarily, as
usual.
```

El programa `info` lee archivos `info`, que están estructurados en nodos individuales (**nodes**), cada uno conteniendo un tema único. Los archivos Info contienen hipervínculos que pueden mover al lector de nodo a nodo. Un hipervínculo puede identificarse por su inicio de asterisco y se activa colocando el cursor sobre él y presionando ENTER.

#### Comandos de info

Para invocar `info`, escribe `info` seguido opcionalmente del nombre de un programa. La siguiente tabla describe los comandos usados para controlar el lector mientras se muestra una página info:

| Comando | Acción |
|---------|--------|
| `?` | Mostrar ayuda de comandos |
| `PAGE UP` o `BACKSPACE` | Mostrar página anterior |
| `PAGE DOWN` o `SPACE` | Mostrar página siguiente |
| `n` | Siguiente: Mostrar el siguiente nodo |
| `p` | Anterior: Mostrar el nodo anterior |
| `u` | Arriba: Mostrar el nodo padre del nodo actualmente mostrado, usualmente un menú |
| `ENTER` | Seguir el hipervínculo en la ubicación del cursor |
| `q` | Salir |

La mayoría de los programas de línea de comandos que hemos discutido hasta ahora son parte del paquete `coreutils` del Proyecto GNU, así que escribir:

```bash
[me@linuxbox ~]$ info coreutils
```

Mostrará una página de menú con hipervínculos a cada programa contenido en el paquete `coreutils`.

### Archivos README y Otra Documentación de Programas

Muchos paquetes de software instalados en nuestro sistema tienen archivos de documentación residiendo en el directorio `/usr/share/doc`. La mayoría están almacenados en formato plaintext (frecuentemente en Markdown, un popular formato de documento plaintext) y pueden ser vistos con `less`.

Algunos archivos están en formato HTML y pueden verse con un navegador web. Podemos encontrar algunos archivos acabando con una extensión `.gz`. Esto indica que han sido comprimidos con el programa de compresión `gzip`. El paquete `gzip` incluye una versión especial de `less` llamada `zless` que mostrará los contenidos de archivos comprimidos con gzip.

## Creando Nuestros Propios Comandos con alias

Ahora para nuestra primera experiencia con programación. Crearemos un comando de nuestro propio usando el comando `alias`. Pero antes de comenzar, necesitamos revelar un pequeño truco de línea de comandos. Es posible poner más de un comando en una línea separando cada comando con un punto y coma. Funciona así:

```
comando1; comando2; comando3...
```

Aquí está el ejemplo que usaremos:

```bash
[me@linuxbox ~]$ cd /usr; ls; cd -
bin  games  include  lib  local  sbin  share  src
/home/me
[me@linuxbox ~]$
```

Como podemos ver, hemos combinado tres comandos en una línea. Primero, cambiamos a `/usr`, luego listamos el directorio, y finalmente regresamos al directorio original (usando `cd -`) así que terminamos donde comenzamos.

Ahora transformemos esta secuencia en un nuevo comando usando `alias`. Lo primero que tenemos que hacer es soñar un nombre para nuestro comando. Intentemos `test`. Antes de hacerlo, sería buena idea averiguar si el nombre `test` ya está siendo usado. Para averiguar, podemos usar el comando `type` nuevamente:

```bash
[me@linuxbox ~]$ type test
test is a shell builtin
```

¡Oops! El nombre `test` ya está tomado. Intentemos `foo`:

```bash
[me@linuxbox ~]$ type foo
bash: type: foo: not found
```

¡Excelente! `foo` no está tomado. Así que vamos a crear nuestro alias:

```bash
[me@linuxbox ~]$ alias foo='cd /usr; ls; cd -'
```

Observa la estructura de este comando mostrado aquí:

```
alias nombre='string'
```

Después del comando `alias`, damos al alias un nombre seguido inmediatamente (sin espacio permitido) por un signo igual, seguido inmediatamente por una cadena entrecomillada que contiene el significado a ser asignado al nombre. Después de definir nuestro alias, podemos usarlo en cualquier lugar donde el shell esperaría un comando. Intentémoslo:

```bash
[me@linuxbox ~]$ foo
bin  games  include  lib  local  sbin  share  src
/home/me
[me@linuxbox ~]$
```

Podemos también usar el comando `type` nuevamente para ver nuestro alias:

```bash
[me@linuxbox ~]$ type foo
foo is aliased to 'cd /usr; ls; cd -'
```

Para remover un alias, el comando `unalias` se usa, así:

```bash
[me@linuxbox ~]$ unalias foo
[me@linuxbox ~]$ type foo
bash: type: foo: not found
```

Aunque intencionalmente evitamos nombrar nuestro alias con un nombre de comando existente, no es incomún hacerlo. Esto a menudo se hace para aplicar una opción comúnmente deseada a cada invocación de un comando común. Por ejemplo, vimos anteriormente cómo el comando `ls` a menudo está aliased para agregar soporte de color:

```bash
[me@linuxbox ~]$ type ls
ls is aliased to 'ls --color=auto'
```

Para ver todos los aliases definidos en el entorno, usa el comando `alias` sin argumentos. Los siguientes son algunos de los aliases definidos por defecto en un sistema Fedora. Intenta averiguar qué hacen:

```bash
[me@linuxbox ~]$ alias
alias l='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
```

Hay un pequeño problema con la definición de aliases en la línea de comandos. Desaparecen cuando nuestra sesión del shell termina. En el Capítulo 11, veremos cómo agregar nuestros propios aliases a los archivos que establecen el entorno cada vez que nos conectamos, pero por ahora, disfruta del hecho de que hemos tomado nuestro primer, aunque pequeño, paso en el mundo de la programación del shell.

## Resumen

Ahora que hemos aprendido cómo encontrar la documentación para comandos, ¡ve y busca la documentación de todos los comandos que hemos encontrado hasta ahora! Estudia qué opciones adicionales están disponibles e ¡inténtalas!

---

## Ver También

- [[wiki/linux/04-manipulacion-archivos-directorios.md|Capítulo 4: Manipulación de Archivos y Directorios]]
- [[wiki/linux/06-redireccion-entrada-salida.md|Capítulo 6: Redirección de Entrada y Salida]]
- [[wiki/linux/comandos-basicos.md|Referencia de Comandos Básicos]]
