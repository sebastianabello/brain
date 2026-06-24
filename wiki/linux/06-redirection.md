---
title: "Capítulo 6: Redirecciónes en Linux"
Sources: raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-85-97.pdf
Raw: Capítulo 6 - REDIRECTION (págs. 51-63)
Updated: 2026-06-22
---

# Redirección (I/O Redirection)

## Concepto Fundamental

**I/O Redirection** es una de las características más poderosas de la línea de comandos de Linux. Permite cambiar dónde proviene la entrada de un comando y dónde va su salida, así como conectar múltiples comandos en **pipelines** para crear operaciones complejas.

El axioma fundamental de Unix: "**Todo es un archivo**". Los programas envían sus resultados a **salida estándar** (stdout) y sus mensajes de estado/error a **salida de error estándar** (stderr).

---

## Entrada, Salida y Error Estándar

En Linux, cada programa maneja tres flujos de datos:

| Flujo | Nombre | Descriptor | Por defecto | Propósito |
|-------|--------|-----------|-----------|-----------|
| **Entrada** | stdin | 0 | Teclado | Lee datos del usuario |
| **Salida** | stdout | 1 | Pantalla | Resultados del programa |
| **Error** | stderr | 2 | Pantalla | Mensajes de estado/error |

**Ejemplo:**
```bash
ls -l /usr/bin
```
Aquí, `ls` es el comando y su salida (el listado) va a stdout (pantalla). Si ocurre un error, se imprime en stderr.

---

## Redirección de Salida Estándar (>)

Cambiar a dónde va la salida de un comando. Por defecto va a la pantalla, pero podemos redirigirla a un archivo.

### Sintaxis
```bash
comando > archivo
```

### Ejemplo
```bash
ls -l /usr/bin > ls-output.txt
```

**Comportamiento importante:**
- Si el archivo **NO existe**, se crea
- Si el archivo **SÍ existe**, se **sobrescribe** completamente (se trunca desde el inicio)

### Append vs Overwrite
- `>` → Sobrescribe (trunca el archivo)
- `>>` → Añade al final (append)

```bash
# Sobrescribir
ls -l /usr/bin > ls-output.txt

# Añadir
ls -l /usr/bin >> ls-output.txt
```

### Comando para Truncar/Crear Archivo Vacío
```bash
> archivo.txt
```

---

## Redirección de Error Estándar (2>)

Los programas bien diseñados envían los mensajes de error a stderr, no a stdout. Por eso ocurren cosas inesperadas:

```bash
ls -l /bin /usr > ls-output.txt
# Error: /bin no existe, el mensaje se imprime en pantalla
```

La redirección `>` solo captura stdout, no stderr. Para redirigir stderr usamos el **descriptor de archivo 2**:

```bash
ls -l /bin /usr 2> ls-error.txt
```

**Descriptores de archivo:**
- `0` = stdin
- `1` = stdout (por defecto, se puede omitir)
- `2` = stderr

```bash
ls -l /bin/usr 1> ls-output.txt 2> ls-error.txt
```

---

## Redirección Combinada: Salida y Error al Mismo Archivo

### Método Tradicional (funciona en bash antiguo)
```bash
ls -l /bin /usr > ls-output.txt 2>&1
```

**Interpretación:**
- `> ls-output.txt` → Redirige stdout al archivo
- `2>&1` → Redirige descriptor 2 (stderr) a descriptor 1 (stdout), que ya está redirigido al archivo

⚠️ **IMPORTANTE:** El orden de las redirecciones es significativo. La redirección de error DEBE ocurrir después de la de salida:
```bash
# ❌ MAL - stderr va a pantalla
2>&1 > ls-output.txt

# ✅ BIEN - ambos van al archivo
> ls-output.txt 2>&1
```

### Método Moderno (bash 4+)
```bash
ls -l /bin /usr &> ls-output.txt
```

O append:
```bash
ls -l /bin /usr &>> ls-output.txt
```

---

## Agrupación de Comandos { }

Para redirigir la salida de múltiples comandos, se pueden agrupar con llaves:

```bash
{ command1; command2; command3; } > logfile.txt
```

**Sintaxis correcta:**
- Espacios alrededor de las llaves
- Punto y coma después de cada comando
- La redirección se aplica a todo el grupo

```bash
{ echo "Inicio"; ls /usr/bin; echo "Fin"; } > salida.txt
```

---

## Descartar Salida No Deseada: /dev/null

`/dev/null` es un archivo especial (dispositivo) que acepta entrada y no hace nada con ella. Es el "cubo de basura" de Unix.

**Uso común:** Silenciar mensajes de error:
```bash
ls -l /bin /usr 2> /dev/null
# Solo muestra la salida válida, los errores desaparecen
```

O descartar toda la salida:
```bash
ls -l /usr/bin > /dev/null 2>&1
```

---

## Redirección de Entrada Estándar (<)

Por defecto, la entrada viene del teclado (stdin). Podemos cambiarla a un archivo.

### Sintaxis
```bash
comando < archivo
```

### Comando: cat — Concatenar Archivos

`cat` lee uno o más archivos y copia su contenido a stdout.

**Sintaxis:**
```bash
cat [archivo...]
```

**Ejemplos:**
```bash
# Mostrar contenido de un archivo
cat archivo.txt

# Concatenar múltiples archivos
cat archivo1.txt archivo2.txt archivo3.txt

# Unir partes de un archivo multimedia
cat movie.mpeg.001 movie.mpeg.002 movie.mpeg.003 > movie.mpeg
```

**Entrada interactiva con cat:**
```bash
cat > lazy_dog.txt
The quick brown fox jumps over the lazy dog.
[Ctrl+D]
# El archivo se crea con el texto ingresado
```

**Redirección de entrada con cat:**
```bash
cat < lazy_dog.txt
# Redirige el archivo como entrada, resultado es igual a `cat lazy_dog.txt`
```

---

## Pipelines (Tuberías) — |

El operador `|` (pipe/tubería) conecta comandos: la salida de uno es entrada para el siguiente.

```bash
comando1 | comando2
```

**Ejemplo:**
```bash
ls -l /usr/bin | less
# Muestra el listado paginado
```

### Concepto: Filtros

Los **filtros** son programas que leen stdin, transforman los datos, y escriben a stdout. Se usan frecuentemente en pipelines.

**Ejemplo de pipeline con filtros:**
```bash
ls /bin /usr/bin | sort | uniq | less
```

Este pipeline:
1. Lista archivos de dos directorios
2. Los ordena alfabéticamente
3. Elimina duplicados
4. Los muestra paginados

---

## Comando: sort — Ordenar Líneas

`sort` ordena líneas de texto alfabéticamente.

**Sintaxis:**
```bash
sort [opciones] [archivo...]
```

**Características:**
- Lee desde stdin o archivo
- Escribe a stdout
- Muy potente, con muchas opciones avanzadas (ver Capítulo 20)

**Ejemplo:**
```bash
ls /bin /usr/bin | sort | less
```

---

## Comando: uniq — Reportar/Omitir Líneas Repetidas

`uniq` reporta o omite líneas repetidas consecutivas.

**Sintaxis:**
```bash
uniq [opciones] [entrada] [salida]
```

⚠️ **Importante:** La entrada DEBE estar ordenada para que funcione correctamente.

**Opciones comunes:**
- `-d` → Muestra solo líneas duplicadas
- `-c` → Cuenta ocurrencias

**Ejemplo:**
```bash
# Eliminar duplicados
ls /bin /usr/bin | sort | uniq | less

# Ver solo duplicados
ls /bin /usr/bin | sort | uniq -d | less

# Contar duplicados
ls /bin /usr/bin | sort | uniq -c
```

---

## Comando: wc — Contar Líneas, Palabras y Bytes

`wc` (word count) imprime número de líneas, palabras y bytes en archivos.

**Sintaxis:**
```bash
wc [opciones] [archivo...]
```

**Salida estándar:**
```bash
wc archivo.txt
7902  64566  503634 archivo.txt
```
Significado: 7902 líneas, 64566 palabras, 503634 bytes

**Opciones:**
- `-l` → Solo líneas
- `-w` → Solo palabras
- `-c` → Solo bytes

**Ejemplo en pipeline:**
```bash
ls /bin /usr/bin | sort | uniq | wc -l
# Cuenta cuántos programas únicos hay
```

---

## Comando: grep — Buscar Patrones

`grep` busca líneas que contengan un patrón específico y las imprime.

**Sintaxis:**
```bash
grep patrón [archivo...]
```

**Funcionalidad:**
- Busca patrones de texto simples
- Patrones avanzados con expresiones regulares (Capítulo 19)
- Lee desde stdin o archivo

**Opciones comunes:**
| Opción | Descripción |
|--------|-------------|
| `-i` | Ignora mayúsculas/minúsculas |
| `-l` | Imprime solo nombres de archivos que contienen el patrón |
| `-v` | Invierte la búsqueda (líneas que NO contienen el patrón) |
| `-w` | Busca solo palabras completas |

**Ejemplo:**
```bash
# Buscar programas con "zip" en el nombre
ls /bin /usr/bin | sort | uniq | grep zip
```

**Búsqueda en archivos:**
```bash
grep "The" libro.txt
```

---

## Comando: head/tail — Primeras/Últimas Líneas

### head — Mostrar Primeras Líneas

```bash
head [opciones] [archivo...]
```

Por defecto, muestra las primeras 10 líneas.

**Opción:**
- `-n N` → Muestra N líneas (o `-N` directamente)

```bash
head -n 5 ls-output.txt
```

### tail — Mostrar Últimas Líneas

```bash
tail [opciones] [archivo...]
```

Muestra las últimas 10 líneas por defecto.

**Opciones:**
- `-n N` → Muestra N líneas (o `-N` directamente)
- `-n +N` → Muestra todo excepto las primeras N-1 líneas (invierte)
- `-f` → Sigue el archivo en tiempo real (útil para logs)

**Ejemplo:**
```bash
# Extraer líneas de la 5 a la 10 de un archivo
head -n 5 texto.txt | tail -n 6 > fragmento.txt
```

**Monitorear logs en tiempo real:**
```bash
tail -f /var/log/messages
```

---

## Comando: tee — Leer de stdin, Escribir a stdout y Archivos

`tee` crea un "acoplamiento tipo T" en la tubería: lee desde stdin, copia a stdout y a uno o más archivos.

**Sintaxis:**
```bash
tee [opciones] archivo...
```

**Útil para:**
- Capturar datos intermedios en un pipeline
- Guardar AND ver resultados simultáneamente

**Ejemplo:**
```bash
ls /usr/bin | tee ls.txt | grep zip
# Guarda listado completo en ls.txt
# Y filtra solo programas con "zip"
```

**Append:**
```bash
tee -a archivo.txt
```

---

## Diferencia entre > y | (Crítica)

Esto es fácil de confundir:

```bash
# ❌ INCORRECTO - sobrescribe archivo ls con output de ls
ls > ls

# ✅ CORRECTO - canaliza output de ls a less
ls | less
```

**Resumen:**
- `comando1 > archivo` → Guarda salida en archivo
- `comando1 | comando2` → Conecta salida de comando1 a entrada de comando2

**Caso de advertencia real:**
Un administrador accidentalmente hizo:
```bash
cd /usr/bin
ls > less
```
Esto sobrescribió el programa `less` con el listado de archivos, ¡destruyendo el programa! La moraleja: el operador `>` crea/sobrescribe archivos silenciosamente, hay que usarlo con respeto.

---

## Resumen de Comandos de Redirección

| Operador | Uso | Ejemplo |
|----------|-----|---------|
| `>` | Redirige stdout a archivo | `cmd > out.txt` |
| `>>` | Append stdout a archivo | `cmd >> out.txt` |
| `2>` | Redirige stderr a archivo | `cmd 2> err.txt` |
| `2>&1` | Redirige stderr a stdout | `cmd > out.txt 2>&1` |
| `&>` | Redirige stdout y stderr (bash 4+) | `cmd &> out.txt` |
| `<` | Redirige archivo a stdin | `cmd < in.txt` |
| `\|` | Conecta stdout a stdin | `cmd1 \| cmd2` |

---

## Comandos Cubiertos

| Comando | Descripción |
|---------|-------------|
| `cat` | Concatena archivos, lee stdin |
| `sort` | Ordena líneas de texto |
| `uniq` | Reporta/omite líneas repetidas |
| `grep` | Busca patrones en texto |
| `wc` | Cuenta líneas, palabras, bytes |
| `head` | Muestra primeras líneas |
| `tail` | Muestra últimas líneas |
| `tee` | Lee stdin, escribe a stdout y archivos |

---

## Ver También

- [[wiki/linux/05-trabajo-con-comandos.md|Capítulo 5: Trabajo con Comandos]] — Identificar y documentar comandos a componer
- [[wiki/linux/07-expansiones.md|Capítulo 7: Expansiones — Cómo el Shell Ve el Mundo]] — Entender cómo el shell procesa los argumentos redirigidos

---

## Filosofía: Linux es Composición

El verdadero poder de Linux no está en comandos individuales, sino en cómo se pueden **componer** usando redirección y pipelines. Casi todos los comandos de línea de comandos usan entrada/salida estándar y pueden trabajar con casi cualquier otro comando. Esta es la esencia de la filosofía Unix: herramientas pequeñas, especializadas, que hacen una cosa bien y se comunican a través de texto.

> "**When I am asked to explain the difference between Windows and Linux, I often use a toy analogy. Windows is like a Game Boy. You go to the store and buy one off the shelf. You take it home, turn it on, and play with it. Pretty graphics, cute sounds. After a while, though, you get tired of the game that came with it, so you go back to the store and buy another one. This cycle repeats over and over. Finally, you go back to the store and say to the person behind the counter, "I want a game that does this!" only to be told that no such game exists because there is no "market demand" for it.
>
> Linux, on the other hand, is like the world's largest Erector Set. You open it up and it's just a huge collection of parts. There's a lot of steel struts, screws, nuts, gears, pulleys, and a few suggestions on what to build. So you start to play with it. You build one of the suggestions and then another. After a while you discover that you have your own ideas of what to make. You don't ever have to go back to the store, as you already have everything you need. The Erector Set takes on the shape of your imagination."

---

*Fuente: The Linux Command Line - A Complete Introduction (William E. Shotts, Jr.)*
