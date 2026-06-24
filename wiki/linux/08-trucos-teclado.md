---
title: "Capítulo 8: Trucos de Teclado en Linux"
Sources: raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-111-118.pdf
Raw: Capítulo 8 - ADVANCED KEYBOARD TRICKS (págs. 77-84)
Updated: 2026-06-22
---

# Trucos Avanzados del Teclado

## Filosofía

Unix se describe burlonamente como "el sistema operativo para gente que le gusta escribir". Sin embargo, incluso los usuarios de línea de comandos no quieren escribir **demasiado**. Por eso muchos comandos tienen nombres cortos: `cp`, `ls`, `mv`, `rm`.

Uno de los objetivos más apreciados de la línea de comandos es la **pereza**: hacer el máximo trabajo con el mínimo número de pulsaciones de teclas. Otro objetivo es no tener que levantar los dedos del teclado para alcanzar el ratón.

Este capítulo explora características de bash que hacen el uso del teclado más rápido y eficiente.

---

## Edición de Línea de Comandos (Command Line Editing)

Bash usa una biblioteca llamada **Readline** para implementar edición de línea de comandos. Ya hemos visto algo de esto: sabemos que las flechas de cursor mueven el cursor, pero hay muchas más características.

⚠️ **Nota:** Algunas secuencias de teclas (particularmente las que usan Alt) pueden ser interceptadas por la GUI o configuración de terminal. Todas las secuencias de teclas deberían funcionar correctamente en una consola virtual.

---

## Movimiento del Cursor (Cursor Movement)

Tabla de atajos para mover el cursor:

| Atajo | Acción |
|-------|--------|
| **Ctrl+A** | Mueve cursor al principio de la línea |
| **Ctrl+E** | Mueve cursor al final de la línea |
| **Ctrl+F** | Mueve cursor hacia adelante un carácter; igual que flecha derecha |
| **Ctrl+B** | Mueve cursor hacia atrás un carácter; igual que flecha izquierda |
| **Alt+F** | Mueve cursor adelante una palabra |
| **Alt+B** | Mueve cursor atrás una palabra |
| **Ctrl+L** | Limpia pantalla y mueve cursor a la esquina superior-izquierda |

**Casos de uso:**
- Ctrl+A: Ir rápidamente al principio para corregir un error al inicio de un comando largo
- Ctrl+E: Ir al final para agregar opciones
- Alt+F/Alt+B: Navegar palabra por palabra en comandos largos

---

## Modificación de Texto (Modifying Text)

Tabla de comandos para editar caracteres en la línea de comandos:

| Atajo | Acción |
|-------|--------|
| **Ctrl+D** | Elimina el carácter en la posición del cursor |
| **Ctrl+T** | Transpone (intercambia) el carácter en la posición del cursor con el anterior |
| **Alt+T** | Transpone la palabra en la posición del cursor con la anterior |
| **Alt+L** | Convierte caracteres desde la posición del cursor al final de la palabra a minúsculas |
| **Alt+U** | Convierte caracteres desde la posición del cursor al final de la palabra a mayúsculas |

**Ejemplos prácticos:**
```bash
# Transponer caracteres (invertir dos letras)
echo hellop
# Posicionar cursor después de la o
Ctrl+T  # Resultado: hello p

# Convertir palabra a mayúsculas
mycommand
# Posicionar al inicio de 'command'
Alt+U   # Resultado: myCOMMAND
```

---

## Cortar y Pegar (Cutting and Pasting / Kill and Yank)

La documentación de Readline usa los términos **killing** y **yanking** para referirse a lo que comúnmente llamamos cortar y pegar.

Los elementos que se cortan se almacenan en un **buffer** (área temporal en memoria) llamado **kill-ring**.

| Atajo | Acción |
|-------|--------|
| **Ctrl+K** | Elimina (kill) texto desde la posición del cursor hasta el final de la línea |
| **Ctrl+U** | Elimina (kill) texto desde la posición del cursor hasta el principio de la línea |
| **Alt+D** | Elimina (kill) texto desde la posición del cursor hasta el final de la palabra actual |
| **Alt+Backspace** | Elimina (kill) texto desde la posición del cursor hasta el principio de la palabra actual. Si el cursor está en el inicio de una palabra, elimina la palabra anterior |
| **Ctrl+Y** | Pega (yank) texto del kill-ring en la posición del cursor |

**Ejemplo práctico: Corregir un comando largo**
```bash
ls -l /usr/bin | sort | uniq | grep zip
# Cambiar de opinión en mitad del comando
Ctrl+K  # Elimina todo desde aquí hacia el final
# Luego escribir algo diferente
```

---

## La Tecla Meta (THE META KEY)

⚠️ **Nota histórica:** En la documentación de Readline encontrarás el término **meta key**. 

En teclados modernos, esto se mapea a Alt. En máquinas antiguas (pre-PC, décadas antes de Linux), no todos los teclados tenían un Alt dedicado. Las máquinas tenían una tecla especial llamada **meta** o un terminal físico con funcionalidad especial.

Presionar Escape se consideraba equivalente a mantener presionada la tecla meta. En el contexto de Linux, simplemente presiona Alt.

---

## Completado de Comandos (Command Completion)

Una forma en que el shell ayuda es a través del **completion** (autocompletado), que ocurre al presionar TAB mientras se escribe un comando.

### Ejemplo Básico

Supongamos que tenemos este directorio:
```
Desktop Documents ls-output.txt Music Pictures Public Templates Videos
```

Escribimos (pero NO presionamos ENTER):
```bash
ls l
```

Presionamos TAB:
```bash
ls ls-output.txt
```

El shell completó la línea automáticamente.

### Completado Ambiguo

Si presionamos TAB con una entrada ambigua:
```bash
ls D
```

Presionamos TAB - nada ocurre. Esto es porque D coincide con múltiples entradas. Presionar TAB de nuevo:

```bash
ls D
```

Muestra todos los completados posibles. Si luego hacemos más específico:
```bash
ls Do
```

Presionamos TAB:
```bash
ls Documents
```

Funciona el completado.

### Comandos de Completado

| Atajo | Acción |
|-------|--------|
| **Alt+?** | Muestra una lista de completados posibles. En muchos sistemas, presionar TAB una segunda vez también funciona (más fácil). |
| **Alt+*** | Inserta todos los completados posibles. Útil cuando quieres usar más de un completado |

### Tipos de Completado

El completado funciona en:
- **Nombres de ruta** — El uso más común
- **Variables** — Si la palabra comienza con `$`
- **Nombres de usuario** — Si la palabra comienza con `~`
- **Nombres de comando** — Si la palabra es el primer palabra en la línea
- **Nombres de host** — Si la palabra comienza con `@`

Completado de hostnames solo funciona para hostnames listados en `/etc/hosts`.

### Completado Programable (Programmable Completion)

Versiones recientes de bash tienen una característica llamada **programmable completion**. Permite agregar reglas de completado para aplicaciones específicas.

Por ejemplo, es posible agregar completados para la lista de opciones de un comando o para tipos específicos de archivos que una aplicación soporta. Ubuntu tiene un conjunto bastante grande definido por defecto.

Si eres curioso, puedes explorar:
```bash
set | less
```

El tema es bastante arcano y está documentado en la sección "READLINE" de la página man de bash. Muchas distribuciones incluyen completados por defecto.

---

## Uso del Historial (Using History)

Bash mantiene un historial de comandos que ya hemos ingresado. Este historial se guarda en un archivo en el directorio home llamado `.bash_history`.

El historial es un recurso útil para reducir la cantidad de escritura necesaria, especialmente cuando se combina con edición de línea de comandos.

### Búsqueda en el Historial (Searching History)

Ver el contenido del historial:
```bash
history | less
```

Por defecto, bash se configura para almacenar los últimos 1000 comandos.

Si queremos encontrar comandos relacionados con `/usr/bin`, podemos hacer:
```bash
history | grep /usr/bin
```

Salida:
```
88 ls -l /usr/bin > ls-output.txt
```

### Expansión del Historial (History Expansion)

El número 88 es la posición en la lista de historial. Podemos usar esto inmediatamente con **history expansion**.

Para usar el comando en la línea 88:
```bash
!88
```

Bash expandirá `!88` al contenido del comando número 88:
```bash
ls -l /usr/bin > ls-output.txt
```

Hay otras formas de expansión de historial, descritas en la Tabla 8-6.

### Comandos de Expansión del Historial

| Secuencia | Acción |
|-----------|--------|
| **!!** | Repite el último comando. Probablemente es más fácil presionar la flecha arriba y ENTER |
| **!number** | Repite el comando número N en el historial |
| **!string** | Repite el último comando en el historial que comienza con el string indicado |
| **!?string** | Repite el último comando en el historial que contiene el string indicado |

**Ejemplo:**
```bash
# Ver que tenemos esto en el historial
history | grep /usr/bin
88 ls -l /usr/bin > ls-output.txt

# Ejecutar ese comando
!88

# O buscar por inicio
!ls

# O buscar por contenido
!?/usr/bin
```

⚠️ **Cuidado:** Usa los formularios `!string` e `!?string` solo si estás absolutamente seguro del contenido del historial. Podría haber sorpresas.

---

## Navegación del Historial (History Commands)

Tabla de atajos para manipular la lista de historial:

| Atajo | Acción |
|-------|--------|
| **Ctrl+P** | Mueve a la entrada anterior del historial. Igual que flecha arriba |
| **Ctrl+N** | Mueve a la siguiente entrada del historial. Igual que flecha abajo |
| **Alt+<** | Mueve al principio (top) de la lista de historial |
| **Alt+>** | Mueve al final (bottom) de la lista de historial, es decir, la línea de comando actual |
| **Ctrl+R** | Búsqueda incremental inversa. Busca incrementalmente desde la línea de comando actual hacia arriba en la lista de historial |
| **Alt+P** | Búsqueda inversa no incremental. Con esta tecla, escribe en la cadena de búsqueda y presiona ENTER antes de que se realice la búsqueda |
| **Alt+N** | Búsqueda adelante no incremental |
| **Ctrl+O** | Ejecuta el elemento actual en la lista de historial y avanza al siguiente. Útil si intentas re-ejecutar una secuencia de comandos en el historial |

### Búsqueda Incremental Inversa (Reverse Incremental Search)

Este es un feature particularmente potente. Veamos cómo funciona:

1. Presiona **Ctrl+R**:
   ```
   (reverse-i-search)'':
   ```

2. El prompt cambia indicando que realizamos búsqueda incremental inversa. Es "inversa" porque buscamos desde "ahora" a algún tiempo en el pasado. Luego comenzamos a escribir nuestro texto de búsqueda.

3. Por ejemplo, si buscamos `ls`:
   ```
   (reverse-i-search)'/usr/bin': ls -l /usr/bin > ls-output.txt
   ```

4. Inmediatamente, la búsqueda retorna nuestro resultado. Con nuestro resultado, podemos:
   - **Presionar ENTER** para ejecutar el comando
   - **Presionar Ctrl+J** para copiar la línea del historial a la línea de comando actual para edición adicional
   - **Presionar Ctrl+G** o **Ctrl+C** para salir de la búsqueda

5. Para encontrar la siguiente ocurrencia del texto (moviendo "up" en la lista de historial), presiona **Ctrl+R** de nuevo.

### Ejemplo Práctico

```bash
# Presiona Ctrl+R
(reverse-i-search)'':

# Escribe "usr/bin"
(reverse-i-search)'/usr/bin': ls -l /usr/bin > ls-output.txt

# Presiona Ctrl+J para copiar a la línea actual
$ ls -l /usr/bin > ls-output.txt

# Tu shell retorna, y tu línea de comando está cargada y lista para actuar
```

---

## Grabación de Sesiones (Script Command)

Además del historial de comandos en bash, la mayoría de distribuciones Linux incluyen un programa llamado `script` que se puede usar para grabar una sesión completa de shell y guardarla en un archivo.

**Sintaxis:**
```bash
script [archivo]
```

Donde `archivo` es el nombre del archivo usado para almacenar la grabación.

Si no se especifica un archivo, se usa `typescript` por defecto.

Consulta la página `man script` para una lista completa de opciones y características del programa.

---

## Resumen de Atajos de Teclado

### Movimiento
| Atajo | Función |
|-------|---------|
| Ctrl+A | Principio de línea |
| Ctrl+E | Final de línea |
| Alt+F | Adelante una palabra |
| Alt+B | Atrás una palabra |

### Edición
| Atajo | Función |
|-------|---------|
| Ctrl+D | Eliminar carácter |
| Ctrl+T | Transponer caracteres |
| Alt+T | Transponer palabras |
| Alt+L | Minúsculas |
| Alt+U | Mayúsculas |

### Cortar y Pegar
| Atajo | Función |
|-------|---------|
| Ctrl+K | Eliminar hasta final de línea |
| Ctrl+U | Eliminar hasta principio de línea |
| Alt+D | Eliminar palabra |
| Ctrl+Y | Pegar |

### Completado
| Atajo | Función |
|-------|---------|
| TAB | Completar comando/archivo |
| Alt+? | Mostrar completados posibles |
| Alt+* | Insertar todos los completados |

### Historial
| Atajo | Función |
|-------|---------|
| Ctrl+P / ↑ | Comando anterior |
| Ctrl+N / ↓ | Comando siguiente |
| Ctrl+R | Búsqueda inversa |
| Alt+< | Inicio de historial |
| Alt+> | Final de historial |
| Alt+P | Búsqueda inversa no incremental |
| !! | Repite último comando |
| !N | Repite comando N |
| !string | Repite comando que comienza con string |

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]] — Primer contacto con la línea de comandos
- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment (Entorno) de Linux]] — Personalización del environment con .bashrc
- [[wiki/linux/12-vi-editor.md|Capítulo 12: Una Introducción Suave a VI(M)]] — Editor modal con bindings de Readline habilitables

---

## Filosofía Final

A medida que pases más tiempo con la línea de comandos, irás adquiriendo más de estos trucos. Para ahora, considéralos opcionales y potencialmente útiles. A medida que te involucres más con la línea de comandos, aprenderás más de estos trucos. Por ahora, considéralos opcionales y potencialmente útiles.

La verdadera belleza de estos atajos es que reducen el trabajo que debes hacer, permitiendo que pases más tiempo pensando en lo que intentas lograr en lugar de pensar en cómo escribirlo.

---

*Fuente: The Linux Command Line - A Complete Introduction (William E. Shotts, Jr.)*
