---
title: "Capítulo 20: Procesamiento de Texto"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 20"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-289-323.pdf"
updated: "2026-06-23"
---

# Procesamiento de Texto

Los sistemas Unix/Linux dependen enormemente de archivos de texto para almacenar datos. Este capítulo cubre las herramientas para "cortar y picar" texto en la línea de comandos. Los comandos tratados son:

| Comando | Descripción |
|---------|-------------|
| `cat`   | Concatenar archivos e imprimir en stdout |
| `sort`  | Ordenar líneas de archivos de texto |
| `uniq`  | Reportar u omitir líneas repetidas |
| `cut`   | Eliminar secciones de cada línea |
| `paste` | Combinar líneas de archivos |
| `join`  | Unir líneas de dos archivos por campo común |
| `tac`   | Concatenar e imprimir archivos en orden inverso |
| `rev`   | Invertir caracteres de cada línea |
| `comm`  | Comparar dos archivos ordenados línea por línea |
| `diff`  | Comparar archivos línea por línea |
| `patch` | Aplicar un diff a un archivo original |
| `tr`    | Traducir o eliminar caracteres |
| `sed`   | Editor de flujo para filtrar y transformar texto |
| `aspell`| Corrector ortográfico interactivo |

## Aplicaciones del Texto

El texto en Unix se usa para: documentos (incluyendo Markdown y lenguajes de marcado), páginas web (HTML/XML), correo electrónico, salida de impresora (PostScript) y código fuente de programas.

---

## Comandos Revisitados

### `cat` — Concatenar y visualizar

`cat` tiene opciones útiles para **visualizar** caracteres especiales y **modificar** el texto:

| Opción | Descripción |
|--------|-------------|
| `-A`   | Muestra caracteres no imprimibles: tabs como `^I`, fin de línea como `$` |
| `-n`   | Numera todas las líneas |
| `-s`   | Suprime líneas en blanco múltiples consecutivas (las colapsa en una) |

**Ejemplo — detectar tabs y espacios finales:**

```bash
[me@linuxbox ~]$ cat > foo.txt
	The quick brown fox jumps over the lazy dog.   
[me@linuxbox ~]$ cat -A foo.txt
^IThe quick brown fox jumps over the lazy dog.   $
```

- `^I` = carácter tab (notación `Ctrl-I`)
- `$` = verdadero fin de línea (los espacios antes del `$` son espacios finales visibles)

**Ejemplo — numerar y compactar líneas en blanco:**

```bash
[me@linuxbox ~]$ cat -ns foo.txt
     1	The quick brown fox
     2
     3	jumps over the lazy dog.
```

> **MS-DOS vs Unix**: Unix termina líneas con linefeed (ASCII 10); MS-DOS usa carriage return + linefeed (ASCII 13 + 10). Para convertir archivos DOS a Unix: `dos2unix` y `unix2dos`. También se puede usar `tr` (ver más abajo).

---

### `sort` — Ordenar líneas

`sort` ordena el contenido de stdin o de uno o más archivos y envía el resultado a stdout. Puede **combinar múltiples archivos** en un único resultado ordenado:

```bash
sort file1.txt file2.txt file3.txt > final_sorted_list.txt
```

**Opciones comunes:**

| Opción | Opción larga | Descripción |
|--------|--------------|-------------|
| `-b` | `--ignore-leading-blanks` | Ignorar espacios iniciales al ordenar |
| `-f` | `--ignore-case` | Ordenar sin distinguir mayúsculas/minúsculas |
| `-n` | `--numeric-sort` | Ordenar numéricamente (no lexicográficamente) |
| `-r` | `--reverse` | Orden descendente |
| `-k` | `--key=campo1[,campo2]` | Ordenar por campo clave |
| `-m` | `--merge` | Combinar archivos ya ordenados sin reordenar |
| `-o` | `--output=archivo` | Enviar salida a archivo en vez de stdout |
| `-t` | `--field-separator=car` | Definir carácter separador de campos |

**Ejemplo — listar los 10 directorios que más espacio usan:**

```bash
[me@linuxbox ~]$ du -s /usr/share/* | sort -nr | head
509940   /usr/share/locale-langpack
242660   /usr/share/doc
197560   /usr/share/fonts
...
```

#### Ordenar por campos (datos tabulares)

`sort` trata el whitespace (espacios y tabs) como delimitadores entre **campos**. Para la salida de `ls -l`, los 8 campos son: permisos, links, usuario, grupo, tamaño, mes, día, hora y nombre.

**Ordenar por tamaño de archivo (campo 5, numérico, reverso):**

```bash
ls -l /usr/bin | sort -nrk 5 | head
```

#### Claves múltiples y offsets

Dado un archivo `distros.txt` con campos: distribución, versión, fecha (MM/DD/YYYY):

```bash
# Ordenar alfabéticamente por distro y numéricamente por versión
sort --key=1,1 --key=2n distros.txt

# Ordenar cronológicamente por fecha MM/DD/YYYY usando offsets
# (campo 3, carácter 7 = año; campo 3.1 = mes; campo 3.4 = día)
sort -k 3.7nbr -k 3.1nbr -k 3.4nbr distros.txt
```

- `1,1` significa "empieza en campo 1, termina en campo 1"
- `2n` significa "campo 2, orden numérico"
- Las letras de opción al final de `-k` son iguales que las globales: `b`, `n`, `r`, etc.

**Separador personalizado** (para `/etc/passwd` separado por `:`):

```bash
sort -t ':' -k 7 /etc/passwd | head
```

---

### `uniq` — Eliminar duplicados

`uniq` recibe una entrada **ya ordenada** y elimina líneas duplicadas adyacentes. Se usa frecuentemente junto con `sort`:

```bash
sort foo.txt | uniq
```

> ⚠️ Si el archivo no está ordenado, `uniq` solo elimina duplicados **consecutivos**.

**Opciones:**

| Opción | Opción larga | Descripción |
|--------|--------------|-------------|
| `-c` | `--count` | Antepone el número de ocurrencias a cada línea |
| `-d` | `--repeated` | Solo muestra líneas duplicadas |
| `-f n` | `--skip-fields=n` | Ignora los primeros n campos |
| `-i` | `--ignore-case` | Ignora mayúsculas/minúsculas |
| `-s n` | `--skip-chars=n` | Ignora los primeros n caracteres |
| `-u` | `--unique` | Solo muestra líneas únicas (sin duplicados) |

**Ejemplo — contar duplicados:**

```bash
[me@linuxbox ~]$ sort foo.txt | uniq -c
      2 a
      2 b
      2 c
```

> **Tip**: La versión GNU de `sort` soporta `-u` para eliminar duplicados directamente sin necesitar `uniq`.

---

## Cortar y Combinar Columnas

### `cut` — Extraer secciones de líneas

`cut` extrae una sección de texto de cada línea y la envía a stdout.

**Opciones:**

| Opción | Opción larga | Descripción |
|--------|--------------|-------------|
| `-c lista` | `--characters=list` | Extrae los caracteres en las posiciones indicadas |
| `-f lista` | `--fields=list` | Extrae los campos indicados |
| `-d delim` | `--delimiter=delim` | Con `-f`, usa `delim` como separador (por defecto: tab) |
| N/A | `--complement` | Extrae todo **excepto** lo especificado |

> `cut` es inflexible: funciona mejor con archivos generados por otros programas (delimitados por tab o posiciones fijas), no con texto escrito a mano.

**Verificar que un archivo usa tabs** (con `cat -A`):

```bash
[me@linuxbox ~]$ cat -A distros.txt
SUSE^I10.2^I12/07/2006$
Fedora^I10^I11/25/2008$
```

**Extraer el tercer campo (fecha) de `distros.txt`:**

```bash
[me@linuxbox ~]$ cut -f 3 distros.txt
12/07/2006
11/25/2008
...
```

**Extraer caracteres 7-10 (el año) del campo de fecha:**

```bash
[me@linuxbox ~]$ cut -f 3 distros.txt | cut -c 7-10
2006
2008
...
```

**Extraer el primer campo de `/etc/passwd`** (separado por `:`):

```bash
[me@linuxbox ~]$ cut -d ':' -f 1 /etc/passwd | head
root
daemon
bin
...
```

> **`expand` y `unexpand`**: Si el archivo usa espacios en lugar de tabs, `expand distros.txt` convierte tabs a espacios; `unexpand` hace lo contrario. Útil para usar `cut -c` en archivos originalmente tabulados.

---

### `paste` — Combinar columnas

`paste` hace lo contrario de `cut`: agrega columnas a un archivo leyendo múltiples archivos y combinando sus campos línea por línea.

**Ejemplo práctico** — crear lista cronológica de distros:

```bash
# Paso 1: ordenar por fecha
sort -k 3.7nbr -k 3.1nbr -k 3.4nbr distros.txt > distros-by-date.txt

# Paso 2: extraer nombre+versión y fechas en archivos separados
cut -f 1,2 distros-by-date.txt > distros-versions.txt
cut -f 3 distros-by-date.txt > distros-dates.txt

# Paso 3: combinar con paste (fecha primero, luego nombre+versión)
paste distros-dates.txt distros-versions.txt
```

```
11/25/2008   Fedora   10
10/30/2008   Ubuntu   8.10
06/19/2008   SUSE     11.0
...
```

---

### `join` — Unir archivos por campo clave

`join` es como `paste` pero combina archivos basándose en un **campo clave compartido** (operación de base de datos relacional). Los archivos **deben estar ordenados** por el campo clave.

**Ejemplo** — dos archivos con campo clave "fecha de lanzamiento":

```bash
# distros-key-names.txt: fecha | nombre
# distros-key-vernums.txt: fecha | versión

join distros-key-names.txt distros-key-vernums.txt | head
```

```
11/25/2008 Fedora 10
10/30/2008 Ubuntu 8.10
06/19/2008 SUSE 11.0
...
```

> Por defecto, `join` usa whitespace como delimitador de entrada y un espacio como delimitador de salida. Ver `man join` para opciones de personalización.

---

## `tac` y `rev` — Invertir texto

### `tac` — Invertir orden de líneas

`tac` funciona como `cat` pero en sentido inverso: concatena archivos imprimiéndolos desde la última línea hasta la primera. Útil para reordenar logs (que suelen mostrar la entrada más reciente al final):

```bash
[me@linuxbox ~]$ tail /var/log/backup.log | tac
# Muestra las entradas más recientes arriba
Fri Aug 16 07:44:11 AM EDT 2025: backup of linuxbox finished.
Fri Aug 16 07:37:57 AM EDT 2025: backup of linuxbox started.
...
```

### `rev` — Invertir caracteres de cada línea

`rev` invierte el orden de los caracteres en cada línea:

```bash
[me@linuxbox ~]$ echo "This is a test." | rev
.tset a si sihT
```

**Uso práctico** — eliminar el último carácter de cada línea (por ejemplo, el `.` final de un log):

```bash
tail /var/log/backup.log | rev | cut -c 2- | rev
```

La técnica: invertir → mover el carácter final al inicio → `cut -c 2-` lo elimina → invertir de nuevo.

---

## Comparación de Texto

### `comm` — Comparar archivos ordenados

`comm` compara dos archivos de texto **ya ordenados** y produce tres columnas:
1. Líneas únicas del primer archivo
2. Líneas únicas del segundo archivo
3. Líneas comunes a ambos

```bash
[me@linuxbox ~]$ comm file1.txt file2.txt
a           # solo en file1
        b   # común
        c   # común
        d   # común
    e       # solo en file2
```

**Suprimir columnas** con `-1`, `-2`, `-3`:

```bash
# Mostrar solo las líneas comunes (suprimir columnas 1 y 2)
comm -12 file1.txt file2.txt
```

---

### `diff` — Comparar diferencias entre archivos

`diff` es más complejo que `comm`: detecta diferencias entre archivos y soporta múltiples formatos de salida. Puede examinar directorios recursivamente (árboles de código fuente).

#### Formato por defecto (normal)

```bash
[me@linuxbox ~]$ diff file1.txt file2.txt
1d0
< a
4a4
> e
```

Cada grupo de cambios va precedido de un **change command** con la forma `rango operación rango`:

| Cambio | Descripción |
|--------|-------------|
| `r1ar2` | Añadir líneas en posición r2 (del archivo 2) a posición r1 (del archivo 1) |
| `r1cr2` | Cambiar (reemplazar) líneas en r1 con líneas en r2 |
| `r1dr2` | Eliminar líneas en r1 del archivo 1 |

#### Formato de contexto (`-c`)

Más detallado, muestra líneas de contexto alrededor de los cambios:

```bash
[me@linuxbox ~]$ diff -c file1.txt file2.txt
*** file1.txt    2025-12-23 06:40:13...
--- file2.txt    2025-12-23 06:40:34...
***************
*** 1,4 ****
- a
  b
  c
  d
--- 1,4 ----
  b
  c
  d
+ e
```

Indicadores dentro de los grupos:

| Indicador | Significado |
|-----------|-------------|
| (en blanco) | Línea de contexto, no indica diferencia |
| `-` | Línea eliminada (solo en el primer archivo) |
| `+` | Línea añadida (solo en el segundo archivo) |
| `!` | Línea cambiada (se muestran ambas versiones) |

#### Formato unificado (`-u`)

Más conciso que el de contexto, elimina las líneas de contexto duplicadas:

```bash
[me@linuxbox ~]$ diff -u file1.txt file2.txt
--- file1.txt    2025-12-23 06:40:13...
+++ file2.txt    2025-12-23 06:40:34...
@@ -1,4 +1,4 @@
-a
 b
 c
 d
+e
```

Indicadores:

| Carácter | Significado |
|----------|-------------|
| (en blanco) | Línea compartida por ambos archivos |
| `-` | Línea eliminada del primer archivo |
| `+` | Línea añadida al primer archivo |

---

### `patch` — Aplicar diferencias a archivos

`patch` aplica archivos `diff` a versiones anteriores de archivos para actualizarlos. Es el mecanismo que usa el kernel de Linux: en lugar de distribuir el árbol completo con cada cambio, se envía un archivo diff.

**Ventajas del flujo diff/patch:**
- El archivo diff es pequeño comparado con el código fuente completo
- El diff muestra claramente qué cambia, facilitando la revisión

**Flujo de trabajo:**

```bash
# Crear el diff (formato recomendado por GNU: -Naur)
diff -Naur archivo_viejo archivo_nuevo > parchefile.txt

# Aplicar el parche
patch < parchefile.txt
```

**Ejemplo:**

```bash
[me@linuxbox ~]$ diff -Naur file1.txt file2.txt > patchfile.txt
[me@linuxbox ~]$ patch < patchfile.txt
patching file file1.txt
[me@linuxbox ~]$ cat file1.txt
b
c
d
e
```

> No es necesario especificar el archivo a parchear: el diff en formato unificado ya contiene los nombres de archivo en el encabezado.

---

## Edición al Vuelo

### `tr` — Traducir caracteres

`tr` **transliteriza** caracteres: convierte un conjunto de caracteres en otro. Opera sobre stdin y produce stdout.

**Sintaxis:** `tr conjunto1 conjunto2`

Los conjuntos se pueden expresar como:
- Lista enumerada: `ABCDEFGHIJKLMNOPQRSTUVWXYZ`
- Rango: `A-Z` (puede verse afectado por locale, usar con cuidado)
- Clases POSIX: `[:upper:]`, `[:lower:]`, `[:digit:]`, etc.

**Ejemplos:**

```bash
# Convertir minúsculas a mayúsculas
echo "lowercase letters" | tr a-z A-Z
# → LOWERCASE LETTERS

# Usando clases POSIX
echo "lowercase letters" | tr [:lower:] A
# → AAAAAAAAA AAAAAAA

# Eliminar carriage returns (convertir texto DOS a Unix)
tr -d '\r' < dos_file > unix_file

# Squeeze: eliminar repeticiones consecutivas de caracteres del set
echo "aaabbbccc" | tr -s ab
# → abccc  (colapsa 'a' y 'b' repetidas, 'c' queda igual por no estar en el set)
```

> **ROT13**: Un cifrado trivial por sustitución. Cada carácter se desplaza 13 posiciones en el alfabeto. Aplicado dos veces restaura el original:
> ```bash
> echo "secret text" | tr a-zA-Z n-za-mN-ZA-M
> # → frperg grkg
> echo "frperg grkg" | tr a-zA-Z n-za-mN-ZA-M
> # → secret text
> ```

---

### `sed` — Editor de flujo (stream editor)

`sed` (stream editor) edita texto en un flujo: puede recibir una lista de archivos o stdin. Es potente y extenso; este capítulo cubre lo esencial.

`sed` recibe un **comando de edición** (en la línea de comandos) o un **script** (archivo con múltiples comandos), y los ejecuta sobre cada línea del flujo.

**Ejemplo básico — sustitución:**

```bash
echo "front" | sed 's/front/back/'
# → back
```

#### Estructura de los comandos sed

Los comandos sed comienzan con una letra. El comando de sustitución `s` va seguido de cadenas separadas por un delimitador (por convención `/`, pero puede ser cualquier carácter que siga inmediatamente al comando):

```bash
echo "front" | sed 's_front_back_'   # usa _ como delimitador
```

#### Direcciones (addresses)

Los comandos pueden ir precedidos de una **dirección** para especificar qué líneas editar. Sin dirección, se aplica a todas las líneas.

| Dirección | Descripción |
|-----------|-------------|
| `n` | Número de línea (entero positivo) |
| `$` | Última línea |
| `/regexp/` | Líneas que coincidan con la expresión regular |
| `addr1,addr2` | Rango de líneas, inclusivo |
| `first~step` | La línea `first` y luego cada `step` líneas (`1~2` = impares, `5~5` = cada quinta desde la 5ª) |
| `addr1,+n` | `addr1` y las siguientes n líneas |
| `addr!` | Todas las líneas **excepto** las que coincidan con `addr` |

**Ejemplos con `distros.txt`:**

```bash
# Imprimir líneas 1-5 (-n suprime auto-print)
sed -n '1,5p' distros.txt

# Solo las líneas que contienen SUSE
sed -n '/SUSE/p' distros.txt

# Todas las líneas EXCEPTO las de SUSE
sed -n '/SUSE/!p' distros.txt
```

#### Comandos básicos de edición

| Comando | Descripción |
|---------|-------------|
| `=`     | Imprime el número de línea actual |
| `a`     | Añade texto después de la línea actual |
| `d`     | Elimina la línea actual |
| `i`     | Inserta texto antes de la línea actual |
| `p`     | Imprime la línea actual (útil con `-n`) |
| `q`     | Sale sin procesar más líneas; imprime la línea si no se usó `-n` |
| `Q`     | Sale sin procesar más líneas ni imprimir |
| `s/regexp/repl/` | Sustituye `regexp` por `repl` en la línea actual |
| `y/set1/set2` | Transliteración de caracteres de `set1` a `set2` (a diferencia de `tr`, requiere conjuntos de igual longitud; no soporta rangos ni clases POSIX) |

#### El comando `s` — sustitución en detalle

- **Flag `g`**: aplica la sustitución globalmente en la línea (no solo a la primera ocurrencia):
  ```bash
  echo "aaabbbccc" | sed 's/b/B/'    # → aaaBbbccc (solo la primera b)
  echo "aaabbbccc" | sed 's/b/B/g'   # → aaaBBBccc (todas las b)
  ```

- **Back references (`\1` a `\9`)**: referencian subexpresiones (grupos entre `\(` y `\)` en BRE) en el replacement:
  ```bash
  # Convertir fecha MM/DD/YYYY → YYYY-MM-DD
  sed 's/\([0-9]\{2\}\)\/\([0-9]\{2\}\)\/\([0-9]\{4\}\)$/\3-\1-\2/' distros.txt
  ```
  Desglose: `\1`=mes, `\2`=día, `\3`=año → resultado: `\3-\1-\2`

- **`&`**: en el replacement, representa el texto completo coincidente con `regexp`.

#### Scripts sed con `-f`

Para comandos complejos o múltiples, se puede guardar un script:

```bash
# Archivo distros.sed
# sed script to produce Linux distributions report

1 i\
\
Linux Distributions Report\

s/\([0-9]\{2\}\)\/\([0-9]\{2\}\)\/\([0-9]\{4\}\)$/\3-\1-\2/
y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/
```

```bash
sed -f distros.sed distros.txt
```

Resultado: inserta un encabezado, convierte fechas a YYYY-MM-DD y pone todo en mayúsculas.

**Notas sobre scripts:**
- Los comentarios empiezan con `#`
- El comando `i` usa `\` + salto de línea como **line-continuation character** para incluir texto multilínea
- El comando `y` (transliteración) no soporta rangos `[a-z]` ni clases POSIX `[:lower:]`; hay que enumerar todos los caracteres

**Edición in-place** con `-i` (reescribe el archivo):

```bash
sed -i 's/lazy/laxy/; s/jumps/jimps/' foo.txt
```

> Los que aprecian `sed` también suelen usar **`awk`** (procesamiento de datos tabulares) y **`perl`** (lenguaje completo para tareas complejas de administración y web).

---

### `aspell` — Corrector ortográfico interactivo

`aspell` es un corrector ortográfico interactivo, sucesor de `ispell`. Puede verificar varios tipos de archivos: prosa, HTML, C/C++, correos electrónicos, etc.

**Sintaxis básica:**

```bash
aspell check textfile
```

**Interfaz interactiva:**

Al ejecutar `aspell check`, aparece una pantalla que muestra:
- La palabra sospechosa resaltada en la parte superior
- Hasta 10 sugerencias numeradas (0-9)
- Opciones de acción: `i` (Ignore), `I` (Ignore all), `r` (Replace), `R` (Replace all), `a` (Add), `l` (Add Lower), `b` (Abort), `x` (Exit)

**Comportamiento por defecto:** crea un archivo de backup con extensión `.bak` antes de modificar el original.

```bash
# Para no crear backup
aspell --dont-backup check textfile
```

**Modo HTML** — ignora las etiquetas HTML, solo revisa el texto:

```bash
aspell -H check foo.txt
```

- Sin `-H`: aspell trata las etiquetas HTML como palabras mal escritas
- Con `-H`: ignora el contenido de las etiquetas (pero sí revisa atributos como `ALT`)

> Por defecto, `aspell` ignora URLs y direcciones de correo electrónico. Este comportamiento se puede cambiar con opciones de línea de comandos. Ver `man aspell` para detalles.

---

## Resumen

Este capítulo cubre las herramientas fundamentales de procesamiento de texto en Linux:

- **Visualización y limpieza**: `cat -A/-n/-s` para detectar caracteres ocultos
- **Ordenamiento**: `sort` con claves múltiples, offsets, separadores personalizados
- **Deduplicación**: `uniq` (siempre después de `sort`)
- **Extracción de columnas**: `cut` por caracteres o campos; `expand/unexpand` para preparar datos
- **Combinación de columnas**: `paste` (columnas en paralelo), `join` (por campo clave compartido)
- **Inversión**: `tac` (líneas), `rev` (caracteres)
- **Comparación**: `comm` (líneas comunes/únicas entre dos archivos ordenados), `diff` (formatos normal, contexto `-c`, unificado `-u`)
- **Aplicación de parches**: `patch` con archivos diff generados por `diff -Naur`
- **Edición de caracteres**: `tr` (transliteración, eliminación, squeeze)
- **Edición de flujo**: `sed` (sustituciones, direcciones, back references, scripts, `-i` in-place)
- **Corrección ortográfica**: `aspell` con soporte para HTML y múltiples tipos de archivo

Estas herramientas se vuelven especialmente poderosas en **shell scripting**, donde se combinan mediante pipes para resolver problemas prácticos de procesamiento de datos.

> **Crédito extra**: `split` (dividir archivos), `csplit` (dividir por contexto), `sdiff` (merge lado a lado de diferencias).

---

## Ver También

- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] — Pipelines y filtros fundamentales
- [[wiki/linux/19-expresiones-regulares.md|Capítulo 19: Expresiones Regulares]] — Patrones usados en `sed`, `grep` y `diff`
- [[wiki/linux/17-busqueda-archivos.md|Capítulo 17: Búsqueda de Archivos]] — `find` y `grep` para localizar y filtrar
