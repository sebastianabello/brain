---
title: "Capítulo 16: Redes (Networking) en Linux"
sources: 
  - "The Linux Command Line: A Complete Introduction, Ch. 16"
raw: "raw/linux/capitulos/The_Linux_Command_Line_A_Complete_Introduction-225-237.pdf"
updated: "2026-06-23"
---

# Redes (Networking) en Linux

## Introducción

Cuando se trata de redes, probablemente no hay nada que no se pueda hacer con Linux. Linux se usa para construir todo tipo de sistemas de redes y dispositivos, incluyendo:

- **Firewalls** (cortafuegos)
- **Routers** (enrutadores)
- **Name servers** (servidores de nombres)
- **Network-attached storage (NAS)** — almacenamiento conectado a red
- Y muchos más

El tema de redes es tan vasto como la cantidad de comandos disponibles para configurarlo y controlarlo. En este capítulo, nos enfocaremos en algunos de los comandos más frecuentemente utilizados para:

- **Monitorear redes** y examinar su rendimiento
- **Transferir datos** a través de SSH (comunicación segura remota)

### Conceptos Clave de Networking

Para aprovechar al máximo este capítulo, deberíamos estar familiarizados con los siguientes términos:

- **Internet protocol (IP) address** — Dirección del protocolo de internet
- **Hostname** y **domain name** — Nombre del host y nombre de dominio
- **Uniform resource identifier (URI)** — Identificador uniforme de recurso

> **Nota**: Algunos comandos que cubriremos pueden requerir (dependiendo de tu distribución) la instalación de paquetes adicionales de los repositorios de tu distribución, y algunos pueden requerir privilegios de superusuario para ejecutarse.

## Examinación y Monitoreo de una Red

Aunque no seas el administrador del sistema, es a menudo útil examinar el rendimiento y operación de una red.

### ping

El comando más básico de red es **ping**. El comando `ping` envía un paquete de red especial llamado **ICMP ECHO_REQUEST** a un host especificado. La mayoría de dispositivos de red que reciben este paquete responderán, permitiendo verificar la conectividad de red.

> **Nota**: Es posible configurar la mayoría de dispositivos de red (incluyendo hosts Linux) para ignorar estos paquetes. Esto normalmente se hace por razones de seguridad, para oscurecer parcialmente un host de un atacante potencial. También es común que los firewalls estén configurados para bloquear tráfico ICMP.

#### Ejemplo Básico

Para ver si podemos llegar a https://linuxcommand.org, podemos usar `ping` así:

```bash
$ ping linuxcommand.org
```

Una vez iniciado, `ping` continúa enviando paquetes en un intervalo especificado (por defecto un segundo) hasta que sea interrumpido.

```bash
PING linuxcommand.org (66.35.250.210) 56(84) bytes of data.
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=1 ttl=43 time=107 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=2 ttl=43 time=108 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=3 ttl=43 time=106 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=4 ttl=43 time=105 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=5 ttl=43 time=107 ms
--- linuxcommand.org ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 6010ms
rtt min/avg/max/mdev = 105.647/107.052/108.118/0.824 ms
```

Después de ser interrumpido (presionando Ctrl+C), `ping` imprime estadísticas de rendimiento. Una red que funciona correctamente mostrará **0 por ciento de pérdida de paquetes**. Un `ping` exitoso indicará que los elementos de la red (tarjetas de interfaz, cableado, enrutamiento y gateways) están en buen orden general.

#### Interpretación de Resultados

Algo a tener en cuenta es que la máquina que responde al `ping` podría ser diferente de la que solicitamos. Esto ocurre porque una máquina (como vhost.sourceforge.net) puede albergar múltiples sitios con diferentes nombres.

### traceroute

El programa **traceroute** lista todos los "saltos" (hops) de tráfico de red necesarios para llegar desde el sistema local a un host especificado. Por ejemplo, para ver la ruta seguida para alcanzar slashdot.org:

```bash
$ traceroute slashdot.org
```

**Salida típica**:

```
traceroute to slashdot.org (216.34.181.45), 30 hops max, 40 byte packets
 1 ipcop.localdomain (192.168.1.1)         1.066 ms 1.366 ms 1.720 ms
 2 * * *
 3 ge-4-13-ur01.rockville.md.bad.comcast.net (68.87.110.9)  14.622 ms 14.885 ms 15.169 ms
 ...
21 557 ms
...
16 slashdot.org (216.34.181.45)            42.727 ms 41.016 ms 41.437 ms
```

#### Interpretación de Resultados

En la salida:

- **Conexión desde nuestro sistema a slashdot.org requiere atravesar 16 routers**
- Para routers que proporcionan información identificable, vemos sus **hostnames, direcciones IP y datos de rendimiento** (tres muestras de tiempo de ida y vuelta del sistema local al router)
- Para **routers que no proporcionan información identificable**, vemos **asteriscos** en la línea para el número de hop 2
- Si la información de enrutamiento se bloquea, podemos intentar superar esto usando las opciones `-T` o `-I` del comando `traceroute`

### ip

El programa **ip** es una herramienta multipropósito de configuración de red que reemplaza el antiguo (y ahora deprecado) programa `ifconfig`. El programa `ip` se usa para examinar varios parámetros de configuración de red en nuestro sistema.

#### Examinando Interfaces de Red

Aquí están las interfaces de red en nuestro sistema:

```bash
$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   inet 127.0.0.1/8 scope host lo
   valid_lft forever preferred_lft forever
inet6 ::1/128 scope host noprefixroute
   valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   link/ether 00:26:6c:26:67:bf brd ff:ff:ff:ff:ff:ff
   inet 192.168.1.223/24 brd 192.168.50.255 scope global dynamic noprefixroute enp1s0
   valid_lft 82366sec preferred_lft 82366sec
inet6 fe80::226:6cff:fe26:67bf/64 scope link noprefixroute
   valid_lft forever preferred_lft forever
3: wlp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
   link/ether 06:8a:59:f4:f6:d3 brd ff:ff:ff:ff:ff:ff
```

En este ejemplo, el sistema prueba tiene tres interfaces de red:

1. **lo** — La **loopback interface**, una interfaz virtual que el sistema usa para "hablar consigo mismo"
2. **enp1s0** — Una **Ethernet interface** (en = Ethernet)
3. **wlp2s0** — Una **wireless interface** (wl = wireless)

#### Indicadores Importantes

Cuando realizas diagnóstico casual de red, los aspectos importantes a buscar son:

- La presencia de la frase **"state UP"** en la primera línea para la interfaz, indicando que está habilitada
- La presencia de una **dirección IP válida** en el campo `inet` en la tercera línea
- Para sistemas que usan Dynamic Host Configuration Protocol (DHCP), una dirección IP válida en este campo verificará que DHCP está funcionando

#### Examinando la Tabla de Enrutamiento

Aquí está la tabla de enrutamiento:

```bash
$ ip route show
default via 192.168.1.1 dev enp1s0 proto dhcp src 192.168.50.223 metric 100
169.254.0.0/16 dev enp1s0 scope link metric 1000
192.168.1.0/24 dev enp1s0 proto kernel scope link src 192.168.1.223 metric 100
```

En este ejemplo simple, vemos una tabla de enrutamiento típica para una máquina cliente en una red de área local (LAN) detrás de un firewall/router. Las direcciones IP que terminan en cero se refieren a redes en lugar de hosts individuales, por lo que esta destinación significa cualquier host en la LAN. La primera línea contiene el **destino por defecto**. Esto significa enviar cualquier tráfico destinado a una red que no esté de otra manera listada en la tabla a este destino. En nuestro ejemplo, vemos que el **gateway por defecto está definido como un router con la dirección 192.168.1.1** (una dirección típica para un home router), que presumiblemente sabe qué hacer con el tráfico de destino.

La sintaxis completa es un programa complicado con muchas opciones y comandos. La sintaxis de comando es de la siguiente:

```bash
ip [-options] object [command]
```

En los ejemplos anteriores, usamos los objetos `address` y `route` con el comando `show`. Por conveniencia, `ip` permite abreviaturas de nombres de objeto y comando a cualquier prefijo no ambiguo (a menudo tan corto como un solo carácter), y ya que el comando por defecto es `show`, podemos abreviar los comandos a `ip a` y `ip r` y obtener resultados idénticos.

### netstat

El comando **netstat** es útil para examinar varios parámetros de red del sistema. Es usado para:

- Mostrar todas las conexiones de red activas
- Mostrar la tabla de enrutamiento
- Mostrar estadísticas de interfaz
- Mostrar conexiones enmascaradas (masqueraded)
- Mostrar membresías multicast

Aunque `netstat` ha sido reemplazado principalmente por `ip` en sistemas modernos, aún sigue siendo ampliamente utilizado para estos exámenes.

## Transferencia de Archivos a Través de una Red

¿De qué sirve una red a menos que podamos mover archivos a través de ella? Hay muchos programas que mueven datos a través de redes. Cubriremos dos de ellos ahora y varios más en secciones posteriores.

### ftp

El programa **ftp** obtiene su nombre del protocolo que utiliza: el **File Transfer Protocol**. FTP fue una vez el método más ampliamente utilizado para descargar archivos en internet. Aunque algunos navegadores aún lo soportan, y vemos URIs comenzando con el protocolo `ftp://`, FTP no es seguro porque transmite todos sus nombres de usuario y contraseñas en **texto sin cifrar**.

#### Problemas de Seguridad

Debido a esto, casi todo FTP se realiza sobre **servidores FTP anónimos**. Un servidor anónimo permite que cualquiera inicie sesión con el nombre de usuario "anonymous" y una contraseña sin sentido.

#### Ejemplo de Sesión FTP

Aquí hay una sesión imaginaria con el programa `ftp` descargando una imagen de Ubuntu desde el servidor FTP anónimo:

```bash
$ ftp fileserver
Connected to fileserver.localdomain.
220 (vsFTPd 2.0.1)
Name (fileserver:me): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub/cd_images/Ubuntu-24.04
250 Directory successfully changed.
ftp> ls
150 Here comes the directory listing.
...ubuntu-24.04-desktop-amd64.iso     733079552 Apr 25 03:53
226 Directory send OK.
ftp> lcd Desktop
Local directory now /home/me/Desktop
ftp> get ubuntu-24.04-desktop-amd64.iso
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ubuntu-24.04-desktop-amd64.iso (733079552 bytes).
226 File send OK.
7330795555 bytes received in 68.56 secs (1041.5 B/s)
ftp> bye
221 Goodbye
```

#### Tabla de Comandos FTP

| Comando | Significado |
|---|---|
| `ftp fileserver` | Invocar el programa `ftp` e intentar conectarse al servidor FTP fileserver |
| `anonymous` | Nombre de usuario. Después del prompt de login, aparecerá un prompt de contraseña. Algunos servidores aceptarán una contraseña en blanco, otros requieren una contraseña en forma de dirección de email. En este caso, prueba algo como `user@example.com`. |
| `cd pub/cd_images/Ubuntu-24.04` | Cambiar al directorio en el sistema remoto que contiene el archivo deseado. Nótese que en la mayoría de servidores FTP anónimos, los archivos para descargar pública están ubicados en algún lugar bajo el directorio `pub`. |
| `ls` | Listar el directorio en el sistema remoto |
| `lcd Desktop` | Cambiar el directorio local de trabajo a `~/Desktop`. Cuando fue invocado, el directorio de trabajo era "~". Este comando cambia el directorio local a `~/Desktop`. |
| `get ubuntu-24.04-desktop-amd64.iso` | Decirle al sistema remoto que transfiera el archivo ubuntu-24.04-desktop-amd64.iso al sistema local. Ya que el directorio de trabajo en el sistema local fue cambiado a `~/Desktop`, el archivo será descargado aquí. |
| `bye` | Desconectar del servidor remoto y terminar el programa `ftp`. Los comandos `quit` y `exit` también pueden ser usados. |

Typing `help` en el prompt de `ftp` mostrará una lista de comandos soportados. Usando `ftp` en un servidor donde permisos suficientes han sido otorgados, es posible realizar muchas tareas ordinarias de gestión de archivos. Es torpe, pero funciona.

### lftp — Un Mejor ftp

`ftp` no es el único cliente FTP de línea de comandos. De hecho, hay muchos. Uno de los mejores (y más populares) es `lftp` por Alexander Lukyanov. Funciona muy parecido al programa tradicional `ftp` pero tiene muchas características adicionales de conveniencia incluyendo:

- Soporte de protocolo múltiple (incluyendo HTTP)
- Reintentos en descargas fallidas
- Procesos en background
- Completado con tabulación de nombres de rutas
- Y muchos más

### curl — Transferir una URL

Otro programa popular de transferencia de archivos es **curl**. Su uso más básico funciona así:

```bash
$ curl https://linuxcommand.org
```

Especificamos una URL, y `curl` descarga la primera página de la URL y la envía a salida estándar. Se pueden especificar múltiples URLs.

`curl` soporta la mayoría de protocolos de red incluyendo HTTP, HTTPS, FTP, IMAP, POP3, SFTP, SMB, y otros.

#### Tabla de Opciones Comunes de curl

| Opción | Descripción |
|---|---|
| `-o, --output file` | Enviar salida al archivo especificado en lugar de salida estándar |
| `-O, --remote-name` | Como `-o`, pero nombra el archivo local igual al archivo remoto |
| `-s, --silent` | Suprimir el medidor de progreso y mensajes de error |
| `-u, --proxy-user u/password` | Especificar una combinación nombre de usuario/contraseña de proxy |
| `-v, --verbose` | Mostrar mensajes verbosos mientras se ejecuta |

La página man de `curl` cubre todos los detalles desagradables.

### wget — Descargador de Red No-Interactivo

Otro programa popular de descarga en línea de comandos para descarga de archivos es **wget**. Es útil para descargar contenido desde sitios web y FTP. Archivos individuales, múltiples archivos, e incluso sitios completos pueden ser descargados. Para descargar la primera página de linuxcommand.org, podríamos hacer esto:

```bash
$ wget http://linuxcommand.org/index.php
--11:02:51--  http://linuxcommand.org/index.php
           => `index.php'
Resolving linuxcommand.org... 66.35.250.210
Connecting to linuxcommand.org|66.35.250.210|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]

[ <=> ] 3,120      --.-K/s

11:02:51 (161.75 MB/s) - `index.php' saved [3120]
```

Las muchas opciones del programa permiten `wget` descargar recursivamente, descargar archivos en background (permitiendo cerrar sesión pero continuar descargando), y completar la descarga de un archivo parcialmente descargado. Estas características están bien documentadas en su página man mejor que el promedio.

## Comunicación Segura con Hosts Remotos

Durante muchos años, los sistemas operativos tipo Unix han tenido la capacidad de ser administrados remotamente a través de una red. En los primeros días, antes de la adopción general de internet, había un par de programas populares usados para conectarse remotamente a hosts. Estos fueron los programas `rlogin` y `telnet`. Estos programas, sin embargo, sufren del mismo defecto fatal que el programa `ftp`: transmiten todas sus comunicaciones (incluyendo nombres de usuario y contraseñas) en texto claro. Esto los hace completamente inapropiados para uso en la era de internet.

### ssh — Secure Shell

Para abordar este problema, se desarrolló un nuevo protocolo llamado **Secure Shell (SSH)**. SSH resuelve los dos problemas básicos de comunicación segura con un host remoto:

1. **Autentica que el host remoto es quien dice que es** (así previniendo los ataques llamados "man-in-the-middle")
2. **Encripta todas las comunicaciones entre el host local y remoto**

SSH consiste en dos partes. Un **servidor SSH** se ejecuta en el host remoto, escuchando conexiones entrantes (por defecto, en puerto 22), mientras que un **cliente SSH** se usa en el sistema local para comunicarse con el servidor remoto.

La mayoría de distribuciones Linux envían una implementación de SSH llamada **OpenSSH** del proyecto OpenBSD. Algunas distribuciones incluyen ambos paquetes de cliente y servidor por defecto, mientras que otros solo suministran el cliente. Para habilitar conexiones remotas en un sistema, debe tener el paquete OpenSSH-server instalado, configurado y ejecutándose, y (si el sistema está ejecutando o está detrás de un firewall) debe permitir conexiones de red entrantes en TCP puerto 22.

> **Nota**: Si no tienes un sistema remoto al que conectarte pero quieres probar estos ejemplos, asegúrate que el paquete OpenSSH-server está instalado en tu sistema y usa `localhost` como el nombre del host remoto. De esa manera, tu máquina creará conexiones de red contigo misma.

#### Conectar a un Host Remoto

El programa cliente SSH usado para conectarse a servidores SSH remotos se llama, apropiadamente, `ssh`. Para conectarte a un host remoto llamado `remote-sys`, usarías el programa cliente `ssh` así:

```bash
$ ssh remote-sys
The authenticity of host 'remote-sys (192.168.1.4)' can't be established.
RSA key fingerprint is 41:ed:7a:df:23:19:bf:3c:35:17:bc:61:b3:7f:d9:bb.
Are you sure you want to continue connecting (yes/no)?
```

La primera vez que se intenta la conexión, se muestra un mensaje indicando que la autenticidad del host remoto no puede ser establecida. Esto se debe a que el cliente nunca ha visto este host remoto antes. Para aceptar las credenciales del host remoto, escribe "yes" cuando se solicite. Una vez establecida la conexión, se solicita una contraseña al usuario.

```bash
Warning: Permanently added 'remote-sys,192.168.1.4' (RSA) to the list of known hosts.
me@remote-sys's password:
```

Después de introducir la contraseña exitosamente, recibimos el prompt del shell del sistema remoto.

```bash
Last login: Sat Aug 30 13:00:48 2025
[me@remote-sys ~]$
```

La sesión del shell remoto continúa hasta que el usuario escribe el comando `exit` en el prompt del shell remoto, cierre la conexión remota. En este punto, la sesión del shell local se reanuda y reaparece el prompt del shell local.

#### Conectar con un Usuario Diferente

También es posible conectarse a sistemas remotos usando un nombre de usuario diferente. Por ejemplo, si el usuario local `me` tuviera una cuenta llamada `bob` en un sistema remoto, el usuario `me` podría conectarse a la cuenta `bob` en el sistema remoto así:

```bash
$ ssh bob@remote-sys
bob@remote-sys's password:
Last login: Sat Aug 30 13:03:21 2025
[bob@remote-sys ~]$
```

#### Verificación de Autenticidad del Host

Como se mencionó antes, `ssh` verifica la autenticidad del host remoto. Si el host remoto no se autentica exitosamente, aparece el siguiente mensaje:

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
Offending RSA key in /home/me/.ssh/known_hosts:42
Host key verification failed.
```

Este mensaje es causado por una de dos situaciones posibles. Primero, un atacante podría estar intentando un ataque man-in-the-middle. Esto es raro, ya que todos saben que `ssh` alerta al usuario a esto. La posibilidad más probable es que el sistema remoto haya sido cambiado de alguna manera; por ejemplo, su sistema operativo o servidor SSH ha sido reinstalado. En los intereses de seguridad y defensa, sin embargo, la primera posibilidad no debe ser descartada de plano. **Siempre verifica con el administrador del sistema remoto cuando este mensaje ocurra.**

Si se determina que el mensaje es debido a una causa benigna, es seguro corregir el problema en el cliente. Esto se hace usando la sugerencia proporcionada por el mensaje de advertencia:

```bash
$ ssh-keygen -f "/home/me/.ssh/known_hosts" -R "remote-sys"
```

Fallando eso, podemos usar un editor de texto (vim quizás) para remover la obsoleta clave del archivo `~/.ssh/known_hosts`. En el ejemplo anterior, vemos esto:

```
Offending key in /home/me/.ssh/known_hosts:42
```

Esto significa que la línea 42 del archivo `known_hosts` contiene la clave ofensiva. Elimina esta línea del archivo, y el programa `ssh` será capaz de aceptar nuevas credenciales de autenticación del sistema remoto.

#### Ejecutar Comandos Remotos

Además de abrir una sesión en un host remoto, `ssh` nos permite ejecutar un comando individual en un host remoto y tener los resultados mostrados en el sistema local. Por ejemplo, para ejecutar el comando `free` en un host remoto llamado `remote-sys` y tener los resultados mostrados en el sistema local:

```bash
$ ssh remote-sys free
me@remote-sys's password:
              total        used        free       shared      buffers       cached
Mem:         775536      507184      268352           0       110068       154596
-/+ buffers/cache:      242520      533016
Swap:       1572856           0     1572856
[me@linuxbox ~]$
```

Es posible usar esta técnica de formas más interesantes, como la siguiente en la que realizamos un `ls` en el sistema remoto y redirigimos la salida a un archivo en el sistema local:

```bash
$ ssh remote-sys 'ls *' > dirlist.txt
me@remote-sys's password:
[me@linuxbox ~]$
```

Nótese el uso de comillas simples en el comando anterior. Esto se hace porque no queremos que la expansión de nombre de ruta se realice en la máquina local; en su lugar, queremos que se realice en el sistema remoto. De la misma manera, si hubiéramos querido que la salida se redirigiera a un archivo en la máquina remota, podríamos haber colocado el operador de redirección y el nombre de archivo dentro de las comillas simples:

```bash
$ ssh remote-sys 'ls * > dirlist.txt'
```

### Tunneling con SSH

Parte de lo que sucede cuando estableces una conexión con un host remoto vía SSH es que se crea un **túnel encriptado** entre el sistema local y remoto. Normalmente, este túnel se usa para permitir que comandos escritos en el sistema local se transmitan de forma segura al sistema remoto y para que los resultados se transmitan de forma segura hacia atrás. Además de esta función básica, el protocolo SSH permite que la mayoría de tráfico de red se envíe a través del túnel encriptado, creando una especie de **red privada virtual (VPN)** entre el sistema local y remoto.

Quizás el caso de uso más común para esta característica es permitir que un cliente X Window se ejecute en el host remoto. Es fácil de hacer; por ejemplo, digamos que estamos sentados en un sistema Linux llamado `linuxbox` que está ejecutando un servidor X, y queremos ejecutar el programa `xload` en un sistema remoto llamado `remote-sys` para ver la salida gráfica en nuestro sistema local. Podríamos hacer esto:

```bash
$ ssh -X remote-sys
me@remote-sys's password:
Last login: Mon Sep 08 13:23:11 2025
[me@remote-sys ~]$ xload
```

Después de que el comando `xload` se ejecuta en el sistema remoto, su ventana aparece en el sistema local. En algunos sistemas, podrías necesitar usar la opción `-Y` en lugar de la opción `-X` para hacer esto.

### scp y sftp

El paquete OpenSSH también incluye dos programas que pueden hacer uso de un túnel encriptado SSH para copiar archivos a través de la red. El primero, **scp** (secure copy), se usa muy similar al programa familiar `cp`. La diferencia más notable es que la fuente o destino del nombre de ruta pueden estar precedidos con el nombre de un host remoto seguido por un carácter de dos puntos. Por ejemplo, si quisiéramos copiar un documento llamado `document.txt` desde nuestro directorio principal en el sistema remoto, `remote-sys`, al directorio de trabajo actual en nuestro sistema local, podríamos hacer esto:

```bash
$ scp remote-sys:document.txt .
me@remote-sys's password:
document.txt                               100S 5581      5.5KB/s  00:00
[me@linuxbox ~]$
```

Como con `ssh`, puedes aplicar un nombre de usuario al inicio del nombre del host remoto si el nombre de cuenta remoto deseado no coincide con el del sistema local:

```bash
$ scp bob@remote-sys:document.txt .
```

El segundo programa SSH file-copying es **sftp**, que, como su nombre implica, es un reemplazo seguro para el programa `ftp`. `sftp` funciona muy similar al programa `ftp` original que usamos anteriormente; sin embargo, en lugar de transmitir todo en texto claro, usa un túnel SSH encriptado. `sftp` tiene una ventaja importante sobre `ftp` convencional en que no requiere un servidor FTP para estar ejecutándose en el host remoto. Requiere solo el servidor SSH. El cliente SSH también puede ser usado como un servidor FTP-like. Aquí hay una sesión de muestra:

```bash
$ sftp remote-sys
Connecting to remote-sys...
me@remote-sys's password:
sftp> ls
ubuntu-24.04-desktop-amd64.iso
sftp> lcd Desktop
sftp> get ubuntu-24.04-desktop-amd64.iso
Fetching /home/me/ubuntu-24.04-desktop-amd64.iso to ubuntu-24.04-desktop-amd64.iso
/home/me/ubuntu-24.04-desktop-amd64.iso 100%   69mB   -7.4MB/s  01:35
sftp> bye
```

> **Nota**: El protocolo SFTP es soportado por muchos gestores de archivos gráficos encontrados en distribuciones Linux. Usando GNOME o KDE, podemos entrar en una URI comenzando con `sftp://` en la barra de ubicación y operar en archivos almacenados en un sistema remoto ejecutando un servidor SSH.

#### SSH Client para Windows

¿Digamos que estás sentado en una máquina Windows, pero necesitas conectarte a tu servidor Linux y hacer algo de trabajo real? ¿Qué haces? ¡Obten un programa cliente SSH para tu box Windows, por supuesto! Hay un número de estos. El más popular es probablemente PuTTY por Simon Tatham y su equipo. El programa muestra una ventana de terminal y permite que un usuario Windows abra una sesión SSH (o telnet) en un host remoto. El programa también proporciona análogos para los programas `scp` y `sftp`.

PuTTY está disponible en http://www.chiark.greenend.org.uk/~sgtatham/putty/.

## Resumen

En este capítulo, hemos examinado una serie de herramientas de redes encontradas en la mayoría de sistemas Linux. Ya que Linux se usa ampliamente en servidores y dispositivos de redes, hay muchos más que pueden ser agregados instalando software adicional. Pero incluso con el conjunto básico de herramientas, es posible realizar muchas tareas útiles relacionadas con redes.

---

## Ver También

- [[wiki/linux/01-introduccion-shell.md|Capítulo 1: Introducción al Shell de Linux]]
- [[wiki/linux/06-redirection.md|Capítulo 6: Redirección (I/O Redirection)]] (Útil para redirigir salida remota)
