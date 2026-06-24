---
title: "Capítulo 13: Personalizando el Prompt"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 13"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-187-194.pdf"
updated: "2026-06-23"
---

# Personalizando el Prompt

## Anatomía de un Prompt

Nuestro prompt por defecto se ve algo como esto:

```
[me@linuxbox ~]$
```

Nota que contiene nuestro nombre de usuario, nuestro nombre de host, y nuestro directorio de trabajo actual, pero ¿cómo llegó a ser de esa manera? Muy simplemente, resulta. El prompt está definido por una variable de environment llamada **PS1** (abreviatura de "prompt string 1"). Podemos ver el contenido de PS1 con el comando echo:

```bash
[me@linuxbox ~]$ echo $PS1
[\u@\h \W]\$
```

> **Nota**: No te preocupes si tus resultados no son los mismos que este ejemplo. Cada distribución de Linux define la cadena de prompt un poco diferente, algunas bastante exóticas.

De los resultados, podemos ver que PS1 contiene algunos de los caracteres que vemos en nuestro prompt, como los corchetes, el signo @ y el signo de dólar, pero el resto son un misterio. Los astutos entre nosotros reconocerán estos como caracteres especiales escapados con barra invertida como los que vimos en el Capítulo 7.

## Códigos de Escape para el Prompt

La siguiente tabla proporciona una lista parcial de los caracteres que bash trata especialmente en la cadena del prompt:

| Secuencia | Valor mostrado |
|-----------|----------------|
| **\a** | Timbre ASCII. Esto hace que el computador emita un beep cuando se encuentra. |
| **\d** | Fecha actual en formato día, mes, fecha; por ejemplo, "Mon May 26". |
| **\e** | Carácter de escape ASCII (033). |
| **\h** | Nombre de host de la máquina local menos el nombre de dominio final. |
| **\H** | Nombre de host completo. |
| **\j** | Número de trabajos (jobs) ejecutándose en la sesión actual del shell. |
| **\l** | Nombre del dispositivo terminal actual. |
| **\n** | Un carácter de nueva línea. |
| **\r** | Un retorno de carro. |
| **\s** | Nombre del programa shell. |
| **\t** | Hora actual en formato 24-horas horas:minutos:segundos. |
| **\T** | Hora actual en formato 12-horas. |
| **\@** | Hora actual en formato 12-horas am/pm. |
| **\A** | Hora actual en formato 24-horas horas:minutos. |
| **\u** | Nombre de usuario del usuario actual. |
| **\v** | Número de versión del shell. |
| **\V** | Números de versión y lanzamiento del shell. |
| **\w** | Nombre del directorio de trabajo actual. |
| **\W** | Última parte del nombre del directorio de trabajo actual. |
| **\!** | Número de historial del comando actual. |
| **\#** | Número de comandos ingresados durante esta sesión del shell. |
| **\$** | Esto muestra un carácter $ a menos que tengamos privilegios de superusuario. En ese caso, muestra un # en su lugar. |
| **\[** | Señala el comienzo de una serie de uno o más caracteres de no-impresión. Esto se usa para incrustar caracteres de control de no-impresión que manipulen el emulador de terminal de alguna manera, como mover el cursor o cambiar colores de texto. Esto es necesario para calcular correctamente la longitud del prompt para el posicionamiento correcto del cursor. |
| **\]** | Señala el final de una secuencia de caracteres de no-impresión. |

## Probando Algunos Diseños de Prompt Alternativos

Con esta lista de caracteres especiales, podemos cambiar el prompt para ver el efecto. Primero, haremos una copia de seguridad de la cadena de prompt existente para que podamos restaurarla más tarde. Para hacer esto, copiaremos la cadena existente en otra variable del shell que creamos nosotros mismos:

```bash
[me@linuxbox ~]$ ps1_old="$PS1"
```

Creamos una nueva variable llamada `ps1_old` y asignamos el valor de PS1 a ella. Podemos verificar que la cadena ha sido copiada usando el comando echo.

```bash
[me@linuxbox ~]$ echo $ps1_old
[\u@\h \W]\$
```

Podemos restaurar el prompt original en cualquier momento durante nuestra sesión de terminal simplemente invirtiendo el proceso.

```bash
[me@linuxbox ~]$ PS1="$ps1_old"
```

### Prompt Vacío

Ahora que estamos listos para proceder, veamos qué pasa si tenemos una cadena de prompt vacía.

```bash
[me@linuxbox ~]$ PS1=
```

Si asignamos nada a la cadena de prompt, obtenemos nada. ¡Sin cadena de prompt en absoluto! El prompt aún está ahí pero no muestra nada, exactamente como le pedimos. Como esto es un poco desconcertante de mirar, lo reemplazaremos con un prompt mínimo.

```bash
PS1="\$ "
```

Eso es mejor. Al menos ahora podemos ver qué estamos haciendo. Nota el espacio final dentro de las comillas dobles. Esto proporciona el espacio entre el signo de dólar y el cursor cuando se muestra el prompt.

### Agregando un Beep

Déjame agregar un beep a nuestro prompt.

```bash
$ PS1="\[\a\]\$ "
```

Ahora deberíamos escuchar un beep cada vez que se muestre el prompt, aunque algunos sistemas desactivan esta "característica". Esto podría ser molesto, pero podría ser útil si necesitáramos notificación cuando se ha ejecutado un comando especialmente largo. Nota que incluimos las secuencias `\[` y `\]`. Como el timbre ASCII (`\a`) no "imprime", es decir, no mueve el cursor, necesitamos decirle a bash para que pueda determinar correctamente la longitud del prompt.

### Prompt con Información

A continuación, intentemos hacer un prompt informativo con información de nombre de host y hora del día.

```bash
$ PS1="\A \h \$ "
17:33 linuxbox $
```

Agregar la hora del día a nuestro prompt será útil si necesitamos mantener un registro de cuándo realizamos ciertas tareas. Finalmente, haremos un nuevo prompt que sea similar al original.

```bash
17:37 linuxbox $ PS1="<\u@\h \W>\$ "
<me@linuxbox ~>$
```

Intenta las otras secuencias listadas en la tabla anterior y ve si puedes crear un prompt brillante nuevo.

## Agregando Color

La mayoría de programas emuladores de terminal responden a ciertas secuencias de caracteres de no-impresión para controlar cosas como atributos de caracteres (como color, texto en negrita y el temido texto parpadeante) y posición del cursor.

### Antecedentes: Confusión de Terminales

En tiempos antiguos, cuando los terminales estaban conectados a computadoras remotas, había muchas marcas de terminales compitiendo, y todas funcionaban de manera diferente. Tenían teclados diferentes, y todas tenían diferentes formas de interpretar información de control. Los sistemas Unix y tipo Unix tienen dos subsistemas bastante complejos para lidiar con la confusión de control terminal (llamados **termcap** y **terminfo**).

En un esfuerzo por hacer que los terminales hablaran algún tipo de idioma común, el Instituto Nacional Estadounidense de Estándares (ANSI) desarrolló un conjunto estándar de secuencias de caracteres para controlar terminales de video. Los antiguos usuarios de DOS recordarán el archivo ANSI.SYS que se usaba para habilitar la interpretación de estos códigos.

### Códigos ANSI para Colores de Texto

El color del carácter está controlado por enviar al emulador de terminal un **código de escape ANSI** incrustado en la secuencia de caracteres a mostrar. El código de control no "se imprime" en la pantalla; en su lugar, es interpretado por el terminal como una instrucción.

Como vimos en la tabla anterior, las secuencias `\[` y `\]` se usan para encapsular caracteres de no-impresión. Un código de escape ANSI comienza con `\e` (octal 033, el código generado por ESC), seguido por un atributo de carácter opcional, seguido por una instrucción. Por ejemplo, el código para establecer el color de texto normal (atributo = 0), texto negro es el siguiente:

```
\e[0;30m
```

La siguiente tabla lista los colores de texto disponibles. Nota que los colores se dividen en dos grupos, diferenciados por la aplicación del atributo de carácter bold (1), que crea la apariencia de colores "light" (claro).

### Tabla de Colores de Texto

| Secuencia | Color | Secuencia | Color |
|-----------|-------|-----------|-------|
| **\e[0;30m** | Negro | **\e[1;30m** | Gris oscuro |
| **\e[0;31m** | Rojo | **\e[1;31m** | Rojo claro |
| **\e[0;32m** | Verde | **\e[1;32m** | Verde claro |
| **\e[0;33m** | Marrón | **\e[1;33m** | Amarillo |
| **\e[0;34m** | Azul | **\e[1;34m** | Azul claro |
| **\e[0;35m** | Púrpura | **\e[1;35m** | Púrpura claro |
| **\e[0;36m** | Cyan | **\e[1;36m** | Cyan claro |
| **\e[0;37m** | Gris claro | **\e[1;37m** | Blanco |

### Prompt Rojo

Intentemos hacer un prompt rojo. Insertaremos el código de escape al comienzo.

```bash
<me@linuxbox ~>$ PS1="\[\e[0;31m\]<\u@\h \W>\$ "
<me@linuxbox ~>$
```

Eso funciona, pero nota que todo el texto que escribimos después del prompt también se mostrará en rojo. Para solucionar esto, agregaremos otro código de escape al final del prompt que le diga al emulador de terminal que regrese al color anterior.

```bash
<me@linuxbox ~>$ PS1="\[\e[0;31m\]<\u@\h \W>\$\[\e[0m\] "
<me@linuxbox ~>$
```

¡Eso es mejor!

### Colores de Fondo

También es posible establecer el color de fondo del texto usando los códigos listados a continuación. Los colores de fondo no soportan el atributo bold.

| Secuencia | Color de fondo | Secuencia | Color de fondo |
|-----------|---|-----------|---|
| **\e[0;40m** | Negro | **\e[0;44m** | Azul |
| **\e[0;41m** | Rojo | **\e[0;45m** | Púrpura |
| **\e[0;42m** | Verde | **\e[0;46m** | Cyan |
| **\e[0;43m** | Marrón | **\e[0;47m** | Gris claro |

Podemos crear un prompt con un fondo rojo aplicando un cambio simple al primer código de escape:

```bash
<me@linuxbox ~>$ PS1="\[\e[0;41m\]<\u@\h \W>\$\[\e[0m\] "
<me@linuxbox ~>$
```

¡Intenta los códigos de color y ve qué puedes crear!

> **Nota**: Además de los atributos normal (0) y bold (1), el texto puede recibir atributos de subrayado (4), parpadeante (5) e inverso (7). En aras del buen gusto, muchos emuladores de terminal se rehúsan a honrar el atributo de parpadeo.

## Moviendo el Cursor

Los códigos de escape pueden ser usados para posicionar el cursor. Esto es comúnmente usado para proporcionar un reloj u otro tipo de información en una ubicación diferente en la pantalla, como en una esquina superior cada vez que se dibuja el prompt.

### Tabla de Movimiento del Cursor

| Código de escape | Acción |
|---|---|
| **\e[l;cH** | Mover el cursor a la línea l y columna c. |
| **\e[nA** | Mover el cursor hacia arriba n líneas. |
| **\e[nB** | Mover el cursor hacia abajo n líneas. |
| **\e[nC** | Mover el cursor hacia adelante n caracteres. |
| **\e[nD** | Mover el cursor hacia atrás n caracteres. |
| **\e[2J** | Limpiar la pantalla y mover el cursor a la esquina superior izquierda (línea 0, columna 0). |
| **\e[K** | Limpiar desde la posición del cursor hasta el final de la línea actual. |
| **\e[s** | Almacenar la posición actual del cursor. |
| **\e[u** | Recuperar la posición del cursor almacenada. |

### Ejemplo Avanzado: Barra de Reloj en la Parte Superior

Usando los códigos anteriores, construiremos un prompt que dibuje una barra roja en la parte superior de la pantalla que contenga un reloj (renderizado en texto amarillo) cada vez que se muestre el prompt. El código para el prompt es esta cadena de aspecto formidable:

```bash
PS1="\[\e[s\e[0;0H\e[0;41m\e[K\e[1;33m\t\e[0m\e[u\]<\u@\h \W>\$ "
```

La siguiente tabla describe qué hace cada parte de la cadena:

| Secuencia | Acción |
|-----------|--------|
| **\[** | Comienza una secuencia de caracteres de no-impresión. El propósito de esto es permitir que bash calcule correctamente el tamaño del prompt visible. Sin un cálculo preciso, las características de edición de línea de comando no pueden posicionar el cursor correctamente. |
| **\e[s** | Almacenar la posición del cursor. Esto es necesario para regresar a la ubicación del prompt después de que la barra y el reloj hayan sido dibujados en la parte superior de la pantalla. Ten en cuenta que algunos emuladores de terminal no reconocen este código. |
| **\e[0;0H** | Mover el cursor a la esquina superior izquierda, que es la línea 0, columna 0. |
| **\e[0;41m** | Establecer el color de fondo a rojo. |
| **\e[K** | Limpiar desde la ubicación actual del cursor (la esquina superior izquierda) hasta el final de la línea. Como el color de fondo ahora es rojo, la línea se limpia a ese color, creando nuestra barra. Ten en cuenta que limpiar hasta el final de la línea no cambia la posición del cursor, que permanece en la esquina superior izquierda. |
| **\e[1;33m** | Establecer el color de texto a amarillo. |
| **\t** | Mostrar la hora actual. Aunque este es un elemento "impreso", aún lo incluimos en la porción de no-impresión del prompt ya que no queremos que bash incluya el reloj al calcular el verdadero tamaño del prompt mostrado. |
| **\e[0m** | Apagar el color. Esto afecta tanto al texto como al fondo. |
| **\e[u** | Restaurar la posición del cursor guardada anteriormente. |
| **\]** | Terminar la secuencia de caracteres de no-impresión. |
| **<\u@\h \W>\$** | Cadena de prompt. |

## Guardando el Prompt

Obviamente, no queremos estar escribiendo ese monstruo todo el tiempo, así que querremos almacenar nuestro prompt en algún lugar. Podemos hacer que el prompt sea permanente agregándolo a nuestro archivo `.bashrc`. Para hacer esto, agrega estas dos líneas al archivo:

```bash
PS1="\[\e[s\e[0;0H\e[0;41m\e[K\e[1;33m\t\e[0m\e[u\]<\u@\h \W>\$ "
export PS1
```

## Resumen

Créalo o no, hay mucho más que se puede hacer con prompts involucrando funciones del shell y scripts que no hemos cubierto aquí, pero este es un buen comienzo. No todos se importarán lo suficiente para cambiar el prompt, ya que el prompt por defecto es usualmente satisfactorio. Pero para aquellos de nosotros que nos gusta modificar cosas, el shell proporciona la oportunidad para muchas horas de diversión casual.

---

## Ver También

- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment (Entorno) de Linux]]
- [[wiki/linux/12-vi-editor.md|Capítulo 12: Una Introducción Suave a VI(M)]]
