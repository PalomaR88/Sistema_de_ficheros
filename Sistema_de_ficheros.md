# Sistema de ficheros (filesystem)
Un sistema de ficheros es una capa entre los ficheros y un sistema de bloques donde se indica la ruta, los directorios y el ugoa (permisos). Pero además, también aporta el índice, la posición y dispoción de los ficheros, etc. 

Controla cómo se almacenan y obtienen los datos. Tiene una capa lógica responsable de la interacción con los usuarios, API y del modelo de seguridad. 

**Fichero**: es un conjunto de datos almacenados en un dispositivo de almacenamiento. Un fichero posee un nombre y metadatos. El tipo de datos contenidos está definido por su formato y en ocasiones por su extensión (en sistemas Linux la extensión es anecdótica).

> Uso del comando **file** y vista de las cabeceras de un fichero con otra extensión que no es la propia.
~~~
paloma@coatlicue:~/Descargas$ ls
 El_delegado.png
paloma@coatlicue:~/Descargas$ file El_delegado.png 
El_delegado.png: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 434x545, components
paloma@coatlicue:~/Descargas$ xxd El_delegado.png | head
00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......
00000010: 0001 0000 ffdb 0043 0002 0101 0101 0102  .......C........
00000020: 0101 0102 0202 0202 0403 0202 0202 0504  ................
00000030: 0403 0406 0506 0606 0506 0606 0709 0806  ................
00000040: 0709 0706 0608 0b08 090a 0a0a 0a0a 0608  ................
00000050: 0b0c 0b0a 0c09 0a0a 0aff db00 4301 0202  ............C...
00000060: 0202 0202 0503 0305 0a07 0607 0a0a 0a0a  ................
00000070: 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a  ................
00000080: 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a  ................
00000090: 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a ffc0  ................
~~~

Normalmente el formato está definido en la cabecera (magic number). Un fichero solo con datos ASCII o UTF8 se conoce como texto plano (plain text). Los fichetos con datos reciben el nombre de ficheros regulares.

**Ficheros especiales**
Aparte de los ficheros regulares, en UNIX “todo es un fichero”:
- Enlaces
- sockets
- Dispositivos (En /dev)
- Ficheros virtuales (En /proco/sys)
- Ficheros virtuales en memoria (Tipo tmpfs o ramfs)

**Directorios**
Los directorios son contenedores de ficheros y se pueden anidar(subdirectorios). El directorio principal es el directorio raız o /. Los directorios se organizan formando un  ́arbol a partir deldirectorio raız. Los ficheros se definen de forma  ́unica por el nombre que incluye suruta completa desde el directorio raız, p.ej. /usr/share/doc/apt/copyright. La estructura de los directorios es estricta y definida en el LinuxFilesystem Hierarchy Standard (FHS) de la Linux Foundation. Las diferentes distros de GNU/Linux deben seguir la FHS

**Dispositivos de bloques**
Son ficheros especiales ubicados en /dev, dispositivos de caracteres.

#### Linux Vistual FileSystem (VFS)
Capa de kérnel que proporciona una interfaz única a usuarios y rpogramas. Implementa un único árbol aunque esté formado por varios dispositivos de bloques y/o tipos de sistemas de ficheros. 

De esta forma, por ejemplo, con un pen con sistema NTFS, que es monousuario y no tiene grupos, cuando se monta aparecerá que el usuario y el grupo es el propio, porquue se "traduce" a sistema multiusuario a través del kérnel Linux. 

Se puede open, stat, read, write y chmod. Y permite la opción montar y desmontar. 

Elementos principales de cada tipo de distema de ficheros:
- Superbloques: contiene la información, los metadatos de cada sistema de fihceros. 
- Inodo: se refleja dondo se encuentra el fichero, la ubicación.

Todos los sitemas de ficheros son diferentes, pero todos tienen superbloques e inodo. 

#### DAS, NAS y SAN
- Direct Attached Storage(DAS) Almacenamiento directo.
- Network Attached Storage(NAS) Almacenamiento en red. Para montar o se accede a un sistema de ficheros remoto.
- Storage Area Network(SAN) Red de almacenamiento específica dedicadda de alta capacidad que permite, no obtener ficheros sino obtener sistemas de ficheros en grupos. Es algo específico de los centros de datos. Se confunde con NAS porque la misma máquina que ofrece SAN puede ofrecer NAS, pero es importante diferenciarlo.


## Sistemas de ficheros clásicos
#### FAT16 - FAT32 (evolución)
- Limitaciones importantes en cuanto a tamaño, porque es de los años 80, y el límite es de 4 GiB. 
- Es monousuario, no guarda información sobre usuarios (porque es de MS2 y este sistema era monousuario).

#### NTFS
Sistema de ficheros de WindowsNT. Incluye mejoras con respecto a FAT32.
- Multiusuario (pero no ugoa, porque no guarda permisos).
- Las limitaciones de tamaño de un fichero es de 16 TiB, que actualmente si puede ser un problema.
- Journaling

#### ext2
Sistema de ficheros tipo UNIX. 
- Multiusuario.
- ugoa (usuarios, privilegios, etc)
- Límite de ficheros 2TiB.
- No dispone de journaling.

#### ext3
Para suplir la limitación de ext2 que no tenía journaling.
- Journaling, es el control transaccional. Si una transacción se interrumpe, los cambios o se han realizado o no, pero no se queda a medias. 

#### ext4
Funciona más rápido que ext3 y son mejores los límites de tamaño. Es el sistema de ficheros por defecto en linux. Características:
- Compatible con ext2/3.
- Utiliza journaling, algo muy importante.
- Hasta 1 EiB con ficheros de 16 TiB.
- Mejoras en el rendimiento frente a ext2/3.

Herramientas del espacio de usuario: e2fsprogs: 
- mkfs.ext4 (formateo)
- fsck.ext4 (chequeo)
- tune2fs (modificación)
- resize2fs (redimensionar)
- dumpe2fs (volcado, para llevarse un sistema de ficheros en bruto de un dispositivo de bloque a otro)
- debugfs...

Pero no es una tecnología rompedora, es el mismo sistema de loso 90 con algunas mejoras. 

#### xfs
Desarrollado originalmente por SGI. Utilizado por defecto en RHEL 7. Hasta 8 EiB. Utiliza journaling. **Permite redimensionado en caliente** y esta es la característica más importnate.

Herramientas del espacio de usuario: xfsprogs:
- mkfs.xfs
- fsck.xfs 
- xfsdb 
- xfsgrowfs 
- xfsinfo ...

#### vfat
Sistema de ficheros de MS-DOS/Windows. Superado. Pero se sigue usando porque estamos completamente seguro de que a cualquier cosa a la que lo conectemos, lo va a reconocer. Tiene grandes limitaciones, pero se utiliza masívamente en dispositivos extraíbles.

Herramientas del espacio de usuario:dosfstools:
- mkfs.vfat
- fsck.vfat
- dosfslabel

## Sistemas de ficheros. Funcionalidad avanzada


#### Copy on Write(CoW)
Se descomponen los datos a almacenar en diferentes bloques. Al cerrar la copia solo se crea un nuevo puntero que apunta al conjunto de datos originales. Cuando los datos copiados se modifican, se van creando nuevos bloques con las modificaciones. Las versiones divergen a medida que van cambiando. 

reflink habilitado en un sistema de ficheros (reflink =1) significa que el copy on write está activo. 

De la siguiente forma se crea una copia copy on write:
~~~
cp --reflink fich fich-cp
~~~

En el sistema de ficheros, esta copia no ocupa el espacio que debería ocupar una copia de este fichero, es decir, el peso del fichero original duplicado, porque no es una copia convencional. 

Si se realiza un mapeo del sistema de ficheros, se ve que los bloques son los mismo.
~~~
xfs_bmap -v fich fich-cp
~~~

Ahora bien, al realizar una modificación ya si hay una diferencia, muy pequeña en el espacio ocupado. Y en un mapeo se observa que algunos bloques ya son diferentes. 


#### Deduplicacion
Utiliza las propiedades Copy on Write de un sistema de ficheros. Identifica bloques idénticos y los reorganiza con Cow. Permite aprovechar la funcionalidad CoW sin intervención del usuario. 

Ejemplo: En un directorio con las cuentas de correo de todos los usuarios de la organizacion se guarda solo una copia del fichero adjunto enviado a todas las cuentas.


#### Cifrado
Permite el cifrado de ficheros al vuelo sin utilizar software adicional. Esto es la implementación al nivel del sistema de ficheros no con una herramienta por encima. 

#### Compresion
Almacena los ficheros comprimidos para optimizar el uso del espacio. 

#### Gestion de volumenes
Equivalente a LVM, pero sin LVM. Permite gestional volúmen y sistemas de ficheros de forma independiente de los dispositivos de bloques físicos. Un dispositivo de bloques sísico puede contener varios volúmenes o un volumen puede estar distribuido en varios dispositivos de bloques físicos. 

#### Instantaneas (snapshots)
Utiliza instantáneas de forma nativa. 

#### Sumas de comprobación (checksums)
Utilizadas para verificar la integridad de los ficheros. Hay sistemas de ficheros que me permiten tener checksums al vuelo. Hay otros que tienes que meter el comando y tarde en realizarse, pero los que tiene esta opción se van guardando instantáneas de cada momento. Y para las restauraciones, las sumas de comprobación, son muy útiles. 

#### Redundancia (RAID)
RAIS software nativo en el sistema de ficheros. Sin necesidad de usar mdadm. 

Relación sistemas de ficheros y funcionalidad de aceptan:
![funcionalidades_sist_fich](/images/aimg.png)

## ZFS
Antes que Btrfs se desarrolla ZFS. Se desarrolla en Solaris, en el año 2001. En el año 2005 aparece OpenSolaris, que es software libre, pero ZFS se libera con CDDL, no GPL. La gente del kernel no peude incluirlo por esto. Y es cuando se desarrolla ZFS on FUSE, es decir que se puede utilizar pero a nivel de usuario, no en el kernel y esto es muy ineficaz para un sistema de ficheros. 

Además, en 2011 Oracle compra Sun Microsustems, desarrollador de OpenSolaris y elimina open de solaris. Pero de OpenSolaris sale un fork llamado Illumos (Oracle sigue con su ZFS en su Solaris) que saca OpenZFS y saca ZFS on linux, que sigue siendo CDDL. 

La limitación con el kernel línux ha hecho que ZFS, considerado por muchos el mejor sistema de ficheros durante mucho tiempo, no triunfara. Pero se han buscado alternativas. Por ejemplo, en el instalador de ubuntu (hay quien dice que no es del todo legal). 

#### El proyecto
- 2001: Comienza el desarrollo en Sun Microsystems para Solaris
- 2005: Se libera con licencia CDDL y se incluye en OpenSolaris. CDDL es incompatible con GPL: ZFS no puede incluirse en el kérnel linux
- 2006: Se desarrolla ZFS sobre FUSE para sistemas linux, sin módulo en el kérnel. Esto tiene penalizaciones importantes en rendimiento.
- 2010: Oracle compra Sun Microsystems y abandona OpenSolaris. ZFS vuelve a ser software cerrado. Pero de lo que está liberado se puede hacer un fork. 
- 2010: Se crea Illumos, fork libre de OpenSolaris
- 2013: Se anuncia OpenZFS como fork libre de ZFS
- 2015: Aparece ZFS on Linux, versión de OpenZFS para linux. Pero sigue teniendo el problema de ser una CDDL.

#### Características de OpenZFS
- Es un sistema completo de almacenamiento que no requiere otras herramientas
- Gestiona los dispositivos de bloques directamente
- Incluye su propia implementación de RAID
- CoW, deduplicación, instantáneas, compresión, cifrado, soportenativo de nfs, cifs o iscsi, . . .
- Siempre consistente sin necesidad de chequeos. No necesita chequeos, porque se autochequea continuamente y se autorepara.
- Se autorepara de forma continua
- Muy escalable
- Exigente en recursos

#### [Práctica: configura ZFS en Debian](URL)


## Btrfs
En 2008 comienza el desarrollo por empleados de Oracle. Se incluye Btrfs en el kérnel linux. No existe suficientes desarrolladores para el proyecto, por lo que ha tardado mucho. Es similar a ZFS pero está en el kérnel Linux. 

# LVM

# RAID
## Intro presentación
## Uso de RAID software
## Uso de RAID hardware

# NAS
## Usos y protocolos habituales
## NFS
## SMB y CIFS

# SAN
## Características Presentación - Vídeo
## iSCSI
## Ceph RBD

# Sistemas de ficheros en clúster
## Características
## Sistemas de dispositivos de bloques compartidos
#### DRBD
#### OCFS2
#### GFS2
## Sistemas de ficheros distribuidos
#### GlusterFS
#### CephFS
#### Windows DFS
#### Lustre

# Almacenamiento de objetos en la nube