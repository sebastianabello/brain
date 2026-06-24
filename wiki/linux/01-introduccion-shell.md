---
title: "Capítulo 1: Introducción al Shell de Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 1"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-37-41.pdf"
updated: "2026-06-22"
---

# Introducción al Shell de Linux

## ¿Qué es el Shell?

El **shell** es un programa que toma comandos del teclado y los pasa al sistema operativo para que los ejecute. Cuando hablamos de "línea de comandos", en realidad nos referimos al shell.

La mayoría de distribuciones Linux proporcionan un programa llamado **bash** (bourne again shell), que es una versión mejorada del shell original `sh` (Bourne shell) de Unix. El nombre "bash" es un acrónimo que hace referencia a que es un reemplazo mejorado del shell original escrito por Steve Bourne.

## Emuladores de Terminal

Para interactuar con el shell en un entorno de interfaz gráfica (GUI), necesitamos un programa llamado **terminal emulator** (emulador de terminal).

Diferentes entornos de escritorio Linux ofrecen distintos emuladores:
- **KDE**: usa `konsole`
- **GNOME**: usa `gnome-terminal` (típicamente llamado "Terminal")
- Y otros más disponibles

Todos estos emuladores cumplen la misma función: darnos acceso al shell. La preferencia por uno u otro generalmente depende de características de personalización.

## Primeros Pasos: El Prompt del Shell

Cuando lanzas un emulador de terminal, verás algo como:

```
[me@linuxbox ~]$
```

Esto se llama **shell prompt** (indicador del shell). Aparece cuando el shell está listo para aceptar entrada. Su formato puede variar según la distribución, pero típicamente incluye:
- **username@machinename** (tu usuario@nombre de la máquina)
- **Current working directory** (directorio actual, el `~` representa el home)
- **A symbol** (`$` para usuarios normales, `#` para root/superusuario)

> **Nota sobre permisos**: Si el último carácter del prompt es `#` en lugar de `$`, la sesión tiene privilegios de superusuario (administrador). Esto significa que o estás logged como root o has seleccionado un emulador de terminal que proporciona esos permisos.

### Primeros Comandos

Para comenzar a practicar, intenta escribir algunos comandos simples. Por ejemplo:

```bash
[me@linuxbox ~]$ kaekfjaelf]
```

Si escribes algo que no es un comando válido:

```
bash: kaekfjaelf]: command not found
[me@linuxbox ~]$
```

El shell te indicará que el comando no existe y te dará otra oportunidad.

## Historial de Comandos

Una característica muy útil del shell es el **command history** (historial de comandos). 

- **Presiona la flecha arriba ↑**: Verás que aparece el comando anterior. La mayoría de distribuciones Linux recuerdan por defecto los últimos 1,000 comandos.
- **Presiona la flecha abajo ↓**: El comando anterior desaparece y vuelves a ver una línea en blanco.

Esto permite recuperar y reutilizar comandos que ya has ejecutado.

## Movimiento del Cursor

Puedes navegar por la línea de comando actual usando las flechas:

- **Flecha izquierda ←** y **flecha derecha →**: Posicionan el cursor en cualquier lugar de la línea de comandos.
- Esto facilita la edición de comandos sin tener que reescribirlos completamente.

## Ratón y Focus en la Terminal

### Mecanismo de Copia y Pegue

Aunque el shell se basa en el teclado, el emulador de terminal permite usar el ratón:
- **Drag and select con el botón izquierdo**: Copia el texto a un buffer especial
- **Botón central del ratón**: Pegua el texto en la ubicación actual del cursor
- Funciona en la mayoría de entornos de escritorio

### Nota sobre Ctrl+C y Ctrl+V

⚠️ **No funcionan en terminal** - Estos atajos de teclado tienen significados diferentes en el shell que fueron asignados hace muchos años, antes del lanzamiento de Windows. En su lugar:
- Usa **Shift+Ctrl+C** y **Shift+Ctrl+V** para copiar/pegar en la terminal

### Focus Policy

- **Entornos como KDE o GNOME** generalmente implementan "focus follows mouse": una ventana obtiene el focus simplemente pasando el ratón sobre ella
- Algunos entornos permiten cambiar la política a "click to focus"
- Esto es útil para la técnica de copia y pegue de la terminal

## Comandos Simples Básicos

### `date` - Fecha y Hora

Muestra la fecha y hora actual:

```bash
[me@linuxbox ~]$ date
Thu Mar  8 15:09:41 EST 2025
```

### `uptime` - Tiempo de Actividad del Sistema

Muestra cuánto tiempo lleva el sistema ejecutándose y la carga promedio:

```bash
[me@linuxbox ~]$ uptime
15:12:22 up 3 days, 23:40,  7 users,  load average: 0.37, 0.37, 0.64
```

Interpreta como:
- **15:12:22**: Hora actual
- **up 3 days, 23:40**: Sistema en ejecución durante 3 días y 23 horas 40 minutos
- **7 users**: 7 usuarios conectados
- **load average**: Carga promedio en los últimos 1, 5 y 15 minutos (0.37, 0.37, 0.64)

### `df` - Espacio en Disco

Muestra la cantidad de espacio libre en los discos:

```bash
[me@linuxbox ~]$ df
Filesystem      1K-blocks     Used Available Use% Mounted on
/dev/sda2       15115452  5012392  9949716  34% /
/dev/sda5       59631908 26545424 30008832  47% /home
/dev/sda1        147764    17370   122756  13% /boot
tmpfs             256856        0   256856   0% /dev/shm
```

Columnas principales:
- **Filesystem**: Nombre del dispositivo
- **1K-blocks**: Tamaño total en bloques de 1KB
- **Used**: Espacio usado
- **Available**: Espacio disponible
- **Use%**: Porcentaje de uso
- **Mounted on**: Punto de montaje

### `free` - Memoria Libre

Muestra la cantidad de memoria libre en el sistema:

```bash
[me@linuxbox ~]$ free
              total        used        free      shared    buffers    cached
Mem:          533712      503976       9736           0       5312     122916
-/+ buffers/cache:      375748      137964
Swap:         1052248     104712      947536
```

Información:
- **total**: Memoria RAM total disponible
- **used**: Memoria en uso
- **free**: Memoria libre
- **buffers/cache**: Memoria usada para cachés (puede ser liberada si es necesaria)
- **Swap**: Memoria virtual en disco

## Consolas Virtuales

Incluso sin un emulador de terminal ejecutándose, varias sesiones de terminal pueden estar activas detrás del escritorio gráfico, llamadas **virtual terminals** o **virtual consoles**.

- **Acceder a consolas virtuales**: Presionar **Ctrl+Alt+F1** a **Ctrl+Alt+F6** en la mayoría de distribuciones Linux
- **Cambiar entre sesiones**: Usa **Alt+F1** a **Alt+F6** para cambiar entre ellas
- **Volver al escritorio gráfico**: En la mayoría de sistemas, presiona **Alt+F7**

En cada consola virtual puedes ver un prompt de login donde puedes entrar tu usuario y contraseña para acceder a otra sesión de terminal.

## Cerrar una Sesión de Terminal

Hay tres formas de terminar una sesión de terminal:

1. **Cerrar la ventana del emulador de terminal** (hacer clic en la X)
2. **Escribir `exit`** en el prompt:
   ```bash
   [me@linuxbox ~]$ exit
   ```
3. **Presionar Ctrl+D**: Señal de fin de archivo (EOF) que termina la sesión

## Resumen

Este primer capítulo introdujo los conceptos fundamentales para trabajar con Linux desde la línea de comandos:

- La naturaleza del shell y bash como el shell estándar de Linux
- Cómo acceder al shell mediante emuladores de terminal
- Cómo escribir comandos y trabajar con el historial
- Primeros comandos útiles para explorar el sistema (`date`, `uptime`, `df`, `free`)
- Edición básica de comandos y navegación
- Acceso a consolas virtuales para múltiples sesiones

En el siguiente capítulo se explorarán más comandos y cómo navegar el sistema de archivos de Linux.

---

## Ver También

- [[wiki/linux/05-trabajo-con-comandos.md|Capítulo 5: Trabajo con Comandos]] — Profundizar en tipos de comandos, búsqueda y ayuda
- [[wiki/linux/08-trucos-teclado.md|Capítulo 8: Trucos Avanzados del Teclado]] — Atajos de Readline para edición eficiente
