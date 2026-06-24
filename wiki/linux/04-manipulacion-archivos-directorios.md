---
title: "Capítulo 4: Manipulación de Archivos y Directorios"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 4"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-59-73.pdf"
updated: "2026-06-22"
---

# Manipulación de Archivos y Directorios

## Introducción

¡Ahora estamos listos para trabajo real! Este capítulo introduce los siguientes comandos fundamentales para manipular archivos y directorios:

- **`cp`** — Copiar archivos y directorios
- **`mv`** — Mover o renombrar archivos y directorios
- **`mkdir`** — Crear directorios
- **`rm`** — Remover archivos y directorios
- **`ln`** — Crear hard y symbolic links

Estos cinco comandos se encuentran entre los comandos de Linux más usados. Aunque es fácil realizar manipulaciones simples con un gestor de archivos gráfico, tareas complicadas pueden ser más fáciles desde la línea de comandos.

## Comodines (Wildcards)

Antes de usar nuestros comandos, debemos hablar de una característica del shell que hace que estos comandos sean tan poderosos. Ya que el shell usa nombres de archivo tan frecuentemente, proporciona caracteres especiales para ayudarnos a especificar rápidamente grupos de nombres de archivo.

Estos caracteres especiales se llaman **wildcards** (también conocidos como **globbing**). Con wildcards, podemos seleccionar nombres de archivo basados en patrones de caracteres.

### Tabla de Comodines

| Comodín | Significado |
|--------|------------|
| `*` | Coincide con cualquier número de caracteres, incluyendo ninguno |
| `?` | Coincide con exactamente un carácter |
| `[caracteres]` | Coincide con cualquier carácter que sea miembro del conjunto *caracteres* |
| `[!caracteres]` o `[^caracteres]` | Coincide con cualquier carácter que NO sea miembro del conjunto *caracteres* |
| `[[clase:]]` | Coincide con cualquier carácter que sea miembro de la clase especificada |

### Clases de Caracteres Comunes

| Clase | Significado |
|-------|------------|
| `[:alnum:]` | Coincide con cualquier carácter alfanumérico |
| `[:alpha:]` | Coincide con cualquier carácter alfabético |
| `[:digit:]` | Coincide con cualquier dígito |
| `[:lower:]` | Coincide con cualquier letra minúscula |
| `[:upper:]` | Coincide con cualquier letra mayúscula |

### Ejemplos de Patrones Comodín

Usando wildcards es posible construir criterios de selección sofisticados para nombres de archivo:

| Patrón | Coincide Con |
|--------|------------|
| `*` | Todos los archivos |
| `b*` | Cualquier archivo que comienza con `b` |
| `b*.txt` | Cualquier archivo que comienza con `b` seguido por cualquier carácter y termina con `.txt` |
| `Data???` | Cualquier archivo que comienza con `Data` seguido por exactamente tres caracteres |
| `[abc]*` | Cualquier archivo que comienza con `a`, `b`, o `c` |
| `BACKUP.[0-9][0-9][0-9]` | Cualquier archivo que comienza con `BACKUP` seguido por exactamente tres dígitos |
| `[[upper:]]*` | Cualquier archivo que comienza con una letra mayúscula |
| `[![:digit:]]*` | Cualquier archivo que no comienza con un numeral |
| `*[[:lower:]]3` | Cualquier archivo que termina con una letra minúscula seguida del número 3 |

### Wildcards en la GUI

Los wildcards son especialmente valiosos no solo porque se usan frecuentemente en la línea de comandos, sino también porque algunos gestores de archivos gráficos los soportan:

- **Nautilus** (gestor de archivos de GNOME): Presiona `Ctrl+S` para entrar en modo de selección con pattern y los archivos en el directorio actualmente mostrado serán seleccionados.
- **Dolphin y Konqueror** (gestores de archivos de KDE): Pueden permitir wildcards directamente en la barra de ubicación.
- **Dolphin** (KDE): Tiene una opción "Filter" para realizar selección comodín.

## Archivos Ocultos (Dot Files)

Si observamos nuestro directorio personal con `ls` usando la opción `-a`, notaremos que hay un número de archivos y directorios cuyos nombres comienzan con un punto. Estos son archivos ocultos.

No es un atributo especial del archivo; significa simplemente que el archivo no aparecerá en la salida de `ls` normal. Los archivos ocultos no aparecerán a menos que uses `ls -a`. Esta característica oculta también se aplica a los comodines. Los archivos ocultos no aparecerán a menos que uses un patrón comodín como `.*`.

Para excluir `.` (directorio actual) y `..` (directorio padre) en los resultados, podemos usar patrones como `.[!.]*` o `..?*`.

## mkdir — Crear Directorios

El comando **`mkdir`** se usa para crear directorios:

```
mkdir directorio...
```

> **Nota**: Los tres puntos (`...`) en la descripción de un comando significan que el argumento puede repetirse. Así, el siguiente comando crearía un único directorio llamado `dir`:
> ```bash
> mkdir dir
> ```
> Y el siguiente comando crearía tres directorios llamados `dir1`, `dir2` y `dir3`:
> ```bash
> mkdir dir1 dir2 dir3
> ```

## cp — Copiar Archivos y Directorios

El comando **`cp`** copia archivos o directorios. Puede usarse de dos formas diferentes:

- Copiar un único archivo o directorio *item1* a archivo o directorio *item2*:
```
cp item1 item2
```

- O copiar múltiples items (archivos o directorios) en un directorio:
```
cp item... directorio
```

### Opciones Útiles de `cp`

| Opción | Opción Larga | Significado |
|--------|------------|------------|
| `-a` | `--archive` | Copia archivos y preserva todos sus atributos, incluyendo ownerships y permisos. Normalmente, las copias toman los atributos del usuario que realiza la copia. Miraremos los permisos de archivo en el Capítulo 9. |
| `-i` | `--interactive` | Antes de sobrescribir un archivo existente, solicita confirmación al usuario. Si esta opción no se especifica, `cp` sobrescribirá silenciosamente (significa no habrá advertencia) archivos. |
| `-r` | `--recursive` | Copia directorios recursivamente y sus contenidos. Esta opción (o la opción `-a`) es requerida cuando se copian directorios. |
| `-u` | `--update` | Al copiar archivos de un directorio a otro, solo copia archivos que o no existen o son más nuevos que los archivos correspondientes existentes en el directorio de destino. Esto es útil cuando se copian grandes números de archivos, ya que salta archivos que no necesitan ser copiados. |
| `-v` | `--verbose` | Muestra mensajes informativos durante la copia. |

### Ejemplos de `cp`

| Comando | Resultado |
|---------|-----------|
| `cp file1 file2` | Copia file1 a file2. Si file2 existe, será sobrescrito con el contenido de file1. Si file2 no existe, será creado. |
| `cp -i file1 file2` | Igual al comando anterior, excepto que si file2 existe, el usuario es solicitado antes de ser sobrescrito. |
| `cp file1 file2 dir1` | Copia file1 y file2 en el directorio dir1. El directorio dir1 debe existir. |
| `cp dir1/* dir2` | Usando comodín, copia todos los archivos en dir1 en dir2. El directorio dir2 debe existir. |
| `cp -r dir1 dir2` | Copia el contenido del directorio dir1 a dir2. Si el directorio dir2 no existe, será creado, y después de la copia, tendrá el mismo contenido que el directorio dir1. Si el directorio dir2 existe, entonces el directorio dir1 (y sus contenidos) será copiado en dir2. |

## mv — Mover y Renombrar Archivos

El comando **`mv`** realiza tanto movimiento de archivos como renombrado, dependiendo de cómo se use.

En cualquiera de los casos, el nombre de archivo original ya no existe después de la operación. `mv` se usa de la misma manera que `cp`:

```
mv item1 item2
```

O para mover uno o más items de un directorio a otro:

```
mv item... directorio
```

### Opciones Útiles de `mv`

| Opción | Opción Larga | Significado |
|--------|------------|------------|
| `-i` | `--interactive` | Antes de sobrescribir un archivo existente, solicita confirmación al usuario. Si esta opción no se especifica, `mv` sobrescribirá silenciosamente archivos. |
| `-u` | `--update` | Al mover archivos de un directorio a otro, solo mueve archivos que o no existen o son más nuevos que los archivos correspondientes existentes en el directorio de destino. |
| `-v` | `--verbose` | Muestra mensajes informativos durante el movimiento. |

### Ejemplos de `mv`

| Comando | Resultado |
|---------|-----------|
| `mv file1 file2` | Mueve file1 a file2. Si file2 existe, es eliminado y file1 es renombrado file2. Si file2 no existe, file1 es simplemente renombrado file2. En cualquier caso, file1 cesa de existir. |
| `mv -i file1 file2` | Igual al comando anterior, excepto que si file2 existe, el usuario es solicitado antes de que sea eliminado. |
| `mv file1 file2 dir1` | Mueve file1 y file2 en el directorio dir1. El directorio dir1 debe existir. |
| `mv dir1 dir2` | Si el directorio dir2 no existe, renombra dir1 a dir2. Si el directorio dir2 existe, mueve dir1 (y sus contenidos) en dir2. |

## rm — Remover Archivos y Directorios

El comando **`rm`** se usa para remover (eliminar) archivos y directorios:

```
rm item...
```

Donde *item* es uno o más archivos o directorios.

### Opciones Útiles de `rm`

| Opción | Opción Larga | Significado |
|--------|------------|------------|
| `-i` | `--interactive` | Antes de eliminar un archivo existente, solicita confirmación al usuario. Si esta opción no se especifica, `rm` eliminará silenciosamente archivos. |
| `-r` | `--recursive` | Elimina directorios recursivamente. Esto significa que si se está eliminando un directorio con subdirectorios, eliminarlos también. Para eliminar un directorio, esta opción debe ser especificada. |
| `-f` | `--force` | Ignora archivos inexistentes y no solicita confirmación. Esto sobrescribe la opción `--interactive`. |
| `-v` | `--verbose` | Muestra mensajes informativos durante la eliminación. |

### ⚠️ ¡CUIDADO CON `rm`!

Los sistemas Unix-like como Linux **no tienen comando "undelete"**. Una vez que elimines algo con `rm`, ¡se ha ido! Linux asume que eres inteligente y sabes lo que estás haciendo.

**Sé particularmente cuidadoso con comodines.** Considera este ejemplo clásico: supongamos que quieres eliminar solo los archivos HTML en un directorio. Para hacerlo, escribirías:

```bash
rm *.html
```

Esto es correcto, pero si accidentalmente pones un espacio entre el `*` y el `.html` como:

```bash
rm * .html
```

El comando `rm` eliminará todos los archivos en el directorio y luego se quejará de que no hay un archivo llamado `.html`.

**Truco útil**: Siempre que uses comodines con `rm` (además de verificar cuidadosamente tu escritura), prueba el comodín primero con `ls`. Esto te permitirá ver los archivos que serán eliminados. Entonces presiona la flecha arriba para recordar el comando y reemplaza `ls` con `rm`.

### Ejemplos de `rm`

| Comando | Resultado |
|---------|-----------|
| `rm file1` | Elimina file1 silenciosamente. |
| `rm -i file1` | Igual al comando anterior, excepto que el usuario es solicitado para confirmación antes de la eliminación. |
| `rm -r file1 dir1` | Elimina file1 y dir1 y sus contenidos. |
| `rm -rf file1 dir1` | Igual al comando anterior, excepto que si file1 o dir1 no existen, `rm` continuará silenciosamente. |

## ln — Crear Enlaces (Links)

El comando **`ln`** se usa para crear either hard or symbolic links. Se usa de dos formas:

- Para crear un hard link:
```
ln file link
```

- Para crear un symbolic link:
```
ln -s item link
```

Donde *item* es ya sea un archivo o un directorio.

### Hard Links

Los hard links son la forma original de Unix de crear links, comparados con los symbolic links que son más modernos. Por defecto, cada archivo tiene un único hard link que le da su nombre.

**Limitaciones de los hard links:**

1. **Un hard link no puede referenciar un archivo fuera de su propio sistema de archivos.** Esto significa que un link no puede referenciar un archivo que no esté en la misma partición de disco que el link mismo.

2. **Un hard link no puede referenciar un directorio.**

Un hard link es indistinguible del archivo mismo. A diferencia de un symbolic link, cuando listamos un directorio conteniendo un hard link, no veremos indicación especial del link. Cuando un hard link es eliminado, el link es removido, pero los contenidos del archivo mismo continúan existiendo (es decir, su espacio no es deallocated) hasta que todos los links al archivo sean eliminados.

**Concepto Interno:**

Es importante ser consciente de hard links porque podrías encontrarlos de vez en cuando. Los archivos consisten de dos partes:

- La parte de datos que contiene los contenidos del archivo
- La parte de nombre que sostiene el nombre del archivo

Cuando creamos hard links, estamos creando partes de nombre adicionales que todas se refieren a la misma parte de datos. El sistema asigna una cadena de bloques de disco a lo que se llama un **inode**, que es entonces asociado con la parte de nombre.

Cada hard link por lo tanto se refiere a un específico inode conteniendo los contenidos del archivo.

El comando `ls` tiene una forma de revelar esta información. Cuando se invoca con la opción `-i`:

```bash
ls -i
```

La salida incluye el número de inode del archivo al inicio del listado.

### Symbolic Links

Los symbolic links fueron creados para superar las limitaciones de los hard links. Trabajan creando un tipo especial de archivo que contiene un pointer de texto al archivo o directorio referenciado.

Un archivo referenciado por un symbolic link y el symbolic link mismo son largamente indistinguibles el uno del otro. Por ejemplo, si escribes algo al symbolic link, el archivo referenciado es escrito a.

Sin embargo, cuando eliminamos un symbolic link, solo el link es eliminado, no el archivo mismo. Si el archivo es eliminado antes que el symbolic link, el link continuará existiendo pero apuntará a nada. En este caso, el link se dice que es **broken**.

Muchas distribuciones modernas de Linux mostrarán broken links en un color distintivo, como rojo, para revelar su presencia.

**Patrones Absolutos vs Relativos:**

Cuando creamos symbolic links, podemos usar ya sea absolute pathnames, como se muestra aquí:

```bash
ln -s /home/me/playground/fun dir1/fun-sym
```

O relative pathnames, como hicimos en nuestro ejemplo anterior. En la mayoría de casos, usar relative pathnames es más deseable porque usualmente permite que un directorio conteniendo symbolic links y sus archivos referenciados sea renombrados o movidos sin romper los links.

Además de archivos regulares, los symbolic links también pueden referenciar directorios:

```bash
ln -s dir1 dir1-sym
```

## Ejemplo Práctico: Construir un Playground

Vamos a practicar nuestras habilidades de manipulación de archivos construyendo un área segura para "jugar" con nuestros comandos.

### Crear Directorios

El comando `mkdir` se usa para crear un directorio. Para crear nuestro directorio playground, primero nos aseguraremos de estar en nuestro directorio personal y luego crearemos el nuevo directorio:

```bash
[me@linuxbox ~]$ cd
[me@linuxbox ~]$ mkdir playground
```

Para hacer nuestro playground un poco más interesante, crearemos un par de directorios dentro llamados `dir1` y `dir2`:

```bash
[me@linuxbox ~]$ cd playground
[me@linuxbox playground]$ mkdir dir1 dir2
```

Nota que el comando `mkdir` aceptará múltiples argumentos permitiéndonos crear ambos directorios con un único comando.

### Copiar Archivos

Ahora, obtengamos algo de datos en nuestro playground. Copiaremos el archivo `/etc/passwd` (el archivo que define todas las cuentas de usuario del sistema) a nuestro directorio de trabajo actual:

```bash
[me@linuxbox playground]$ cp /etc/passwd .
```

Observa cómo usamos la abreviatura para el directorio de trabajo actual, el período simple. Así ahora si realizamos un `ls`, veremos nuestro archivo:

```bash
[me@linuxbox playground]$ ls -l
total 12
drwxrwxr-x 2 me me 4096 2025-01-10 16:40 dir1
drwxrwxr-x 2 me me 4096 2025-01-10 16:40 dir2
-rw-r--r-- 1 me me 1650 2025-01-10 16:07 passwd
```

Ahora, para diversión, repitamos la copia usando la opción `-v` (verbose) para ver qué hace:

```bash
[me@linuxbox playground]$ cp -v /etc/passwd .
'/etc/passwd' -> './passwd'
```

El comando `cp` realizó la copia nuevamente pero esta vez mostró un mensaje conciso indicando qué operación estaba realizando. Observa que cp sobrescribió la primera copia sin advertencia. Nuevamente, esto es un caso de `cp` siendo silenciosamente (no habrá advertencia) sobrescribir archivos.

### Mover y Renombrar Archivos

Ahora, el nombre `passwd` no parece muy lúdico, y este es un playground, así que cambiémoslo a algo más.

```bash
[me@linuxbox playground]$ mv passwd fun
```

Pasemos el "fun" alrededor un poco moviendo nuestro archivo renombrado a cada uno de los directorios y de vuelta. Primero, lo movemos al directorio `dir1`:

```bash
[me@linuxbox playground]$ mv fun dir1
```

Luego, lo movemos de `dir1` a `dir2`:

```bash
[me@linuxbox playground]$ mv dir1/fun dir2
```

Finalmente, lo traemos de vuelta al directorio de trabajo actual:

```bash
[me@linuxbox playground]$ mv dir2/fun .
```

A continuación, veamos el efecto de `mv` en directorios. Primero, moveremos nuestro archivo de datos en `dir1` nuevamente:

```bash
[me@linuxbox playground]$ mv fun dir1
```

Luego movemos `dir1` en `dir2` y lo confirmamos con `ls`:

```bash
[me@linuxbox playground]$ mv dir1 dir2
[me@linuxbox playground]$ ls -l dir2
total 4
drwxrwxr-x 2 me me 4096 2025-01-10 16:33 dir1
-rw-r--r-- 1 me me 1650 2025-01-10 16:33 fun
```

Observa que desde que `dir2` ya existía, `mv` movió `dir1` en `dir2`. Si `dir2` no hubiera existido, `mv` habría renombrado `dir1` a `dir2`. Finalmente, pongamos todo nuevamente:

```bash
[me@linuxbox playground]$ mv dir2/dir1 .
[me@linuxbox playground]$ mv dir1/fun .
```

### Crear Hard Links

Ahora intentaremos algunos links. Crearemos algunos hard links a nuestro archivo de datos así:

```bash
[me@linuxbox playground]$ ln fun fun-hard
[me@linuxbox playground]$ ln fun dir1/fun-hard
[me@linuxbox playground]$ ln fun dir2/fun-hard
```

Así ahora tenemos cuatro instancias del archivo `fun`. Veamos nuestro directorio playground:

```bash
[me@linuxbox playground]$ ls -l
total 16
drwxrwxr-x 2 me me 4096 2025-01-14 16:17 dir1
drwxrwxr-x 2 me me 4096 2025-01-14 16:17 dir2
-rw-r--r-- 4 me me 1650 2025-01-10 16:33 fun
-rw-r--r-- 4 me me 1650 2025-01-10 16:33 fun-hard
```

Una cosa que notamos es que ambos archivos `fun` y `fun-hard` tienen un 4 en el segundo campo, que es el número de hard links que ahora existen para el archivo.

Cuando pensamos sobre hard links, es útil imaginar que los archivos consisten de dos partes:

- La parte de datos que contiene los contenidos del archivo
- La parte de nombre que sostiene el nombre del archivo

Cuando creamos hard links, estamos creando partes de nombre adicionales que todos se refieren a la misma parte de datos.

Para investigar más, podemos usar el comando `ls -i` para ver el número de inode. Cuando se invoca con esta opción, la salida inicial muestra el campo inode (número inode):

```bash
[me@linuxbox playground]$ ls -i
total 16
12353539 drwxrwxr-x 2 me me 4096 2025-01-14 16:17 dir1
12353540 drwxrwxr-x 2 me me 4096 2025-01-14 16:17 dir2
12353538 -rw-r--r-- 4 me me 1650 2025-01-10 16:33 fun
12353538 -rw-r--r-- 4 me me 1650 2025-01-10 16:33 fun-hard
```

En esta versión del listado, el primer campo es el número de inode, y como podemos ver, ambos `fun` y `fun-hard` comparten el mismo número de inode, lo que confirma que son el mismo archivo.

### Crear Symbolic Links

Los symbolic links fueron creados para superar las dos desventajas de los hard links.

Los symbolic links son un tipo especial de archivo que contiene un pointer de texto al archivo o directorio referenciado.

Crear symbolic links es similar a crear hard links:

```bash
[me@linuxbox playground]$ ln -s fun fun-sym
[me@linuxbox playground]$ ln -s ../fun dir1/fun-sym
[me@linuxbox playground]$ ln -s ../fun dir2/fun-sym
```

El primer ejemplo es bastante directo; simplemente agregamos la opción `-s` para crear un symbolic link. Pero ¿qué pasa con los próximos dos?

Cuando creamos un symbolic link, creamos una descripción textual de dónde está el archivo relativo al symbolic link. Es más fácil ver si miramos la salida de `ls` mostrada aquí:

```bash
[me@linuxbox playground]$ ls -l dir1
total 4
-rw-r--r-- 4 me me 1650 2025-01-10 16:33 fun-hard
lrwxrwxrwx 1 me me 6 2025-01-15 15:37 fun-sym -> ../fun
```

El listado para `fun-sym` en `dir1` muestra que es un symbolic link por el leading `l` en el primer campo y que apunta a `../fun`, que es correcto. Relativo a la ubicación de `fun-sym`, `fun` está en el directorio arriba. También, tenga en cuenta, que la longitud del symbolic link es 6, la cantidad de caracteres en la string `../fun` en lugar de la longitud del archivo del cual apunta.

Cuando creamos symbolic links, podemos usar ya sea absolute pathnames:

```bash
ln -s /home/me/playground/fun dir1/fun-sym
```

O relative pathnames, como hicimos en nuestro ejemplo anterior. En la mayoría de casos, usar relative pathnames es más deseable porque usualmente permite que un directorio conteniendo symbolic links y sus archivos referenciados sea renombrados o movidos sin romper los links.

Además de archivos regulares, los symbolic links también pueden referenciar directorios:

```bash
ln -s dir1 dir1-sym
```

### Removiendo Archivos y Directorios

Como cubrimos antes, el comando `rm` se usa para eliminar archivos y directorios. Vamos a usar para limpiar nuestro playground un poco. Primero, eliminemos uno de nuestros hard links:

```bash
[me@linuxbox playground]$ rm fun-hard
[me@linuxbox playground]$ ls -l
total 12
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir1
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir2
-rw-r--r-- 3 me me 1650 2025-01-10 16:33 fun
lrwxrwxrwx 1 me me 3 2025-01-15 15:15 fun-sym -> fun
```

Esto funcionó como se esperaba. El archivo `fun-hard` se ha ido, y el link count mostrado para `fun` se redujo de cuatro a tres, como se indica en el segundo campo.

A continuación, eliminemos el archivo `fun`, y solo para disfrutarlo, incluiremos la opción `-i` para mostrar qué hace:

```bash
[me@linuxbox playground]$ rm -i fun
rm: remove regular file 'fun'?
```

Ingresa `y` en el prompt y el archivo es eliminado. Pero miremos la salida de `ls` ahora. Observa qué sucedió con `fun-sym`. Desde que es un symbolic link apuntando a un archivo ahora inexistente, el link es **broken**.

```bash
[me@linuxbox playground]$ ls -l
total 8
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir1
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir2
lrwxrwxrwx 1 me me 3 2025-01-15 15:15 fun-sym -> fun
```

La mayoría de distribuciones de Linux configuran `ls` para mostrar broken links. La presencia de un broken link no es inherentemente peligrosa, pero es bastante desordenado.

Si intentamos usar un broken link, veremos esto:

```bash
[me@linuxbox playground]$ less fun-sym
fun-sym: No such file or directory
```

Limpiemos un poco. Eliminaremos los symbolic links aquí:

```bash
[me@linuxbox playground]$ rm fun-sym dir1-sym
[me@linuxbox playground]$ ls -l
total 8
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir1
drwxrwxr-x 2 me me 4096 2025-01-15 15:17 dir2
```

Una cosa a recordar sobre los symbolic links es que la mayoría de operaciones de archivo se realizan en el target del link, no en el link mismo. Es una excepción. Cuando eliminamos un link, es el link el que es eliminado, no el target.

Finalmente, eliminaremos nuestro playground. Para hacer esto, regresaremos a nuestro directorio personal y usaremos `rm` con la opción recursiva (`-r`) para eliminar `playground` y todos sus contenidos, incluyendo subdirectorios:

```bash
[me@linuxbox playground]$ cd
[me@linuxbox ~]$ rm -r playground
```

## Creando Symlinks con la GUI

Los gestores de archivos en GNOME y KDE proporcionan un método fácil y automático de crear symbolic links. Con GNOME, sostener `Ctrl+Shift` mientras arrastras un archivo creará un link en lugar de copiar (o mover) el archivo. En KDE, un pequeño menú aparece cuando se arroja un archivo, ofreciendo una opción de copiar, mover o hacer link del archivo.

## Resumen

Cubrimos mucho terreno aquí, y tomará un tiempo para que todo se asiente completamente. Realiza el ejercicio playground una y otra vez hasta que tenga sentido. Es importante obtener una buena comprensión de los comandos básicos de manipulación de archivos y comodines. Siéntete libre de expandir el ejercicio playground agregando más archivos y directorios, usando comodines para especificar archivos para varias operaciones. El concepto de links es un poco confuso al principio, pero tómate el tiempo para aprender cómo funcionan. Pueden ser un verdadero ahorro de vidas.

---

## Ver También

- [[wiki/linux/02-navegacion-sistema-archivos.md|Capítulo 2: Navegación del Sistema de Archivos]] — Rutas y directorios base para manipulación
- [[wiki/linux/03-explorando-el-sistema.md|Capítulo 3: Explorando el Sistema]] — Entender estructura antes de modificar
- [[wiki/linux/09-permisos.md|Capítulo 9: Permisos en Linux]] — Control de acceso a archivos modificados
