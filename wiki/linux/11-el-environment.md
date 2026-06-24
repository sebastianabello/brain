---
title: "Capítulo 11: El Environment (Entorno) de Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 11"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-155-167.pdf"
updated: "2026-06-23"
---

# El Environment (Entorno) de Linux

## ¿Qué es el Environment?

El **environment** (entorno) es un conjunto de información que el shell mantiene durante la sesión del usuario. Los programas utilizan datos almacenados en el environment para determinar la configuración del sistema. Aunque la mayoría de programas usan archivos de configuración para almacenar ajustes, algunos programas también consultan valores en el environment para modificar su comportamiento.

El environment contiene principalmente dos tipos de datos:

- **Environment variables** (variables de environment): Son datos colocados por la instancia actual de bash. Están disponibles en el shell.
- **Shell variables**: Son datos colocados por bash. Además, el shell almacena algunos datos programáticos, como `aliases` (alias) y `shell functions` (funciones del shell).

## Comandos para Trabajar con el Environment

En este capítulo utilizaremos los siguientes comandos:

- **printenv**: Imprime parte o todo el environment
- **set**: Establece opciones del shell
- **export**: Exporta environment a procesos ejecutados posteriormente
- **alias**: Crea un alias para un comando
- **source**: Ejecuta comandos de un archivo en el shell actual

## Examinando el Environment

### Usando `printenv` y `set`

Para ver qué está almacenado en el environment, podemos usar el comando `set` (integrado en bash) o el programa `printenv`. 

El comando `set` muestra tanto variables del shell como del environment, mientras que `printenv` muestra solo las variables de environment.

```bash
[me@linuxbox ~]$ printenv | less
```

Este comando nos mostrará algo como:

```
USER=me
PAGER=less
LSCOLORS=Gxfxcxdxbxegedabagacad
XDG_CONFIG_DIRS=/etc/xdg:/xdg-ubuntu:/usr/share/xdg:/etc/xdg
PATH=/home/me/.local/sbin:/bin:/usr/local/sbin:/bin:/usr/games:/usr
...
```

### Ver una Variable Específica

También podemos ver el valor de una variable específica:

```bash
[me@linuxbox ~]$ printenv USER
me
```

O usar el comando `echo`:

```bash
[me@linuxbox ~]$ echo $HOME
/home/me
```

### Ver Todos los Alias

Los alias no aparecen en `set` ni `printenv`. Para verlos:

```bash
[me@linuxbox ~]$ alias
alias l.='ls -d .* --color=tty'
alias ll='ls -l --color=tty'
alias ls='ls --color=tty'
alias vi='vim'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

## Variables de Environment Interesantes

Existen varias variables de environment estándar. La siguiente tabla muestra las más importantes:

| Variable | Contenido |
|----------|-----------|
| **DISPLAY** | Nombre del display si estás usando un ambiente gráfico. Usualmente es `:0`, especificando que el primer display generado por el servidor X. En sistemas con wayland, hay una variable relacionada llamada WAYLAND_DISPLAY. |
| **EDITOR** | Nombre del programa a ser usado para edición de texto. |
| **SHELL** | Nombre del programa shell del usuario por defecto. |
| **HOME** | Ruta del directorio home del usuario. |
| **LANG** | Juego de caracteres y orden de collación del lenguaje humano. |
| **OLDPWD** | Directorio de trabajo anterior. |
| **PAGER** | Nombre del programa a usar para paging de salida. Usualmente es `/usr/bin/less`. |
| **PATH** | Lista separada por dos puntos de directorios que se buscan cuando ingresas el nombre de un programa ejecutable. |
| **PS1** | Define el contenido del prompt del shell. Como veremos en el Capítulo 13, esto puede ser altamente personalizado. |
| **PWD** | Directorio de trabajo actual. |
| **TERM** | Nombre del tipo de terminal. Los sistemas Unix soportan muchos protocolos de terminal; esta variable establece cuál será usado por tu emulador de terminal. |
| **TZ** | Especifica tu zona horaria. Los sistemas Unix mantienen el reloj interno del computador en Coordinated Universal Time (UTC) y luego muestran la hora local aplicando un offset especificado por esta variable. |
| **USER** | Tu nombre de usuario. |

> **Nota**: No te preocupes si algunos de estos valores están faltantes. Varían según la distribución.

## ¿Cómo Se Establece el Environment?

Cuando accedes al sistema, el programa bash comienza y lee una serie de archivos de configuración llamados **startup files** (archivos de inicio), que definen el environment por defecto compartido por todos los usuarios. Esto es seguido por archivos de startup en tu directorio home que definen tu environment personal.

La secuencia exacta depende del tipo de sesión shell que se inicia. Hay dos tipos:

### Login Shell Session (Sesión Shell de Login)

Una sesión en la que se te pide tu nombre de usuario y contraseña. Esto ocurre cuando accedes a un ambiente gráfico o cuando inicias una consola virtual.

Los shells de login leen uno o más archivos de startup, como se muestra en la tabla siguiente:

| Archivo | Contenido |
|---------|-----------|
| **/etc/profile** | Script de configuración global que se aplica a todos los usuarios. |
| **~/.bash_profile** | Archivo de startup personal del usuario. Puede ser usado para extender o anular las configuraciones globales. |
| **~/.bash_login** | Si `~/.bash_profile` no se encuentra, bash intenta leer este script. |
| **~/.profile** | Si ni `~/.bash_profile` ni `~/.bash_login` se encuentran, bash intenta leer este archivo. Es el default en las distribuciones Debian, como Ubuntu. |

### Non-Login Shell Session (Sesión Shell Sin Login)

Usualmente ocurre cuando inicias una sesión de terminal en la interfaz gráfica con tu emulador de terminal.

Los shells sin login leen los archivos de startup listados en la tabla siguiente:

| Archivo | Contenido |
|---------|-----------|
| **/etc/bash.bashrc** | Script de configuración global que se aplica a todos los usuarios. |
| **~/.bashrc** | Archivo de startup personal del usuario. Puede ser usado para extender o anular las configuraciones globales. |

Además, los shells sin login heredan las variables de environment de su proceso padre, usualmente un shell de login.

## ¿Qué Hay en un Archivo de Startup?

Si observamos un archivo típico `.bash_profile`, se verá algo como esto:

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin
export PATH
```

Las líneas que comienzan con `#` son **comentarios** y no son ejecutadas por el shell. Estas están ahí para la legibilidad humana.

La primera cosa interesante ocurre en la cuarta línea, con el siguiente código:

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

Esta es una **if compound command** (sentencia if compuesta), que cubriremos completamente cuando lleguemos al shell scripting en la Parte IV, pero por ahora, aquí hay una traducción:

```
Si el archivo "~/.bashrc" existe, entonces
    lee el archivo "~/.bashrc".
```

El siguiente elemento interesante es el manejo de la variable `PATH`:

```bash
PATH=$PATH:$HOME/bin
export PATH
```

### Expansión de Parámetros

La variable `PATH` se modifica para añadir el directorio `$HOME/bin` al final de la lista. Esto es un ejemplo de **parameter expansion** (expansión de parámetros), que tocamos en el Capítulo 7. Para demostrar cómo funciona, intenta lo siguiente:

```bash
[me@linuxbox ~]$ foo="This is some "
[me@linuxbox ~]$ echo $foo
This is some
[me@linuxbox ~]$ foo=$foo"text."
[me@linuxbox ~]$ echo $foo
This is some text.
```

Al añadir la cadena `$HOME/bin` al final del contenido de la variable `PATH`, el directorio `$HOME/bin` se añade a la lista de directorios buscados cuando se ingresa un comando. Esto significa que cuando queremos crear un directorio dentro de tu directorio home para almacenar tus propios programas privados, el shell está listo para acomodarnos. Todo lo que tenemos que hacer es llamarlo `bin`, y estamos listos para ir.

> **Nota**: Muchas distribuciones proporcionan este ajuste PATH por defecto. Las distribuciones basadas en Debian, como Ubuntu, prueban la existencia del directorio `~/bin` al login y dinámicamente lo añaden a la variable PATH si el directorio se encuentra.

### Comando `export`

Finalmente, tenemos:

```bash
export PATH
```

El comando `export` le dice al shell que haga el contenido de `PATH` disponible para procesos hijo de este shell. En esencia, lo convierte en una variable de environment.

## Explorando Cómo los Procesos Hijo Heredan Su Environment

Este último punto merece cierta elaboración. Las variables del shell son locales a la instancia actual del shell y no se copian a ningún proceso hijo que lance el shell. Demostremos esto.

Primero, establece una variable del shell en tu shell actual:

```bash
[me@linuxbox ~]$ foo="bar"
```

A continuación, lanza otra copia del shell:

```bash
[me@linuxbox ~]$ bash
[me@linuxbox ~]$
```

Ahora parece que no pasó nada, pero estamos en realidad ejecutando otra instancia de bash. Podemos mostrar esto mirando los resultados del comando `ps`:

```bash
[me@linuxbox ~]$ ps
PID TTY          TIME CMD
1011638 pts/9     00:00:00 bash
1011650 pts/9     00:00:00 bash
1011662 pts/9     00:00:00 ps
```

Aquí vemos que estamos ejecutando dos copias de bash. Como no pusimos la nueva instancia en el background cuando la lanzamos, es ahora la tarea en primer plano, y la instancia original está esperando que esta nueva shell termine. Ahora intenta ver el valor de la variable `foo` que establecemos un momento atrás:

```bash
[me@linuxbox ~]$ echo $foo

[me@linuxbox ~]$
```

No hay resultado, lo que indica que la variable `foo` está vacía. La razón es que no la dimos como valor en esta instancia del shell. Las variables del shell no se copian y se dan a un proceso hijo cuando se crea el proceso hijo. Las variables de environment, por el otro lado, se copian para convertirse en el environment del proceso hijo.

Para demostrar otra característica divertida de los procesos y su environment, vamos a definir `foo` en esta instancia del shell:

```bash
[me@linuxbox ~]$ foo="barbar"
```

A continuación, saldremos de esta instancia de bash y regresaremos a la instancia padre, que ha estado pacientemente esperando que este nuevo shell termine. Ahora intenta ver el valor de `foo` en esta instancia:

```bash
[me@linuxbox ~]$ exit
[me@linuxbox ~]$
```

Correremos `ps` nuevamente para ver que hemos regresado a la primera instancia:

```bash
[me@linuxbox ~]$ ps
PID TTY          TIME CMD
1011638 pts/9     00:00:00 bash
1014900 pts/9     00:00:00 ps
```

Vemos que todavía contiene el valor que le dimos, no el nuevo valor que establecimos en la instancia hijo. Esto muestra una regla importante sobre procesos hijo: **Un proceso hijo no puede alterar el environment de su padre. Cuando un proceso hijo termina, lleva su environment consigo. Este hecho será importante cuando empecemos a escribir shell scripts.**

## Lanzar un Programa con un Environment Temporal

Otro truco conveniente que el shell proporciona es la capacidad de ejecutar un comando y darle una variable de environment temporal. A veces queremos ejecutar un programa y darle un valor especial de environment. Un buen ejemplo es el comando `man`, que busca una variable de environment llamada `MANWIDTH` que le dice a man cuán ancho formatear su salida. Por ejemplo, para tener man formatear su salida a un máximo de 75 caracteres de ancho (un ajuste práctico para lectura fácil), podemos hacer esto:

```bash
[me@linuxbox ~]$ MANWIDTH=75 man ls
```

Esto genera la página man del comando `ls` bonito formateado a un ancho cómodo. Por cierto, esto es una buena cosa para un alias:

```bash
[me@linuxbox ~]$ alias man="MANWIDTH=75 man"
```

Ahora siempre que usemos el comando `man`, siempre limitará la longitud de línea a 75 caracteres.

Para ambientes temporales más avanzados, ver el comando `env`.

## Modificando el Environment

Ahora que sabemos dónde están los archivos de startup y qué contienen, podemos modificarlos para personalizar nuestro environment.

### ¿Qué Archivos Debemos Modificar?

Como regla general, para añadir directorios a tu `PATH` o definir variables de environment adicionales, coloca esos cambios en `.bash_profile` (o equivalente, según tu distribución; por ejemplo, Ubuntu usa `.profile`). Para todo lo demás, coloca los cambios en `.bashrc`.

> **Nota importante**: A menos que seas el administrador del sistema y necesites cambiar los defaults para todos los usuarios del sistema, restringe tus modificaciones a los archivos en tu directorio home. Es ciertamente posible cambiar los archivos en `/etc` como `profile`, y en muchos casos sería sensible hacerlo, pero por ahora, juguemos a lo seguro.

## Editores de Texto

Para editar los archivos de startup del shell, así como la mayoría de otros archivos de configuración en el sistema, usamos un programa llamado **text editor** (editor de texto). Un editor de texto es un programa que, de algunas maneras, es como un procesador de palabras en que nos permite editar palabras en la pantalla con un cursor móvil. Difiere de un procesador de palabras soportando solo texto puro y a menudo contiene características diseñadas para escribir programas.

Muchos editores de texto diferentes están disponibles para Linux; la mayoría de los sistemas tienen varios instalados. Por qué hay tantos editores diferentes? Porque los programadores les encanta escribirlos y como los programadores los utilizan extensamente, escriben editores que expresan sus deseos sobre cómo deberían funcionar.

### Categorías de Editores de Texto

Los editores de texto caen en dos categorías básicas:

#### Editores Gráficos

GNOME y KDE ambos incluyen algunos editores gráficos populares:
- **gedit**: Incluido en GNOME. Usualmente llamado "Text Editor" en el menú GNOME.
- **KDE**: Usualmente viene con dos, que son `kwrite` y el más avanzado `kate`.

#### Editores Basados en Texto

Los populares que encontraremos a menudo son `nano`, `vi`, y `emacs`:
- **nano**: Un editor simple y fácil de usar diseñado como reemplazo para el editor `pico` suministrado con el cliente de correo PINE. El editor `vi` (que en la mayoría de sistemas Linux es reemplazado por un programa llamado `vim`, que significa "vi improved") es el editor tradicional para sistemas similares a Unix. Será el sujeto de nuestro siguiente capítulo. El editor `emacs` fue originalmente escrito por Richard Stallman. Es un gigante, editor de propósito completamente general, que es programable en gran medida. Si bien está disponible, rara vez se instala por defecto en la mayoría de sistemas Linux.

## Usando un Editor de Texto

Los editores de texto se invocan desde la línea de comandos escribiendo el nombre del editor seguido del nombre del archivo que queremos editar. Si el archivo no existe ya, el editor asumirá que queremos crear un nuevo archivo. Aquí hay un ejemplo usando gedit:

```bash
[me@linuxbox ~]$ gedit some_file
```

Este comando iniciará el editor gedit y cargará el archivo llamado `some_file`, si existe.

### Editando `.bashrc` con `nano`

Los editores gráficos son bastante autoexplicativos, así que no los cubriremos aquí. En su lugar, nos enfocaremos en nuestro primer editor basado en texto, nano. Vamos a iniciar nano para editar el archivo `.bashrc`. Pero antes de hacerlo, practicemos algo de "computación segura". Cuando edites un archivo de configuración importante, es una buena idea crear una copia de backup del archivo primero. Esto nos protege en caso de que arruinemos el archivo mientras lo editamos.

Para crear un backup del archivo `.bashrc`, haz esto:

```bash
[me@linuxbox ~]$ cp .bashrc .bashrc.bak
```

Ahora que tenemos un archivo de backup, iniciemos el editor:

```bash
[me@linuxbox ~]$ nano .bashrc
```

Una vez que nano inicia, obtendremos una pantalla como esta:

```
GNU nano 8.6     File: .bashrc

# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi

# User specific aliases and functions

[ Read 8 lines ]
Get Help       WriteOut      Read Fil       Prev Pag      Cut Text      Cur Pos
Exit           Justify       Where Is       Next Pag      UnCut Te      To Spell
```

La pantalla consiste en un encabezado en la parte superior, el texto del archivo siendo editado en el medio, y un menú de comandos en la parte inferior. Como nano fue diseñado para reemplazar el editor de texto suministrado con un cliente de correo, es bastante corto en características de edición.

El primer comando que debemos aprender en cualquier editor de texto es cómo salir del programa. En el caso de nano, presionamos Ctrl+X para salir. Esto se indica en el menú en la parte inferior de la pantalla. La notación `^X` significa Ctrl+X. Este es un convención común para la notación de caracteres de control usados en muchos programas.

El segundo comando que necesitamos saber es cómo guardar nuestro trabajo. Con nano es Ctrl+O. Con este conocimiento, estamos listos para hacer algo de edición. Usando la flecha hacia abajo y/o Page Down, mueve el cursor al final del archivo, y luego agrega las siguientes líneas al archivo `.bashrc`:

```bash
umask 0002
export HISTCONTROL=ignoredups
export HISTSIZE=1000
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
```

### Explicación de las Adiciones

La siguiente tabla detalla el significado de nuestras adiciones:

| Línea | Significado |
|-------|-------------|
| `umask 0002` | Establece el umask para resolver el problema con los directorios compartidos discutidos en el Capítulo 9. |
| `export HISTCONTROL=ignoredups` | Causa que el feature de grabación de historial del shell ignore un comando si el mismo comando fue grabado justo anteriormente. |
| `export HISTSIZE=1000` | Incrementa el tamaño del historial de comandos del default usual de 500 líneas a 1,000 líneas. |
| `alias l.='ls -d .* --color=auto'` | Crea un nuevo comando llamado `l.`, que muestra todas las entradas de directorio que comienzan con un punto. |
| `alias ll='ls -l --color=auto'` | Crea un nuevo comando llamado `ll`, que muestra un listado de formato largo de directorio. |

### La Importancia de los Comentarios

Cuando modifiques archivos de configuración, es una buena idea agregar comentarios para documentar tus cambios. Seguro que probablemente recordarás qué cambiaste mañana, pero ¿qué hay de dentro de seis meses? ¡Hazle un favor a ti mismo y agrega algunos comentarios! Mientras estés en ello, no es una mala idea mantener un registro de qué cambios haces.

Los shell scripts y archivos de startup bash usan un símbolo `#` para comenzar un comentario. Otros archivos de configuración pueden usar otros símbolos. La mayoría de los archivos de configuración tendrán comentarios. Úsalos como una guía.

Verás a menudo líneas en archivos de configuración que están **commented out** (comentadas) para prevenir que sean usadas por el programa afectado. Esto se hace para darle al lector sugerencias para posibles opciones de configuración o ejemplos de sintaxis de configuración correcta. Por ejemplo, el archivo `.bashrc` de Ubuntu 24.04 contiene estas líneas:

```bash
# some more ls aliases
#alias ll='ls -lF'
#alias la='ls -A'
#alias l='ls -CF'
```

Las últimas tres líneas son definiciones de alias válidas que han sido comentadas. Si remuevas los símbolos `#` al comienzo de estas tres líneas, una técnica llamada **uncommenting** (descomentar), activarás los alias. Conversamente, si agregas un símbolo `#` al comienzo de una línea, puedes desactivar una línea de configuración mientras preservas la información que contiene.

## Activando Nuestros Cambios

Los cambios que hemos hecho a nuestro `.bashrc` no tendrán efecto hasta que cerremos nuestra sesión de terminal y iniciemos una nueva porque el archivo `.bashrc` es leído solo al principio de una sesión. Sin embargo, podemos forzar a bash a releer el archivo `.bashrc` modificado con el siguiente comando:

```bash
[me@linuxbox ~]$ source ~/.bashrc
```

Después de hacer esto, deberíamos ser capaces de ver el efecto de nuestros cambios. Intenta uno de los nuevos alias:

```bash
[me@linuxbox ~]$ ll
```

## Sobre el Comando `source`

El comando `source` (que puede ser abreviado como `.`) es un shell builtin que **lee un archivo directamente en el shell actual como si sus contenidos hubieran sido ingresados en el teclado**. Sí, todas esas cosas extrañas que hemos visto en los archivos de startup son simplemente cosas que el shell entiende y puede actuar sobre. Muchos sistemas operativos antiguos, como DOS, CP/M, y así sucesivamente, funcionaban principalmente como simples lanzadores de programas. Los shells Unix pueden hacer mucho más.

## Resumen

En este capítulo, aprendimos una habilidad esencial: editar archivos de configuración con un editor de texto. Avanzando, a medida que leemos páginas de manual para comandos, tomaremos nota de las variables de environment que los comandos soportan. Puede haber una joya o dos. En capítulos posteriores, aprenderemos sobre shell functions, una característica poderosa que también puedes incluir en los archivos de startup de bash para añadir a tu arsenal de comandos personalizados.

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]] — Fundamentos del shell y línea de comandos
- [[wiki/linux/08-trucos-teclado.md|Capítulo 8: Trucos Avanzados del Teclado]] — Configuración de Readline en .bashrc
- [[wiki/linux/10-procesos.md|Capítulo 10: Procesos en Linux]] — Cómo el environment se hereda en procesos hijo
