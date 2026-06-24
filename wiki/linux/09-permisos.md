---
title: "Capítulo 9: Permisos en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 9"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-119-135.pdf"
updated: "2026-06-22"
---

# Permisos en Linux

## ¿Por Qué los Permisos?

Los sistemas operativos en la tradición Unix/Linux son **multitarea** y **multiusuario**. Esto significa que:
- Múltiples usuarios pueden usar la computadora simultáneamente
- Los usuarios pueden acceder remotamente vía SSH y ejecutar comandos
- Es necesario proteger a los usuarios entre sí y contra cambios no autorizados

Para hacer esto práctico, se implementó un sistema de **seguridad de permisos** que controla quién puede leer, escribir y ejecutar archivos y directorios. Este sistema es fundamental en la arquitectura de Linux.

## Usuarios, Grupos y "Others"

En el modelo de seguridad de Unix/Linux:

- **Usuario (User)**: El propietario de un archivo o directorio. Cada usuario tiene un **user ID (uid)**.
- **Grupo (Group)**: Uno o más usuarios que comparten acceso a archivos y directorios. Cada grupo tiene un **group ID (gid)**.
- **Others (Otros)**: El resto de usuarios del sistema, sin relación especial con el propietario del archivo.

### El Comando `id` - Identidad del Usuario

Para ver la información sobre tu identidad en el sistema:

```bash
[me@linuxbox ~]$ id
uid=1000(me) gid=1000(me) groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(lpadmin),114(admin),1000(me)
```

Interpretar como:
- **uid=1000(me)**: ID único del usuario y su nombre
- **gid=1000(me)**: ID del grupo principal
- **groups=...**: Grupos adicionales a los que pertenece el usuario

### Archivos de Sistema de Usuarios

Los datos de usuarios y grupos se almacenan en archivos de texto:

| Archivo | Contenido |
|---------|----------|
| `/etc/passwd` | Información de cuentas de usuario (login, uid, gid, home directory, shell) |
| `/etc/group` | Información de grupos y sus miembros |
| `/etc/shadow` | Información de contraseñas de usuario (encriptada) |

**Nota**: En Linux moderno, es común que cada usuario tenga su propio grupo con el mismo nombre. Esto simplifica la gestión de permisos.

## Permisos de Archivos y Directorios

### Tipos de Archivo

Cuando ejecutas `ls -l`, el primer carácter indica el **tipo de archivo**:

| Atributo | Tipo de archivo | Descripción |
|----------|-----------------|-------------|
| `-` | Regular file | Archivo regular |
| `d` | Directory | Directorio |
| `l` | Symbolic link | Enlace simbólico (permisos dummy, reales en el archivo destino) |
| `c` | Character special | Dispositivo de caracteres (terminal, /dev/null) |
| `b` | Block special | Dispositivo de bloques (disco duro, SSD) |

### Atributos de Permisos (rwx)

Los 9 caracteres restantes de los atributos de archivo representan los permisos para el propietario, grupo y otros:

```
-rw-r--r--
 ||| ||| |||
 +--+--+--+--- propietario, grupo, otros
```

**Significado de r, w, x:**

| Permiso | En Archivos | En Directorios |
|---------|------------|----------------|
| **r** (read) | Permite abrir y leer el archivo | Permite listar el contenido del directorio |
| **w** (write) | Permite escribir o truncar el archivo; no permite renombrar o eliminar (controlado por permisos del directorio) | Permite crear, eliminar y renombrar archivos dentro del directorio si el permiso execute también está activo |
| **x** (execute) | Permite ejecutar el archivo como programa (scripts y binarios requieren permiso read también) | Permite entrar en el directorio (necesario para acceder a `ls -l`, `cd`, etc.) |

### Ejemplos de Permisos Comunes

| Atributos | Significado |
|-----------|-------------|
| `-rw-------` | Archivo legible, escribible y ejecutable solo por el propietario |
| `-rw-r--r--` | Archivo legible y escribible por el propietario; legible por grupo y otros |
| `-rwxr-xr-x` | Archivo ejecutable por todos; escribible solo por propietario |
| `-rw-rw----` | Archivo legible y escribible por propietario y grupo solo |
| `lrwxrwxrwx` | Enlace simbólico (los permisos siempre son dummy) |
| `drwxr-xr-x` | Directorio: propietario puede crear/eliminar archivos; grupo y otros solo pueden listar |
| `drwxr-x---` | Directorio: propietario puede crear/eliminar; grupo solo puede entrar y listar |

## Chmod - Cambiar Permisos de Archivo

El comando `chmod` modifica los permisos. Solo el propietario o root puede cambiar el modo.

### Notación Octal

Cada dígito octal representa 3 bits (rwx):

```
r w x = 4 2 1 = 7 (read + write + execute)
r - x = 4 0 1 = 5 (read + execute)
r w - = 4 2 0 = 6 (read + write)
- - x = 0 0 1 = 1 (execute only)
```

**Sintaxis**: `chmod NNN archivo` donde NNN son tres dígitos octales para usuario, grupo, otros.

```bash
[me@linuxbox ~]$ chmod 600 foo.txt
# Propietario: rwx (6)
# Grupo: --- (0)
# Otros: --- (0)

[me@linuxbox ~]$ chmod 755 script.sh
# Propietario: rwx (7)
# Grupo: r-x (5)
# Otros: r-x (5)
```

### Notación Simbólica

Formato: `chmod [u|g|o|a][+|-|=][r|w|x] archivo`

- **u** = usuario (propietario), **g** = grupo, **o** = otros, **a** = todos
- **+** = agregar, **-** = remover, **=** = establecer exactamente
- **r**, **w**, **x** = permisos

```bash
[me@linuxbox ~]$ chmod u+x script.sh      # Agregar execute para propietario
[me@linuxbox ~]$ chmod o-rw file.txt      # Remover read y write para otros
[me@linuxbox ~]$ chmod a+x all_users.sh   # Todos pueden ejecutar
[me@linuxbox ~]$ chmod u=rw,g=r,o= file   # Establece exactamente estos permisos
```

### Entender Números Octales y Hexadecimales

**Contexto**: Las computadoras modernas fueron diseñadas con 1 dígito (binario), pero los humanos trabajamos con 10 dígitos. Para representar datos de manera compacta:

- **Octal (base 8)**: Usa dígitos 0-7. Cada dígito representa 3 bits en binario. Útil para permisos (3 permisos = 3 bits)
- **Hexadecimal (base 16)**: Usa 0-9 y A-F. Cada dígito representa 4 bits. Común para colores RGB (ej: FF0000 para rojo)

Ejemplo de conversión octal a binario:
```
Octal:   6   2   3
Binario: 110 010 011
```

## Umask - Permisos por Defecto

El comando `umask` controla los permisos predeterminados cuando se crea un archivo o directorio. Usa notación octal para especificar qué permisos **remover** de los permisos por defecto.

```bash
[me@linuxbox ~]$ umask
0002
```

El valor `0002` significa remover el permiso write (2) para "others". Los permisos por defecto se calculan así:

```
Permisos por defecto: 666 (archivos) o 777 (directorios)
Mask:                 0002
Resultado:            664 (archivos) o 775 (directorios)
```

**Cambiar umask**:

```bash
[me@linuxbox ~]$ umask 0022
# Ahora archivos se crean con 644 (rw-r--r--)
# Y directorios con 755 (rwxr-xr-x)
```

Nota: Los cambios a `umask` duran solo para la sesión actual. Para hacerlos permanentes, se deben agregar al archivo de configuración del shell.

## Permisos Especiales

Además de read, write, execute, hay tres permisos especiales:

### Setuid (Set User ID)

**Octal 4000**: Cuando se ejecuta un archivo con setuid, el programa corre con los privilegios del propietario, no del usuario que lo ejecuta.

Uso común: Programas que necesitan acceso privilegiado pero no queremos dar acceso root completo.

```bash
[me@linuxbox ~]$ chmod u+s program
# Resultado: -rwsr-xr-x
```

### Setgid (Set Group ID)

**Octal 2000**: Similar a setuid, pero usa el grupo del archivo. En directorios, causa que archivos creados hereden el grupo del directorio.

```bash
[me@linuxbox ~]$ chmod g+s directory
# Resultado: drwxr-sr-x
```

### Sticky Bit

**Octal 1000**: En directorios, previene que usuarios eliminen archivos de otros usuarios, incluso si tienen permiso write en el directorio.

Uso común: Directorios compartidos como `/tmp`.

```bash
[me@linuxbox ~]$ chmod +t directory
# Resultado: drwxrwxrwt
```

## Cambiar Identidades

Hay tres formas de asumir la identidad de otro usuario:

1. **Logout y login nuevamente** - Inconveniente
2. **Comando `su`** - Start a new shell como otro usuario
3. **Comando `sudo`** - Execute a single command as another user (método moderno)

### Su - Ejecutar Shell como Otro Usuario

**Sintaxis**: `su [-l] [usuario]`

- `-l` (login): Carga el ambiente del usuario (env vars, working directory)
- Sin `-l`: Solo cambia el UID, mantiene el ambiente actual

```bash
[me@linuxbox ~]$ su -
Password:
[root@linuxbox ~]# exit
[me@linuxbox ~]$
```

Para ejecutar un único comando sin abrir shell interactivo:

```bash
[me@linuxbox ~]$ su -c 'ls -l /root/*'
Password:
```

**Nota**: En distribuciones modernas, `su` está siendo reemplazado por `sudo`.

### Sudo - Ejecutar Comando como Otro Usuario

**Sintaxis**: `sudo [opciones] comando`

Características:
- Requiere la contraseña del usuario actual (no del usuario destino)
- Puede configurarse para permitir ciertos comandos sin contraseña
- Los comandos ejecutados se registran en logs
- Más seguro que `su` porque solo da privilegios para comandos específicos

```bash
[me@linuxbox ~]$ sudo backup_script
Password:
System Backup Starting...
```

Ver qué comandos puedes ejecutar con sudo:

```bash
[me@linuxbox ~]$ sudo -l
User me may run the following commands on this host:
    (ALL) ALL
```

**Nota sobre distribuciones modernas**: Ubuntu y muchas otras distribuciones modernas desactivan la cuenta root por defecto y confían en `sudo` para tareas administrativas. Esto es más seguro que permitir login directo como root.

## Chown - Cambiar Propietario y Grupo

**Sintaxis**: `chown [propietario][:grupo] archivo...`

Solo root puede usar este comando.

| Argumento | Resultado |
|-----------|----------|
| `bob` | Cambia el propietario a bob |
| `bob:users` | Cambia el propietario a bob y el grupo a users |
| `:admins` | Cambia solo el grupo a admins |
| `bob:` | Cambia el propietario a bob y el grupo al grupo login de bob |

**Ejemplo**:

```bash
[janet@linuxbox ~]$ sudo cp myfile.txt ~tony/myfile.txt
[janet@linuxbox ~]$ sudo ls -l ~tony/myfile.txt
-rw-r--r-- 1 root root 754 2025-03-20 14:30 /home/tony/myfile.txt

[janet@linuxbox ~]$ sudo chown tony: ~tony/myfile.txt
[janet@linuxbox ~]$ sudo ls -l ~tony/myfile.txt
-rw-r--r-- 1 tony tony 754 2025-03-20 14:30 /home/tony/myfile.txt
```

## Chgrp - Cambiar Grupo

En versiones antiguas de Unix, `chown` solo cambiaba el propietario. Para cambiar el grupo se usaba `chgrp`. En sistemas modernos, esto se hace con `chown`, pero `chgrp` sigue disponible.

```bash
[me@linuxbox ~]$ chgrp newgroup file
```

## Caso Práctico: Directorio Compartido

Scenario: Dos usuarios (janet y tony) quieren compartir un directorio de música.

**Paso 1**: Crear el grupo y agregar usuarios

```bash
[janet@linuxbox ~]$ sudo groupadd music
[janet@linuxbox ~]$ sudo usermod -a -G music janet
[janet@linuxbox ~]$ sudo usermod -a -G music tony
```

**Paso 2**: Crear el directorio

```bash
[janet@linuxbox ~]$ sudo mkdir /usr/local/share/Music
```

**Paso 3**: Cambiar propietario y grupo

```bash
[janet@linuxbox ~]$ sudo chown music /usr/local/share/Music
[janet@linuxbox ~]$ sudo chmod 2775 /usr/local/share/Music
```

Esto establece:
- Setgid (2) - Archivos creados heredarán el grupo "music"
- rwx para usuario (7)
- rwx para grupo (7)
- r-x para otros (5)

**Paso 4**: Cambiar umask para crear archivos compartibles

```bash
[janet@linuxbox ~]$ umask 0002
[janet@linuxbox ~]$ > /usr/local/share/Music/test_file
[janet@linuxbox ~]$ mkdir /usr/local/share/Music/test_dir
```

Ahora ambos usuarios pueden crear y modificar archivos en el directorio.

## Passwd - Cambiar Contraseña

**Sintaxis**: `passwd [usuario]`

```bash
[me@linuxbox ~]$ passwd
Changing password for me.
Current password:
New password:
Retype new password:
passwd: password updated successfully
```

El comando `passwd` intenta forzar contraseñas "fuertes":
- Advertencia si son muy cortas
- Advertencia si son similares a contraseñas anteriores
- Advertencia si son palabras de diccionario

**Administrador**: Si tienes privilegios, puedes cambiar contraseñas de otros usuarios:

```bash
[root@linuxbox ~]$ passwd username
```

Otros comandos útiles (parte de shadow-utils):

| Comando | Descripción |
|---------|------------|
| `lastlog` | Muestra el último login de usuarios |
| `useradd` | Crear nuevo usuario |
| `userdel` | Eliminar usuario |
| `usermod` | Modificar cuenta de usuario |
| `groupadd` | Crear nuevo grupo |
| `groupdel` | Eliminar grupo |
| `groupmod` | Modificar grupo |

## Resumen

Los permisos en Linux son fundamentales para la seguridad del sistema:

- Los archivos tienen propietario, grupo y "otros" con permisos rwx
- `chmod` cambia permisos (octal o simbólico)
- `umask` controla los permisos por defecto
- Permisos especiales (setuid, setgid, sticky bit) para casos específicos
- `su` y `sudo` para asumir otras identidades
- `chown` y `chgrp` para cambiar propietario y grupo
- El sistema de permisos refleja el origen multiusuario de Unix

---

## Ver También

- [[wiki/linux/02-navegacion-sistema-archivos.md|Capítulo 2: Navegación del Sistema de Archivos]] — Conceptos de rutas para operaciones de permisos
- [[wiki/linux/04-manipulacion-archivos-directorios.md|Capítulo 4: Manipulación de Archivos y Directorios]] — Operaciones con archivos que requieren permisos
- [[wiki/linux/10-procesos.md|Capítulo 10: Procesos en Linux]] — Control de procesos con privilegios y cambio de identidad
