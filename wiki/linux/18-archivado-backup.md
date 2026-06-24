---
title: "Capítulo 18: Archivado y Copias de Seguridad (Backup)"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 18"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-253-267.pdf"
updated: "2026-06-23"
---

# Archivado y Copias de Seguridad (Backup)

## Introducción

Una de las tareas principales del administrador de sistemas es **mantener los datos del sistema seguros**. Una forma de hacerlo es realizar **copias de seguridad periódicas** de los archivos del sistema. Aunque no todos los usuarios son administradores de sistemas, es común necesitar:

- Hacer copias de colecciones de archivos
- Mover grandes cantidades de archivos de un lugar a otro
- Transferir archivos entre dispositivos

En este capítulo cubrimos varias herramientas comunes para **gestionar colecciones de archivos**:

**Herramientas de compresión:**
- **`gzip`**: Comprime o expande archivos
- **`bzip2`**: Compresor de archivos con ordenamiento por bloques

**Herramientas de archivado:**
- **`tar`**: Utilidad clásica de archivado de cintas (tape archive)
- **`zip`**: Empaqueta y comprime archivos

**Herramienta de sincronización:**
- **`rsync`**: Sincronización remota de archivos y directorios

---

## Compresión de Archivos

### Concepto Fundamental: Redundancia en Datos

A lo largo de la historia de la informática, siempre ha existido una lucha por **guardar la mayor cantidad de datos en el menor espacio disponible**, ya sea memoria, dispositivos de almacenamiento o ancho de banda de red.

**Compresión de datos** es el proceso de **eliminar redundancia** de los datos.

#### Ejemplo de Compresión

Imagina una imagen completamente negra con dimensiones de 100 × 100 píxeles. Si cada píxel se almacena con 3 bytes (formato de color RGB), la imagen ocuparía:

```
100 × 100 × 3 = 30,000 bytes
```

Sin embargo, si la imagen es completamente negra, todos los píxeles contienen datos redundantes. Podríamos:
- Describir el hecho: "tenemos un bloque de 10,000 píxeles negros"
- Usar **run-length encoding** (codificación por longitud de serie): representar los datos con "10,000 seguido de cero"

De este modo, se reduce significativamente el tamaño almacenado.

### Tipos de Compresión

**Compresión lossless (sin pérdida):**
- Preserva todos los datos del original
- Cuando se restaura un archivo desde una versión comprimida, es idéntico al original sin comprimir
- Ejemplo: archivos de texto, datos de software

**Compresión lossy (con pérdida):**
- Elimina datos durante la compresión
- El archivo restaurado no coincide con el original; es una aproximación
- Ejemplos: JPEG (imágenes), MP3 (música)

En este capítulo nos enfocamos **exclusivamente en compresión lossless**, ya que la mayoría de datos en computadoras no pueden tolerar pérdida de datos.

---

## gzip — Compresión Rápida

### Concepto General

**`gzip`** es un programa usado para **comprimir o expandir archivos**. Cuando se ejecuta:
- Reemplaza el archivo original con una versión comprimida
- El archivo comprimido tiene la extensión `.gz`
- El programa `gunzip` se usa para restaurar archivos comprimidos a su forma original sin comprimir

### Ejemplo Básico

```bash
[me@linuxbox ~]$ ls -l /etc | head -n 1
-rw-r--r-- 1 me  me  15738 2025-10-14 07:15 foo.txt
```

Comprimimos el archivo:

```bash
[me@linuxbox ~]$ gzip foo.txt
[me@linuxbox ~]$ ls -l foo.*
-rw-r--r-- 1 me  me  3231 2025-10-14 07:15 foo.txt.gz
```

El archivo original ha sido reemplazado con una versión comprimida aproximadamente **una quinta parte** del tamaño original.

Descomprimimos el archivo:

```bash
[me@linuxbox ~]$ gunzip foo.txt
[me@linuxbox ~]$ ls -l foo.*
-rw-r--r-- 1 me  me  15738 2025-10-14 07:15 foo.txt
```

El archivo comprimido ha sido reemplazado con el original. Nótese que **los permisos y la marca de tiempo se han preservado**.

### Opciones de gzip

| Opción | Opción Larga | Descripción |
|--------|--------------|-------------|
| `-c` | `--to-stdout` | Escribe salida a stdout y conserva los archivos originales |
| `-d` | `--decompress` / `--uncompress` | Descomprime. Causa que gzip actúe como gunzip |
| `-f` | `--force` | Fuerza compresión incluso si ya existe una versión comprimida del archivo original |
| `-h` | `--help` | Muestra información de uso |
| `-l` | `--list` | Lista estadísticas de compresión para cada archivo comprimido |
| `-r` | `--recursive` | Si uno o más argumentos en línea de comandos es un directorio, comprime recursivamente archivos contenidos dentro |
| `-t` | `--test` | Prueba la integridad de un archivo comprimido |
| `-v` | `--verbose` | Muestra mensajes detallados mientras se comprime |
| `-number` | — | Establece cantidad de compresión. El `number` es un entero en el rango 1 (compresión mínima/más rápida) a 9 (máxima compresión/más lenta). Los valores 1 y 9 también se pueden expresar como `--fast` y `--best`, respectivamente. El valor predeterminado es 6 |

### Usos Avanzados de gzip

Crear una versión comprimida de un listado de directorio:

```bash
[me@linuxbox ~]$ ls -l /etc | gzip > foo.txt.gz
```

Descomprimir un archivo a stdout para verlo:

```bash
[me@linuxbox ~]$ gunzip -c foo.txt | less
```

Alternativamente, existe un programa llamado **`zcat`** equivalente a `gunzip -c` que puede usarse como el comando `cat` para archivos comprimidos con gzip:

```bash
[me@linuxbox ~]$ zcat foo.txt.gz | less
```

> **Tip**: También existe un programa `zless` que realiza la misma función que el pipeline anterior.

---

## bzip2 — Compresión de Bloque

### Concepto General

**`bzip2`**, escrito por Julian Seward, es similar a `gzip` pero usa un **algoritmo de compresión diferente** que logra niveles **más altos de compresión** a costa de una **velocidad de compresión más lenta**. En casi todos los aspectos, funciona de la misma manera que `gzip`.

Un archivo comprimido con `bzip2` se denota con la extensión `.bz2`.

### Ejemplo

```bash
[me@linuxbox ~]$ ls -l /etc | head -n 1
-rw-r--r-- 1 me  me  15738 2025-10-14 07:15 foo.txt
```

Comprimimos con bzip2:

```bash
[me@linuxbox ~]$ bzip2 foo.txt
[me@linuxbox ~]$ ls -l foo.txt.bz2
-rw-r--r-- 1 me  me  2792 2025-10-17 13:51 foo.txt.bz2
```

El archivo comprimido con `bzip2` es aún más pequeño que con `gzip` (2,792 bytes vs 3,231 bytes).

### Comparación y Compatibilidad

`bzip2` puede usarse de la misma manera que `gzip`. Todas las opciones (excepto `-r`) que discutimos para `gzip` también son soportadas por `bzip2`. Sin embargo, la opción de nivel de compresión (`-number`) tiene un significado algo diferente en `bzip2`.

`bzip2` viene con:
- **`bunzip2`**: para descomprimir archivos
- **`bzcat`**: equivalente a `bzip2 -c`
- **`bzip2recover`**: intentará recuperar archivos `.bz2` dañados

### Advertencia: No Comprimir Dos Veces

⚠️ **Compresión Compulsiva**

A veces se ve gente intentando comprimir un archivo que ya ha sido comprimido efectivamente:

```bash
$ gzip picture.jpg
```

**¡No lo hagas!** Es probable que solo estés malgastando tiempo y espacio. Si aplicas compresión a un archivo que ya está comprimido, generalmente terminarás con un archivo **más grande**. Esto es porque todos los algoritmos de compresión incluyen una sobrecarga agregada al archivo. Si intentas comprimir un archivo que ya no contiene datos redundantes, la compresión **fallará la mayoría de las veces** y no resultará en ahorros para compensar la sobrecarga adicional.

---

## Archivado de Archivos

### Concepto General

Una tarea común de gestión de archivos es **archivar** — el proceso de **reunir muchos archivos y agruparlos en un solo archivo grande**. El archivado se hace a menudo como parte de copias de seguridad del sistema. También se usa cuando se mueven datos antiguos de un sistema a almacenamiento a largo plazo.

---

## tar — Utilidad Clásica de Archivado

### Historia y Concepto

En el mundo del software similar a Unix, **`tar`** es la herramienta clásica para archivar archivos. Su nombre, abreviatura de **tape archive**, revela sus raíces como herramienta para crear cintas de backup. Si bien aún se usa para esa tarea tradicional, es igualmente apto para otros dispositivos de almacenamiento.

Archivos de `tar` a menudo tienen extensiones que terminan en `.tar` (archive plano) o `.tar.gz` / `.tar.bz2` (archive comprimido).

Un archivo de `tar` puede consistir en:
- Un grupo de archivos separados
- Una o más jerarquías de directorios
- O una mezcla de ambos

### Modos de tar

La sintaxis de `tar` es ligeramente inusual, así que necesitamos algunos ejemplos. Primero, recreemos nuestro **playground** del capítulo anterior:

```bash
[me@linuxbox ~]$ mkdir -p playground/dir-{001..100}
[me@linuxbox ~]$ touch playground/dir-{001..100}/file-{A..Z}
```

El `mode` es uno de los siguientes modos de operación (tabla parcial; ver página del manual de `tar` para lista completa):

| Modo | Descripción |
|------|-------------|
| `c` | Crear un archive a partir de una lista de archivos y/o directorios |
| `x` | Extraer un archive |
| `r` | Agregar rutas especificadas al final de un archive. Si el archive no existe, se crea |
| `t` | Listar los contenidos de un archive |

`tar` usa una forma ligeramente extraña de expresar opciones, así que necesitaremos algunos ejemplos para mostrar cómo funciona. Primero, recreemos nuestro **playground** del capítulo anterior:

```bash
[me@linuxbox ~]$ mkdir -p playground/dir-{001..100}
[me@linuxbox ~]$ touch playground/dir-{001..100}/file-{A..Z}
```

Ahora, vamos a crear un archive tar de todo el playground:

```bash
[me@linuxbox ~]$ tar cf playground.tar playground
```

Este comando crea un archive tar llamado `playground.tar` que contiene toda la jerarquía de directorios `playground`. Podemos ver que el modo y la opción `f`, que se usa para especificar el nombre del archive tar, pueden unirse juntos y no requieren un guión inicial. Nótese, sin embargo, que el modo debe estar **siempre especificado primero**, antes de cualquier opción.

Para listar los contenidos del archive, podemos hacer esto:

```bash
[me@linuxbox ~]$ tar tf playground.tar
```

Para un listado más detallado, podemos agregar la opción `v` (verbose):

```bash
[me@linuxbox ~]$ tar tvf playground.tar
```

Ahora, vamos a extraer el **playground** en una nueva ubicación. Haremos esto creando un nuevo directorio llamado `foo`, cambiando el directorio y extrayendo el archive tar.

```bash
[me@linuxbox ~]$ mkdir foo
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ tar xf ../playground.tar
[me@linuxbox foo]$ ls
playground
```

Si examinamos los contenidos de `~/foo/playground`, veremos que el archive fue instalado exitosamente, creando una reproducción precisa de los archivos originales. Hay una advertencia, sin embargo. A menos que estemos operando como el superusuario, los archivos y directorios extraídos de archives toman la propiedad del usuario que está realizando la extracción, en lugar del propietario original.

### Manejo de Pathnames en tar

Un comportamiento interesante de `tar` es la forma en que maneja pathnames en archives. El predeterminado para pathnames es **relativo**, en lugar de absoluto. `tar` hace esto simplemente removiendo cualquier barra inicial del pathname cuando crea el archive. Para demostrar esto, vamos a recrear nuestro archive, esta vez especificando un pathname absoluto:

```bash
[me@linuxbox foo]$ cd
[me@linuxbox ~]$ tar cf playground2.tar ~/playground
```

Recuerda, `~/playground` se expandirá a `/home/me/playground` cuando presionemos enter, así que obtendremos un pathname absoluto para nuestra demostración. Ahora, vamos a extraer el archive como lo hicimos antes y ver qué sucede:

```bash
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ tar xf ../playground2.tar
[me@linuxbox foo]$ ls
home  playground
[me@linuxbox foo]$ ls home/me
playground
```

Aquí podemos ver que cuando extrajimos nuestro segundo archive, recreó el directorio `/home/me/playground` relativo a nuestro directorio de trabajo actual, `~/foo`. Esto puede parecer una forma extraña de funcionar, pero es en realidad muy útil porque nos permite extraer archives a cualquier ubicación en lugar de forzarlos a extraerse a su ubicación original.

### Extracción Selectiva

Cuando extraemos un archive, es posible limitar lo que se extrae del archive. Por ejemplo, si queremos extraer un solo archivo de un archive:

```bash
tar xf archive.tar pathname
```

Al agregar el `pathname` al final del comando, `tar` restaurará solo el archivo especificado. Múltiples pathnames pueden especificarse. Nótese que el pathname debe ser la ruta exacta y relativa tal como se almacena en el archive.

Cuando se especifican pathnames, los wildcards generalmente no son soportados; sin embargo, la versión más reciente de GNU `tar` (que es la más común en distribuciones Linux) los soporta con la opción `--wildcards`. Aquí hay un ejemplo usando nuestro archivo `playground.tar` anterior:

```bash
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ tar xf ../playground2.tar --wildcards 'home/me/playground/dir-*/file-A'
```

Este comando extraerá solo los archivos que coincidan con el pathname especificado incluyendo el wildcard `dir-*`.

### Uso Práctico: Backups Incrementales

`tar` se usa a menudo en conjunto con `find` para producir archives. En este ejemplo, usaremos `find` para generar una lista de archivos que coincidan con un criterio de búsqueda específico y luego, usando la acción `-exec`, invocar `tar` en modo de anexado (`r`) para agregar los archivos coincidentes al archive `playground.tar`.

Aquí usamos `find` para seleccionar todos los archivos en `playground` nombrados `file-A`:

```bash
[me@linuxbox foo]$ find playground -name 'file-A' -exec tar rf playground.tar '{}' '+'
```

Aquí usamos `find` para buscar todos los archivos en `playground` nombrados `file-A` y luego, usando la acción `-exec`, invocamos `tar` en el modo de anexado (r) para agregar los archivos coincidentes al archive `playground.tar`.

### Compresión con tar

Mientras que usamos el programa `gzip` externamente para producir nuestro archive comprimido en ejemplos anteriores, las versiones modernas de GNU tar soportan tanto compresión `gzip` como `bzip2` directamente con el uso de las opciones `z` y `j`, respectivamente.

Usando nuestro ejemplo anterior como base, podemos simplificar así:

**Con gzip:**

```bash
[me@linuxbox ~]$ find playground -name 'file-A' | tar czf playground.tgz -T -
```

**Con bzip2:**

```bash
[me@linuxbox ~]$ find playground -name 'file-A' | tar cjf playground.tbz -T -
```

Simplemente cambiando la opción de compresión de `z` a `j` (y cambiando la extensión del archivo de salida a `.tbz` para indicar compresión tar bzip2), habilitamos compresión bzip2.

### Transferencia Remota con tar y ssh

Un uso interesante de `tar` es en transferencias de archivos entre sistemas a través de una red. Imagina que tenemos dos máquinas ejecutando un sistema similar a Unix equipado con `tar` y `ssh`. En tal escenario, podríamos transferir un directorio desde un sistema remoto (llamado `remote-sys` para este ejemplo) a nuestro sistema local.

```bash
[me@linuxbox ~]$ mkdir remote-stuff
[me@linuxbox ~]$ cd remote-stuff
[me@linuxbox remote-stuff]$ ssh remote-sys 'tar cf - Documents' | tar xf -
me@remote-sys's password:
[me@linuxbox remote-stuff]$ ls
Documents
```

Aquí fuimos capaces de copiar un directorio llamado `Documents` desde el sistema remoto `remote-sys` a un directorio dentro del directorio llamado `remote-stuff` en el sistema local. ¿Cómo hicimos esto? Primero, lanzamos el programa `tar` en `remote-sys` usando `ssh`. Recuerda, `ssh` nos permite ejecutar un programa remotamente en una computadora en red y "ver" los resultados en el sistema local — la salida estándar producida en el sistema remoto se envía al sistema local para visualización. Podemos aprovechar esto teniendo `tar` crear un archive (el modo `c`) y enviarlo a salida estándar, en lugar de un archivo (el modo `f` con el guión), por lo tanto transportando el archive sobre el túnel encriptado proporcionado por `ssh` al sistema local. En el sistema local, ejecutamos `tar` y le indicamos que expanda un archive (el modo `x`) suministrado desde entrada estándar (nuevamente, el modo `f` con el guión).

---

## zip — Empaquetado y Compresión

### Concepto General

El programa **`zip`** es tanto una **herramienta de compresión** como un **archivador**. El formato de archivo usado por el programa es familiar para usuarios de Windows, ya que lee y escribe archivos (extensión `.zip`). Su sintaxis básica es:

```bash
zip options zipfile file...
```

### Ejemplo Básico

Para hacer un archive zip de nuestro **playground**, haríamos esto:

```bash
[me@linuxbox ~]$ zip -r playground.zip playground
```

A menos que incluyamos la opción `-r` para recursión, solo el directorio `playground` (pero no sus contenidos) se almacenaría. Aunque la adición de la extensión `.zip` es automática, la incluiremos por claridad.

Durante la creación del archive zip, `zip` normalmente mostrará una serie de mensajes como este:

```
adding: playground/dir-020/file-Z (stored 0%)
adding: playground/dir-020/file-Y (stored 0%)
adding: playground/dir-020/file-X (stored 0%)
adding: playground/dir-087/ (stored 0%)
adding: playground/dir-087/file-S (stored 0%)
```

Estos mensajes muestran el estado de cada archivo agregado al archive. `zip` agregará archivos al archive usando uno de dos métodos de almacenamiento: almacenará un archivo sin compresión, como se muestra aquí, o **deflará** el archivo (lo comprimirá). El valor numérico mostrado después del método de almacenamiento indica la cantidad de compresión lograda. Como nuestro playground contiene solo archivos vacíos, no se realiza compresión en sus contenidos.

### Extracción de zip

Extraer los contenidos de un archivo zip es sencillo cuando se usa el programa `unzip`:

```bash
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ unzip ../playground.zip
```

Una cosa a notar sobre `zip` (a diferencia de `tar`) es que si se especifica un archive existente, se actualiza en lugar de ser reemplazado. Esto significa que el archive existente se preserva, pero se agregan archivos nuevos y coincidentes que se reemplazan.

Los archivos pueden listarse y extraerse selectivamente desde un archive zip especificándolos a `unzip`:

```bash
[me@linuxbox ~]$ unzip -l playground.zip playground/dir-087/file-Z
Archive: ../playground.zip
  Length      Date    Time      Name
  --------  ---------- -----  --------
       0    10-05-25 09:25   playground/dir-087/file-Z
  --------                   -------
       0                     1 file
```

Usando la opción `-l`, `unzip` simplemente lista el contenido del archive sin extraer el archivo. Si no se especifican archivos, `unzip` listará todos los archivos en el archive. La opción `-v` puede agregarse para aumentar la verbosidad del listado.

### Uso con stdin/stdout

Como `tar`, `zip` puede hacer uso de entrada y salida estándar, aunque su implementación es algo menos útil. Es posible canalizar una lista de nombres de archivos a `zip` a través de la opción `-@`:

```bash
[me@linuxbox foo]$ cd
[me@linuxbox ~]$ find playground -name 'file-A' | zip -@ file-A.zip
```

Aquí usamos `find` para generar una lista de archivos que coincidan con el test `-name 'file-A'` y luego canalizamos la lista a `zip`, que crea el archive `file-A.zip` que contiene los archivos seleccionados.

`zip` también soporta escribir su salida a salida estándar, pero su uso es limitado porque pocos programas pueden hacer uso de la salida. Desafortunadamente, el programa `zip` y `unzip` no aceptan entrada estándar. Esto previene que `zip` y `unzip` se usen juntos para realizar copias de archivos de red como `tar`.

`zip` puede, sin embargo, aceptar entrada estándar para comprimir la salida de otros programas:

```bash
[me@linuxbox ~]$ ls -l /etc | zip ls-etc.zip -
adding: - (deflated 80%)
```

En este ejemplo, canalizamos la salida de `ls` en `zip`. Como `tar` interpreta el guión como "use entrada estándar para la lista de archivos", zip interpreta el guión como "use entrada estándar como un archivo único a ser comprimido". Cuando se especifica un archivo a `unzip`, debes especificar el archivo actual relativo como se almacena en el archive:

```bash
[me@linuxbox ~]$ unzip -l ls-etc.zip
Archive: ls-etc.zip
  Length      Date      Time    Name
  --------  ---------- -----  --------
    0       10-05-25 09:25   playground/dir-087/file-Z
  --------                   -------
    0                       1 file
```

Usaremos `unzip` para ver los contenidos del archive. Para ver la salida resultante, usaremos el programa `unzip` como permite que su salida sea enviada a salida estándar cuando se especifica la opción `-p` (para pipe):

```bash
[me@linuxbox ~]$ unzip -p ls-etc.zip | less
```

Tocamos algunos de las tareas básicas que `zip`/`unzip` pueden hacer. Ambos tienen muchas opciones que agregan a su flexibilidad, aunque algunas son específicas de plataforma a otros sistemas. Las páginas de manual para `zip` y `unzip` son bastante buenas y contienen ejemplos útiles. Sin embargo, el uso principal de estos programas es para intercambiar archivos con sistemas Windows, en lugar de realizar compresión y archivado en Linux, donde `tar` y `gzip` son muy preferidos.

---

## Sincronización de Archivos y Directorios

### Concepto General

Una estrategia común para mantener una copia de backup de un sistema implica mantener uno o más directorios **sincronizados** con otro directorio (o directorios) ubicado ya sea en el sistema local (usualmente un dispositivo de almacenamiento removible de algún tipo) o en un sistema remoto.

En el mundo de Unix, la herramienta preferida para esta tarea es **`rsync`**. Este programa puede sincronizar tanto directorios locales como remotos usando el **protocolo remote-update**, que permite a `rsync` detectar rápidamente diferencias entre dos directorios y realizar la cantidad mínima de copia requerida para traerlos a sincronización. Esto hace que `rsync` sea muy rápido y económico de usar, comparado con otros tipos de programas de copia.

### Sintaxis Básica

`rsync` se invoca así:

```bash
rsync options source destination
```

Donde `source` y `destination` pueden ser uno de los siguientes:

- Un archivo o directorio local
- Un archivo o directorio remoto en la forma de `[user@]host:path`
- Un servidor rsync remoto especificado con un URI de `rsync://[user@]host[:port]/path`

**Nota importante**: Ya sea el `source` o el `destination` debe ser un archivo local.

La copia remota a remota no es soportada.

### Ejemplo Práctico: Sincronización Local

Primero, limpiemos nuestro directorio `foo`:

```bash
[me@linuxbox ~]$ rm -rf foo/*
```

Ahora, vamos a sincronizar el directorio `playground` con una copia correspondiente en `foo`:

```bash
[me@linuxbox ~]$ rsync -av playground foo
```

Hemos incluido tanto la opción `-a` (para archiving — causa recursión y preservación de atributos de archivo) como la opción `-v` (verbose output) para hacer un **mirror** del directorio `playground` dentro de `foo`. Mientras que el comando se ejecuta, veremos un listado de archivos y directorios siendo copiados. Al final, veremos un mensaje de resumen como este indicando la cantidad de copia realizada:

```
sent 135759 bytes received 57870 bytes 387258.00 bytes/sec
total size is 3230 speedup is 0.02
```

Si ejecutamos el comando nuevamente, veremos un resultado diferente:

```bash
[me@linuxbox ~]$ rsync -av playground foo
building file list ... done

sent 22635 bytes received 20 bytes 45310.00 bytes/sec
total size is 3230 speedup is 0.14
```

Nótese que no hay un listado de archivos. Esto es porque `rsync` detectó que no había diferencias entre `~/playground` y `~/foo/playground`, por lo tanto no necesitaba copiar nada. Si modificamos un archivo en `playground` y ejecutamos `rsync` nuevamente:

```bash
[me@linuxbox ~]$ touch playground/dir-099/file-Z
[me@linuxbox ~]$ rsync -av playground foo
building file list ... done
playground/dir-099/file-Z
sent 22685 bytes received 42 bytes 45454.00 bytes/sec
total size is 3230 speedup is 0.14
```

Veremos que `rsync` detectó el cambio y copió solo el archivo actualizado.

### Comportamiento con Trailing Slash

Hay una característica sutil pero útil que podemos usar cuando especificamos un `rsync` source. Consideremos estos dos directorios:

```bash
[me@linuxbox ~]$ ls
source  destination
```

El directorio `source` contiene un archivo llamado `file1`, y el directorio `destination` está vacío. Si realizamos una copia de `source` a `destination` así:

```bash
[me@linuxbox ~]$ rsync source destination
```

Entonces `rsync` copia el directorio `source` en `destination`:

```bash
[me@linuxbox ~]$ ls destination
source
```

Sin embargo, si agregamos un trailing `/` al nombre del directorio source, `rsync` copiará solo los contenidos del directorio source y no el directorio en sí:

```bash
[me@linuxbox ~]$ rsync source/ destination
[me@linuxbox ~]$ ls destination
file1
```

Esto es práctico si queremos solo los contenidos de un directorio copiados sin crear otro nivel de directorios dentro de destination. Podemos pensar como si `source/*` en su resultado, pero este método copiará todos los contenidos del directorio source incluyendo archivos ocultos.

### Ejemplo Práctico: Backup en Dispositivo Externo

Como ejemplo práctico, consideremos el imaginario disco duro externo que usamos anteriormente con `tar`. Si adjuntamos la unidad a nuestro sistema y está montada en `/media/BigDisk` una vez más, podemos realizar un backup útil del sistema creando primero un directorio llamado `/backup` en la unidad externa y luego usando `rsync` para copiar los directorios más importantes desde nuestro sistema a la unidad externa.

```bash
[me@linuxbox ~]$ mkdir /media/BigDisk/backup
[me@linuxbox ~]$ sudo rsync -av --delete /etc /home /usr/local /media/BigDisk/backup
```

En este ejemplo, copiamos los directorios `/etc`, `/home`, y `/usr/local` desde nuestro sistema a nuestro dispositivo de almacenamiento imaginario. Incluimos la opción `--delete` para eliminar archivos que pueden haber existido en el dispositivo de backup que no existen más en el sistema fuente (esto es irrelevante la primera vez que hacemos un backup pero será útil en copias posteriores).

Repetir el procedimiento de adjuntar la unidad externa y ejecutar este comando `rsync` sería una forma útil (aunque no ideal) de mantener una pequeña copia de sistema backed up. Por supuesto, un alias sería útil aquí también. Podríamos crear un alias y agregarlo a nuestro archivo de configuración:

```bash
alias backup='sudo rsync -av --delete /etc /home /usr/local /media/BigDisk/backup'
```

Ahora todo lo que tendríamos que hacer es adjuntar nuestra unidad externa y ejecutar el comando de backup para hacer el trabajo.

### Sincronización a Través de Red

Una de las bellezas reales de `rsync` es que puede usarse para copiar archivos a través de una red. Después de todo, la "r" en `rsync` es para "remoto". La copia remota puede hacerse de una de dos formas. La primera es con otro sistema que tenga `rsync` instalado, junto con un programa de shell remoto como `ssh`.

Digamos que tenemos dos máquinas en nuestra red local con mucho espacio en disco disponible y queríamos realizar una operación de backup usando el sistema remoto en lugar de una unidad externa. Asumiendo que ya teníamos un directorio llamado `/backup` donde podríamos entregar nuestros archivos, podríamos hacer esto:

```bash
[me@linuxbox ~]$ sudo rsync -av --delete --rsh=ssh /etc /home /usr/local remote-sys:/backup
```

Hicimos dos cambios a nuestro comando para facilitar la copia de red. Primero, agregamos la opción `--rsh=ssh`, que instruye `rsync` a usar el programa `ssh` como su shell remoto. De esta manera, pudimos usar un túnel encriptado para transferir datos de manera segura entre sistemas. En sistemas modernos ssh es el predeterminado, así que esta opción es raramente necesitada.

Segundo, especificamos el host remoto por prefijando su nombre (en este caso el host remoto es llamado `remote-sys`) al pathname de destino.

### rsync como Daemon

La segunda forma que `rsync` puede usarse para sincronizar archivos a través de una red es configurándolo para ejecutarse como daemon y escuchar solicitudes entrantes de sincronización. Esto se hace a menudo para permitir mirroring de un sistema remoto. Por ejemplo, Red Hat Software mantiene un gran repositorio de paquetes de software bajo desarrollo para su distribución Fedora. Es deseable para testers de software mantener un mirror local de este colección durante la fase de testing de la distribución de lanzamiento.

Desde la perspectiva de un usuario `rsync`, las cosas son un poco diferentes. En lugar de usar la sintaxis habitual, usamos un URI del servidor `rsync`:

```bash
[me@linuxbox ~]$ mkdir fedora-devel
[me@linuxbox ~]$ rsync -av --delete rsync://archive.linux.duke.edu/fedora/linux/development/x86_64/os/ fedora-devel
```

En este ejemplo, usamos el URI del servidor rsync remoto, que consiste de un protocolo (`rsync://`), seguido por el hostname remoto (`archive.linux.duke.edu`), seguido por el pathname del repositorio.

---

## Resumen

Hemos observado los programas comunes de compresión y archivado usados en Linux y otros sistemas operativos similares a Unix.

Para archivado de archivos, la combinación **`tar`/`gzip`** es el método preferido en sistemas similares a Unix, mientras que **`zip`/`unzip`** se usa para interoperabilidad con sistemas Windows.

Finalmente, examinamos el programa **`rsync`** (un favorito personal), que es muy útil para sincronización eficiente de archivos y directorios a través de sistemas.

---

## Ver También

- [[wiki/linux/17-busqueda-archivos.md|Capítulo 17: Búsqueda de Archivos en Linux]]
- [[wiki/linux/15-medios-almacenamiento.md|Capítulo 15: Medios de Almacenamiento en Linux]]
