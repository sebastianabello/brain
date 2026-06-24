---
title: "Capítulo 22: Impresión en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 22"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-343-353.pdf"
updated: "2026-06-23"
---

# Impresión en Linux

## Una Breve Historia de la Impresión

Para entender mejor los sistemas de impresión en Unix/Linux, es útil conocer cómo evolucionó la tecnología de impresión.

### Impresoras en la Era Centralizada

En los años 1980, en la era anterior a las computadoras personales, los usuarios accedían a computadoras a través de terminales remotos conectados a un servidor central. Las impresoras eran dispositivos costosos y centralizados, supervisados por operadores de computadora.

Como las impresoras eran compartidas por múltiples usuarios, fue necesario **crear una cola de impresión (print queue)**: un mecanismo para programar trabajos de impresión y entregarlos a los usuarios individuales. Este es el concepto que prevalece hasta hoy en Unix.

### Impresoras de Caracteres

La tecnología de impresora de los años 1980 se basaba en **impact printers** (impresoras de impacto). Estos dispositivos utilizaban un mecanismo mecánico (como una rueda de margarita o matriz de puntos) que golpeaba cinta contra papel.

**Características principales:**
- Las impresoras solo podían imprimir caracteres fijos prediseñados en la rueda o matriz
- Usaban **fuentes monoespaciadas** (ancho fijo) — cada carácter ocupaba el mismo espacio
- Una página US-Letter (8.5" × 11") de 66 líneas y 80 caracteres de ancho = **4,800 bytes de datos**

**Códigos de control:**
Para la impresión de caracteres, los datos se enviaban como un stream simple de bytes con códigos ASCII. Para funciones especiales (avance de línea, cambio de fuente), se utilizaban códigos de control ASCII de bajo número.

Ejemplo: Para imprimir texto en **negrita** mediante impact printer:
```
- Imprimir el carácter
- Enviar backspace
- Imprimir el carácter nuevamente
```

### Impresoras Gráficas

Con la llegada de las interfaces gráficas (GUI), la impresión evolucionó de caracteres fijos a gráficos.

**Cambio tecnológico:**
El desarrollo de **impresoras láser de bajo costo** permitió:
- Imprimir en cualquier lugar de la página (no solo caracteres predefinidos)
- Usar fuentes proporcionales
- Crear diagramas y gráficos de alta calidad
- Mayor flexibilidad general

**Desafío técnico:**
Una página de 300 DPI (dots per inch) requiere **900,000 bytes de datos**. Las redes de 1980 no podían manejar esto, así que fue necesario un nuevo enfoque.

### PostScript y Lenguajes de Descripción de Página

La solución fue el **lenguaje PostScript (PDL)** — un lenguaje de programación que describe el contenido de una página:

```
Go to this position, draw the character 'a' in 10 point Helvetica, 
go to this position...
```

**PostScript** de Adobe Systems:
- Lenguaje de programación completo para tipografía y gráficos
- Incluye 35 fuentes estándar, posibilidad de agregar más en tiempo de ejecución
- La impresora contiene su propio procesador que ejecuta el programa PostScript
- Mucho más pequeño que el stream de datos de bitmap

**RIP (Raster Image Processor):**
El proceso se denomina **rendering** — la impresora ejecuta el programa PostScript y genera un patrón de bits que se transfiere al papel.

## Impresión en Linux

Los sistemas Linux modernos emplean dos suites de software para impresión:

### CUPS (Common Unix Printing System)

**CUPS** es el sistema estándar de impresión en Linux moderno. Proporciona:
- **Gestión de colas de impresión** — donde se programan los trabajos hasta poder imprimirse
- **Gestión de trabajos de impresión** — capacidad para programar, pausar, reanudar o cancelar trabajos
- **Reconocimiento de tipos de datos** — puede convertir archivos a un formato imprimible
- **Conversión automática** — puede convertir datos a un formato que la impresora entienda

CUPS también tiene la capacidad de reconocer diferentes tipos de datos (dentro de lo razonable) y convertir archivos a un formato imprimible.

### Ghostscript

**Ghostscript** es un intérprete de PostScript que actúa como un **RIP** (Raster Image Processor). Convierte programas PostScript en archivos imprimibles para impresoras que no entienden PostScript de forma nativa.

## Preparación de Archivos para Impresión

### Comando: `pr` — Convertir Archivos de Texto para Impresión

El comando `pr` ajusta texto para fit en un tamaño de página específico, con paginación opcional, headers y márgenes.

**Sintaxis:**
```bash
pr [opciones] [archivo]
```

**Opciones comunes de `pr`:**

| Opción | Descripción |
|--------|-------------|
| `+first[:last]` | Imprimir rango de páginas comenzando con `first` y, opcionalmente, terminando con `last` |
| `-columns` | Organizar contenido en número de columnas especificadas |
| `-a` | De forma predeterminada, la salida de múltiples columnas se lista verticalmente. Al agregar `-a`, el contenido se lista horizontalmente |
| `-d` | Salida de espacio doble |
| `-D "format"` | Formatear la fecha mostrada en headers de página usando formato. Ver man page del comando `date` |
| `-f` | Usar form feeds en lugar de carriage returns para separar páginas |
| `-h "header"` | En la porción central del header de página, usar `header` en lugar del nombre del archivo siendo procesado |
| `-l length` | Establecer longitud de página a `length`. Predeterminado es 66 (US letter de 6 líneas por pulgada) |
| `-n` | Numerar líneas |
| `-o offset` | Crear margen izquierdo de `offset` caracteres de ancho |
| `-w width` | Establecer ancho de página a `width`. Predeterminado es 72 |

**Ejemplo: Listar directorio con formateo para impresión**

Crear una salida formateada de 3 columnas, 65 caracteres de ancho:

```bash
[me@linuxbox ~]$ ls /usr/bin | pr -3 -w 65 | head
```

Este comando:
1. Lista contenido de `/usr/bin`
2. Formatea en 3 columnas con ancho de 65 caracteres
3. Agrega headers automáticos de fecha, nombre de archivo y página
4. Muestra solo primeras líneas con `head`

### Comando: `lp` / `lpr` — Enviar Trabajos de Impresión

CUPS soporta dos métodos históricos de impresión:

#### `lpr` — Estilo Berkeley

El programa `lpr` (del Berkeley Software Distribution) envía archivos a la impresora.

**Sintaxis:**
```bash
lpr [opciones] archivo
```

**Enviar a una impresora específica:**
```bash
lpr -P nombre_impresora archivo
```

**Ver impresoras disponibles:**
```bash
lpstat -a
```

**Opciones comunes de `lpr`:**

| Opción | Descripción |
|--------|-------------|
| `-# number` | Establecer número de copias a `number` |
| `-p` | Imprimir cada página con header sombreado con fecha, hora, nombre de trabajo y número de página. Esta opción "pretty print" se utiliza al imprimir archivos de texto |
| `-P printer` | Especificar nombre de la impresora usada para salida. Si no se especifica impresora, se usa la impresora predeterminada del sistema |
| `-r` | Eliminar archivos después de imprimir. Sería útil para programas que producen salida temporal de impresora |

**Ejemplo:**
```bash
[me@linuxbox ~]$ ls /usr/bin | pr -3 | lpr
```

#### `lp` — Estilo System V

El programa `lp` (de System V) es similar a `lpr` pero soporta un conjunto de opciones más sofisticado.

**Sintaxis:**
```bash
lp [opciones] archivo
```

**Opciones comunes de `lp`:**

| Opción | Descripción |
|--------|-------------|
| `-d printer` | Establecer destino (impresora) a `printer`. Si no se especifica `-d`, se usa la impresora predeterminada del sistema |
| `-n number` | Establecer número de copias a `number` |
| `-o landscape` | Establecer salida a orientación apaisada |
| `-o fitplot` | Escalar el archivo para ajustarlo a la página. Útil al imprimir imágenes como archivos JPEG |
| `-o scaling=number` | Escalar archivo a `number`. Valores menores a 100 se reducen, mientras que valores mayores a 100 causan que el archivo se imprima en múltiples páginas |
| `-o cpi=number` | Establecer caracteres de salida por pulgada a `number`. Predeterminado es 10 |
| `-o lpi=number` | Establecer líneas de salida por pulgada a `number`. Predeterminado es 6 |
| `-P pages` | Especificar lista de páginas. `pages` puede ser expresada como lista separada por comas y/o rango, por ejemplo, `1,3,5,7-10` |

**Ejemplo con múltiples opciones:**

```bash
[me@linuxbox ~]$ ls /usr/bin | pr -4 -w 90 -l 88 | lp -o page-left=36 -o cpi=12 -o lpi=8
```

Este comando:
1. Lista `/usr/bin`
2. Formatea en 4 columnas, 90 caracteres de ancho, 88 líneas por página
3. Envía a impresora con ajustes: 36 caracteres de margen izquierdo, 12 CPI, 8 LPI

### Comando: `a2ps` — Formato de Archivos para Impresión en PostScript

El programa `a2ps` (ASCII a PostScript) es un convertidor y formateador de archivos. Diferente de `pr` que solo formatea texto:

**Características principales:**
- Convierte texto a PostScript para mejor presentación
- Envía salida a la impresora predeterminada por defecto (comportamiento de "pretty printer")
- Mejora significativa en apariencia de salida

**Ejemplo: Convertir listado de directorios a PostScript con 66 líneas:**

```bash
[me@linuxbox ~]$ ls /usr/bin | pr -3 -t | a2ps -o ~/Desktop/ls.ps -L 66
[stdin (plain): 11 pages on 6 sheets]
[Total: 11 pages on 6 sheets] saved into the file `/home/me/Desktop/ls.ps'
```

Interpretación:
- Filtramos con `pr` usando `-3` (3 columnas) y `-t` (sin headers/footers)
- Con `a2ps` especificamos:
  - `-o ~/Desktop/ls.ps`: archivo de salida (PostScript)
  - `-L 66`: 66 líneas por página (para coincidir con formateo de `pr`)
- Resultado: 11 páginas en 6 hojas de papel (formato "two-up")

**Opciones comunes de `a2ps`:**

| Opción | Descripción |
|--------|-------------|
| `--center-title=text` | Establecer título de página centrado a `text` |
| `--columns=number` | Organizar páginas en `number` columnas. Predeterminado es 2 |
| `--footer-text` | Establecer pie de página a `text` |
| `--pages=range` | Imprimir páginas en `range` |
| `--left-footer=text` | Establecer pie de página izquierdo a `text` |
| `--left-title=text` | Establecer título de página izquierdo a `text` |
| `-P printer` | Usar `printer`. Si no se especifica, se usa salida estándar |
| `-R` | Orientación retrato |
| `-r` | Orientación apaisada |
| `-T number` | Establecer tabulaciones cada `number` caracteres |
| `-u text` | Subrayar (watermark) páginas con `text` |

**Nota:** `a2ps` tiene muchas más opciones. Use `a2ps --help` o `man a2ps` para referencia completa.

## Monitoreo y Control de Trabajos de Impresión

CUPS proporciona varios programas de línea de comandos para gestionar el estado de la impresora y colas de impresión.

### Comando: `lpstat` — Mostrar Estado del Sistema de Impresión

El programa `lpstat` es útil para determinar nombres y disponibilidad de impresoras en el sistema.

**Ejemplo: Ver estado general de impresoras**

```bash
[me@linuxbox ~]$ lpstat -a
PDF accepting requests since Mon 08 Dec 2024 03:05:59 PM EST
printer accepting requests since Tue 24 Feb 2025 08:43:22 AM EST
```

**Ver configuración detallada del sistema:**

```bash
[me@linuxbox ~]$ lpstat -s
system default destination: printer
device for PDF: cups-pdf:/
device for printer: ipp://print-server:631/printers/printer
```

En este ejemplo:
- `printer` es la impresora predeterminada del sistema
- Es una impresora de red usando protocolo IPP (Internet Printing Protocol) conectada a un servidor llamado `print-server`

**Opciones comunes de `lpstat`:**

| Opción | Descripción |
|--------|-------------|
| `-a [printer...]` | Mostrar estado de cola de impresión para `printer`. Nota: esto es estado de la cola de impresora, no estado de impresoras físicas. Si no se especifican impresoras, se muestran todas las colas |
| `-d` | Mostrar nombre de la impresora predeterminada del sistema |
| `-p [printer...]` | Mostrar estado de impresora especificada `printer`. Si no se especifican impresoras, se muestran todas |
| `-r` | Mostrar estado del servidor de impresión |
| `-s` | Mostrar resumen de estado |
| `-t` | Mostrar reporte de estado completo |

### Comando: `lpq` — Mostrar Estado de la Cola de Impresión

Para ver el estado de una cola de impresión, se usa `lpq`.

```bash
[me@linuxbox ~]$ lpq
printer is ready
no entries
```

Si enviamos un trabajo y luego verificamos la cola:

```bash
[me@linuxbox ~]$ ls > /tmp/file.txt && pr -3 | lp
request id is printer-603 (1 file(s))
[me@linuxbox ~]$ lpq
printer is ready and printing
Rank   Owner   Job   File(s)        Total Size
active  me    603   (stdin)        1024 bytes
```

Tabla de interpretación:
- **Rank**: Posición en cola (active = actualmente imprimiendo)
- **Owner**: Usuario que envió trabajo
- **Job**: Número de trabajo (ID)
- **File(s)**: Nombre de archivo(s)
- **Total Size**: Tamaño total en bytes

### Comando: `lprm` / `cancel` — Cancelar Trabajos de Impresión

Para terminar trabajos de impresión y removerlos de la cola:

**Estilo Berkeley (`lprm`):**
```bash
lprm job_id
```

**Estilo System V (`cancel`):**
```bash
cancel job_id
```

**Ejemplo: Cancelar trabajo de impresión**

```bash
[me@linuxbox ~]$ cancel 603
[me@linuxbox ~]$ lpq
printer is ready
no entries
```

Cada comando tiene opciones para remover todos los trabajos de un usuario particular, una impresora particular, y múltiples números de trabajo. Las páginas de man tienen todos los detalles.

## Resumen

Este capítulo exploró cómo los sistemas de impresión de Unix/Linux heredaron conceptos de los primeros días de la computación centralizada, y cómo la tecnología moderna de impresoras gráficas (especialmente PostScript) ha influido en el diseño de sistemas de impresión actuales.

**Puntos clave:**
- Entender la historia proporciona contexto para las opciones de control de impresión disponibles en la línea de comandos
- CUPS es el sistema de impresión estándar en Linux moderno
- Múltiples herramientas para preparar archivos (`pr`, `lp`/`lpr`, `a2ps`)
- Herramientas para monitorear y controlar trabajos (`lpstat`, `lpq`, `lprm`/`cancel`)

---

## Ver También

- [[wiki/linux/20-procesamiento-texto.md|Capítulo 20: Procesamiento de Texto]] — Herramientas de filtrado y transformación
- [[wiki/linux/21-formato-salida.md|Capítulo 21: Formato de Salida]] — Herramientas de formateo (nl, fold, fmt, pr, printf, groff)
