---
title: "Capítulo 10: Procesos en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 10"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-137-151.pdf"
updated: "2026-06-22"
---

# Procesos en Linux

## ¿Qué es un Proceso?

Los sistemas operativos modernos son **multitarea**, lo que significa que crean la ilusión de ejecutar múltiples programas simultáneamente. El kernel de Linux gestiona esto mediante **procesos**: la manera en que Linux organiza los diferentes programas esperando su turno en la CPU.

Cuando la máquina se vuelve lenta o una aplicación no responde, es necesario examinar qué está pasando. Este capítulo cubre herramientas para:
- Examinar qué programas están en ejecución
- Monitorear el rendimiento del sistema
- Terminar procesos problemáticos

## Comandos de Gestión de Procesos

Los comandos a cubrir en este capítulo son:

| Comando | Descripción |
|---------|-------------|
| `ps` | Reportar una instantánea de procesos actuales |
| `top` | Mostrar tareas (actualización dinámica) |
| `jobs` | Listar trabajos activos |
| `bg` | Poner un trabajo en segundo plano |
| `fg` | Poner un trabajo en primer plano |
| `kill` | Enviar una señal a un proceso |
| `killall` | Matar procesos por nombre |
| `nice` | Ejecutar programa con prioridad de planificación modificada |
| `renice` | Cambiar prioridad de procesos en ejecución |
| `nohup` | Ejecutar comando inmune a cierres de terminal |
| `halt/poweroff/reboot` | Detener, apagar o reiniciar el sistema |
| `shutdown` | Apagar o reiniciar el sistema |

## Cómo Funciona un Proceso

Cuando el sistema inicia, el kernel lanza sus propias actividades como procesos e inicia un programa llamado **init**. Init, a su vez, inicia los servicios del sistema. En distribuciones Linux antiguas, init ejecuta una serie de scripts de shell (ubicados en `/etc` llamados **init scripts**) para realizar una función similar.

### Procesos Padres e Hijos

Un programa puede lanzar otros programas, lo que se expresa en el esquema de procesos como un **proceso padre** produciendo un **proceso hijo**.

El kernel mantiene información sobre cada proceso:
- **PID (Process ID)**: Número único asignado a cada proceso en orden ascendente, con init siempre recibiendo PID 1
- **Memoria asignada**: El kernel hace seguimiento de la memoria asignada a cada proceso y la preparación para reanudar la ejecución
- **Propietario y UID efectivo**: Como los archivos, los procesos tienen propietarios y UIDs efectivos

### Daemons

Muchos servicios del sistema se implementan como **daemon programs**: programas que simplemente se sientan en segundo plano y realizan su trabajo sin interfaz visible. Incluso si no está logged in, el sistema ejecuta al menos algunos daemons.

## Visualizar Procesos

### El Comando `ps` - Instantánea de Procesos

El comando `ps` es la herramienta más común para ver procesos. En su forma más simple:

```bash
[me@linuxbox ~]$ ps
  PID TTY      TIME CMD
 5198 pts/1    00:00:00 bash
10129 pts/1    00:00:00 ps
```

Esto lista dos procesos: bash (el shell) y ps (el comando en sí). La salida muestra:
- **PID**: Process ID
- **TTY**: Tipo de terminal (identificación del terminal de control)
- **TIME**: Cantidad de CPU consumida por el proceso
- **COMMAND**: Nombre del comando

**Nota**: Por defecto, `ps` solo muestra procesos asociados con la sesión actual del terminal.

### Estados del Proceso

Cada proceso tiene un **estado** indicado en la columna STAT. Los estados principales son:

| Estado | Significado |
|--------|-------------|
| `R` | Running (En ejecución o listo para ejecutarse) |
| `S` | Sleeping (En reposo, esperando un evento como tecla o paquete de red) |
| `D` | Uninterruptible sleep (Esperando I/O como lectura de disco) |
| `T` | Stopped (Indicado para detenerse; más adelante en el capítulo) |
| `Z` | Defunct/"zombie" (Proceso hijo terminado pero no limpiado por su padre) |
| `<` | Proceso de alta prioridad (se le da más importancia a nivel de CPU) |
| `N` | Proceso de baja prioridad (nice process, obtendrá tiempo de CPU solo después de procesos de mayor prioridad) |

### Opciones Comunes de ps

**Opción `x`** - Listar procesos propios:
```bash
[me@linuxbox ~]$ ps x
  PID TTY      STAT   TIME COMMAND
 2799 ?        Ss     0:00 /usr/libexec/bonobo-activation-server -ac
 2820 ?        S1     0:01 /usr/libexec/evolution-data-server-1.10 --
```

La opción `x` muestra todos nuestros procesos sin importar qué terminal (si hay) los controla. El `?` en la columna TTY indica no hay terminal de control.

**Opción `aux`** - Listar todos los procesos con información adicional:
```bash
[me@linuxbox ~]$ ps aux
USER      PID %CPU %MEM    VSZ   RSS STAT START   TIME COMMAND
root        1 0.0  0.0   2136   644 ?   Ss   Mar05  0:31 init
root        2 0.0  0.0      0     0 ?   S<   Mar05  0:00 [ks]
```

La opción `aux` proporciona columnas adicionales:

| Columna | Significado |
|---------|-------------|
| `USER` | ID del propietario del proceso |
| `%CPU` | Uso de CPU en porcentaje |
| `%MEM` | Uso de memoria en porcentaje |
| `VSZ` | Tamaño de memoria virtual |
| `RSS` | Tamaño de conjunto residente (memoria física RAM en kilobytes) |
| `STAT` | Estado del proceso |
| `START` | Hora de inicio del proceso (valores > 24 horas muestran fecha) |
| `TIME` | Cantidad de CPU consumido por el proceso |

**Ver un proceso específico**:
```bash
[me@linuxbox ~]$ ps uw 44719
USER PID  %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
me  44719  0.0  0.0  13480  6492 pts/1  S    15:57  0:00 bash
```

### Visualizar Procesos Dinámicamente con `top`

Mientras que `ps` proporciona una instantánea, el comando `top` muestra una vista dinámica y continuamente actualizada (por defecto cada 3 segundos):

```bash
[me@linuxbox ~]$ top
```

El programa top muestra procesos en orden de actividad. La salida tiene dos partes:
1. Resumen del sistema en la parte superior
2. Tabla de procesos ordenados por actividad de CPU

**Resumen del sistema** (ejemplo):
```
top - 14:59:20 up 6:30,  2 users,  load average: 0.07, 0.02, 0.00
Tasks: 109 total,    1 running, 106 sleeping,    0 stopped,    2 zombie
Cpu(s): 0.7%us, 1.0%sy, 0.05%ni, 98.3%id, 0.0%wa, 0.0%hi, 0.0%si
Mem:   533712k total,  334606k used,  134806k free,   46386k buffers
Swap:   875500k total,  149128k used,  726372k free,  114676k cached
```

Campos informativos del top:

| Campo | Significado |
|-------|-------------|
| **top** | Nombre del programa |
| **14:59:20** | Hora actual |
| **up 6:30** | Uptime: el sistema ha estado en ejecución 6 horas 30 minutos |
| **2 users** | Dos usuarios logged in |
| **load average** | Promedio de procesos esperando ejecución en últimos 1, 5, 15 minutos (< 1.0 indica que la máquina no está ocupada) |
| **Tasks** | Resumen de procesos y diversos estados |
| **Cpu(s)** | Desglosa del uso de CPU (us=usuario, sy=sistema, ni=nice, id=idle, wa=waiting for I/O, hi=hardware interrupts, si=software interrupts) |
| **Mem** | Uso de RAM física |
| **Swap** | Uso de memoria virtual |

**Comandos de teclado en top**:
- `h` - Muestra pantalla de ayuda
- `q` - Sale de top
- Otros comandos disponibles en la página man

## Controlar Procesos

### Interrumpir un Proceso

Para interrumpir un programa en ejecución en el foreground, presiona **Ctrl+C**. Esto envía una señal SIGINT al proceso pidiendo que termine.

```bash
[me@linuxbox ~]$ xlogo
[me@linuxbox ~]$
```

Presionar Ctrl+C interrumpe el programa (xlogo en este caso).

### Poner un Proceso en Segundo Plano

Para lanzar un programa de modo que se coloque inmediatamente en background sin usar el prompt de shell, agregar un ampersand (`&`) al final:

```bash
[me@linuxbox ~]$ xlogo &
[1] 28236
[me@linuxbox ~]$
```

El shell imprime el jobspec `[1]` y el PID del proceso background `28236`. El prompt retorna inmediatamente.

### Control de Trabajos (Jobs)

El comando `jobs` lista los trabajos lanzados desde la terminal:

```bash
[me@linuxbox ~]$ jobs
[1]+ Running        xlogo &
```

El resultado muestra:
- **[1]** - Número del trabajo (jobspec)
- **+** - Indica este es el "trabajo actual"
- **Running** - Estado del trabajo
- **xlogo &** - El comando

Puedes ejecutar múltiples trabajos en background:

```bash
me@linuxbox:~$ xlogo & gedit &
[1] 47211
[2] 47212
```

### Traer un Proceso al Foreground

Para traer un proceso del background al foreground, usa el comando `fg`:

```bash
[me@linuxbox ~]$ jobs
[1]+ Running        xlogo &
[me@linuxbox ~]$ fg %1
xlogo
```

El comando `fg %1` trae el trabajo 1 al foreground. El jobspec `%1` es opcional si solo hay un trabajo.

### Detener (Pausar) un Proceso

Presiona **Ctrl+Z** para enviar una señal TSTP (terminal stop) a un proceso, pausándolo sin terminarlo:

```bash
[me@linuxbox ~]$ xlogo
[1]+ Stopped        xlogo
[me@linuxbox ~]$
```

Ahora el proceso está pausado. Puedes verificar presionando Ctrl+Z nuevamente en el prompt actual:

```bash
[me@linuxbox ~]$ bg %1
[1]+ xlogo &
[me@linuxbox ~]$
```

Usar `bg` para reanudar el proceso en background, o `fg` para reanudarlo en foreground.

## Cambiar Prioridad del Proceso

En la salida de `ps`, hay un atributo de proceso llamado **niceness**, que se refiere a la prioridad de planificación dada a un proceso. En ciertas circunstancias (como transcoding de video o ray tracing basado en CPU), podemos querer dar más prioridad a un proceso. Alternativamente, si queremos que un proceso use menos tiempo de CPU, podemos darle mayor niceness.

### El Comando `nice`

El comando `nice` lanza un proceso con una niceness especificada. La niceness se ajusta de -20 (más favorable) a 19 (menos favorable) con un valor predeterminado de 0 (sin ajuste).

```bash
[me@linuxbox ~]$ nice -n 10 cpu-hog
```

Esto ejecuta `cpu-hog` con niceness 10 (menor prioridad).

Para aumentar prioridad (solo como superuser):
```bash
[me@linuxbox ~]$ sudo nice -n -10 must-run-fast
```

**Nota**: Es raro ejecutar comandos con prioridad aumentada, ya que esto riesga desmesurar procesos esenciales del sistema.

### El Comando `renice`

El comando `renice` ajusta la prioridad de un proceso en ejecución:

```bash
[me@linuxbox ~]$ ps
  PID TTY      TIME CMD
379087 pts/9   00:00:00 bash
379215 pts/9   00:00:00 cpu-hog
379213 pts/9   00:00:00 ps

[me@linuxbox ~]$ renice -n 19 379215
```

Esto aumenta la niceness del proceso 379215 a 19 (menor prioridad). Solo el propietario o root puede cambiar prioridades.

## Señales

El comando `kill` no exactamente "mata" procesos; envía **señales** a programas. Signals son una forma en que el sistema operativo se comunica con los programas. Ya hemos visto señales en acción:
- **Ctrl+C** envía INT (interrupt)
- **Ctrl+Z** envía TSTP (terminal stop)

Los programas pueden escuchar y actuar sobre señales, permitiendo que un programa haga cosas como guardar trabajo en progreso cuando recibe una señal de terminación.

### Enviar Señales a Procesos con `kill`

**Sintaxis**: `kill [-signal] PID...`

Si no se especifica una señal, por defecto se envía la señal **TERM** (terminate).

```bash
[me@linuxbox ~]$ xlogo &
[1] 28401
[me@linuxbox ~]$ kill 28401
[1]+ Terminated    xlogo
```

Primero lanzamos xlogo en background, luego usamos `kill` con el PID. El proceso termina.

**Señales Comunes**:

| Número | Nombre | Significado |
|--------|--------|-------------|
| 1 | HUP (Hangup) | Señal histórica de sistemas conectados a líneas telefónicas. Se usa para indicar a programas que el terminal controlador ha cerrado. Muchos daemons escuchan esta señal para recargar archivos de configuración (ej: Apache web server). Es posible hacer un proceso inmune a HUP con `nohup` |
| 2 | INT (Interrupt) | Igual que Ctrl+C. Usualmente termina un programa |
| 9 | KILL | No se envía al proceso; el kernel lo termina inmediatamente. El proceso no tiene oportunidad de "limpiar" después de sí mismo o guardar trabajo. Solo usar como último recurso |
| 15 | TERM (Terminate) | Señal por defecto de `kill`. Si un programa es lo suficientemente "vivo" para recibir señales, terminará |
| 18 | CONT (Continue) | Reanuda proceso después de STOP o TSTP. Enviada por `bg` y `fg` |
| 19 | STOP | Pausa el proceso sin terminar. No se envía al proceso; el kernel lo pausa inmediatamente |
| 20 | TSTP (Terminal Stop) | Señal enviada por Ctrl+Z. A diferencia de STOP, el TSTP es recibido por el programa, pero el programa puede elegir ignorarlo |

Las señales pueden especificarse por número o por nombre:

```bash
[me@linuxbox ~]$ xlogo &
[1] 13546
[me@linuxbox ~]$ kill -1 13546
[1]+ Hangup    xlogo

[me@linuxbox ~]$ xlogo &
[1] 13601
[me@linuxbox ~]$ kill -INT 13601
[1]+ Interrupt    xlogo
[me@linuxbox ~]$ xlogo &
[1] 13608
[me@linuxbox ~]$ kill -SIGINT 13608
[1]+ Interrupt    xlogo
```

Señales adicionales frecuentemente usadas:

| Número | Nombre | Significado |
|--------|--------|-------------|
| 3 | QUIT | Quit (similar a INT pero produce un dump del core) |
| 11 | SEGV | Segmentation violation. Indica acceso ilegal a memoria |
| 28 | WINCH | Window change. Enviada por el sistema cuando cambia tamaño de ventana. Programas como `top` y `less` responden redibujándose |

Ver todas las señales disponibles:
```bash
[me@linuxbox ~]$ kill -l
```

### Hacer un Proceso Inmune a Desconexión

Muchos programas de línea de comandos responden a HUP terminando cuando su terminal controlador se cierra. Para prevenir esto, lanzar el programa con `nohup`:

```bash
[me@linuxbox ~]$ nohup xlogo
```

Si cerramos la ventana del terminal, xlogo continuará en ejecución.

### Enviar Señales a Múltiples Procesos con `killall`

Es posible enviar señales a múltiples procesos que coincidan con un programa o nombre de usuario usando `killall`:

```bash
[me@linuxbox ~]$ xlogo &
[1] 18801
[me@linuxbox ~]$ xlogo &
[2] 18802
[me@linuxbox ~]$ killall xlogo
[1]- Terminated    xlogo
[2]+ Terminated    xlogo
```

Lanzamos dos instancias de xlogo y luego las terminamos con una sola orden `killall`.

**Nota**: Se requieren privilegios de superusuario para enviar señales a procesos de otros usuarios.

## Apagar el Sistema

El apagado del sistema implica la terminación ordenada de todos los procesos, así como la realización de tareas vitales de housekeeping (como sincronización de todos los filesystems montados) antes de que el sistema se apague. Hay cuatro comandos que pueden realizar esta función: `halt`, `poweroff`, `reboot`, y `shutdown`.

Los primeros tres son bastante autoexplicativos y generalmente se usan sin opciones de línea de comandos:

```bash
[me@linuxbox ~]$ sudo reboot
```

El comando `shutdown` es más interesante. Con él, podemos especificar qué acción realizar (halt, power down, o reboot) y proporcionar un retraso de tiempo para el evento. La mayoría de las veces se usa así:

```bash
[me@linuxbox ~]$ sudo shutdown -h now
```

Detener el sistema ahora.

```bash
[me@linuxbox ~]$ sudo shutdown -r now
```

Reiniciar el sistema ahora.

El retraso puede especificarse de varias maneras. Cuando se ejecuta el comando `shutdown`, se "broadcasts" un mensaje a todos los usuarios logged-in advirtiendo del evento inminente.

## Más Comandos Relacionados con Procesos

Dado que monitorear procesos es una tarea importante de administración del sistema, hay muchos comandos para ello:

| Comando | Descripción |
|---------|-------------|
| `pstree` | Salida de lista de procesos dispuesta en patrón de árbol mostrando relaciones padre-hijo entre procesos |
| `vmstat` | Instantánea de uso de recursos del sistema incluyendo memoria, swap, e I/O de disco. Para ver una pantalla continua, seguir el comando con un retraso de tiempo (en segundos). Ejemplo: `vmstat 5`. Terminar salida con Ctrl+C |
| `xload` | Programa gráfico que dibuja gráfico mostrando carga del sistema sobre el tiempo |
| `tload` | Similar al programa xload pero dibuja gráfico en terminal. Terminar salida con Ctrl+C |

## Resumen

Los procesos en Linux son la manera fundamental en que el kernel organiza el trabajo:

- Un proceso es una instancia en ejecución de un programa con su propia memoria y conjunto de recursos
- El comando `ps` muestra instantánea de procesos; `top` muestra vista dinámica actualizada
- Los procesos pueden ejecutarse en foreground o background
- Job control permite suspender, reanudar, y cambiar procesos entre foreground y background
- Las señales son el mecanismo de comunicación del sistema con programas
- El comando `kill` envía señales (generalmente para terminar procesos)
- La prioridad del proceso puede ajustarse con `nice` y `renice`
- El sistema puede apagarse ordenadamente con `halt`, `poweroff`, `reboot`, o `shutdown`

---

## Ver También

- [[wiki/linux/09-permisos.md|Capítulo 9: Permisos en Linux]] — Identidades de usuario y control de acceso a procesos
- [[wiki/linux/11-el-environment.md|Capítulo 11: El Environment (Entorno) de Linux]] — Variables de environment heredadas por procesos hijo
