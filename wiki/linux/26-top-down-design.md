---
title: "Capítulo 26: Diseño de Arriba Hacia Abajo (Top-Down Design)"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 26"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-385-394.pdf"
updated: "2026-06-23"
---

# Diseño de Arriba Hacia Abajo (Top-Down Design)

## Introducción al Problema

A medida que los programas se hacen más grandes y complejos, se vuelven más difíciles de diseñar, codificar y mantener. Como en cualquier proyecto grande, a menudo es una buena idea **romper tareas grandes y complejas en una serie de tareas pequeñas y simples**.

### Ejemplo: Ir al Mercado

Imaginemos que estamos tratando de describir una tarea común y cotidiana, ir al mercado a comprar comida, a una persona de Marte. Podríamos describir el proceso general como la siguiente serie de pasos:

1. Subirse al auto
2. Conducir al mercado
3. Estacionar el auto
4. Entrar al mercado
5. Comprar comida
6. Volver al auto
7. Conducir a casa
8. Estacionar el auto
9. Entrar a la casa

Sin embargo, una persona de Marte probablemente necesitaría más detalle. Podríamos desglosar aún más la subtarea "Estacionar el auto" en esta serie de pasos:

1. Encontrar espacio de estacionamiento
2. Conducir el auto al espacio
3. Apagar el motor
4. Poner el freno de estacionamiento
5. Salir del auto
6. Bloquear el auto

La subtarea "Apagar el motor" podría desglosarse aún más en pasos como "Apagar el encendido", "Sacar la llave del encendido", y así sucesivamente, hasta que cada paso del proceso completo de ir al mercado haya sido completamente definido.

### Definición de Top-Down Design

Este proceso de **identificar los pasos de nivel superior y desarrollar vistas cada vez más detalladas de esos pasos** se llama **top-down design** (diseño de arriba hacia abajo).

Esta técnica nos permite **romper tareas grandes y complejas en muchas tareas pequeñas y simples**. El diseño de arriba hacia abajo es un método común de diseño de programas y es uno que está particularmente bien adaptado a la programación del shell.

En este capítulo, usaremos el diseño de arriba hacia abajo para desarrollar aún más nuestro script generador de reportes.

---

## Shell Functions (Funciones de Shell)

### Estructura Actual del Programa

Nuestro script actualmente realiza los siguientes pasos para generar el documento HTML:

1. Abrir página
2. Abrir encabezado de página
3. Establecer título de página
4. Cerrar encabezado de página
5. Abrir cuerpo de página
6. Outputear encabezado de página
7. Outputear timestamp
8. Cerrar cuerpo de página
9. Cerrar página

### Tareas a Agregar

Para la siguiente etapa de desarrollo, agregaremos algunas tareas entre los pasos 7 y 8:

**System uptime and load** — Cantidad de tiempo desde el último apagado o reinicio y el promedio de tareas actualmente ejecutándose en el procesador en varios intervalos de tiempo.

**Disk space** — Uso general de espacio en los dispositivos de almacenamiento del sistema.

**Home space** — Cantidad de espacio de almacenamiento utilizado por cada usuario.

### Visión del Script con Comandos

Si tuviéramos un comando para cada una de estas tareas, podríamos agregarlas a nuestro script simplemente a través de sustitución de comandos:

```bash
#!/bin/bash
# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

### Dos Enfoques Posibles

Podríamos crear estos comandos adicionales de dos formas:

1. **Escribir tres scripts separados** y colocarlos en un directorio listado en nuestro PATH
2. **Embeber los scripts dentro de nuestro programa como shell functions** (funciones de shell)

Las **shell functions** son "mini-scripts" que se encuentran dentro de otros scripts y pueden actuar como programas autónomos.

---

## Definición de Shell Functions

### Sintaxis de Functions

Las shell functions tienen dos formas sintácticas comunes.

**Forma más formal:**

```bash
function name {
  commands
  return
}
```

**Forma más simple (generalmente preferida):**

```bash
name () {
  commands
  return
}
```

En este ejemplo, `name` es el nombre de la función, y `commands` es una serie de comandos contenidos dentro de la función. Ambas formas son equivalentes y pueden usarse indistintamente.

### Ejemplo Básico de Function

El siguiente script demuestra el uso de una shell function:

```bash
#!/bin/bash

# Shell function demo

function step2 {
  echo "Step 2"
  return
}

# Main program starts here

echo "Step 1"
step2
echo "Step 3"
```

**Flujo de ejecución:**

- Cuando el shell lee el script, pasa sobre las líneas iniciales porque consisten en comentarios y la definición de función
- La ejecución comienza en `echo "Step 1"` (primer comando principal)
- La línea `step2` **llama a la shell function**, y el shell ejecuta la función como lo haría con cualquier otro comando
- El control del programa se mueve a la línea `echo "Step 2"` dentro de la función
- `return` termina la función y devuelve el control al programa en la línea siguiente a la llamada de función
- Se ejecuta el último `echo "Step 3"`

### Reglas Importantes

- Los **nombres de shell functions siguen las mismas reglas que las variables**
- Una función **debe contener al menos un comando**
- El comando `return` (que es opcional) satisface este requisito
- **Las definiciones de función deben aparecer en el script antes de que se llamen**, de lo contrario se interpretarán como nombres de programas externos

### Agregando Functions al Script

Aquí agregamos definiciones mínimas de shell functions a nuestro script:

```bash
#!/bin/bash
# Program to output a system information page

TITLE="System Information Report For $HOSTNAME"
CURRENT_TIME="$(date +"%x %r %Z")"
TIMESTAMP="Generated $CURRENT_TIME, by $USER"

report_uptime () {
  return
}

report_disk_space () {
  return
}

report_home_space () {
  return
}

cat << _EOF_
<html>
  <head>
    <title>$TITLE</title>
  </head>
  <body>
    <h1>$TITLE</h1>
    <p>$TIMESTAMP</p>
    $(report_uptime)
    $(report_disk_space)
    $(report_home_space)
  </body>
</html>
_EOF_
```

---

## Local Variables (Variables Locales)

### Problema de Scope

En los scripts que hemos escrito hasta ahora, todas las variables (incluyendo constantes) han sido **global variables** (variables globales). Las variables globales **mantienen su existencia durante todo el programa**.

Esto está bien para muchas cosas, pero a veces puede complicar el uso de shell functions. Dentro de shell functions, a menudo es deseable tener **local variables** (variables locales).

### Definición de Local Variables

Las **variables locales** son accesibles solo dentro de la shell function en la que se definen y en cualquier otra función que la shell function pueda llamar. **Dejan de existir una vez que la shell function termina**.

Tener variables locales permite al programador usar variables con nombres que quizás ya existan, ya sea globalmente en el script o en otras shell functions, **sin tener que preocuparse por posibles conflictos de nombres**.

### Ejemplo de Variables Locales

Aquí hay un script de ejemplo que demuestra cómo se definen y usan las variables locales:

```bash
#!/bin/bash
# local-vars: script to demonstrate local variables

foo=0  # global variable foo

funct_1 () {
  local foo  # variable local to funct_1
  foo=1
  echo "funct_1: foo = $foo"
}

funct_2 () {
  local foo  # variable local to funct_2
  foo=2
  echo "funct_2: foo = $foo"
}

echo "global: foo = $foo"
funct_1
echo "global: foo = $foo"
funct_2
echo "global: foo = $foo"
```

**Salida:**

```
global: foo = 0
funct_1: foo = 1
global: foo = 0
funct_2: foo = 2
global: foo = 0
```

### Mecánica de Variables Locales

Las variables locales se definen precediéndolas con la palabra **`local`**. Esto crea una variable que es local a la shell function en la que se define. Una vez afuera de la shell function, la variable ya no existe.

Vemos que la asignación de valores a la variable local `foo` dentro de ambas shell functions **no tiene efecto en el valor de `foo` definido afuera de las funciones**.

### Ventajas

Esta característica permite que las shell functions se escriban de modo que **permanezcan independientes unas de otras y del script en el que aparecen**. Esto es valioso porque:

- Ayuda a prevenir que una parte de un programa interfiera con otra
- Permite que las shell functions se escriban de manera **portable**
- Las funciones pueden ser **cortadas y pegadas de script a script** según sea necesario (reutilización)

---

## Shell Functions y Redirección

### Group Commands en Functions

Si miramos más de cerca cómo se escriben las shell functions, podemos notar algo:

```bash
my_funct () {
  command1
  command2
  command3
}
```

Los tres comandos dentro de las llaves forman un **group command** (comando de grupo). Como recordamos del Capítulo 6, los comandos de grupo combinan múltiples comandos en una sola entidad cuando se trata de redirección.

Con comandos de grupo podemos hacer:

```bash
{ command1; command2; command3; } > some_output.txt
```

Y también:

```bash
{ command1; command2; command3; } < some_input.txt
```

### Redirección en Functions

Lo mismo es cierto para las shell functions. Consideremos el siguiente código:

```bash
my_funct () {
  echo "My Documents"
  ls ~/Documents
  echo "My Music"
  ls ~/Music
  echo "My Videos"
  ls ~/Videos
  return
}
```

Es fácil ver qué hace esta función, pero ¿dónde va su salida? Va a donde sea que la dirrijamos.

**Cuando llamamos a esta función, envía su salida combinada a standard output**, y si queremos, podemos dirigirla a un archivo:

```bash
my_funct > my_directories.txt
```

O podemos dirigirla a un pipeline:

```bash
my_funct | sort
```

Podemos incluso almacenar la salida en una variable usando sustitución de comandos:

```bash
my_var="$(my_funct)"
```

**La redirección también se aplica a standard input**. Si la función contiene un comando que acepta standard input (por ejemplo, `cat` sin argumentos), podemos hacer fácilmente:

```bash
my_funct < input.txt
```

---

## Mantener Scripts en Funcionamiento (Keep Scripts Running)

### Desarrollo Incremental

Mientras desarrollamos nuestro programa, es útil **mantener el programa en un estado ejecutable**. Al hacer esto y **testeando frecuentemente**, podemos detectar errores temprano en el proceso de desarrollo. Esto hará que la depuración de problemas sea mucho más fácil.

Por ejemplo, si ejecutamos el programa, hacemos un pequeño cambio, y luego ejecutamos el programa nuevamente y encontramos un problema, **es probable que el cambio más reciente sea la fuente del problema**.

### Usando Stubs para Testing

Al agregar las funciones vacías, llamadas **stubs** (en jerga de programador), podemos verificar el flujo lógico de nuestro programa en una etapa temprana.

Cuando se construye un stub, es una buena idea incluir algo que proporcione **feedback al programador**, que muestre que el flujo lógico se está llevando a cabo.

#### Ejemplo sin Feedback

Si miramos la salida de nuestro script ahora sin feedback:

```
<html>
  <head>
    <title>System Information Report For linuxbox</title>
  </head>
  <body>
    <h1>System Information Report For linuxbox</h1>
    <p>Generated 03/19/2009 04:02:10 PM EDT, by me</p>
  </body>
</html>
```

Vemos que hay algunas líneas en blanco en nuestro output después del timestamp, pero no podemos estar seguros de la causa.

#### Con Feedback

Si cambiamos las funciones para incluir feedback:

```bash
report_uptime () {
  echo "Function report_uptime executed."
  return
}

report_disk_space () {
  echo "Function report_disk_space executed."
  return
}

report_home_space () {
  echo "Function report_home_space executed."
  return
}
```

Y ejecutamos el script nuevamente:

```
<html>
  <head>
    <title>System Information Report For linuxbox</title>
  </head>
  <body>
    <h1>System Information Report For linuxbox</h1>
    <p>Generated 03/20/2025 05:17:26 AM EDT, by me</p>
    Function report_uptime executed.
    Function report_disk_space executed.
    Function report_home_space executed.
  </body>
</html>
```

Ahora vemos que, de hecho, nuestras tres funciones se están ejecutando.

---

## Implementación de Functions Funcionales

### report_uptime

Ahora es hora de rellenar el código de la función. Primero, la función `report_uptime`:

```bash
report_uptime () {
  cat << _EOF_
  <h2>System Uptime</h2>
  <pre>$(uptime)</pre>
_EOF_
  return
}
```

Es bastante sencillo. Usamos un here document para outputear un encabezado de sección y la salida del comando `uptime`, rodeada de etiquetas `<pre>` para preservar el formato del comando.

### report_disk_space

La función `report_disk_space` es similar:

```bash
report_disk_space () {
  cat << _EOF_
  <h2>Disk Space Utilization</h2>
  <pre>$(df -h)</pre>
_EOF_
  return
}
```

Esta función usa el comando `df -h` para determinar la cantidad de espacio en disco.

### report_home_space

Finalmente, construimos la función `report_home_space`:

```bash
report_home_space () {
  cat << _EOF_
  <h2>Home Space Utilization</h2>
  <pre>$(du -sh /home/*)</pre>
_EOF_
  return
}
```

Usamos el comando `du` con las opciones `-sh` para realizar esta tarea.

#### Nota sobre Permisos

Sin embargo, esto **no es una solución completa al problema**. Si bien funcionará en algunos sistemas (Ubuntu, por ejemplo), **no funcionará en otros**.

La razón es que muchos sistemas establecen los **permisos de los directorios de inicio para prevenir que sean legibles por el mundo**, que es una medida de seguridad razonable.

En estos sistemas, la función `report_home_space` tal como está escrita, solo funcionará si nuestro script se ejecuta con **privilegios de superusuario**. Una solución mejor sería que el script ajuste su comportamiento según los privilegios del usuario. Abordaremos este problema en el siguiente capítulo.

---

## Shell Functions en tu Archivo .bashrc

Las shell functions hacen excelentes **reemplazos para alias** y son en realidad el **método preferido para crear pequeños comandos para uso personal**.

Los alias son limitados en el tipo de comandos y características de shell que soportan, mientras que las shell functions permiten cualquier cosa que pueda ser escrita en script.

Por ejemplo, si nos gustara la shell function `report_disk_space` que desarrollamos para nuestro script, podríamos crear una función similar llamada `ds` para nuestro archivo `.bashrc`:

```bash
ds () {
  echo "Disk Space Utilization For $HOSTNAME"
  df -h
}
```

Ahora podemos simplemente escribir `ds` en la línea de comandos para obtener un reporte de espacio en disco rápidamente.

---

## Resumen

En este capítulo, introdujimos un método común de diseño de programas llamado **top-down design**, y vimos cómo las **shell functions** se utilizan para construir el refinamiento gradual que requiere.

Puntos clave:

- **Top-down design** rompe problemas complejos en tareas más pequeñas
- **Shell functions** son "mini-scripts" reutilizables dentro de programas mayores
- **Local variables** hacen funciones independientes y portables
- **Group commands** (las llaves en funciones) permiten redirección combinada
- **Stubs** con feedback ayudan a mantener programas en estado ejecutable durante el desarrollo
- Las funciones son mejores que alias para comandos personales en `.bashrc`

También vimos cómo las **variables locales** pueden usarse para hacer que las shell functions sean independientes entre sí y del programa en el que se colocan. Esto hace posible que las shell functions se escriban de manera **portable y reutilizable**, lo que es un gran ahorro de tiempo.

---

## Ver También

- [[wiki/linux/25-iniciando-un-proyecto.md|Capítulo 25: Iniciando un Proyecto]] — Variables, constantes y here documents
- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] — Group commands y redirección
- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment]] — Archivo .bashrc y configuración del shell
