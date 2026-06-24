---
title: "Capítulo 15: Medios de Almacenamiento en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 15"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-207-224.pdf"
updated: "2026-06-23"
---

# Medios de Almacenamiento en Linux

## Introducción

En capítulos anteriores, hemos trabajado con **manipulación de datos a nivel de archivo**. En este capítulo consideramos datos a **nivel de dispositivo**. Linux tiene capacidades increíbles para manejar dispositivos de almacenamiento, ya sean:

- Discos duros físicos
- Almacenamiento en red
- Dispositivos de almacenamiento virtual como RAID (Redundant Array of Independent Disks) y LVM (Logical Volume Manager)

Para los propósitos de este capítulo, introduciremos conceptos y comandos clave para gestionar dispositivos de almacenamiento.

### Comandos a Tratar

- **mount** — Montar un filesystem
- **umount** — Desmontar un filesystem
- **parted** — Programa de manipulación de particiones
- **mkfs** — Crear un filesystem
- **fsck** — Verificar y reparar un filesystem
- **dd** — Convertir y copiar un archivo
- **genisoimage** — Crear un archivo de imagen ISO 9660
- **wodim** — Escribir datos en medios ópticos
- **sha256sum** — Calcular y verificar checksums SHA256

## Montaje y Desmontaje de Dispositivos de Almacenamiento

### El Concepto de Mounting

El **mounting** es el proceso de **adjuntar un dispositivo al árbol del filesystem**. Esto permite que el dispositivo interactúe con el sistema operativo.

A diferencia de otros sistemas operativos (MS-DOS, Windows) que mantienen **árboles de filesystem separados para cada dispositivo** (C:\, D:\), Linux mantiene **un único árbol de filesystem** con dispositivos adjuntos en varios puntos.

### El Archivo /etc/fstab

El archivo `/etc/fstab` (filesystem table) lista los dispositivos (típicamente particiones de disco duro) que deben montarse en el arranque. Ejemplo de un archivo `/etc/fstab`:

```
LABEL=/12              /           ext4    defaults    1 1
LABEL=/home            /home       ext4    defaults    1 2
LABEL=/boot            /boot       ext4    defaults    1 2
tmpfs                  /dev/pts    devpts  gid=5,mode=620 0 0
sysfs                  /sys        sysfs   defaults    0 0
proc                   /proc       proc    defaults    0 0
LABEL=SWAP-sda3        swap        swap    defaults    0 0
```

#### Campos de /etc/fstab

Cada línea del archivo consta de seis campos, como se describe en la Tabla 15-1:

| Campo | Contenido | Descripción |
|---|---|---|
| 1 | Device | Nombre del dispositivo (ej. `/dev/sda1`) o etiqueta UUID. En distribuciones modernas se usa etiqueta LABEL o UUID. |
| 2 | Mount point | Directorio donde se adjunta el dispositivo al árbol del filesystem |
| 3 | Filesystem type | Tipo de filesystem a montar. Linux ext4, FAT16, FAT32, NTFS, CD-ROM (iso9660), y otros |
| 4 | Options | Opciones de montaje (ej. ro para read-only, prevenir ejecución de programas) |
| 5 | Frequency | Número que especifica si y cuándo hacer backup con `dump` |
| 6 | Order | Número que especifica el orden en que `fsck` verifica filesystems |

### Visualizar Filesystems Montados

El comando `mount` sin argumentos muestra todos los filesystems actualmente montados:

```bash
$ mount
/dev/sda2 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda3 on /home type ext4 (rw,relatime)
/dev/sdc on /media/me/C911-C314 type vfat (rw,nosuid,nodev,relatime,...)
```

El formato es: `device on mount_point type filesystem_type (options)`

### Ejemplo Práctico: Montaje de USB Flash Drive

Primero, listamos los dispositivos montados con `mount | grep /dev/sd`:

```bash
$ mount | grep /dev/sd
/dev/sda2 on / type ext4 (rw,relatime,errors=remount-ro)
/dev/sda1 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,...)
/dev/sdc on /media/me/C911-C314 type vfat (rw,nosuid,nodev,relatime,...)
```

Cuando insertamos un USB flash drive, el sistema agregará una entrada nueva. El dispositivo tendrá un nombre como `/dev/sdc` (el nombre exacto variará en tu sistema).

Para desmontar y montar el drive en otra ubicación:

**Paso 1: Convirtiéndose en superusuario (si es necesario)**
```bash
$ sudo -i
[sudo] password for me:
$ umount /dev/sdc
```

**Paso 2: Crear un nuevo mount point**
```bash
$ mkdir /mnt/flash
```

**Paso 3: Montar el dispositivo en el nuevo punto**
```bash
$ mount -t vfat /dev/sdc /mnt/flash
```

**Paso 4: Examinar el contenido**
```bash
$ cd /mnt/flash
$ ls
```

**Paso 5: Desmontar el dispositivo**
```bash
$ umount /dev/sdc
umount: /mnt/flash: device is busy
```

> **Problema Común**: No puedes desmontar un dispositivo si está siendo usado. Si obtienes "device is busy", cambia el directorio de trabajo a algo que no sea el mount point.

```bash
$ cd
$ umount /dev/sdc
```

Ahora el dispositivo se desmonta exitosamente.

### Por Qué Desmontar es Importante

El desmontaje es crítico por la razón de los **buffers** en sistemas operativos:

- Los datos escritos en dispositivos se guardan en memoria (**buffers**) antes de ser escritos físicamente
- El sistema mantiene esta información en memoria el máximo tiempo posible
- Al desmontar, **toda la información pendiente se escribe al dispositivo**
- Si removes un dispositivo sin desmontar primero, **podrías perder datos** y causar **corrupción del filesystem**

## Determinación de Nombres de Dispositivos

### Tabla de Patrones de Dispositivos Linux

| Patrón | Dispositivo | Descripción |
|---|---|---|
| `/dev/fd*` | Floppy disk drives | Unidades de disquete (antiguas) |
| `/dev/hd*` | IDE (PATA) disks | Discos IDE en sistemas antiguos. Motherboards modernos usan dos canales IDE con dos puntos de conexión cada uno. El primer drive en el cable se llama "master" (maestro), el segundo se llama "slave" (esclavo). `/dev/hda` es el master en el primer canal, `/dev/hdb` es el slave en el primer canal. Los dígitos al final indican número de partición. Ej: `/dev/hda1` es la primera partición del primer disco IDE. |
| `/dev/ip*` | Printers | Impresoras |
| `/dev/sd*` | SCSI disks | Discos SCSI. En sistemas Linux modernos, el kernel trata todos los discos similares a SCSI (SATA, USB, portable music players, digital cameras) como SCSI. El esquema de nombres es similar a `/dev/hd*`. |
| `/dev/sr*` | Optical drives | Lectores y grabadores de CD/DVD |

### Encontrar Dispositivos Removibles

Si estás trabajando con sistemas que **no montan automáticamente** dispositivos removibles, puedes usar estas técnicas:

**Opción 1: Ver logs en tiempo real**
```bash
$ sudo tail -f /var/log/syslog
```

O en sistemas basados en systemd:
```bash
$ sudo journalctl -f
```

Cuando inseres un dispositivo removible, verás mensajes como:

```
Jul 25 13:15:07 ratel mtp-probe[21318]: checking bus 3, device 8: "/sys/devices/pci0000:00/0000:00:14.0/usb3/3-6"
Jul 25 13:15:08 ratel kernel: sd 6:0:0:0: [sdc] Write Protect is off
Jul 25 13:15:08 ratel kernel: sd 6:0:0:0: Attached SCSI removable disk
```

**Opción 2: Usar lsblk**

El comando `lsblk` lista todos los dispositivos de bloque del sistema:

```bash
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda    8:0    0 111.8G 0 disk
├─sda1 8:1    0  976M 0 part /boot/efi
├─sda3 8:3    0 14.9G 0 part [SWAP]
├─sdb   8:16  0 931.5G 0 disk
├─sdb1 8:17  0 922.2G 0 part /home
└─sdb2 8:18  0 9.3G 0 part
sdd    8:48  0 232.9G 0 disk
└─sdd1 8:49  0 111.8G 0 part
└─sdd2 8:50  0 111.8G 0 part
sro    11:0   1 1024M 0 rom
```

Cuando insertas el USB flash drive:

```bash
$ lsblk
sdc    8:32   1 3.8G 0 disk
```

El dispositivo sigue siendo `/dev/sdc` mientras permanece conectado a la computadora.

## Creación de Nuevos Filesystems

Para trabajar con un disco duro USB externo y dividirlo en dos particiones (ext4 para Linux, NTFS para Windows), necesitamos:

1. Crear un diseño de nueva partición
2. Crear nuevos filesystems vacíos en el disco

### Manipulación de Particiones con parted

⚠️ **ADVERTENCIA**: En los siguientes ejercicios, vamos a formatear un disco duro externo. **¡Asegúrate de especificar el nombre correcto del dispositivo en tu sistema, no el mostrado en el texto!** El no prestar atención podría resultar en formatear (borrar) el disco equivocado.

`parted` es un programa que permite interactuar con dispositivos similares a discos duros (como discos duros y USB) a muy bajo nivel.

Primero, verificamos el disco USB y desmontamos las particiones:

```bash
$ sudo umount /dev/sdd1
```

Luego invocamos `parted` en el dispositivo de disco duro completo (no por número de partición):

```bash
$ sudo parted /dev/sdd
GNU Parted 3.4
Using /dev/sdd
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

El comando `help` muestra opciones disponibles. Algunos comandos clave:

| Comando | Descripción |
|---|---|
| **print** | Mostrar tabla de particiones, espacios disponibles, todos los dispositivos encontrados |
| **mklabel, mktable LABEL-TYPE** | Crear una nueva table de etiquetas de disco |
| **mkpart PART-TYPE [FS-TYPE] START END** | Hacer una partición nueva |
| **name NUMBER NAME** | Nombrar partición NUMBER como NAME |
| **rm NUMBER** | Eliminar partición NUMBER |
| **resizepart NUMBER END** | Redimensionar partición NUMBER |
| **quit, rescue START END** | Salir del programa; rescatar una partición perdida entre START y END |
| **select DEVICE** | Elegir el dispositivo a editar |
| **disk_set FLAG STATE** | Cambiar el FLAG en el dispositivo seleccionado |
| **set NUMBER FLAG STATE** | Cambiar el FLAG en la partición NUMBER |
| **toggle [NUMBER [FLAG]]** | Toggle (alternar) el estado de FLAG en partición NUMBER |
| **unit UNIT** | Establecer la unidad por defecto a UNIT |
| **version** | Mostrar el número de versión de GNU Parted |

#### Ejemplo Práctico: Crear Particiones

**Paso 1: Ver diseño actual de particiones**
```bash
(parted) print
Model: WDC WD25 00BEVT-00ARDF0 (scsi)
Disk /dev/sdd: 250GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number Start  End    Size    Type     File system Flags
1      104kB  250GB  250GB   primary  fat32       lba
```

El disco tiene 250GB en una sola partición FAT32.

**Paso 2: Eliminar la partición actual**
```bash
(parted) rm 1
(parted) print
```

**Paso 3: Crear nueva primera partición (ext4)**
```bash
(parted) mkpart
Partition type? primary/extended? primary
File system type? [ext2]? ext4
Start? 1
End? 120000
```

**Paso 4: Crear segunda partición (NTFS)**
```bash
(parted) mkpart
Partition type? primary/extended? primary
File system type? [ext2]? ntfs
Start? 120001
End? 240000
```

**Paso 5: Ver resultado**
```bash
(parted) print
Model: WDC WD25 00BEVT-00ARDF0 (scsi)
Disk /dev/sdd: 250GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number Start  End    Size    Type     File system Flags
1      104kB  120GB  120GB   primary  ext4        lba
2      120GB  240GB  120GB   primary  ntfs        lba
```

**Paso 6: Salir de parted**
```bash
(parted) quit
```

### Creación de Filesystems con mkfs

Una vez completada la partición, es hora de crear (formatear) nuevos filesystems. Usaremos `mkfs` (make filesystem) para crear filesystems en el disco.

Para crear un filesystem ext4 en `/dev/sdd1`:

```bash
$ sudo mkfs -t ext4 -L EXT4_Disk /dev/sdd1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 29296640 4k blocks and 7331840 inodes
Filesystem UUID: 3365d135-d8d9-4b91-a57a-ec3567c8548a
Superblock backups stored on blocks:
  32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632,
  2654208, 4096000, 7962624, 11239424, 20480000, 23887872

Allocating groups: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done
```

Para crear un filesystem NTFS en `/dev/sdd2`:

```bash
$ sudo mkfs -t ntfs --quick -L NTFS_Disk /dev/sdd2
Cluster size has been automatically set to 4096 bytes.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
```

La opción `--quick` se usa para saltar la verificación de bloques defectuosos (que toma mucho tiempo).

Si volvemos a ejecutar `lsblk`, veremos:

```bash
$ lsblk
...
├─sdd1 8:49  0 111.8G 0 part
└─sdd2 8:50  0 111.8G 0 part
```

Las etiquetas de volumen se utilizan para crear nombres de puntos de montaje en el directorio `/media` donde los dispositivos de almacenamiento removibles se montan automáticamente.

## Verificación y Reparación de Filesystems

El comando `fsck` (filesystem check) verifica y repara filesystems. En `/etc/fstab`, cada entrada especifica el orden en que `fsck` debe verificar los filesystems en el arranque.

### Cómo Funciona fsck

El sistema verifica la integridad de los filesystems antes de montarlos. Lo realiza el programa `fsck` que lee el archivo `/etc/fstab`.

Ejemplo de verificación:

```bash
$ sudo umount /dev/sdd1
$ sudo fsck /dev/sdd1
fsck from util-linux 2.37.2
EXT4_Disk: clean, 11/7331840 files, 606693/29296640 blocks
```

> **Nota sobre fsck**: En la cultura Unix, el acrónimo "fsck" a menudo se usa en lugar de una palabra popular de tres letras. ¡Es especialmente apropiado cuando te encuentras en una situación donde te ves obligado a ejecutar fsck!

## Movimiento de Datos Directamente a y desde Dispositivos

Mientras normalmente pensamos en datos en nuestras computadoras organizados en archivos, también es posible pensar en los datos en "forma bruta". Si observamos un disco duro, veremos que consta de un gran número de "bloques" de datos que el sistema operativo ve como directorios y archivos.

El comando `dd` realiza esta tarea. Copia bloques de datos de un lugar a otro. Usa una sintaxis única:

```bash
dd if=input_file of=output_file [bs=block_size] [count=blocks]
```

⚠️ **ADVERTENCIA**: El comando `dd` es poderoso. Aunque su nombre se deriva de "data definition," a veces se llama "destroy disk" porque los usuarios a menudo cometén errores con `if` u `of`. **¡Siempre verifica dos veces tus especificaciones de entrada y salida antes de presionar Enter!**

### Ejemplo: Copiar Contenidos entre USB Drives

Si tenemos dos USB flash drives del mismo tamaño y queremos copiar exactamente el primero al segundo:

```bash
$ sudo dd if=/dev/sdb of=/dev/sdc
```

Alternativamente, si solo quieres copiar el contenido a un archivo para restauración posterior:

```bash
$ sudo dd if=/dev/sdb of=flash_drive.img
```

## Creación de Imágenes CD-ROM

### Concepto de Imagen ISO

Escribir un CD-ROM grabable (CD-R o CD-RW) consiste en dos pasos:

1. Construir un archivo de **imagen ISO** que sea la copia exacta del filesystem del CD-ROM
2. Escribir el archivo de imagen en el medio CD-ROM

### Copia de ISO desde CD-ROM Existente

Si tenemos un CD de Ubuntu y queremos hacer una copia ISO:

```bash
$ dd if=/dev/cdrom of=ubuntu.iso
```

Esta técnica funciona para DVDs de datos pero **no para CDs de audio**, ya que no usan filesystem para almacenamiento.

### Creación de Imagen ISO desde Colección de Archivos

Para crear un archivo de imagen ISO que contenga el contenido de un directorio, usamos el programa `genisoimage`. Primero creamos un directorio con los archivos que queremos incluir en la imagen:

```bash
$ genisoimage -o cd-rom.iso -R -J /cd-rom-files
```

Opciones:
- `-o` — Especifica el nombre del archivo de imagen
- `-R` — Agrega metadatos para extensiones de Rock Ridge (nombres de archivo largos y permisos POSIX)
- `-J` — Habilita extensiones Joliet (nombres de archivo largos para Windows)

### Montaje Directo de Imagen ISO

Existe un truco para montar una imagen ISO mientras aún está en el disco duro y tratarla como si fuera un CD-ROM o DVD ya montado. Agregando la opción `-o loop` a `mount` (junto con el tipo `-t iso9660`), podemos montar el archivo de imagen:

```bash
$ mkdir /mnt/iso_image
$ mount -t iso9660 -o loop image.iso /mnt/iso_image
```

En este ejemplo, creamos un punto de montaje `/mnt/iso_image` y luego montamos el archivo de imagen en ese punto. Después de montarse, se puede tratar como un CD-ROM o DVD real.

> **Recuerda**: Desmonta la imagen cuando ya no la necesites.

## Escritura de Imágenes CD-ROM

### Borrado de CD-RW Reescribibles

Los medios CD-RW reescribibles necesitan ser **borrados** (blanking) antes de reutilizarse. Para hacer esto, usamos `wodim`, especificando el dispositivo del grabador de CD y el tipo de blanking deseado. El tipo más mínimo (y más rápido) es "fast":

```bash
$ wodim dev=/dev/cdw blank=fast
```

### Escritura de Imagen

Para escribir una imagen en un medio óptico, usamos `wodim`, especificando el nombre del dispositivo grabador de CD y el nombre del archivo de imagen:

```bash
$ wodim dev=/dev/cdw image.iso
```

Además del nombre de dispositivo e imagen, `wodim` soporta un gran conjunto de opciones. Dos comunes son:
- `-v` — Para salida verbose (detallada)
- `-dao` — Escribe el disco en modo disc-at-once (útil si estamos preparando un disco para producción comercial)

El modo por defecto de `wodim` es **track-at-once**, que es útil para grabar pistas de música.

## Verificación de Datos

Cuando descargamos un archivo grande como una imagen de instalación de Linux, queremos asegurar que el archivo descargado esté completo y no corrupto. Una herramienta útil es `sha256sum`, un reemplazo moderno de un programa anterior llamado `md5sum`.

### Ejemplo: Verificación de Descarga

Digamos que queremos descargar una imagen de Linux Mint 22. Vamos al sitio web de Linux Mint y descargamos `linuxmint-22-cinnamon-64bit.iso`. Mientras estamos ahí, también descargamos un archivo de **checksum** llamado `sha256sum.txt`. Cuando vemos su contenido:

```bash
$ cat sha256sum.txt
7a04b5483000049945c1eda6ed6ec8c57ff4b249de4b31bd021a3d969f29b8f *lin unmint-22-cinnamon-64bit.iso
55e917b99206187564d294764f21b9b8f5a8a0b6e54c49ff6acb39dcfcb4bd80 *lin unmint-22-mate-64bit.iso
55e917b99206187564d294764f21b9b8f5a8a0b6e54c49ff6acb39dcfcb4bd80 *lin unmint-22-xfce-64bit.iso
```

**Checksums** son una representación matemática precisa de archivos ISO. Si el archivo descargado se altera aunque sea un bit del archivo original, el checksum será significativamente diferente.

Para verificar que el archivo descargado es completo, usamos `sha256sum` generando un checksum del archivo descargado y comparándolo con uno en la lista. Podemos hacer esto ejecutando `sha256sum` así:

```bash
$ sha256sum -c --ignore-missing sha256sum.txt
linuxmint-22-cinnamon-64bit.iso: OK
```

La opción `-c` (short for "check") invoca el modo de verificación, mientras que la opción `--ignore-missing` le dice a `sha256sum` que no se queje si archivos en la lista no fueron descargados (linuxmint-22-mate-64bit.iso y linuxmint-22-xfce-64bit.iso no están presentes).

## Resumen

En este capítulo, hemos visto tareas básicas de gestión de almacenamiento. Linux soporta una amplia gama de dispositivos y esquemas de filesystems. También ofrece muchas características para interoperabilidad con otros sistemas.

Con el conocimiento adquirido sobre montaje de dispositivos, particionamiento, creación de filesystems, verificación de datos y manejo de medios ópticos, deberías poder trabajar efectivamente con dispositivos de almacenamiento en Linux.

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]]
- [[wiki/linux/09-permisos.md|Capítulo 9: Permisos en Linux]] (Permisos necesarios para muchas operaciones)
- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] (Útil para entender pipes con dd)
