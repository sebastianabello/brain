---
title: "Capítulo 30: Troubleshooting (Resolución de Problemas en Scripts)"
sources:
  - "The Linux Command Line: A Complete Introduction, Ch. 30"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-433-446.pdf"
updated: "2026-06-23"
---

# Troubleshooting: Resolución de Problemas en Scripts Bash

A medida que los scripts se vuelven más complejos, es inevitable encontrarse con errores. Este capítulo cubre los tipos de errores más comunes en scripts y las técnicas para detectarlos y eliminarlos.

---

## Errores Sintácticos

Los **errores sintácticos** (*syntactic errors*) implican escribir incorrectamente algún elemento de la sintaxis del shell. El shell detiene la ejecución del script cuando encuentra este tipo de error.

Script de demostración (funciona correctamente):

```bash
#!/bin/bash
# trouble: script para demostrar errores comunes

number=1

if [ $number = 1 ]; then
    echo "Number is equal to 1."
else
    echo "Number is not equal to 1."
fi
```

### Comillas Faltantes (*Missing Quotes*)

Si eliminamos la comilla de cierre del primer `echo`:

```bash
echo "Number is equal to 1.   # ← comilla faltante
```

El error que obtenemos **no apunta a la línea del problema**, sino mucho más tarde:

```
/home/me/bin/trouble: line 10: unexpected EOF while looking for matching `"'
/home/me/bin/trouble: line 13: syntax error: unexpected end of file
```

**Por qué ocurre esto:** bash continúa buscando la comilla de cierre hasta que la encuentra (justo después del segundo `echo`). Después de eso, el shell queda muy confundido y la sintaxis del `if` subsiguiente se rompe porque el `fi` ya está dentro de un string entrecomillado (pero abierto).

> **Tip:** En scripts largos, un editor con *syntax highlighting* ayuda enormemente. En vim, habilitar con:
> ```
> :syntax on
> ```

### Tokens Faltantes o Inesperados (*Missing or Unexpected Tokens*)

Otro error común es olvidar completar un comando compuesto, por ejemplo, eliminar el punto y coma (`;`) antes del `then`:

```bash
if [ $number = 1 ] then    # ← falta el ;
```

Error resultante:

```
/home/me/bin/trouble: line 9: syntax error near unexpected token `else'
/home/me/bin/trouble: line 9: `else'
```

**Por qué ocurre:** `[` acepta una lista de comandos y evalúa el exit status del último. Sin el `;`, la palabra `then` se agrega a la lista de argumentos de `[` (lo que es sintácticamente válido). El `echo` siguiente también es legal. Cuando el shell llega a `else`, no sabe qué hacer con él porque es una **reserved word** (palabra reservada) fuera de contexto.

### Expansiones No Anticipadas (*Unanticipated Expansions*)

Si la variable `number` está vacía:

```bash
number=    # variable vacía
if [ $number = 1 ]; then
```

Al expandirse, el comando queda:

```bash
[ = 1 ]    # inválido: falta el lado izquierdo del operador =
```

Error:

```
/home/me/bin/trouble: line 7: [: =: unary operator expected
```

**Solución:** Siempre encerrar variables entre comillas dobles:

```bash
[ "$number" = 1 ]
# Con variable vacía queda: [ "" = 1 ]  ← válido
```

> **Regla de oro:** Siempre encerrar variables y sustituciones de comandos en comillas dobles, a menos que sea necesario el *word splitting*.

---

## Errores Lógicos

A diferencia de los errores sintácticos, los **errores lógicos** (*logical errors*) no impiden que el script se ejecute, pero producen resultados incorrectos por un problema en la lógica. Tipos comunes:

| Tipo | Descripción |
|------|-------------|
| **Expresiones condicionales incorrectas** | Codificar mal un `if/then/else`, con lógica invertida o incompleta |
| **Errores de "off by one"** | En bucles con contadores, empezar en el valor incorrecto (0 vs 1) o terminar una iteración demasiado pronto/tarde |
| **Situaciones no anticipadas** | Datos o condiciones no previstas por el programador (como expansiones inesperadas de variables) |

---

## Programación Defensiva

Es importante **verificar los supuestos** al programar: evaluar cuidadosamente el exit status de programas y comandos.

### El Problema Clásico: `cd` + `rm`

Este fragmento es peligroso:

```bash
cd $dir_name
rm *
```

Si `cd` falla (porque `dir_name` no existe), `rm *` elimina los archivos del **directorio de trabajo actual**. Hay tres niveles de protección:

**Nivel 1:** Usar `&&` para que `rm` solo se ejecute si `cd` tuvo éxito:

```bash
cd "$dir_name" && rm *
```

**Nivel 2:** Verificar además que el directorio existe:

```bash
[[ -d "$dir_name" ]] && cd "$dir_name" && rm *
```

**Nivel 3 (recomendado):** Script completo con manejo de errores explícito:

```bash
# Eliminar archivos en el directorio $dir_name
if [[ ! -d "$dir_name" ]]; then
    echo "No such directory: '$dir_name'" >&2
    exit 1
fi
if ! cd "$dir_name"; then
    echo "Cannot cd to '$dir_name'" >&2
    exit 1
fi
if ! rm *; then
    echo "File deletion failed. Check results" >&2
    exit 1
fi
```

Cada fallo envía un mensaje descriptivo a **stderr** (`>&2`) y termina el script con exit status 1.

### `set -e`, `set -u` y `set -o pipefail`

Bash ofrece estas opciones para intentar manejar errores automáticamente:

- **`set -e`**: termina el script si cualquier comando retorna exit status no-cero
- **`set -u`**: termina si se usa una variable no inicializada
- **`set -o pipefail`**: termina si el elemento final de un pipeline falla

**⚠️ No se recomienda usarlas.** El Bash FAQ #105 advierte que `set -e` fue un intento de agregar "detección automática de errores", pero tiene reglas extremadamente complicadas y cambiantes entre versiones de bash (especialmente en torno a `if`, tests y pipelines). Es mejor diseñar manejo de errores explícito en lugar de depender de estas opciones.

### ShellCheck: Tu Amigo

**ShellCheck** es un programa disponible en la mayoría de repositorios que analiza scripts y detecta muchos tipos de fallos y malas prácticas:

```bash
# Para scripts con shebang
shellcheck my_script

# Para librerías de funciones sin shebang (especificar dialecto con -s)
shellcheck -s bash my_library
```

Más información: https://www.shellcheck.net

### Cuidado con los Nombres de Archivos

Unix permite casi cualquier carácter en los nombres de archivos. Solo están prohibidos:
- `/` (separador de ruta)
- El carácter nulo (byte cero)

Esto incluye espacios, tabs, saltos de línea, guiones iniciales, retornos de carro, etc.

**El peligro de los guiones iniciales:** Un archivo llamado `-rf ~` pasado a `rm` se interpreta como opciones del comando, no como un nombre de archivo.

**Solución:** Usar `./` antes de los wildcards:

```bash
# Peligroso
rm *

# Seguro: previene que nombres con guión inicial sean interpretados como opciones
rm ./*
```

> **Regla general:** Siempre preceder wildcards como `*` y `?` con `./` para evitar malinterpretaciones.

#### Nombres de Archivo Portables (POSIX)

El **POSIX Portable Filename Character Set** define los únicos caracteres que garantizan portabilidad entre sistemas:

- Letras mayúsculas: `A`–`Z`
- Letras minúsculas: `a`–`z`
- Dígitos: `0`–`9`
- Punto (`.`), guión (`-`), guión bajo (`_`)
- **Los nombres no deben comenzar con guión**

### Validación del Input

Una regla del buen programar es que si un programa acepta input, debe poder manejar cualquier cosa que reciba. El input debe filtrarse para aceptar solo datos válidos.

Ejemplo de validación de una selección de menú (acepta solo números del 0 al 3):

```bash
[[ $REPLY =~ ^[0-3]$ ]]
```

Este test retorna exit status cero **solo si** el string es un número en el rango de 0 a 3. Nada más es aceptado.

---

## Testing (Pruebas)

En el mundo open source existe el dicho *"release early, release often"* (lanza temprano, lanza a menudo). La experiencia muestra que los bugs son más fáciles y baratos de encontrar cuanto antes se detectan en el ciclo de desarrollo.

### Hacer las Pruebas Seguras

Para probar código potencialmente peligroso (como eliminación de archivos), se modifica el código para hacerlo seguro durante las pruebas:

```bash
if [[ -d $dir_name ]]; then
    if cd $dir_name; then
        echo rm * # TESTING ← echo en lugar de rm
    else
        echo "cannot cd to '$dir_name'" >&2
        exit 1
    fi
else
    echo "no such directory: '$dir_name'" >&2
    exit 1
fi
exit # TESTING ← termina el script antes de otras partes
```

Cambios clave:
- Reemplazar el `rm` real por `echo rm` para mostrar qué se eliminaría sin hacerlo
- Agregar `exit # TESTING` al final del fragmento para impedir que el resto del script se ejecute
- Marcar con comentarios `# TESTING` para facilitar encontrar y eliminar los cambios al terminar

### Casos de Prueba (*Test Cases*)

Para realizar pruebas útiles, es importante desarrollar y aplicar buenos *test cases* que cubran los **casos límite y extremos** (*edge and corner cases*).

Para el fragmento de eliminación de archivos, los casos relevantes son:

| Condición | Descripción |
|-----------|-------------|
| `dir_name` = directorio existente | Caso normal, debería funcionar |
| `dir_name` = directorio inexistente | Debe detectar el error y salir |
| `dir_name` vacío | Caso extremo peligroso |

Al probar con cada una de estas condiciones se logra una buena **cobertura de pruebas** (*test coverage*).

> **Nota de diseño:** El testing (como el diseño) es función del tiempo disponible. No todo necesita pruebas extensas, pero el código potencialmente destructivo merece atención especial tanto en diseño como en pruebas.

---

## Debugging (Depuración)

Cuando las pruebas revelan un problema, el siguiente paso es la depuración. Un "problema" generalmente significa que el script no se comporta como el programador espera.

### Encontrar el Área del Problema

En scripts largos, es útil aislar el área relacionada con el problema mediante **comentar secciones** para desactivarlas temporalmente:

```bash
if [[ -d $dir_name ]]; then
    if cd $dir_name; then
            rm *
    else
        echo "cannot cd to '$dir_name'" >&2
        exit 1
    fi
# else                                           ← sección comentada
#     echo "no such directory: '$dir_name'" >&2
#     exit 1
fi
```

Al comentar una sección lógica y volver a probar, podemos determinar si esa sección está relacionada con el error.

### Tracing (Rastreo)

Los bugs suelen ser casos de flujo lógico inesperado: partes del script que nunca se ejecutan, o que se ejecutan en el orden o momento incorrecto.

**Método 1:** Insertar mensajes `echo` informativos que se envían a stderr:

```bash
echo "preparing to delete files" >&2
if [[ -d $dir_name ]]; then
    if cd $dir_name; then
echo "deleting files" >&2
        rm *
    else
        echo "cannot cd to '$dir_name'" >&2
        exit 1
    fi
else
    echo "no such directory: '$dir_name'" >&2
    exit 1
fi
echo "file deletion complete" >&2
```

> Los mensajes de tracing se envían a **stderr** para separarlos del output normal y porque no se indentan (facilita encontrarlos y eliminarlos después).

**Método 2: `bash -x`** — activa el tracing para todo el script agregando `-x` al shebang:

```bash
#!/bin/bash -x
```

Con tracing activo, cada comando se muestra con sus expansiones aplicadas, precedido por `+`:

```
[me@linuxbox ~]$ trouble
+ number=1
+ '[' 1 = 1 ']'
+ echo 'Number is equal to 1.'
Number is equal to 1.
```

El `+` es el carácter predeterminado del prefijo, controlado por la variable **`PS4`**.

**Personalizar PS4 para incluir número de línea:**

```bash
[me@linuxbox ~]$ export PS4='$LINENO + '
[me@linuxbox ~]$ trouble
5 + number=1
7 + '[' 1 = 1 ']'
8 + echo 'Number is equal to 1.'
Number is equal to 1.
```

> Nota: usar comillas simples en `export PS4='$LINENO + '` para prevenir la expansión hasta que el prompt sea usado.

**Método 3: `set -x` / `set +x`** — activar/desactivar tracing solo para porciones específicas del script:

```bash
#!/bin/bash

number=1

set -x # Activar tracing
if [ $number = 1 ]; then
    echo "Number is equal to 1."
else
    echo "Number is not equal to 1."
fi
set +x # Desactivar tracing
```

Esto permite examinar múltiples porciones problemáticas de un script sin ver el tracing completo.

### Examinar Valores Durante la Ejecución

Además del tracing, es útil mostrar el contenido de variables para ver el funcionamiento interno del script. Se agregan sentencias `echo` con comentarios `# DEBUG`:

```bash
#!/bin/bash

number=1

echo "number=$number" # DEBUG
set -x # Activar tracing
if [ $number = 1 ]; then
    echo "Number is equal to 1."
else
    echo "Number is not equal to 1."
fi
set +x # Desactivar tracing
```

Marcar las líneas de debug con `# DEBUG` facilita encontrarlas y eliminarlas cuando el debugging esté completo. Esta técnica es especialmente útil para observar el comportamiento de bucles y expresiones aritméticas.

---

## Resumen de Técnicas

| Técnica | Cuándo Usar |
|---------|-------------|
| Comillas en variables `"$var"` | Siempre; previene expansiones inesperadas |
| `&&` para encadenar comandos | Cuando el segundo comando depende del éxito del primero |
| Verificación explícita de condiciones + `exit 1` | Scripts de producción con operaciones peligrosas |
| `shellcheck` | Antes de desplegar cualquier script |
| `rm ./*` en lugar de `rm *` | Siempre que se usen wildcards con `rm` |
| Validar input con regex | Cuando el script acepta input del usuario |
| Stubs con `echo` + `exit # TESTING` | Probar flujo sin ejecutar código peligroso |
| Casos de prueba (normal, inexistente, vacío) | Garantizar cobertura de casos límite |
| `echo "..." >&2` para tracing | Depuración ligera y localizada |
| `#!/bin/bash -x` | Tracing completo del script |
| `set -x` / `set +x` | Tracing de secciones específicas |
| `export PS4='$LINENO + '` | Ver número de línea en el tracing |
| `echo "var=$var" # DEBUG` | Inspeccionar valores de variables en ejecución |

---

## Ver También

- [[wiki/linux/24-primer-script.md|Capítulo 24: Escribiendo tu Primer Script]] — fundamentos de scripts bash
- [[wiki/linux/26-top-down-design.md|Capítulo 26: Top-Down Design]] — stubs para testing incremental
- [[wiki/linux/27-control-flujo-if.md|Capítulo 27: Control de Flujo con if]] — exit status y expresiones condicionales
- [[wiki/linux/29-control-flujo-while-until.md|Capítulo 29: Bucles while/until]] — contexto de scripts complejos
