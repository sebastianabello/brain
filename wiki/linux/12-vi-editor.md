---
title: "Capítulo 12: Una Introducción Suave a VI(M)"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 12"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-169-185.pdf"
updated: "2026-06-23"
---

# Una Introducción Suave a VI(M)

## ¿Por Qué Aprender vi?

En esta era moderna de editores gráficos y editores de texto fáciles de usar como nano, ¿por qué deberíamos aprender vi? Hay tres buenas razones:

1. **vi está disponible en casi todas partes**: Esto puede ser un salvavidas si tienes un sistema sin interfaz gráfica, como un servidor remoto o un sistema con una configuración GUI rota. nano, aunque cada vez más popular, aún no es universal. POSIX, un estándar para compatibilidad entre programas en sistemas Unix, requiere que vi esté presente.

2. **vi es ligero y rápido**: Para muchas tareas, es más fácil traer vi que esperar a que el editor gráfico se cargue desde el menú. Además, vi está diseñado para escribir rápido. Como veremos, usuarios expertos nunca tienen que levantar sus dedos del teclado mientras editan.

3. **No queremos que otros usuarios de Linux y Unix piensen que somos cobardes**: Bueno, tal vez dos buenas razones.

## Un Poco de Antecedentes

La primera versión de vi fue escrita en 1976 por Bill Joy, un estudiante de la Universidad de California-Berkeley que más tarde fue a co-fundar Sun Microsystems. vi toma su nombre de la palabra "visual" (visual), porque fue concebido para permitir edición en un terminal de video con un cursor móvil. Antes de los editores "visuales", había **line editors** (editores de línea) que operaban en una sola línea de texto a la vez.

Para especificar un cambio, le dirías al editor que vaya a una línea particular y describa qué cambio hacer, como agregar o eliminar texto. Con el advenimiento de los terminales de video (en lugar de terminales basadas en impresoras como teletypes), la edición visual se hizo posible. vi es un editor de línea visual muy potente, y podemos usar comandos de edición de línea mientras usamos vi.

En la mayoría de distribuciones Linux no incluyen vi real; en su lugar, usan una versión mejorada llamada **vim** (que es la abreviatura de "vi improved"), escrita por Bram Moolenaar. vim es una mejora sustancial sobre el vi tradicional y a menudo está simbólicamente vinculado (o aliased) al nombre vi en los sistemas Linux.

## Iniciando y Deteniendo vi

Para iniciar vi, simplemente ingresa lo siguiente:

```bash
[me@linuxbox ~]$ vi
```

Una pantalla como esta debería aparecer:

```
~
~
~
                    VIM - Vi Improved
~
~
~
                version 9.1.607
            by Bram Moolenaar et al.
Vim is open source and freely distributable

        Sponsor Vim development!
      type :help sponsorCinter>    for information

        type :q<Enter>          to exit
        type :help<Enter> or <F1> for on-line help
        type :help version8<Enter>   for version info

        Running in VI compatible mode
        type :set nocp<Enter>     for Vim defaults
        type :help cp-default<Enter>  for info on this
```

Para salir, ingresa el siguiente comando (nota que el carácter de dos puntos es parte del comando):

```
:q
```

El prompt del shell debería regresar. Si, por alguna razón, vi no quiere salir (usualmente porque hicimos un cambio a un archivo que aún no ha sido guardado), puedes decirle a vi que realmente lo dices agregando un signo de exclamación al comando.

```
:q!
```

> **Nota**: Si obtienes "lost" en vi, intenta presionar ESC dos veces para encontrar tu camino nuevamente.

### Modo Compatible

En la pantalla de inicio del ejemplo, vemos que el texto dice que vi se ejecutará en un modo que es más cercano al comportamiento normal de vi en lugar del comportamiento mejorado de vim. Para los propósitos de este capítulo, querremos ejecutar vim con su comportamiento mejorado. Para hacer esto, tienes algunas opciones. Intenta ejecutar vim en lugar de vi. Si eso funciona, considera agregar una línea:

```bash
alias vi='vim'
```

a tu archivo `.bashrc`. Alternativamente, usa este comando para agregar una línea a tu archivo de configuración de vim:

```bash
echo "set nocp" >> ~/.vimrc
```

> **Nota**: Diferentes distribuciones de Linux empacan vim de diferentes maneras. Algunas distribuciones instalan una versión mínima de vim (vim-tiny) que por defecto solo soporta un conjunto limitado de características de vim. Mientras realizas las lecciones que siguen, puedes encontrar características faltantes. Si este es el caso, instala la versión completa de vim.

## Modos de Edición

Vamos a iniciar vi nuevamente, esta vez pasándole el nombre de un archivo inexistente. De esta manera podemos crear un nuevo archivo con vi:

```bash
[me@linuxbox ~]$ rm -f foo.txt
[me@linuxbox ~]$ vi foo.txt
```

Si todo va bien, deberíamos obtener una pantalla como esta:

```
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
~
"foo.txt" [New File]
```

Los caracteres tilde (~) líderes indican que no hay texto en esa línea. Esto muestra que tenemos un archivo vacío. **No escribas nada aún!**

La segunda cosa más importante para aprender sobre vi (después de aprender a salir) es que vi es un **modal editor** (editor modal). Cuando vi inicia, comienza en **normal mode** (modo normal). En este modo, casi cada tecla es un comando, así que si empezáramos a escribir, vi sería loco y haría un lío.

## Modos de Operación

### Modo Normal

En el modo normal, casi cada pulsación de tecla es un comando.

### Entrando en el Modo Insert (Inserción)

Para agregar texto a nuestro archivo, debemos primero entrar en **insert mode** (modo insert). Para hacer esto, escribimos `i`. Después, deberíamos ver lo siguiente en la parte inferior de la pantalla si vi se ejecuta en su modo mejorado usual (esto no aparecerá en el modo compatible de vi):

```
-- INSERT --
```

Ahora podemos ingresar algún texto. Intenta esto:

```
The quick brown fox jumps over the lazy dog.
```

Para salir del modo insert y regresar al modo normal, presiona ESC.

### Guardando Nuestro Trabajo

Para guardar el cambio que acabamos de hacer a nuestro archivo, debemos entrar en **command mode** (modo de comando). Esto se hace presionando el carácter `:` mientras estamos en el modo normal. Después de hacer esto, un carácter de dos puntos debería aparecer en la parte inferior de la pantalla:

```
:
```

Para escribir nuestro archivo modificado, seguimos el dos puntos con una `w` y luego presionamos ENTER:

```
:w
```

El archivo se escribirá en el disco duro, y deberíamos obtener un mensaje de confirmación en la parte inferior de la pantalla, como esto:

```
"foo.txt" [New] 1L, 45C written
```

> **Nota**: Mientras vi llama a los tres modos principales de edición, modo normal, insert, y comando, vi real (y su documentación) llama a estos modos comando, insert, y ex, respectivamente. Muchos tutoriales en línea usarán los nombres del modo tradicional vi, y sí, puede ser confuso.

## Moviendo el Cursor

Mientras está en modo normal, vi ofrece una gran cantidad de comandos de movimiento, algunos de los cuales comparte con less. La tabla 12-1 enumera un subconjunto.

### Tabla de Movimiento del Cursor

| Tecla | Mueve el cursor |
|-------|-----------------|
| **l** o **flecha derecha** | Un carácter a la derecha |
| **h** o **flecha izquierda** | Un carácter a la izquierda |
| **j** o **flecha abajo** | Una línea hacia abajo |
| **k** o **flecha arriba** | Una línea hacia arriba |
| **0** (cero) | Al comienzo de la línea actual |
| **^** | Al primer carácter de no-whitespace en la línea actual |
| **$** | Al final de la línea actual |
| **w** | Al comienzo de la siguiente palabra o carácter de puntuación |
| **W** | Al comienzo de la siguiente palabra, ignorando caracteres de puntuación |
| **b** | Al comienzo de la palabra anterior o carácter de puntuación |
| **B** | Al comienzo de la palabra anterior, ignorando caracteres de puntuación |
| **Ctrl+F** o **PAGE DOWN** | Una página hacia abajo |
| **Ctrl+B** o **PAGE UP** | Una página hacia arriba |
| **numberG** | A la línea número. Por ejemplo, 1G mueve a la primera línea del archivo |
| **G** | A la última línea del archivo |

¿Por qué `h`, `j`, `k`, y `l` se usaron para el movimiento del cursor? Cuando vi se escribió originalmente, no todos los terminales de video tenían teclas de flecha, y los tipistas expertos podían usar las teclas de flecha regulares del teclado para moverse sin tener que levantar sus dedos del teclado.

Muchos comandos en vi pueden ser prefijados con un número, como sucedió con el comando `G` anteriormente. Al prefijar un comando con un número, especificamos cuántas veces se debe ejecutar el comando. Por ejemplo, el comando `5j` hace que vi se mueva el cursor hacia abajo cinco líneas.

## Edición Básica

La mayoría de la edición consiste en pocas operaciones básicas como insertar texto, eliminar texto, y mover texto alrededor cortando y pegando. vi, por supuesto, soporta todas estas operaciones de su propia manera única. vi también proporciona una forma limitada de deshacer. Si escribimos `u` mientras estamos en modo normal, vi deshará el último cambio que hiciste. Esto será práctico a medida que intentes algunos de los comandos de edición básicos.

### Anexando Texto (Appending)

vi tiene varias formas diferentes de entrar en modo insert. Ya hemos usado el comando `i` para insertar texto. Déjame ir de nuevo a nuestro archivo foo.txt para un momento:

```
The quick brown fox jumps over the lazy dog.
```

Si querríamos agregar texto al final de esta oración, descubriríamos que el comando `i` no lo hará, ya que no podemos mover el cursor más allá del final de la línea. vi proporciona un comando para anexar texto, el cual se nombra apropiadamente como comando. Si movemos el cursor al final de la línea y escribimos `a`, el cursor se moverá más allá del final de la línea, y vi entrará en modo insert. Esto nos permitirá agregar más texto.

Primero, moveremos el cursor al comienzo de la línea usando el comando `0` (cero). Ahora escribiremos `a` y agregaremos el siguiente texto:

```
The quick brown fox jumps over the lazy dog. It was cool.
```

Presiona ESC para salir del modo insert.

Ya que casi siempre querremos anexar texto al final de una línea, vi ofrece un atajo para mover al final de la línea actual y comenzar a anexar. Es el comando `A` (mayúscula). Intenta usarlo y agrega algunas líneas más a nuestro archivo.

Primero, moveremos el cursor al comienzo de la línea usando el comando `0` (cero). Ahora escribiremos `A` y agregaremos las siguientes líneas de texto:

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3
Line 4
Line 5
```

Presiona ESC para salir del modo insert.

Como podemos ver, el comando `A` es más útil ya que mueve el cursor al final de la línea antes de comenzar el modo insert.

### Abriendo una Línea (Opening a Line)

Otra manera en que podemos insertar texto es "abriendo" una línea. Esto inserta una línea en blanco entre dos líneas existentes y entra en modo insert. Esto tiene dos variantes, como se describe en la siguiente tabla:

| Comando | Abre |
|---------|------|
| **o** | La línea debajo de la línea actual |
| **O** | La línea arriba de la línea actual |

Podemos demostrar esto de la siguiente manera: Coloca el cursor en la línea 3 y luego escribe `o`:

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3

Line 4
Line 5
```

Una nueva línea fue abierta debajo de la tercera línea, y entramos en modo insert. Presiona ESC para salir del modo insert y deshacer nuestro cambio presionando `U`.

### Eliminando Texto (Deleting Text)

Como podemos esperar, vi ofrece una variedad de formas de eliminar texto, todas las cuales contienen uno de dos pulsaciones de teclas. Primero, el comando `x` eliminará un carácter en la ubicación del cursor. `x` puede ser precedido por un número especificando cuántos caracteres se eliminarán. El comando `d` es más general.

Como `x`, puede ser precedido por un número especificando el número de veces que la eliminación es para ser realizada. Además, `d` siempre es seguido por un movimiento comando que controla el tamaño de la eliminación.

| Comando | Elimina |
|---------|---------|
| **x** | El carácter actual |
| **3x** | El carácter actual y los siguientes dos caracteres |
| **dd** | La línea actual |
| **5dd** | La línea actual y las siguientes cuatro líneas |
| **dw** | Desde la posición actual del cursor hasta el comienzo de la siguiente palabra |
| **d$** | Desde la posición actual del cursor hasta el final de la línea actual |
| **d0** | Desde la posición actual del cursor hasta el comienzo de la línea |
| **d^** | Desde la posición actual del cursor hasta el primer carácter de no-whitespace en la línea |
| **dG** | Desde la línea actual hasta el final del archivo |
| **d20G** | Desde la línea actual hasta la vigésima línea del archivo |

Coloca el cursor en la palabra "It" en la primera línea de nuestro texto. Presiona `X` repetidamente hasta que el resto de la oración sea eliminada. A continuación, presiona `U` repetidamente para deshacer la eliminación.

> **Nota**: Real vi soporta solo un único nivel de deshacer. vim soporta múltiples niveles.

Intenta la eliminación nuevamente, esta vez usando el comando `d`. Nuevamente, mueve el cursor a la palabra "It" y escribe `d$` para eliminar desde el cursor posición hasta el final de la línea.

```
The quick brown fox jumps over the lazy dog.
Line 2
Line 3
Line 4
Line 5
```

Escribe `d$` para eliminar desde la posición del cursor hasta el final de la línea.

```
The quick brown fox jumps over the lazy dog.
Line 2
Line 3
Line 4
Line 5
```

Escribe `d0` para eliminar desde la posición actual del cursor hasta el comienzo de la línea.

```
~
~
```

Presiona `u` tres veces para deshacer la eliminación.

### Cortando, Copiando y Pegando Texto (Cutting, Copying, and Pasting)

El comando `d` no solo elimina texto sino que también lo "corta". Cada vez que usamos el comando `d`, el texto eliminado se copia a un buffer de pegue (piensa en clipboard) que podemos recordar más tarde con el comando `p` para pegar el contenido del buffer después del cursor o con el comando `P` comando para pegar los contenidos antes del cursor.

El comando `y` se usa para "yankar" (copiar) texto mucho en la misma manera en que el comando `d` se usa para cortar texto. La tabla 12-4 proporciona algunos ejemplos de comandos de combinación del comando `y` con varios comandos de movimiento.

| Comando | Copia |
|---------|-------|
| **yy** | La línea actual |
| **5yy** | La línea actual y las siguientes cuatro líneas |
| **yw** | Desde la posición actual del cursor hasta el comienzo de la siguiente palabra |
| **y$** | Desde la posición actual del cursor hasta el final de la línea actual |
| **y0** | Desde la posición actual del cursor hasta el comienzo de la línea |
| **y^** | Desde la posición actual del cursor hasta el primer carácter de no-whitespace en la línea |
| **yG** | Desde la línea actual hasta el final del archivo |
| **y20G** | Desde la línea actual hasta la vigésima línea del archivo |

Intenta copiar y pegar. Coloca el cursor en la primera línea del texto y escribe `yy` para copiar la línea actual. A continuación, mueve el cursor a la última línea e introduce `p` para pegar la línea debajo de la línea actual.

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3
Line 4
Line 5
The quick brown fox jumps over the lazy dog. It was cool.
```

Como antes, el comando `u` deshará nuestro cambio. Con el cursor aún posicionado en la última línea del archivo, escribe `?` para pegar el texto arriba de la línea actual.

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3
Line 4
The quick brown fox jumps over the lazy dog. It was cool.
Line 5
```

Intenta algunos de los otros comandos `y` en la tabla 12-4 y ponte familiar con el comportamiento de ambos comandos `p` y `?`. Cuando haya terminado, devuelve el archivo a su estado original.

### Uniendo Líneas (Joining Lines)

vi es bastante estricto sobre su idea de una línea. Normalmente, no es posible mover el cursor al final de una línea y eliminar el carácter de fin de línea para unirse a una línea con la de abajo. Debido a esto, vi proporciona un comando específico, `J` (no confundir con `j`, que es para movimiento del cursor), para unir líneas.

Si colocamos el cursor en la línea 3 y escribimos el comando `J`, aquí es lo que sucede:

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3 Line 4
Line 5
```

## Búsqueda y Reemplazo (Search-and-Replace)

vi tiene la capacidad de mover el cursor a ubicaciones basadas en búsquedas. Puede hacer esto en una sola línea o en todo un archivo. También puede realizar reemplazos de texto con o sin confirmación del usuario.

### Búsqueda Dentro de una Línea

El comando `f` busca una línea y mueve el cursor a la siguiente instancia del carácter especificado. Por ejemplo, el comando `fa` movería el cursor a la siguiente ocurrencia del carácter `a` dentro de la línea actual. Después de realizar una búsqueda de carácter dentro de una línea, la búsqueda puede ser repetida escribiendo un punto y coma.

### Búsqueda en Todo el Archivo

Para mover el cursor a la siguiente ocurrencia de una palabra o frase, se usa el comando `/`. Esto funciona de la misma manera que aprendimos anteriormente en el programa less. Cuando escribes el comando `/`, una `/` aparecerá en la parte inferior de la pantalla. A continuación, escribe la palabra o frase a ser buscada, seguida por ENTER.

El cursor se moverá a la siguiente ubicación que contiene la cadena de búsqueda. Una búsqueda puede ser repetida usando el comando anterior `n` con la cadena de búsqueda anterior.

Aquí hay un ejemplo:

```
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3
Line 4
Line 5
```

Coloca el cursor en la primera línea del archivo. Escribe esto y presiona ENTER:

```
/Line
```

El cursor se moverá a la línea 2. Presiona `n` y el cursor se moverá a la línea 3. Repetir el comando `n` moverá el cursor hacia abajo en el archivo hasta que se agote las coincidencias. Aunque hemos usado hasta ahora solo palabras y frases para nuestros patrones de búsqueda, vi permite el uso de **regular expressions** (expresiones regulares), un método poderoso de expresar patrones de texto complejos. Cubriremos expresiones regulares completamente en el Capítulo 19.

### Búsqueda y Reemplazo Global

vi usa el modo de comando para realizar operaciones de búsqueda y reemplazo (llamadas **substitution** en vi) en un rango de líneas o en todo el archivo. Para cambiar la palabra "Line" en la línea 1 al "Line" en todo el archivo, ingresaríamos el siguiente comando:

```
:%s/Line/line/g
```

Desglosar este comando en elementos separados y ver qué hace cada uno (ver tabla 12-5):

| Elemento | Significado |
|----------|-------------|
| **:** | El carácter de dos puntos entra en modo de comando |
| **%** | Especifica el rango de líneas para la operación. `%` es un atajo que significa desde la primera línea hasta la última línea. Alternativamente, el rango podría haber sido `1,5` (desde la línea 1 al final del archivo) o `1,5`, lo que significa "desde la línea 1 a la última línea del archivo". Si el rango de líneas es omitido, la operación es realizada en solo la línea actual. |
| **s** | Esto especifica la operación. En este caso, es la sustitución (búsqueda y reemplazo) |
| **/Line/line/** | Esto especifica el patrón de búsqueda y el texto de reemplazo |
| **g** | Esto significa "global" en el sentido que la búsqueda y reemplazo es realizada en cada instancia de la cadena de búsqueda en la línea. Si es omitido, solo la primera instancia de la cadena de búsqueda en cada línea es reemplazada. |

Después de ejecutar nuestro comando de búsqueda y reemplazo, nuestro archivo se verá así:

```
The quick brown fox jumps over the lazy dog. It was cool.
line 2
line 3
line 4
line 5
```

También podemos especificar un comando de sustitución con confirmación del usuario. Esto se hace agregando una `c` al final del comando. Aquí hay un ejemplo:

```
:%s/Line/line/gc
```

Este comando cambiará nuestro archivo nuevamente a su forma anterior; sin embargo, antes de cada sustitución, vi se detiene y nos pide que confirmemos la sustitución con este mensaje:

```
replace with Line (y/n/a/l) /?
```

Cada uno de los caracteres dentro de los paréntesis es una opción posible, como se describe en la siguiente tabla:

| Tecla | Acción |
|-------|--------|
| **y** | Realizar la sustitución |
| **n** | Saltar esta instancia del patrón |
| **a** | Realizar la sustitución en esta y en todas las instancias subsecuentes del patrón |
| **q** o **ESC** | Dejar de sustituir |
| **l** | Realizar esta sustitución y luego dejar de sustituir. Esto es la abreviatura de "last" |
| **Ctrl+E, Ctrl+Y** | Desplazarse hacia abajo y desplazarse hacia arriba, respectivamente. Esto es útil para ver el contexto de la sustitución propuesta |

Si escribes `y`, la sustitución será realizada. `n` hará que vi saltee esta instancia y se mueva a la siguiente.

## Edición de Múltiples Archivos

Es a menudo útil editar más de un archivo a la vez. Puedes abrir múltiples archivos para editar especificándolos en la línea de comandos.

```bash
vi file1 file2 file3...
```

Comencemos con solo un archivo. Intenta salir de vi y crear un nuevo archivo para editar. Escribe lo siguiente:

```bash
[me@linuxbox ~]$ vi foo.txt
```

Para agregar nuestro segundo archivo, ingresa lo siguiente:

```
:e ls-output.txt
```

Debería aparecer en la pantalla. El primer archivo aún está presente como podemos verificar de esta manera:

```
:buffers
```

Esto mostrará una lista de los archivos en la parte inferior de la pantalla.

### Cambiando Entre Archivos

Para cambiar de un archivo a otro, usa este comando `ex`:

```
:bn
```

Para moverte al archivo anterior, usa lo siguiente:

```
:bp
```

Aunque podemos moveros de un archivo a otro, vi hace cumplir una política que previene cambiar archivos si el archivo actual tiene cambios no guardados. Para forzar vi a cambiar archivos y abandonar tus cambios, agrega un signo de exclamación (!) al comando.

Además de cambiar archivos con el método descrito anteriormente, vim (y algunas versiones de vi) proporciona algunos comandos del modo de comando que hacen edición de múltiples archivos más fácil de manejar. Podemos ver una lista de archivos siendo editados con el comando `:buffers`.

### Abriendo Archivos Adicionales para Editar

También es posible agregar archivos a nuestra sesión actual de edición. El comando del modo de comando `:e` (abreviatura de "edit") seguido por un nombre de archivo abrirá un archivo adicional. Déjamos salir de nuestra sesión actual de edición y regresemos a la línea de comandos.

Iniciar vi nuevamente con solo un archivo:

```bash
[me@linuxbox ~]$ vi foo.txt
```

Para agregar nuestro segundo archivo, ingresa lo siguiente:

```
:e ls-output.txt
```

Debería aparecer en la pantalla. El primer archivo aún está presente como podemos verificar de esta manera:

```
:buffers
1 %a "foo.txt"                    Line 1
2 %a "ls-output.txt"              Line 0
Press ENTER or type command to continue
```

### Copiando Contenido de Un Archivo a Otro

A menudo mientras editamos múltiples archivos, querremos copiar una porción de un archivo a otro que estamos editando. Esto se hace fácilmente usando los comandos de yank-y-paste usuales que usamos anteriormente. Podemos demostrar de la siguiente manera.

Primero, usando nuestros dos archivos, cambia al buffer 1 (foo.txt) por ingresando esto:

```
:buffer 1
```

Nuestro archivo debería mostrarse ahora.

A continuación, mueve el cursor a la primera línea y escribe `yy` para copiar (yank) la línea. Cambia al segundo buffer por ingresando lo siguiente:

```
:buffer 2
```

La pantalla ahora contendrá listados de archivos como este (solo una porción se muestra aquí):

```
-total 343700
-rwxr-xr-x 1 root root 31316 2024-12-05 08:58 [
-rwxr-xr-x 1 root root  8240 2024-12-09 13:39 4itopmm
-rwxr-xr-x 1 root root 111276 2025-01-31 13:36 a2p
```

Mueve el cursor a la primera línea y pega la línea que copiamos del archivo anterior escribiendo el comando `p`.

```
-total 343700
The quick brown fox jumps over the lazy dog. It was cool.
-rwxr-xr-x 1 root root 31316 2024-12-05 08:58 [
```

Como antes, el comando `u` deshará nuestro cambio. Con el cursor aún posicionado en la última línea del archivo, escribe `?` para pegar el texto arriba de la línea actual.

### Insertando un Archivo Completo en Otro

También es posible insertar un archivo completo en uno que estamos editando. Para ver esto en acción, déjemos nuestra sesión de edición anterior y comenzemos un nuevo con solo un archivo.

```bash
[me@linuxbox ~]$ vi ls-output.txt
```

Veremos nuestro listado de archivos de nuevo.

```
-total 343700
-rwxr-xr-x 1 root root 31316 2024-12-05 08:58 [
-rwxr-xr-x 1 root root  8240 2024-12-09 13:39 4itopmm
-rwxr-xr-x 1 root root 111276 2025-01-31 13:36 a2p
```

Mueve el cursor a la tercera línea, y luego ingresa el siguiente comando del modo de comando:

```
:r foo.txt
```

El comando `:r` (abreviatura de "read") inserta el archivo especificado debajo de la posición del cursor. Nuestra pantalla ahora debería verse así:

```
-total 343700
-rwxr-xr-x 1 root root 31316 2024-12-05 08:58 [
-rwxr-xr-x 1 root root  8240 2024-12-09 13:39 4itopmm
The quick brown fox jumps over the lazy dog. It was cool.
Line 2
Line 3
Line 4
Line 5
-rwxr-xr-x 1 root root 111276 2025-01-31 13:36 a2p
```

## Guardando Nuestro Trabajo

Como todo lo demás en vi, hay varias formas diferentes de guardar nuestros archivos editados. Ya hemos cubierto el comando `:w`, pero hay algunos otros que también podemos encontrar útiles.

En el modo normal, escribir `ZZ` guardará el archivo actual y saldrá de vi. Asimismo, el comando del modo de comando `:wq` combinará los comandos `:w` y `:q` en uno que guardará el archivo y saldrá.

El comando `:w` también puede especificar un nombre de archivo opcional. Esto actúa como "Save As". Por ejemplo, si estuviéramos editando `foo.txt` y querríamos guardar una versión alternativa llamada `fool.txt`, ingresaríamos lo siguiente:

```
:w fool.txt
```

> **Nota**: Aunque este comando guarda el archivo bajo un nuevo nombre, no cambia el nombre del archivo que estamos editando. A medida que continuamos editando, aún estaremos editando foo.txt, no fool.txt.

## bash También Usa vi

En el Capítulo 8 vimos las varias maneras en que podríamos editar el contenido de la línea de comandos. Los comandos de edición particulares que bash usa no son arbitrarios. Están inspirados por el editor de texto emacs. Esta es la predeterminada en bash, pero bash también soporta edición de línea de comando al estilo vi también. Esta característica se activa fácilmente con el siguiente comando:

```bash
[me@linuxbox ~]$ set -o vi
```

Una vez esto se hace, podemos usar muchos de los comandos de estilo vi que hemos aprendido. Intenta. En el prompt del comando, escribe el siguiente texto de ejemplo:

```bash
[me@linuxbox ~]$ the quick brown fox jumps over the lazy dog
```

Podemos mover el cursor con las teclas de flecha como antes, y podemos escribir caracteres de la manera normal. Se comporta de esta manera porque cuando iniciamos una nueva línea de comandos, el editor está en modo insert y se comporta como en vi. Para obtener el material bueno, tenemos que cambiar al modo normal. Presionamos ESC. Todos los comandos de movimiento, yank, delete, y paste funcionan justo como si estuviéramos editando un archivo de texto de una línea en vi. Para regresar al modo insert, hacemos uso de las teclas A o I. Entonces, mientras establezca bash para usar edición de línea de comando al estilo vi, te permite usar la mayoría de los comandos de edición de vi en la línea de comandos.

Configurar bash para usar edición de línea de comando al estilo vi es una buena manera de reforzar tus habilidades de teclado vi, y tiene el beneficio adicional de reducir el número de comandos de edición que tienes que recordar. Dale un intento. Para hacerlo permanente, podemos agregar la línea `set -o vi` comando a nuestro archivo `.bashrc`.

Para regresar al modo de edición al estilo emacs, ingresa este comando:

```bash
[me@linuxbox ~]$ set -o emacs
```

> **Nota**: Muchos tutoriales en línea están disponibles para esta característica, pero ten en cuenta que la mayoría usará los nombres del modo tradicional vi (command, insert, y ex) en lugar de los nombres del modo vi (normal, insert, y command).

## Resumen

Con este conjunto básico de habilidades, ahora podemos realizar la mayoría de la edición de texto necesaria para mantener un sistema Linux típico. Aprender a usar vi en una base regular te pagará en el largo plazo. Como los editores al estilo vi están tan profundamente integrados en la cultura Unix, veremos muchos otros programas que han sido influenciados por su diseño. less es un buen ejemplo de esta influencia.

---

## Ver También

- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment (Entorno) de Linux]]
- [[wiki/linux/08-trucos-teclado.md|Capítulo 8: Trucos Avanzados del Teclado]]
- [[wiki/linux/comandos-basicos.md|Referencia de Comandos Básicos]]
