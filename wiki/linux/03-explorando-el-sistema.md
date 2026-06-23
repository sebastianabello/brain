---
title: "Capítulo 3: Explorando el Sistema"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 3"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-49-58.pdf"
updated: "2026-06-22"
---

# Explorando el Sistema Linux

## Introducción

Ahora que sabemos cómo movernos por el sistema de archivos, es momento de hacer un tour guiado por nuestro sistema Linux. Pero antes, aprenderemos algunos comandos más útiles:

- **`ls`** — Listar contenidos de directorio (formas avanzadas)
- **`file`** — Determinar el tipo de archivo
- **`less`** — Ver contenidos de archivo

## Más Diversión con el Comando `ls`

El comando `ls` es probablemente el comando más usado en Linux. Nos permite ver contenidos de directorio y determinar varios atributos importantes de archivos y directorios.

### Sintaxis Básica

Simplemente escribiendo `ls` obtenemos una lista de archivos y subdirectorios en el directorio actual:

```bash
[me@linuxbox ~]$ ls
Desktop  Documents  Music  Pictures  Public  Templates  Videos
```

### Listar Múltiples Directorios

Podemos especificar diferentes directorios para listar:

```bash
[me@linuxbox ~]$ ls /usr
bin  games  include  lib  local  sbin  share  src
```

Incluso puedes listar múltiples directorios. En este ejemplo, listamos el directorio home del usuario (representado por `~`) y el directorio `/usr`:

```bash
[me@linuxbox ~]$ ls ~ /usr
/home/me:
Desktop  Documents  Music  Pictures  Public  Templates  Videos
/usr:
bin  games  include  lib  local  sbin  share  src
```

## Opciones y Argumentos

Esto nos lleva a un punto importante sobre cómo funcionan la mayoría de comandos en Linux. Los comandos generalmente van seguidos por una o más **opciones** que modifican su comportamiento y, además, por uno o más **argumentos** (los elementos sobre los cuales el comando actúa).

La estructura general es:

```
comando -opciones argumentos
```

### Tipos de Opciones

La mayoría de comandos usan opciones que consisten en un carácter precedido por un guion, por ejemplo `-l`. Muchos comandos, particularmente los del Proyecto GNU, también soportan **long options** (opciones largas), que consisten en una palabra precedida por dos guiones.

Por ejemplo, el comando `ls` recibe dos opciones:
- `-l` produce formato largo
- `-t` ordena resultados por tiempo de modificación

Muchos comandos permiten encadenar múltiples opciones cortas. En el siguiente ejemplo, `ls -lt` combina las dos opciones anteriores:

```bash
[me@linuxbox ~]$ ls -lt
```

También puedes agregar long options. Por ejemplo, para revertir el orden de la clasificación, agrega `--reverse`:

```bash
[me@linuxbox ~]$ ls -lt --reverse
```

> **Nota**: Las opciones de comandos, como los nombres de archivos en Linux, distinguen entre mayúsculas y minúsculas.

## Opciones Comunes de `ls`

| Opción | Opción Larga | Descripción |
|--------|-------------|-------------|
| `-a` | `--all` | Lista todos los archivos, incluso los ocultos (los que comienzan con punto) |
| `-A` | `--almost-all` | Como la opción `-a` excepto que no lista `.` (directorio actual) y `..` (directorio padre) |
| `-d` | `--directory` | Ordinariamente, si se especifica un directorio, `ls` lista los contenidos del directorio, no el directorio mismo. Usa esta opción con `-l` para ver detalles del directorio |
| `-F` | `--classify` | Agrega un carácter indicador al final de cada nombre listado. Por ejemplo, una barra inclinada `/` si el nombre es un directorio |
| `-h` | `--human-readable` | En listados de formato largo, muestra tamaños de archivo en formato legible por humanos en lugar de en bytes |
| `-l` | — | Muestra resultados en formato largo |
| `-r` | `--reverse` | Muestra resultados en orden inverso. Normalmente, `ls` muestra resultados en orden alfabético ascendente |
| `-S` | — | Ordena resultados por tamaño de archivo |
| `-t` | — | Ordena por tiempo de modificación |

## Una Mirada Más Profunda al Formato Largo

La opción `-l` causa que `ls` muestre sus resultados en formato largo. Este formato contiene mucha información útil:

```bash
[me@linuxbox ~]$ ls -l
-rw-r--r-- 1 root root 3576296 2017-04-03 11:05 Experience.ubuntu.ogg
-rw-r--r-- 1 root root 1186219 2017-04-03 11:05 ubuntu-leaflet.png
-rw-r--r-- 1 root root  47584  2017-04-03 11:05 logo-Edubuntu.png
-rw-r--r-- 1 root root  44355  2017-04-03 11:05 logo-Ubuntu.png
-rw-r--r-- 1 root root  34391  2017-04-03 11:05 logo-Ubundutu.png
-rw-r--r-- 1 root root  32059  2017-04-03 11:05 oo-cd-cover.pdf
-rw-r--r-- 1 root root 159744  2017-04-03 11:05 oo-derivatives.doc
-rw-r--r-- 1 root root  27837  2017-04-03 11:05 oo-maxwell.odt
-rw-r--r-- 1 root root  98816  2017-04-03 11:05 oo-trig.xls
```

### Desglose de los Campos del Formato Largo

| Campo | Significado |
|-------|------------|
| `-rw-r--r--` | **Permisos del archivo**. El primer carácter indica el tipo de archivo (el guion significa archivo regular). Los próximos tres caracteres son permisos para el propietario, los siguientes tres para miembros del grupo, y los últimos tres para otros. El Capítulo 9 discute el significado completo de esto. |
| `1` | **Número de hard links** del archivo. Ver las secciones "Enlaces Simbólicos" y "Enlaces Duros" más adelante. |
| `root` | **Nombre del usuario propietario** del archivo. |
| `root` | **Nombre del grupo propietario** del archivo. |
| `3576296` | **Tamaño del archivo** en bytes. |
| `2017-04-03 11:05` | **Fecha y hora de la última modificación** del archivo. |
| `Experience.ubuntu.ogg` | **Nombre del archivo**. |

## Determinando el Tipo de Archivo con `file`

Mientras exploramos el sistema, será útil saber qué tipo de datos contienen los archivos. Para hacer esto, usamos el comando **`file`** (archivo) para determinar el tipo de un archivo.

El comando se invoca de la siguiente manera:

```
file nombre_archivo
```

Cuando se invoca, el comando `file` imprimirá una breve descripción de los contenidos del archivo. Por ejemplo:

```bash
[me@linuxbox ~]$ file picture.jpg
picture.jpg: JPEG image data, JFIF standard 1.01
```

### Filosofía Unix: "Todo es un Archivo"

Una de las ideas básicas en sistemas Unix-like como Linux es que "todo es un archivo". A medida que progresemos con nuestras lecciones, veremos cuán cierto es esto.

Existen muchos tipos de archivos. De hecho, uno de los conceptos básicos de Unix es que "todo es un archivo". A medida que progresemos, veremos exactamente cuán cierto es esto.

Aunque muchos archivos en nuestro sistema son familiares (como MP3 y JPEG), hay muchos tipos que son un poco menos obvios, y algunos que son bastante extraños.

## Viendo Contenidos de Archivos con `less`

### Introducción a `less`

El comando **`less`** es un programa para ver archivos de texto. A lo largo de nuestro sistema Linux, muchos archivos contienen texto legible por humanos.

¿Por qué querríamos examinar archivos de texto? Porque muchos archivos que configuran los ajustes del sistema (llamados **archivos de configuración**) se almacenan en este formato, y algunos de los programas reales que usa el sistema (llamados **scripts**) también se almacenan en este formato.

El comando `less` se usa como:

```
less nombre_archivo
```

### ¿Qué es "Texto"?

Existen muchas formas de representar información en una computadora. Todos los métodos implican definir una relación entre la información y algún número que se usará para representarla.

Las computadoras, después de todo, solo entienden números, y todos los datos se convierten a representación numérica.

Existen sistemas de representación complejos (como archivos de vídeo comprimido) y otros simples. Uno de los más antiguos y simples se llama **texto ASCII**.

**ASCII** (American Standard Code for Information Interchange, pronunciado "as-key") es una simple codificación que fue utilizada por primera vez en máquinas de teletipo para mapear caracteres del teclado a números.

El texto es una forma de mapeo simple de caracteres a números. Es compacto: cincuenta caracteres de texto se traducen a 50 bytes de datos.

Es importante entender que el texto contiene solo un mapeo simple de caracteres a números. A diferencia de un documento de procesador de texto como los creados por Microsoft Word o LibreOffice Writer (que contienen muchos elementos no imprimibles para describir su estructura y formato), los archivos ASCII de texto simple contienen solo los caracteres mismos y códigos de control rudimentarios como tabulaciones, retornos de carro y saltos de línea.

A lo largo del sistema Linux, muchos archivos se almacenan en formato de texto, y hay muchas herramientas de Linux que funcionan con archivos de texto. Incluso Windows reconoce la importancia de este formato. El conocido programa NOTEPAD.EXE es un editor para archivos de texto ASCII simples.

### Usando `less`

Una vez que comienza el programa `less`, podemos ver los contenidos del archivo. Si el archivo es más largo que una página, podemos desplazarnos hacia arriba y hacia abajo.

Para salir de `less`, presiona `q`.

### Comandos de Navegación en `less`

| Comando | Acción |
|---------|--------|
| `PAGE UP` o `b` | Desplaza hacia atrás una página |
| `PAGE DOWN` o `SPACE` | Desplaza hacia adelante una página |
| Flecha arriba | Desplaza hacia arriba una línea |
| Flecha abajo | Desplaza hacia abajo una línea |
| `G` | Va al final del archivo de texto |
| `1G` o `g` | Va al principio del archivo de texto |
| `/caracteres` | Busca hacia adelante la próxima ocurrencia de *caracteres* |
| `n` | Busca la próxima ocurrencia de la búsqueda anterior |
| `h` | Muestra pantalla de ayuda |
| `q` | Salir de `less` |

### `less` es Más

El programa `less` fue diseñado como un reemplazo mejorado de un programa Unix anterior llamado `more`.

El nombre `less` es un juego de palabras sobre la frase "less is more" —un lema de arquitectos modernistas y diseñadores.

`less` cae en la clase de programas llamados **pagers**, programas que permiten la visualización fácil de documentos de texto largo de manera página por página. Mientras que el programa más antiguo solo podía avanzar hacia adelante, el programa `less` permite desplazarse tanto hacia adelante como hacia atrás y tiene muchas otras características también.

## Tour Guiado del Sistema

El diseño del sistema de archivos de un sistema Linux es muy similar al encontrado en otros sistemas Unix-like. El diseño está realmente especificado en un estándar publicado llamado el **Linux Filesystem Hierarchy Standard**. No todas las distribuciones de Linux se ajustan exactamente al estándar, pero la mayoría lo hace bastante bien.

Ahora vamos a explorar nuestro sistema Linux para ver qué lo hace funcionar. Esto nos dará una oportunidad de practicar nuestras habilidades de navegación. Una cosa que descubriremos es que muchos archivos interesantes están en texto plano legible por humanos.

### Metodología del Tour

1. `cd` en un directorio dado
2. Lista los contenidos del directorio con `ls -l`
3. Si ves un archivo interesante, determina su contenido con `file`
4. Si parece que podría ser texto, intenta verlo con `less`

Si accidentalmente intentas ver un archivo que no es de texto y se revuelve la ventana del terminal, puedes recuperarte ingresando el comando `reset`.

> **Nota**: Recuerda el truco de copiar y pegar. Si estás usando un ratón, puedes hacer doble clic en un nombre de archivo para copiarlo, y hacer clic con el botón central para pegarlo en comandos.

Como exploramos, no tengas miedo de buscar cosas. Los usuarios regulares están prohibidos de desordenar cosas. ¡Ese es el trabajo del administrador del sistema! Si un comando se queja de algo, simplemente continúa hacia algo más. Pasa tiempo explorando. El sistema es nuestro para explorar. Recuerda: en Linux, no hay secretos.

## Directorios del Sistema Linux

La siguiente tabla lista solo algunos de los directorios que podemos explorar. Puede haber algunas ligeras diferencias dependiendo de nuestra distribución Linux. ¡Así que no tengas miedo de buscar y probar más!

| Directorio | Descripción |
|-----------|------------|
| `/` | El directorio raíz, donde todo comienza. |
| `/bin` | Contiene binarios (programas) que deben estar presentes para que el sistema inicie y se ejecute. Nota que en las distribuciones modernas de Linux se ha deprecado `/bin` en favor de `/usr/bin` (ver `/usr/bin` entry). |
| `/boot` | Contiene el kernel de Linux, la imagen RAM del disco inicial (para drivers necesarios en tiempo de inicio), y el cargador de arranque. Archivos interesantes en `/boot/grub/grub.cfg` o `menu.lst`, que son usados para configurar el cargador de arranque, y `/boot/vmlinuz`, que es el kernel de Linux. |
| `/dev` | Este es un directorio especial que contiene nodos de dispositivo. "Todo es un archivo" también aplica a dispositivos. Aquí es donde el kernel mantiene una lista de todos los dispositivos que comprende. |
| `/etc` | El directorio `/etc` contiene todos los archivos de configuración del sistema. Todo en este directorio debería ser legible. Aunque todo en `/etc` es interesante, aquí hay algunos todos los tiempos favoritos: `/etc/crontab`, que define cuándo se ejecutarán trabajos automatizados; `/etc/hosts`, una tabla de dispositivos de almacenamiento y sus puntos de montaje asociados; y `/etc/passwd`, una lista de las cuentas de usuario. |
| `/home` | En configuraciones normales, cada usuario recibe un directorio en `/home`. Usuarios ordinarios solo pueden escribir archivos en sus directorios personales. Esta limitación protege el sistema de errores de usuario. |
| `/lib` | Contiene archivos de biblioteca compartida utilizados por los programas básicos del sistema. Estos son similares a las librerías de enlace dinámico (DLLs) en Windows. Este directorio ha sido deprecado en distribuciones modernas en favor de `/usr/lib`. |
| `/lost+found` | Cada partición o dispositivo usando un sistema de archivos tradicional de Linux, como ext4, tendrá este directorio. Se usa en caso de recuperación parcial de un sistema de corrupción de sistema de archivos. A menos que algo realmente malo haya sucedido a tu sistema, este directorio permanecerá vacío. |
| `/media` | En sistemas Linux modernos, el directorio `/media` contendrá puntos de montaje para medios removibles como unidades USB, CD-ROMs, y similares, que se montan automáticamente en la inserción. |
| `/mnt` | En sistemas Linux más antiguos, el directorio `/mnt` contiene puntos de montaje para dispositivos que han sido montados manualmente. |
| `/opt` | El directorio `/opt` se usa para instalar software "opcional". Esto es principalmente usado para sostener productos de software comercial que pueden haber sido instalados en el sistema. |
| `/proc` | El directorio `/proc` es especial. No es un sistema de archivos real en el sentido de archivos almacenados en el disco duro. Más bien, es un sistema de archivos virtual mantenido por el kernel de Linux. Los "archivos" aquí son orificios de ventilación en el kernel. Los archivos son legibles y te darán una imagen de cómo el kernel ve la computadora. Explorar este directorio puede revelar muchos detalles sobre el hardware. |
| `/root` | Este es el directorio principal para la cuenta de root. |
| `/run` | En sistemas modernos, `/run` se utiliza para almacenar datos temporales y transitorios creados por varios programas. Algunos distribuciones están vacías `/tmp` cada vez que el sistema se reinicia. |
| `/sbin` | Este directorio contiene "binarios del sistema". Estos son programas que realizan tareas vitales del sistema que generalmente están reservadas para el superusuario. Nota que en distribuciones modernas de Linux se ha deprecado `/sbin` en favor de `/usr/sbin` (ver entrada `/usr/sbin`). |
| `/sys` | El directorio `/sys` contiene información sobre dispositivos que han sido detectados por el kernel. Esto es mucho como los contenidos del directorio `/dev` pero más detallado, incluyendo cosas de direcciones de hardware reales. |
| `/tmp` | El directorio `/tmp` está destinado a almacenar archivos temporales y transitorios creados por varios programas. Algunas distribuciones vacían este directorio cada vez que el sistema se reinicia. |
| `/usr` | El árbol de directorio `/usr` es probablemente el más grande en un sistema Linux. Contiene todos los programas y datos de apoyo utilizados por usuarios regulares. |
| `/usr/bin` | `/usr/bin` contiene los programas ejecutables instalados por la distribución de Linux. No es común sostener miles de programas aquí. |
| `/usr/lib` | Las bibliotecas compartidas para los programas en `/usr/bin`. |
| `/usr/local` | El árbol `/usr/local` es donde los programas que no se incluyen en la distribución pero que están destinados para el uso de todo el sistema se instalan. Los programas compilados a partir del código fuente normalmente se instalan en `/usr/local/bin`. En un sistema Linux recién instalado, este árbol existe, pero estará vacío hasta que el administrador del sistema agregue algo. |
| `/usr/sbin` | Contiene más programas de administración del sistema. |
| `/usr/share` | `/usr/share` contiene todos los datos compartidos utilizados por programas en `/usr/bin`. Esto incluye cosas como archivos de configuración predeterminados, iconos, fondos de pantalla, archivos de sonido, y así sucesivamente. |
| `/usr/share/doc` | La mayoría de los paquetes instalados en el sistema incluirán algún tipo de documentación. En `/usr/share/doc`, encontraremos documentación organizada por paquete. |
| `/var` | Con la excepción de `/tmp` y `/home`, el directorio `/var` tree es donde los datos que probablemente cambian están almacenados. Esto incluye bases de datos, archivos de correo, archivos de registro, y etc. |
| `/var/log` | El directorio `/var/log` contiene registros de actividades. Estos son archivos importantesy deberían ser monitoreados de vez en cuando. El más útil es `/var/log/messages` (y en algunos sistemas `/var/log/syslog` a través de estos no están disponibles en todos los sistemas). Nota que por razones de seguridad, algunos sistemas solo permiten al superusuario ver registros. |
| `~/.config` y `~/.local` | Estos dos directorios están ubicados en el directorio personal de cada usuario de escritorio. Se usan para almacenar configuración específica del usuario y datos de estado para aplicaciones de escritorio. |

## Enlaces Simbólicos (Symbolic Links)

Mientras exploramos, es probable que veamos un listado de directorio (por ejemplo, en `/usr/lib`) con una entrada como esta:

```
lrwxrwxrwx 1 root root  11 2025-08-11 07:34 libc.so.6 -> libc-2.6.so
```

Observa cómo la primera letra del listado es `l` y la entrada parece tener dos nombres de archivo. Esto es un tipo especial de archivo llamado un **enlace simbólico** (también conocido como **soft link** o **symlink**).

### ¿Qué es un Enlace Simbólico?

En la mayoría de sistemas Unix-like, es posible tener un archivo referenciado por múltiples nombres. Aunque el valor de esto puede no ser obvio, es realmente una característica útil.

**Imagina este escenario**: Un programa requiere el uso de un recurso compartido de algún tipo contenido en un archivo llamado `foo`, pero `foo` tiene versiones frecuentes. Sería bueno incluir el número de versión en el nombre del archivo para que el administrador u otro interesado pudiera ver qué versión de `foo` está instalada.

Esto presenta un problema. Si cambiamos el nombre del recurso compartido, tenemos que rastrear cada programa que podría usar e cambiar para buscar un nombre de archivo nuevo cada vez que se instala una nueva versión del recurso.

**Aquí es donde los enlaces simbólicos ahorran el día.** Supongamos que instalamos versión 2.6 de `foo`, la cual tiene el nombre de archivo `foo-2.6`, y luego creamos un enlace simbólico simplemente llamado `foo` que apunta a `foo-2.6`. Esto significa que cuando un programa abre el archivo `foo`, en realidad está abriendo el archivo `foo-2.6`. Ahora todos los programas que se basan en `foo` pueden encontrarlo, y los programas que se basan en `foo-2.6` también lo pueden encontrar.

Cuando es hora de actualizar a `foo-2.7`, simplemente agregamos el archivo a nuestro sistema, eliminamos el enlace simbólico `foo` anterior y creamos uno nuevo apuntando a la nueva versión. No solo resuelve el problema de actualización de versión, sino que también te permite mantener versiones anteriores en el sistema. Imagina que `foo-2.7` tiene un bug (¡sorpresa, desarrolladores!) y necesitamos revertir a la versión anterior. Solo eliminamos el enlace simbólico apuntando a la versión nueva y creamos uno nuevo apuntando a la versión anterior. No solo resuelve el problema de actualización de versión, sino que también te permite mantener versiones anteriores en el sistema. Imagina que `foo-2.7` tiene un bug (¡sorpresa, desarrolladores!) y necesitamos revertir a la versión anterior. Solo eliminamos el enlace simbólico apuntando a la versión nueva y creamos uno nuevo apuntando a la versión anterior.

El listado de directorio al principio de esta sección (del directorio `/usr/lib` de un sistema Fedora) muestra un enlace simbólico llamado `libc.so.6` que apunta a un archivo compartido de biblioteca llamado `libc-2.6.so`. Esto significa que los programas que buscan el archivo `libc.so.6` en realidad obtendrán el archivo `libc-2.6.so`. Aprenderemos cómo crear enlaces simbólicos en el siguiente capítulo.

## Enlaces Duros (Hard Links)

Mientras estamos en el tema de enlaces, necesitamos mencionar que hay un segundo tipo de enlace llamado **hard link**. Los hard links también permiten archivos que tengan múltiples nombres, pero lo hacen de una manera diferente.

Hablaremos más sobre las diferencias entre enlaces simbólicos y hard links en el siguiente capítulo.

---

## Ver También

- [[wiki/linux/02-navegacion-sistema-archivos.md|Capítulo 2: Navegación del Sistema de Archivos]]
- [[wiki/linux/04-manipulacion-de-archivos.md|Capítulo 4: Manipulación de Archivos]]
- [[wiki/linux/comandos-basicos.md|Referencia de Comandos Básicos]]
