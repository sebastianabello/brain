# Registro de Operaciones

## [2026-06-22] ingest | Procesos en Linux

- **Nuevo artículo creado**: `wiki/linux/10-procesos.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-137-151.pdf` (Capítulo 10)
- **Contenido**: Traducción y síntesis completa en español sobre gestión de procesos en Linux. Cubre: (1) Concepto de procesos en sistemas multitarea y multidaemon. (2) Comando `ps` con diferentes opciones (x, aux) y estados de proceso. (3) Visualización dinámica con `top`: resumen de sistema, campos informativos, uso de CPU/memoria/swap. (4) Control de procesos: interrumpir (Ctrl+C), background/foreground (&, fg, bg), pausa (Ctrl+Z), job control (jobs). (5) Cambiar prioridad: comandos `nice` y `renice`. (6) Señales: explicación de cómo funcionan, comando `kill`, tabla de señales comunes (HUP, INT, KILL, TERM, STOP, TSTP, CONT), `nohup` para inmunidad a desconexión, `killall` para múltiples procesos. (7) Apagar el sistema: comandos halt, poweroff, reboot, shutdown. (8) Comandos adicionales: pstree, vmstat, xload, tload.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 10
  - Artículo completo con tablas extensas, ejemplos ejecutables, explicación de conceptos de sistemas operativos

## [2026-06-22] ingest | Permisos en Linux

- **Nuevo artículo creado**: `wiki/linux/09-permisos.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-119-135.pdf` (Capítulo 9)
- **Contenido**: Traducción y síntesis completa en español sobre sistema de permisos Unix/Linux. Cubre: (1) Concepto de usuarios, grupos y "others", comando `id`. (2) Tipos de archivo (regular, directorio, link simbólico, character/block special). (3) Permisos rwx para usuario/grupo/otros. (4) Comando `chmod` con notación octal y simbólica. (5) Explicación detallada de números octales, hexadecimales y binarios. (6) Comando `umask` para permisos por defecto. (7) Permisos especiales: setuid (4000), setgid (2000), sticky bit (1000). (8) Cambiar identidades con `su` y `sudo`, diferencias y casos de uso. (9) Comandos `chown` y `chgrp`. (10) Caso práctico completo: directorio compartido entre dos usuarios. (11) Comando `passwd` y utilidades shadow-utils.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 9
  - Artículo extenso con tablas de referencia, ejemplos ejecutables, explicaciones de conceptos matemáticos (octal/binario), y caso práctico paso a paso

## [2026-06-22] ingest | Trabajo con Comandos

- **Nuevo artículo creado**: `wiki/linux/05-trabajo-con-comandos.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-75-84.pdf` (Capítulo 5)
- **Contenido**: Traducción y síntesis en español sobre 4 tipos de comandos, herramientas para identificar comandos (type, which), obtención de documentación (help, man, info, apropos, whatis), y creación de alias como introducción a la programación del shell.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 5
  - Capítulo fundamental para aprender a buscar ayuda y documentación en Linux

## [2026-06-22] ingest | Manipulación de Archivos y Directorios

- **Nuevo artículo creado**: `wiki/linux/04-manipulacion-archivos-directorios.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-59-73.pdf` (Capítulo 4)
- **Contenido**: Traducción y síntesis en español sobre comodines (wildcards), archivos ocultos, comandos mkdir/cp/mv/rm/ln, opciones y ejemplos de uso, diferencia entre hard links y symbolic links, con ejemplo práctico completo de creación de playground.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 4
  - Capítulo extenso con muchas tablas de opciones y ejemplos prácticos

## [2026-06-22] ingest | Explorando el Sistema

- **Nuevo artículo creado**: `wiki/linux/03-explorando-el-sistema.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-49-58.pdf` (Capítulo 3)
- **Contenido**: Traducción y síntesis en español sobre comandos avanzados de ls, interpretación del formato largo, comando file para determinar tipos de archivo, comando less para visualización de contenidos, tour completo del sistema de directorios Linux, y explicación de enlaces simbólicos y duros.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 3
  - Tabla extensa de directorios del sistema incluida

## [2026-06-22] ingest | Navegación del Sistema de Archivos

- **Nuevo artículo creado**: `wiki/linux/02-navegacion-sistema-archivos.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-43-48.pdf` (Capítulo 2)
- **Contenido**: Traducción y síntesis en español sobre estructura jerárquica del sistema de archivos Linux, directorios actuales, comandos pwd/cd/ls, rutas absolutas y relativas, y convenciones de nombres de archivos en Linux.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 2

## [2026-06-22] ingest | Introducción al Shell de Linux

- **Nuevo artículo creado**: `wiki/linux/01-introduccion-shell.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-37-41.pdf` (Capítulo 1)
- **Contenido**: Traducción y síntesis en español de conceptos fundamentales sobre el shell bash, emuladores de terminal, comandos básicos (date, uptime, df, free), historial de comandos, movimiento del cursor y consolas virtuales.
- **Resumen de cambios**:
  - Creado índice inicial en `wiki/index.md`
  - Agregada entrada en el registro

## [2026-06-22] ingest | Redirección (I/O Redirection)

- **Nuevo artículo creado**: `wiki/linux/06-redirection.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-85-97.pdf` (Capítulo 6)
- **Contenido**: Traducción y síntesis completa en español sobre I/O redirection. Cubre conceptos fundamentales de stdin/stdout/stderr, redirección con `>`, `>>`, `2>`, `2>&1`, `&>`. Explica agrupación de comandos, `/dev/null`, redirección de entrada con `<`. Sección extensa sobre pipelines y filtros. Describe en detalle 7 comandos: `cat`, `sort`, `uniq`, `grep`, `wc`, `head/tail`, `tee`. Incluye casos de uso, opciones principales, advertencia sobre diferencia entre `>` y `|`, y filosofía Unix de composición.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 6
  - Articulo estructurado con tablas de opciones, ejemplos ejecutables y notas importantes

## [2026-06-22] ingest | Expansiones: Cómo el Shell Ve el Mundo

- **Nuevo artículo creado**: `wiki/linux/07-expansiones.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-99-109.pdf` (Capítulo 7)
- **Contenido**: Traducción y síntesis completa en español sobre expansiones shell. Cubre 7 tipos de expansión: pathname expansion (wildcards), tilde expansion (~), arithmetic expansion ($((...))), brace expansion ({...}), parameter expansion ($var), command substitution ($(...)). Sección extensa sobre quoting: double quotes, single quotes, escaping characters, y backslash escape sequences (\n, \t, \a, etc.). Incluye ejemplos ejecutables, tablas comparativas, notas sobre comportamientos especiales (archivos ocultos, word-splitting, archivos con espacios), y casos de uso prácticos como crear estructuras de directorios.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 7
  - Articulo con estructura clara de conceptos progresivos, tablas de referencia y ejemplos de comando real

## [2026-06-22] ingest | Trucos Avanzados del Teclado

- **Nuevo artículo creado**: `wiki/linux/08-trucos-teclado.md`
- **Fuente**: `raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-111-118.pdf` (Capítulo 8)
- **Contenido**: Traducción y síntesis completa en español sobre atajos de teclado usando Readline. Cubre 5 secciones principales: (1) Movimiento del cursor (Ctrl+A, Ctrl+E, Alt+F, Alt+B). (2) Modificación de texto (Ctrl+D, Ctrl+T, Alt+T, Alt+L, Alt+U). (3) Cortar y pegar/Kill and Yank (Ctrl+K, Ctrl+U, Alt+D, Alt+Backspace, Ctrl+Y). (4) Completado de comandos con TAB, tipos de completado (variables, usuarios, hostnames), y completado programable. (5) Historial: navegación (Ctrl+P/N, Alt+</>, búsqueda incremental inversa (Ctrl+R), expansión del historial (!!,!N,!string,!?string). Incluye comando script para grabar sesiones, ejemplos prácticos, tabla de referencia rápida de todos los atajos.
- **Resumen de cambios**:
  - Actualizado `wiki/index.md` con entrada del Capítulo 8
  - Artículo con tablas extensas de referencia rápida, ejemplos paso a paso, explicación de conceptos históricos (Meta key)

---

*Registro append-only de operaciones de ingest, query archivado y lint*
