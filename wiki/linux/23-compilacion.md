---
title: "Capítulo 23: Compilación de Programas"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 23"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-355-364.pdf"
updated: "2026-06-23"
---

# Compilación de Programas

## Introducción

La disponibilidad del código fuente es la libertad esencial que hace posible Linux. Todo el ecosistema de desarrollo de Linux se basa en el libre intercambio de código entre desarrolladores.

**Históricamente**, la compilación era una habilidad común entre usuarios de Linux. Sin embargo, hoy en día, los proveedores de distribuciones mantienen enormes repositorios de binarios precompilados listos para descargar e instalar. Por ejemplo, el repositorio Debian (uno de los más grandes) contiene más de 68,000 paquetes.

**¿Por qué compilar software entonces?**

### Razones para Compilar

**Disponibilidad (Availability):**
Aunque hay muchos programas precompilados en repositorios, algunos programas deseados pueden no estar disponibles en la distribución específica. En este caso, la única forma de obtener el programa deseado es compilarlo desde el código fuente.

**Versiones actualizadas (Timeliness):**
Algunas distribuciones se especializan en versiones de última generación de programas, pero muchas no lo hacen. Esto significa que para tener la última versión de un programa, compilar puede ser necesario.

**Complejidad:**
La compilación de software desde código fuente puede volverse bastante compleja y técnica, más allá del alcance de muchos usuarios. Sin embargo, muchos casos de compilación son fáciles e implican solo unos pocos pasos. Examinaremos un caso simple como punto de partida.

**Nueva herramienta:**
```
make    Utilidad para mantener programas
```

---

## ¿Qué es la Compilación?

Simplemente dicho, la **compilación** es el proceso de traducir **código fuente** (la descripción legible por humanos de un programa escrita por un programador) en el lenguaje nativo del procesador de la computadora.

### Lenguajes de Máquina

El procesador de la computadora (o **CPU**) ejecuta programas en lo que se llama **lenguaje máquina**. Este es un código numérico que describe operaciones extremadamente pequeñas, tales como:
- "Agregar este byte"
- "Apuntar a esta ubicación en memoria"
- "Copiar este byte"

Cada una de estas instrucciones se expresa en binario (unos y ceros). Los primeros programas de computadora se escribían usando este código numérico, lo que puede explicar por qué los programadores que lo escribían fumaban mucho, bebían galones de café y sufrían ataques.

### Lenguajes de Ensamblaje

Este problema fue superado por el advenimiento del **lenguaje de ensamblaje**, que reemplazó los códigos numéricos con características mnemonics (abreviaciones fáciles de usar) como:
- **CPY** (para copy)
- **MOV** (para move)

Los programas escritos en lenguaje de ensamblaje son procesados en lenguaje máquina por un programa llamado un **ensamblador**. El lenguaje de ensamblaje todavía se usa hoy en día para ciertas tareas especializadas de programación, tales como:
- **Device drivers** (controladores de dispositivos)
- **Embedded systems** (sistemas embebidos)

### Lenguajes de Programación de Alto Nivel

A continuación, llegamos a los llamados **lenguajes de programación de alto nivel**. Se llaman así porque permiten al programador preocuparse menos por los detalles de lo que el procesador está haciendo y más con la resolución del problema en cuestión.

**Lenguajes clásicos históricos:**
- **FORTRAN** (diseñado para tareas científicas y técnicas)
- **COBOL** (diseñado para aplicaciones de negocio)

Ambos siguen siendo utilizados hoy en día.

**Lenguajes modernos:**
Actualmente, la mayoría de programas escritos para sistemas modernos se escriben en **C** o **C++**. En los ejemplos que seguiremos, compilaremos un programa C.

### Compiladores

Los programas escritos en lenguajes de programación de alto nivel se convierten en lenguaje máquina por un programa llamado **compilador**. Algunos compiladores traducen instrucciones de alto nivel en lenguaje de ensamblaje y luego usan un ensamblador para realizar la etapa final de traducción en lenguaje máquina.

### Linking (Vinculación)

Un proceso a menudo utilizado en conjunción con la compilación se llama **linking** (vinculación). Hay muchas tareas comunes realizadas por programas. Por ejemplo:
- Abrir un archivo
- Muchos programas realizan esta tarea, pero sería desperdicio tener cada programa implementar su propia rutina para abrir archivos

Es más sensato tener una pieza de programación que sabe cómo abrir archivos y permitir que todos los programas que lo necesiten la compartan.

**Bibliotecas (Libraries):**
El apoyo para tareas comunes se logra a través de lo que se llaman **bibliotecas** (libraries). Contienen múltiples **rutinas**, cada una realizando alguna tarea común que múltiples programas pueden compartir.

**Linker:**
Un programa llamado **linker** es utilizado para formar las conexiones entre la salida del compilador y las bibliotecas que el programa compilado requiere. El resultado final de este proceso es el **archivo ejecutable**, listo para usar.

---

## ¿Están Todos los Programas Compilados?

No. Como hemos visto, hay programas como shell scripts que no requieren compilación. Se ejecutan directamente. Estos se escriben en lo que se conoce como **lenguajes de script** o **lenguajes interpretados**. Estos lenguajes han crecido en popularidad en años recientes e incluyen **Perl, Python, PHP, Ruby** y muchos otros.

### Lenguajes Interpretados vs Compilados

Los **lenguajes interpretados** son ejecutados por un programa especial llamado un **intérprete**. Un intérprete lee el archivo del programa e interpreta (lee y ejecuta) cada instrucción contenida en él.

**Ventajas de programas interpretados:**
- Generalmente más lentos que programas compilados (porque cada instrucción de código fuente se traduce cada vez que se ejecuta)
- Más rápido y fácil desarrollar (programas interpretados)

**Ventajas de programas compilados:**
- Mucho más rápidos que programas interpretados (porque se completa la traducción una sola vez)
- La ventaja real es que es generalmente más rápido y más fácil desarrollar programas interpretados

**Ciclo de desarrollo:**
Los programas son generalmente desarrollados en un ciclo repetitivo de código, compilación, prueba. A medida que un programa crece en tamaño, la fase de compilación puede volverse bastante larga. Los lenguajes interpretados eliminan el paso de compilación y aceleran el desarrollo de programas.

---

## Compilación de un Programa C

Vamos a compilar algo. Antes de hacerlo, sin embargo, necesitaremos algunas herramientas como el compilador, el linker y make. El compilador C utilizado casi universalmente en el entorno Linux se llama **gcc** (GNU C Compiler), originalmente escrito por Richard Stallman. La mayoría de distribuciones no instalan gcc por defecto.

### Verificar si el Compilador está Instalado

Podemos verificar si el compilador está presente así:

```bash
[me@linuxbox ~]$ which gcc
/usr/bin/gcc
```

El resultado indica que el compilador está instalado.

> **Nota:** Tu distribución puede tener un meta-package (una colección de paquetes) para software de desarrollo. Si es así, considera instalar si deseas compilar programas en tu sistema. Si tu sistema no proporciona un meta-package, intenta instalar los paquetes gcc y make. En muchas distribuciones, esto es suficiente para realizar el siguiente ejercicio.

---

## Obtención del Código Fuente

Para nuestro ejercicio de compilación, vamos a compilar un programa del GNU Project llamado **diction**. Este programa útil verifica archivos de texto para calidad de escritura y estilo. Como programas van, es bastante pequeño y será fácil de build.

### Crear Directorio para Código Fuente

Siguiendo convención, primero vamos a crear un directorio para nuestro código fuente llamado `src` y luego descargar el código fuente en él usando ftp.

```bash
[me@linuxbox ~]$ mkdir src
[me@linuxbox ~]$ cd src
[me@linuxbox src]$ ftp ftp.gnu.org
Connected to ftp.gnu.org.
220 GNU FTP server ready.
Name (ftp.gnu.org:me): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd gnu/diction
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 1003  65534   65940 Aug 28  1998 diction-0.7.tar.gz
-rw-r--r--    1 1003  65534   90957 Mar 04  2002 diction-1.02.tar.gz
-rw-r--r--    1 1003  65534  141062 Sep 17  2007 diction-1.11.tar.gz
226 Directory end OK.
ftp> get diction-1.11.tar.gz
local: diction-1.11.tar.gz remote: diction-1.11.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for diction-1.11.tar.gz (141062 bytes).
226 File send OK.
141062 bytes received in 0.16 secs (887.4 kB/s).
ftp> bye
221 Goodbye.
[me@linuxbox src]$ ls
diction-1.11.tar.gz
```

### Descargas Alternativas

Aunque usamos ftp en el ejemplo anterior, que es tradicional, hay otras formas de descargar código fuente. Por ejemplo, el GNU Project también soporta descarga usando HTTPS. Podemos descargar el código fuente diction usando el programa `curl`:

```bash
[me@linuxbox ~]$ curl -O https://ftp.gnu.org/gnu/diction/diction-1.11.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Total    Spent
  Left Speed
100  137k  100  137k    0    0    652k      0 --:--:-- --:--:--
  --:--:--  652k
```

> **Nota:** Dado que somos el "mantenedor" de este código fuente mientras lo compilamos, lo mantendremos en `~/src`. El código fuente instalado por tu distribución se instalará en `/usr/src`, mientras que el código fuente destinado para uso por múltiples usuarios generalmente se instala en `/usr/local/src`.

### Descompresión del Archivo de Código Fuente

Como podemos ver, el código fuente generalmente se suministra en forma de archivo comprimido. A veces llamado un **tarball**, este archivo contiene el **source tree** (árbol de fuentes), o jerarquía de directorios y archivos que comprenden el código fuente.

Después de llegar al sitio FTP, examinamos la lista de archivos disponibles y seleccionamos la versión más nueva para descargar. Usando el comando `get` dentro de ftp, copiamos el archivo del servidor FTP a la máquina local.

Una vez que el archivo tar se descarga, debe desempaquetarse. Esto se realiza con el programa tar.

```bash
[me@linuxbox src]$ tar xzf diction-1.11.tar.gz
[me@linuxbox src]$ ls
diction-1.11.tar.gz
```

> **Nota:** El programa diction, como todo software del GNU Project, sigue ciertos estándares para el código fuente. Un estándar es que cuando se desempaqueta el archivo de código fuente tar, se crea un directorio que contiene el source tree, y este directorio se nombra project-x.xx, encontrado containiendo tanto el nombre del proyecto como su número de versión. Este esquema permite la instalación fácil de múltiples versiones del mismo programa.
>
> Sin embargo, algunos proyectos no crearán el directorio pero en su lugar utilizarán los archivos directamente en el directorio actual. Esto hará un lío en tu directorio src bien organizado. Para evitar esto, usa el siguiente comando para examinar los contenidos del archivo tar:
>
> ```bash
> tar tzf tarfile | head
> ```

---

## Examinación del Árbol de Fuentes

Desempaquetar el archivo tar resulta en la creación de un nuevo directorio, llamado `diction-1.11`. Este directorio contiene el source tree. Veamos adentro:

```bash
[me@linuxbox src]$ cd diction-1.11
[me@linuxbox diction-1.11]$ ls
config.guess    diction.c        getopt.c        nl
config.h.in     diction.pot      getopt.h        nl.po
config.sub      diction.spec     getopt_int.h    README
configure       diction.spec.in  INSTALL         sentence.c
configure.in    diction.texi.in  INSTALL-sh      sentence.h
COPYING         en                Makefile.in     style.i.in
de              en_GB            en_GB.po        style.c
de.po           en_GB.mo          misc.c          style.i.c
diction.texi    en_GB.po          misc.h          test
```

En este directorio, veremos una serie de archivos. Los programas pertenecientes al GNU Project, así como muchos otros, suministrarán archivos de documentación **README, INSTALL, NEWS** y **COPYING**. Estos archivos contienen la descripción del programa, información sobre cómo construirlo e instalarlo, y términos de licencia. **Siempre es una buena idea leer cuidadosamente los archivos README e INSTALL antes de intentar construir el programa.**

### Archivos de Código Fuente

Los otros archivos interesantes en este directorio son los que terminan con `.c` y `.h`:

```bash
[me@linuxbox diction-1.11]$ ls *.c
diction.c  getopt.c  getopt.c  misc.c  sentence.c  style.c
[me@linuxbox diction-1.11]$ ls *.h
getopt.h  getopt_int.h  misc.h  sentence.h
```

Los archivos `.c` contienen los dos programas C suministrados por el paquete (style y diction), divididos en módulos. Es práctica común para grandes programas dividirse en piezas más pequeñas y más fáciles de administrar. Los archivos de código fuente son texto ordinario y pueden examinarse con less.

```bash
[me@linuxbox diction-1.11]$ less diction.c
```

### Archivos Header

Los archivos `.h` se conocen como **header files** (archivos de encabezamiento). Estos, también, son texto ordinario. Los archivos de encabezamiento contienen descripciones de las rutinas incluidas en un archivo de código fuente o biblioteca. Para que el compilador conecte los módulos, debe recibir una descripción de todos los módulos necesarios para completar el programa entero.

En la parte superior del archivo `diction.c`, vemos esta línea:

```c
#include "getopt.h"
```

Esto instruye al compilador a leer el archivo `getopt.h` mientras lee el código fuente en `diction.c` para "conocer" qué está en `getopt.c`. El archivo `getopt.c` suministra rutinas que son compartidas por los programas style y diction.

### Includes del Sistema

Antes de la declaración `#include` para `getopt.h`, vemos algunas otras declaraciones `#include` como estas:

```c
#include <regex.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
```

Estos también refieren a archivos header, pero se refieren a archivos header que viven fuera del source tree actual. Son suministrados por el sistema para soportar la compilación de cada programa. Si los buscamos en `/usr/include`, podemos verlos.

```bash
[me@linuxbox diction-1.11]$ ls /usr/include
```

Los archivos header en este directorio fueron instalados cuando instalamos el compilador.

---

## Compilación del Programa

La mayoría de programas se construyen con una secuencia simple de dos comandos.

```bash
./configure
make
```

### El Script `configure`

El programa `configure` es un shell script que es suministrado con el código fuente. Su trabajo es analizar el **build environment**. La mayoría del código fuente está diseñado para ser **portable**. Esto es, está diseñado para construir en más de un tipo de sistema Unix-like. Sin embargo, para hacer eso, el código fuente puede necesitar someterse a ajustes leves durante la construcción para acomodar diferencias entre sistemas.

`configure` también verifica para ver que herramientas externas necesarias y componentes están instalados. Dado que `configure` no se encuentra donde el shell normalmente espera programas, debemos explícitamente decirle al shell su ubicación prefijando el comando con `./` para indicar que el programa se encuentra en el directorio de trabajo actual.

```bash
[me@linuxbox diction-1.11]$ ./configure
```

Cuando se ejecuta, `configure` mostrará una gran cantidad de mensajes mientras prueba y configura la construcción. Cuando termina, se verá algo como esto:

```
checking libintl.h presence... yes
checking for libintl.h... yes
checking for library containing gettext... none required
configure: creating ./config.status
config.status: creating Makefile
config.status: creating diction.i
config.status: creating diction.texi
config.status: creating diction.spec
config.status: creating style.i
config.status: creating test/runidiction
config.status: creating config.h
[me@linuxbox diction-1.11]$
```

Lo importante aquí es que no hay mensajes de error. Si hay alguno, la configuración habrá fallado, y el programa no se construirá hasta que los errores sean corregidos.

Vemos que `configure` creó varios archivos nuevos en nuestro directorio source. El más importante es el **makefile**. El makefile es un archivo de configuración que instruye al programa make exactamente cómo construir el programa.

### Ver el Makefile

Sin el makefile, make se negará a ejecutar. El makefile es un archivo ordinario, así que podemos verlo.

```bash
[me@linuxbox diction-1.11]$ less Makefile
```

### Estructura del Makefile

El programa `make` toma como entrada un **makefile** (normalmente nombrado Makefile), que describe las relaciones y dependencias entre componentes que comprenden el programa terminado.

La primera parte del makefile define variables que son sustituidas en secciones posteriores del makefile. Por ejemplo, vemos la siguiente línea:

```
CC=        gcc
```

Que define el compilador C como gcc. Más tarde en el makefile, veremos una instancia donde se obtiene usado.

```
diction:    diction.o sentence.o misc.o getopt.o getopt1.o \
$(CC) -o $@ $(LDFLAGS) diction.o sentence.o misc.o \
getopt.o getopt1.o $(LIBS)
```

Una sustitución se realiza aquí, y el valor `$(CC)` es reemplazado por gcc en tiempo de ejecución.

### Targets y Dependencies

La mayoría del makefile consiste en líneas, que definen un **target** (objetivo), en este caso el archivo ejecutable `diction` y los archivos en los que depende. El makefile restante describe los comandos necesarios para crear el target de sus componentes.

Vemos en este ejemplo que el archivo ejecutable `diction` (uno de los productos finales) depende de la existencia de `diction.o, sentence.o, misc.o, getopt.o,` y `getopt1.o`. Las líneas restantes describen cada una de éstos como targets.

Sin embargo, no vemos ningún comando especificado para ellos. Esto es manejado por un target general, más temprano en el archivo, que describe el comando usado para compilar cualquier archivo `.c` en un archivo `.o`:

```
.c.o:
    $(CC) -c $(CPPFLAGS) $(CFLAGS) $<
```

Esto suena muy complicado. ¿Por qué no simplemente enumerar todos los pasos para compilar las piezas y estar hecho con esto? La respuesta es que se hará clara en un momento. Mientras tanto, ejecutemos make y construyamos nuestros programas.

```bash
[me@linuxbox diction-1.11]$ make
```

El programa `make` ejecutará, usando los contenidos de Makefile para guiar sus acciones. Producirá una gran cantidad de mensajes.

### Verificación de Compilación Exitosa

Cuando termina, veremos que todos los targets ahora están presentes en nuestro directorio.

```bash
[me@linuxbox diction-1.11]$ ls
config.guess    de.po          en_GB            sentence.c
config.h        diction         en_GB.mo         sentence.h
config.h.in     en_GB.po        Makefile         sentence.o
config.log      diction.i       diction.i.in     sentence.c
config.sub      diction.o       Makefile.in      style
config.status   diction.spec    misc.c           style.i.in
diction.texi    diction.spec.in misc.h           test
```

Entre los archivos, vemos `diction` y `style`, los programas que configuramos para construir. ¡Felicitaciones, estamos en orden! Acabamos de compilar nuestros primeros programas desde código fuente!

### Magic de Make: Actualización Inteligente

Pero solo por curiosidad, ejecutemos make nuevamente para ver qué hace. Ejecutemos make y construyamos nuestros programas.

```bash
[me@linuxbox diction-1.11]$ make
make: Nothing to be done for `all'.
```

Solo produce este extraño mensaje. ¿Qué está pasando? ¿Por qué no se construyó el programa nuevamente? ¡Ah, aquí está la magia de make! En lugar de simplemente construir todo de nuevo, make construye solo lo que necesita construir. Con todos los targets presentes, make determinó que no había nada que hacer. Podemos demostrar esto borrando uno de los targets intermedios y ejecutar make nuevamente para ver qué hace.

```bash
[me@linuxbox diction-1.11]$ rm getopt.o
[me@linuxbox diction-1.11]$ make
```

Vemos que make reconstruye y revincula los programas `diction` y `style`, ya que dependen del módulo faltante. Este comportamiento también señala otra característica importante de make: mantiene los targets actualizados.

### Actualización Automática de Targets

`make` insiste en que los targets sean más nuevos que las dependencias. Esto tiene sentido perfecto, porque un programador a menudo actualizará un bit de código fuente y luego usar make para construir una nueva versión del producto terminado. `make` asegura que todo lo que necesita ser construido basado en el código fuente actualizado sea construido.

Si utilizamos el programa `touch` para "actualizar" uno de los archivos de código fuente, veremos que esto suceda:

```bash
[me@linuxbox diction-1.11]$ ls -l diction getopt.c
-rwxr-xr-x 1 me    me   37164 2025-03-05 06:14 diction
-rw-r--r-- 1 me    me   33125 2007-03-30 17:45 getopt.c
[me@linuxbox diction-1.11]$ touch getopt.c
[me@linuxbox diction-1.11]$ ls -l diction getopt.c
-rwxr-xr-x 1 me    me   37164 2009-03-05 06:23 diction
-rw-r--r-- 1 me    me   33125 2025-03-05 06:23 getopt.c
[me@linuxbox diction-1.11]$ make
```

Después de que make se ejecuta, vemos que ha restaurado el target a ser más nuevo que la dependencia.

```bash
[me@linuxbox diction-1.11]$ ls -l diction getopt.c
-rwxr-xr-x 1 me    me   37164 2009-03-05 06:23 diction
-rw-r--r-- 1 me    me   33125 2009-03-05 06:23 getopt.c
```

La capacidad de make para inteligentemente construir solo lo que necesita construir es un gran beneficio para programadores. Aunque el ahorro de tiempo puede no ser aparente con nuestro pequeño proyecto, es significante con proyectos más grandes. Recuerda, el kernel de Linux (un programa que sufre modificación continua y mejora) contiene varios **millones** de líneas de código.

---

## Instalación del Programa

Los paquetes de código fuente a menudo incluirán un objetivo especial de make llamado `install`. Este objetivo instalará el producto final en un directorio de sistema para uso. Usualmente, este directorio es `/usr/local/bin`, la ubicación tradicional para software construido localmente. Sin embargo, este directorio no es normalmente escribible por usuarios ordinarios, así que debemos convertirnos en el superusuario para realizar la instalación.

```bash
[me@linuxbox diction-1.11]$ sudo make install
```

Después de realizar la instalación, podemos verificar que el programa esté listo para ir.

```bash
[me@linuxbox diction-1.11]$ which diction
/usr/local/bin/diction
[me@linuxbox diction-1.11]$ man diction
```

¡Ahí lo tenemos!

---

## Resumen

En este capítulo, vimos cómo estos tres comandos simples pueden ser usados para construir muchos paquetes de código fuente: `./configure && make && make install`. También vimos el importante papel que make juega en el mantenimiento de programas.

Podemos usar el programa `make` para cualquier tarea que necesite mantener una relación target/dependencia, no solo para compilar código fuente.

---

## Ver También

- [[wiki/linux/14-gestion-paquetes.md|Capítulo 14: Gestión de Paquetes en Linux]] — Instalación de paquetes precompilados desde repositorios
- [[wiki/linux/05-trabajo-con-comandos.md|Capítulo 5: Trabajo con Comandos]] — Cómo usar help, man, info para documentación
