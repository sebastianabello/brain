---
title: "Capítulo 2: Navegación del Sistema de Archivos"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 2"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-43-48.pdf"
updated: "2026-06-22"
---

# Navegación del Sistema de Archivos

## Introducción

Lo primero que debemos aprender (además de cómo escribir comandos) es cómo navegar el sistema de archivos en Linux. En este capítulo introducimos los siguientes comandos fundamentales:

- **`pwd`** — Imprime el nombre del directorio actual
- **`cd`** — Cambia de directorio
- **`ls`** — Lista contenidos del directorio

## Entendiendo la Estructura del Sistema de Archivos

### Estructura Jerárquica

Como en Windows, un sistema Unix-like como Linux organiza sus archivos en lo que se llama una **estructura jerárquica de directorios**. Esto significa que están organizados en un patrón de árbol de **directorios** (a veces llamados **carpetas** en otros sistemas).

El primer directorio en el sistema de archivos se llama **root directory** (directorio raíz, representado por `/`). El directorio raíz contiene archivos y subdirectorios, que a su vez contienen más archivos y subdirectorios.

### Diferencia con Windows

Una diferencia importante con Windows: en sistemas Unix-like como Linux siempre hay un único árbol del sistema de archivos, **sin importar cuántas unidades de almacenamiento estén conectadas**. Los dispositivos de almacenamiento se **montan** (attach) en varios puntos del árbol de directorios, según los deseos del **administrador del sistema** (la persona responsable de mantener el sistema).

## El Directorio Actual (Current Working Directory)

La mayoría de interfaces gráficas usan un gestor de archivos que representa el árbol del sistema de archivos. Sin embargo, en la línea de comandos necesitamos pensar en él de manera diferente.

Imagina el sistema de archivos como un **laberinto con forma de árbol invertida**: estamos de pie en el medio del laberinto. En cualquier momento dado, estamos dentro de un único directorio. Podemos ver:
- Los archivos contenidos en el directorio
- El camino al directorio padre (llamado **parent directory**)
- Cualquier subdirectorio debajo de nosotros

El directorio en el que estamos ubicados se llama **current working directory** (directorio actual). Para mostrar el nombre del directorio actual, usamos el comando **`pwd`**.

### Comando: `pwd` (Print Working Directory)

Imprime la ruta del directorio actual:

```bash
[me@linuxbox ~]$ pwd
/home/me
```

Cuando iniciamos sesión por primera vez en nuestro sistema (o abrimos un emulador de terminal), nuestro directorio actual se establece a nuestro **home directory** (directorio personal). Cada usuario cuenta con su propio directorio personal, y es el único lugar donde un usuario regular puede escribir archivos.

## Listando Contenidos del Directorio

### Comando: `ls` (List Directory Contents)

Para listar los archivos y directorios en el directorio actual, usamos el comando `ls`:

```bash
[me@linuxbox ~]$ ls
Desktop  Documents  Music  Pictures  Public  Templates  Videos
```

El comando `ls` lista los archivos y directorios en cualquier directorio, no solo en el actual. Hay muchas otras cosas que puede hacer `ls`. Profundizaremos más en el siguiente capítulo.

## Cambiando el Directorio Actual

### Comando: `cd` (Change Directory)

Para cambiar nuestro directorio actual (donde estamos parados en el árbol en forma de laberinto), usamos el comando `cd`. Para hacerlo, escribe `cd` seguido de un espacio y el **nombre de ruta** (pathname) del directorio deseado.

Un **pathname** es la ruta que seguimos por las ramas del árbol para llegar al directorio que queremos. Los nombres de ruta se pueden especificar de dos formas diferentes: como **absolute pathnames** (rutas absolutas) o como **relative pathnames** (rutas relativas).

## Rutas Absolutas (Absolute Pathnames)

Una ruta absoluta comienza con el directorio raíz y sigue la rama del árbol rama por rama hasta que se alcanza la ruta al directorio o archivo deseado.

Por ejemplo, en la mayoría de sistemas Linux, hay un directorio `/usr/bin` que contiene muchos de los programas del sistema. La ruta absoluta es `/usr/bin`. Esto significa:
- Desde el directorio raíz (representado por la barra diagonal al principio)
- Hay un directorio llamado `usr`
- Que contiene un directorio llamado `bin`

### Ejemplo de Ruta Absoluta

```bash
[me@linuxbox ~]$ cd /usr/bin
[me@linuxbox bin]$ pwd
/usr/bin
[me@linuxbox bin]$ ls
... listing of many, many files...
```

Ahora podemos ver que hemos cambiado el directorio actual a `/usr/bin` y está lleno de archivos.

Observa también cómo el prompt del shell ha cambiado: ahora muestra que estamos en el directorio `bin` en lugar del directorio personal.

Como conveniencia, generalmente está configurado para mostrar automáticamente el nombre del directorio de trabajo actual.

## Rutas Relativas (Relative Pathnames)

Donde una ruta absoluta comienza con el directorio raíz y conduce a su destino, una ruta relativa comienza con el directorio actual. Para hacer esto, utiliza un par de notaciones especiales para representar posiciones relativas en el árbol del sistema de archivos. Estas notaciones especiales son:

- **`.`** (punto) — Refiere al directorio actual
- **`..`** (dos puntos) — Refiere al directorio padre del directorio actual

### Ejemplo 1: Cambiar a Directorio Padre con Ruta Relativa

Cambiemos el directorio actual a `/usr/bin` nuevamente:

```bash
[me@linuxbox ~]$ cd /usr/bin
[me@linuxbox bin]$ pwd
/usr/bin
```

Ahora digamos que queremos cambiar el directorio actual al padre de `/usr/bin`, que es `/usr`. Podríamos hacerlo de dos maneras diferentes, usando una ruta absoluta:

```bash
[me@linuxbox bin]$ cd /usr
[me@linuxbox usr]$ pwd
/usr
```

O usando una ruta relativa:

```bash
[me@linuxbox bin]$ cd ..
[me@linuxbox usr]$ pwd
/usr
```

Dos métodos diferentes con resultados idénticos. ¿Cuál debemos usar? **La que requiera menos escritura**.

### Ejemplo 2: Cambiar Múltiples Niveles

Del mismo modo, podemos cambiar el directorio de trabajo de `/usr` a `/usr/bin` de dos maneras diferentes, usando una ruta absoluta:

```bash
[me@linuxbox usr]$ cd /usr/bin
[me@linuxbox bin]$ pwd
/usr/bin
```

O usando una ruta relativa:

```bash
[me@linuxbox usr]$ cd ./bin
[me@linuxbox bin]$ pwd
/usr/bin
```

### Omitiendo el Punto (.)

Ahora bien, hay algo importante que señalar aquí. En casi todos los casos, podemos omitir el `./`. Es implícito. Si escribimos:

```bash
[me@linuxbox usr]$ cd bin
```

Hace lo mismo. En general, si no especificamos un nombre de ruta para algo, el directorio actual será asumido.

## Datos Importantes sobre Nombres de Archivos

En sistemas Linux, los archivos se nombran de una manera similar a otros sistemas como Windows, pero hay algunas diferencias importantes:

### Archivos Ocultos

- **Los archivos que comienzan con un punto (.) están ocultos.** Esto significa que `ls` no los mostrará a menos que escribas `ls -a`. 
- Cuando se creó tu cuenta de usuario, se colocaron varios archivos ocultos en tu directorio personal para configurar cosas de tu cuenta. 
- En el Capítulo 11, examinaremos algunos de estos archivos para ver cómo puedes personalizar tu entorno.
- Además, algunas aplicaciones almacenan sus archivos de configuración y preferencias en tu directorio personal como archivos ocultos.

### Sensibilidad a Mayúsculas/Minúsculas

- **Los nombres de archivos y comandos en Linux, como en Unix, distinguen entre mayúsculas y minúsculas.** Los nombres de archivo `File1` y `file1` se refieren a archivos diferentes.

### Sin Extensiones de Archivo

- **Linux no tiene concepto de "extensión de archivo" como algunos otros sistemas operativos.** Puedes nombrar archivos de cualquier forma que desees. El contenido o propósito de un archivo se determina por otros medios. Aunque los sistemas operativos tipo Unix no usan extensiones de archivo para determinar el contenido o propósito de archivos, muchos programas de aplicación sí lo hacen.

### Nombres Largos con Espacios

- **Linux permite nombres de archivo largos que pueden contener espacios en blanco y caracteres de puntuación.** Limita los caracteres de puntuación en los nombres de los archivos que creas a puntos, guiones y guiones bajos. 
- **Lo más importante, evita incrustar espacios en los nombres de archivos.** Si quieres representar espacios entre palabras en un nombre de archivo, usa caracteres de guion bajo. Te lo agradecerás más tarde.

## Atajos Útiles para el Comando `cd`

La siguiente tabla muestra algunas formas útiles de cambiar rápidamente el directorio actual:

| Atajo | Resultado |
|-------|-----------|
| `cd` | Cambia el directorio actual al directorio personal |
| `cd -` | Cambia el directorio actual al directorio anterior |
| `cd ~usuario` | Cambia el directorio actual al directorio personal de *usuario* (por ejemplo, `cd ~bob` cambiará el directorio al directorio personal del usuario bob) |

## Resumen

Este capítulo explicó cómo el shell trata la estructura de directorios del sistema. Aprendimos sobre rutas absolutas y relativas, y los comandos básicos que usamos para movernos alrededor de esa estructura. En el siguiente capítulo, exploraremos una aplicación de estos conocimientos para explorar un sistema Linux moderno.

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]]
- [[wiki/linux/comandos-basicos.md|Referencia de Comandos Básicos]]
