---
title: "Capítulo 17: Búsqueda de Archivos en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 17"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-239-252.pdf"
updated: "2026-06-23"
---

# Búsqueda de Archivos en Linux

## Introducción

Un sistema Linux típico contiene una cantidad **muy grande de archivos**. Aunque el sistema de archivos está bien organizado siguiendo convenciones heredadas de Unix, la cantidad de archivos puede presentar un desafío: ¿cómo encontramos lo que buscamos?

En este capítulo cubrimos dos herramientas principales para localizar archivos en un sistema Linux:

- **`locate`**: Encuentra archivos por nombre (búsqueda rápida)
- **`find`**: Busca archivos en una jerarquía de directorios (búsqueda potente y flexible)

También cubriremos comandos útiles para procesar los resultados de las búsquedas:

- **`touch`**: Cambia los tiempos de archivo
- **`stat`**: Muestra estado del archivo o sistema de archivos
- **`xargs`**: Construye y ejecuta líneas de comando desde entrada estándar

---

## locate — Búsqueda Rápida por Nombre

### Concepto General

El comando **`locate`** realiza una búsqueda rápida en una **base de datos de rutas** (pathnames). En lugar de buscar en el sistema de archivos en tiempo real, consulta una base de datos precompilada que es mucho más rápida.

Por ejemplo, para encontrar todos los programas con nombres que comiencen con `zip`:

```bash
[me@linuxbox ~]$ locate bin/zip
/usr/bin/zip
/usr/bin/zipcloak
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/bin/zipnote
/usr/bin/zipsplit
```

### Combinación con grep

Si el criterio de búsqueda es más complejo, podemos combinar `locate` con `grep`:

```bash
[me@linuxbox ~]$ locate zip | grep bin
/bin/bunzip2
/bin/bzip2
/bin/bzip2recover
/bin/gunzip
/bin/gzip
/usr/bin/funzip
/usr/bin/gps-zip
/usr/bin/preunzip
/usr/bin/prezip
/usr/bin/prezip-bin
/usr/bin/unzip
/usr/bin/unzipsfx
/usr/bin/zip
/usr/bin/zipcloak
/usr/bin/zipgrep
```

### Base de Datos de locate

La base de datos de `locate` se **actualiza periódicamente** mediante un cron job (tarea programada):

- Por defecto, la mayoría de sistemas ejecutan `updatedb` una vez al día
- Si acabas de instalar archivos nuevos y no aparecen en `locate`, puedes ejecutar manualmente `updatedb` con privilegios de superusuario
- Los archivos muy recientes no aparecerán en las búsquedas de `locate`

**Nota**: En algunas distribuciones, `locate` falla inmediatamente después de instalar el sistema, pero funciona correctamente al día siguiente cuando `updatedb` se ejecuta por primera vez.

---

## find — Búsqueda Potente y Flexible

### Concepto General

Mientras que `locate` es rápido pero limitado a búsquedas por nombre, **`find`** es un comando mucho más potente que:

- Busca en jerarquías de directorios en tiempo real
- Permite filtrar por atributos de archivos (tipo, tamaño, permisos, tiempo de modificación, etc.)
- Puede ejecutar acciones sobre los archivos encontrados
- Combina múltiples criterios de búsqueda con operadores lógicos

En su forma más simple:

```bash
[me@linuxbox ~]$ find ~
```

Esto produce una lista de todos los archivos y subdirectorios en el directorio home. En sistemas activos, esto genera una lista muy grande.

### Conteo de Archivos

Para contar cuántos archivos hay, usamos `wc -l`:

```bash
[me@linuxbox ~]$ find ~ | wc -l
47068
```

---

## Pruebas (Tests) en find

El poder de `find` proviene de su capacidad de usar **pruebas** para filtrar resultados. Las pruebas permiten buscar archivos según criterios específicos.

### Pruebas por Tipo de Archivo

La prueba más común es **`-type`**, que limita los resultados a archivos de un tipo específico:

```bash
[me@linuxbox ~]$ find ~ -type d | wc -l
1695
```

**Tabla de tipos de archivo:**

| Tipo | Descripción |
|------|-------------|
| `b` | Archivo especial de bloque |
| `c` | Archivo especial de carácter |
| `d` | Directorio |
| `f` | Archivo regular |
| `l` | Enlace simbólico |

Ejemplo: Buscar solo archivos regulares:

```bash
[me@linuxbox ~]$ find ~ -type f | wc -l
38737
```

### Pruebas por Nombre y Tamaño

Podemos buscar archivos por nombre con **`-name`** y usar wildcards:

```bash
[me@linuxbox ~]$ find ~ -type f -name "*.JPG" -size +1M | wc -l
840
```

Este comando encuentra:
- Archivos regulares (`-type f`)
- Con extensión `.JPG` (`-name "*.JPG"`)
- Mayores de 1 MB (`-size +1M`)

**Tabla de unidades de tamaño:**

| Carácter | Unidad |
|----------|--------|
| `b` | Bloques de 512 bytes (por defecto) |
| `c` | Bytes |
| `w` | Palabras de 2 bytes |
| `k` | Kilobytes (1,024 bytes) |
| `M` | Megabytes (1,048,576 bytes) |
| `G` | Gigabytes (1,073,741,824 bytes) |

**Nota**: El signo `+` indica "mayor que", `-` indica "menor que", y sin signo significa "exactamente".

### Pruebas Comunes de find

La siguiente tabla lista las pruebas más comunes:

| Prueba | Descripción |
|--------|-------------|
| `-cmin n` | Archivos cuyo contenido o atributos fueron modificados hace exactamente `n` minutos. Use `-n` para "menos de n minutos", use `+n` para "más de n minutos" |
| `-cnewer file` | Archivos cuyo contenido o atributos fueron modificados más recientemente que `file` |
| `-ctime n` | Archivos cuyo contenido o atributos fueron modificados hace n×24 horas |
| `-empty` | Archivos y directorios vacíos |
| `-group name` | Archivos o directorios pertenecientes al grupo `group`. Puede expresarse como nombre de grupo o ID numérico |
| `-iname pattern` | Como `-name` pero sin distinción de mayúsculas/minúsculas |
| `-inum n` | Archivos con número de inodo `n`. Útil para encontrar todos los enlaces duros a un inodo particular |
| `-mmin n` | Archivos cuyo contenido fue modificado hace n minutos |
| `-mtime n` | Archivos cuyo contenido fue modificado hace n×24 horas |
| `-name pattern` | Archivos y directorios con patrón wildcard especificado |
| `-newer file` | Archivos y directorios modificados más recientemente que `file`. Útil en scripts de backup |
| `-nogroup` | Archivos y directorios que no pertenecen a un grupo válido |
| `-nouser` | Archivos y directorios que no pertenecen a un usuario válido. Útil para detectar cuentas eliminadas |
| `-perm mode` | Archivos o directorios con permisos establecidos al `mode` especificado. Mode puede expresarse en notación octal o simbólica |
| `-samefile name` | Similar a `-inum`. Coincide con archivos que comparten el mismo número de inodo que `name` |
| `-size n` | Archivos de tamaño n |
| `-type c` | Archivos de tipo c |
| `-user name` | Archivos o directorios pertenecientes al usuario `name` |

---

## Operadores Lógicos en find

Para expresar relaciones lógicas más complejas entre pruebas, `find` proporciona **operadores lógicos**:

| Operador | Descripción |
|----------|-------------|
| `-and` | Coincide si las pruebas a ambos lados del operador son verdaderas. Puede abreviarse como `-a`. Si no hay operador presente, `-and` está implícito por defecto |
| `-or` | Coincide si una prueba en cualquier lado del operador es verdadera. Puede abreviarse como `-o` |
| `-not` | Coincide si la prueba siguiente al operador es falsa. Puede abreviarse con un signo de exclamación `!` |
| `( )` | Agrupa pruebas y operadores para formar expresiones mayores. Se utiliza para controlar el orden de evaluación. Por defecto, `find` evalúa izquierda a derecha. Dado que los paréntesis tienen significado especial para el shell, deben escaparse con backslash. También es importante que exista un espacio después de `(` y antes de `)` |

### Ejemplo: Combinación de Pruebas

Para buscar archivos con permisos "malos" en el directorio home:

```bash
find ~ \( -type f -not -perm 0600 \) -or ( -type d -not -perm 0700 \)
```

Esta expresión busca:
- Archivos regulares con permisos distintos de 0600 (`-type f -not -perm 0600`)
- **O** (`-or`) directorios con permisos distintos de 0700 (`-type d -not -perm 0700`)

**Nota importante**: Los paréntesis deben escaparse con backslash `\(` y `\)` para evitar que el shell los interprete.

---

## Acciones Predefinidas en find

Las pruebas son útiles, pero lo que realmente queremos es **actuar** sobre los archivos encontrados. `find` proporciona acciones predefinidas y la capacidad de definir acciones personalizadas.

### Acciones Predefinidas

| Acción | Descripción |
|--------|-------------|
| `-delete` | Elimina el archivo actualmente coincidente |
| `-ls` | Realiza `ls -dils` en el archivo coincidente. Output enviado a stdout |
| `-print` | Muestra la ruta completa del archivo coincidente en stdout. Esta es la acción predeterminada si no se especifica otra |
| `-quit` | Sale una vez que coincide un archivo |

### Ejemplos de Acciones

**Ejemplo 1: Listar archivos**

```bash
[me@linuxbox ~]$ find ~ -print
```

Esto es equivalente a simplemente `find ~` ya que `-print` es la acción predeterminada.

**Ejemplo 2: Eliminar archivos de backup**

Para eliminar archivos con extensión `.bak` (archivos de backup):

```bash
find ~ -type f -name "*.bak" -delete
```

⚠️ **Advertencia**: Usa **extrema cautela** con `-delete`. Siempre prueba primero usando `-print` en lugar de `-delete` para confirmar los resultados de búsqueda.

**Ejemplo 3: Usar `-print` con operadores lógicos**

```bash
find ~ -type f -name "*.bak" -print
```

Se puede expresar más explícitamente como:

```bash
find ~ -type f -and -name "*.bak" -and -print
```

---

## Acciones Definidas por el Usuario

Además de las acciones predefinidas, `find` permite ejecutar comandos arbitrarios. La forma tradicional es usar la acción **`-exec`**:

```bash
-exec command {} \;
```

Donde:
- `command` es el nombre del comando a ejecutar
- `{}` es una representación simbólica de la ruta del archivo actual
- `;` es un delimitador requerido que indica el fin del comando

### Ejemplo con -exec

```bash
find ~ -type f -name "foo*" -exec ls -l '{}' ';'
-rwxr-xr-x 1 me me  224 2016-10-29 18:44 /home/me/bin/foo
-rw-r--r-- 1 me me    0 2025-09-19 12:53 /home/me/foo.txt
```

Este comando busca archivos con nombres que comienzan con `foo` y ejecuta `ls -l` en cada uno.

### La Acción -ok (Interactiva)

Para ejecutar acciones interactivamente (el usuario es consultado antes de ejecutar cada comando), usa **`-ok`** en lugar de `-exec`:

```bash
find ~ -type f -name "foo*" -ok ls -l '{}' ';'
< ls ... /home/me/bin/foo > ? y
-rwxr-xr-x 1 me me  224 2016-10-29 18:44 /home/me/bin/foo
< ls ... /home/me/foo.txt > ? y
-rw-r--r-- 1 me me    0 2025-09-19 12:53 /home/me/foo.txt
```

El usuario debe responder con `y` para ejecutar cada comando.

---

## Mejora de Eficiencia: xargs

Cuando se usa `-exec`, se lanza una **nueva instancia del comando** para cada archivo encontrado, lo que puede ser ineficiente. El comando **`xargs`** resuelve este problema.

### Concepto de xargs

`xargs` acepta entrada desde stdin y la convierte en una **lista de argumentos** para un comando especificado. Ejecuta el comando una sola vez con múltiples argumentos.

### Ejemplo: xargs vs -exec

**Con -exec (crea múltiples instancias de `ls`):**

```bash
find ~ -type f -name "*.JPG" -exec ls -l '{}' ';'
ls -l /home/me/bin/foo
ls -l /home/me/foo.txt
```

**Con xargs (ejecuta `ls` una sola vez):**

```bash
find ~ -type f -name "*.JPG" -print | xargs ls -l
-rwxr-xr-x 1 me me  224 2016-10-29 18:44 /home/me/bin/foo
-rw-r--r-- 1 me me    0 2025-09-19 12:53 /home/me/foo.txt
```

**Con la sintaxis alternativa de find (más eficiente):**

```bash
find ~ -type f -name "foo*" -exec ls -l '{}' +
```

Al cambiar el punto y coma `;` por un signo más `+`, `find` combina los resultados de búsqueda en una lista de argumentos para una sola ejecución del comando.

### Manejo de Nombres con Espacios

Los nombres de archivos con espacios pueden causar problemas. `find` proporciona la acción **`-print0`** que produce salida separada por caracteres nulos (ASCII 0), y `xargs` tiene la opción **`--null` (o `-0`)** para aceptar entrada con separadores nulos:

```bash
find ~ -iname "*.jpg" -print0 | xargs --null ls -l
```

De esta forma, todos los archivos, incluso los con espacios incrustados, se manejan correctamente.

---

## Ejemplo Práctico: Playground

Para demostrar el poder de `find`, creamos un directorio de prueba ("playground") con muchos subdirectorios y archivos:

```bash
[me@linuxbox ~]$ mkdir -p playground/dir-{001..100}
[me@linuxbox ~]$ touch playground/dir-{001..100}/file-{A..Z}
```

Con estas dos líneas:
- Creamos un directorio `playground` con 100 subdirectorios (usando expansión de brace)
- Creamos 26 archivos vacíos en cada subdirectorio

Esto produce 100 directorios + 2,600 archivos.

### Búsquedas en el Playground

**Encontrar todos los archivos `file-A`:**

```bash
[me@linuxbox ~]$ find playground -type f -name 'file-A'
```

**Encontrar archivos modificados después de un archivo de referencia:**

```bash
[me@linuxbox ~]$ touch playground/timestamp
[me@linuxbox ~]$ find playground -type f -newer playground/timestamp
```

**Buscar archivos con permisos "malos":**

```bash
[me@linuxbox ~]$ find playground \( -type f -not -perm 0600 \) -or \( -type d -not -perm 0700 \)
```

Este comando lista todos los directorios y 2,600 archivos en playground (porque ninguno tiene los permisos definidos como "buenos").

**Cambiar permisos y ejecutar acciones:**

```bash
[me@linuxbox ~]$ find playground \( -type f -exec chmod 0600 '{}' '+' \) -or \( -type d -exec chmod 0700 '{}' '+' \)
```

---

## Opciones de find

Finalmente, `find` proporciona **opciones** que controlan el alcance de una búsqueda. Se incluyen con las pruebas, acciones y operadores:

| Opción | Descripción |
|--------|-------------|
| `-depth` | Dirige `find` a procesar los archivos de un directorio antes del directorio en sí. Esta opción se aplica automáticamente cuando se especifica la acción `-delete` |
| `-maxdepth levels` | Establece el número máximo de niveles que `find` descenderá en un árbol de directorios al realizar pruebas y acciones |
| `-mindepth levels` | Establece el número mínimo de niveles que `find` descenderá en un directorio antes de aplicar pruebas y acciones |
| `-mount` | Dirige `find` a no atravesar directorios montados en otros sistemas de archivos |
| `-noleaf` | Dirige `find` a no optimizar su búsqueda asumiendo que busca un sistema de archivos similar a Unix. Esto es necesario cuando escanea sistemas DOS/Windows y CD-ROMs |

---

## Resumen

`locate` es simple y rápido para búsquedas por nombre, pero `find` es mucho más potente:

- Puede filtrar por una variedad de atributos de archivo
- Permite combinar múltiples criterios con operadores lógicos
- Puede ejecutar acciones sobre los archivos encontrados
- Proporciona opciones para controlar el alcance de la búsqueda

Con práctica y exploración de la página del manual de `find`, desarrollarás una mejor comprensión de las operaciones del sistema de archivos de Linux.

---

## Ver También

- [[wiki/linux/02-navegacion-sistema-archivos.md|Capítulo 2: Navegación del Sistema de Archivos]]
- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]]
