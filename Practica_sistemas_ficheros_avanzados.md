# ZFS

### Creación de escenario
Se va a utilizar una máquina con Debian 10, 1024MB de RAM. Además, se van a añadir 4 discos de 1GB para hace3r pruebas con el sistema de fichero ZFS.
~~~
vagrant@nfs:~$ lsblk -l
NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda    8:0    0 19.8G  0 disk 
sda1   8:1    0 18.8G  0 part /
sda2   8:2    0    1K  0 part 
sda5   8:5    0 1021M  0 part [SWAP]
sdb    8:16   0    1G  0 disk 
sdc    8:32   0    1G  0 disk 
sdd    8:48   0    1G  0 disk 
sde    8:64   0    1G  0 disk 
~~~

### Instalación de ZFS
Para instalar ZFS en Debian se tiene que compilar un paquete que se descargará de un reposotorio de gitHub. Pero antes se necesitan descargar los siguientes paquetes:
~~~
vagrant@nfs:~$ sudo apt install build-essential autoconf automake libtool gawk alien fakeroot ksh zlib1g-dev uuid-dev libattr1-dev libblkid-dev libselinux-dev libudev-dev libacl1-dev libaio-dev libdevmapper-dev libssl-dev libelf-dev
vagrant@nfs:~$ sudo apt install linux-headers-$(uname -r)
~~~

Tras instalar todos los paquetes necesarios se descarga el contenido del repositorio:
~~~
vagrant@nfs:~$ wget https://github.com/zfsonlinux/zfs/releases/download/zfs-0.8.2/zfs-0.8.2.tar.gz
~~~

A conitnuación, se utilizar tar y se cambian los permisos del directorio:
~~~
vagrant@nfs:~$ tar axf zfs-0.8.2.tar.gz
vagrant@nfs:~$ cd zfs-0.8.2
vagrant@nfs:~/zfs-0.8.2$ sudo chown -R root:root ./
vagrant@nfs:~/zfs-0.8.2$ sudo chmod -R o-rwx ./
~~~

Se establecen las opciones de configuración según el sistema en el que se vaya a instalar:
~~~
root@nfs:/home/vagrant/zfs-0.8.2# ./configure \
> --disable-systemd \
> --enable-sysvinit \
> --disable-debug \
> --with-spec=generic \
> --with-linux=$(ls -1dtr /usr/src/linux-headers-*.*.*-common | tail -n 1) \
> --with-linux-obj=$(ls -1dtr /usr/src/linux-headers-*.*.*-amd64 | tail -n 1)
~~~

Y se compila:
~~~
root@nfs:/home/vagrant/zfs-0.8.2# make -j1 && make install
~~~

Por último, se instala el script de inicio del sercicio:
~~~
root@nfs:/home/vagrant/zfs-0.8.2# cd /etc/init.d/
root@nfs:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-import /etc/init.d/
root@nfs:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-mount /etc/init.d/
root@nfs:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-share /etc/init.d/
root@nfs:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-zed /etc/init.d/
~~~

Y se establece el servicio y se activa el módulo:
~~~
root@nfs:/etc/init.d# update-rc.d zfs-import defaults
root@nfs:/etc/init.d# update-rc.d zfs-mount defaults
root@nfs:/etc/init.d# update-rc.d zfs-share defaults
root@nfs:/etc/init.d# update-rc.d zfs-zed defaults
root@nfs:/etc/init.d# modprobe zfs
~~~

> Comprobación:
~~~
root@nfs:/etc/init.d# lsmod | grep zfs
zfs                  3760128  0
zunicode              335872  1 zfs
zlua                  172032  1 zfs
zcommon                90112  1 zfs
znvpair                90112  2 zfs,zcommon
zavl                   16384  1 zfs
icp                   311296  1 zfs
spl                   114688  5 zfs,icp,znvpair,zcommon,zavl
~~~

Tras esto se debe reiniciar el equipo.

### Gestiona los discos adicionales con ZFS
La salida del comando de comprobación del estado de los discos indica que no están disponibles porque aún no se han configurado:
~~~
root@nfs:/home/vagrant# zpool status
no pools available
~~~

#### Configuración de los RAID
Se va a configura los discos en RAID, haciendo pruebas de fallo de algún disco y sustitución, restauración del RAID. 

##### Creción del RAID1
Se va a crear un pool con RAID 1 con los discos sdb, sdc y sdd:
~~~
vagrant@nfs:~$ sudo zpool create -f RAID1 mirror /dev/sdb /dev/sdc /dev/sdd
~~~

Se comprueba el estado de los pools:
~~~
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0

errors: No known data errors
~~~

Se va a añadir el cuarto disco como reserva:
~~~
vagrant@nfs:~$ sudo zpool add -f RAID1 spare /dev/sde
~~~

Se vuelve a comprobar el estado, ahora con el nuevo disco como reserva y con la etiqueta spare:
~~~
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	spares
	  sde       AVAIL   

errors: No known data errors
~~~

##### Fallo en RAID1
Se marca un disco como fallido:
~~~
vagrant@nfs:~$ sudo zpool offline -f RAID1 /dev/sdc
~~~

Y se vuelve a comprobar el estado, viendo como ahora el disco marcado como fallado está en estado FAULTED:
~~~
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
	repaired.
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       DEGRADED     0     0     0
	  mirror-0  DEGRADED     0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     FAULTED      0     0     0  external device fault
	    sdd     ONLINE       0     0     0
	spares
	  sde       AVAIL   

errors: No known data errors
~~~

Para reemplazar el disco fallido por el disco de reserva se utiliza el siguiente comando:
~~~
vagrant@nfs:~$ sudo zpool replace -f RAID1 /dev/sdc /dev/sde
~~~

> Comprobación:
~~~
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
	repaired.
  scan: resilvered 286K in 0 days 00:00:00 with 0 errors on Mon Jan 20 17:17:38 2020
config:

	NAME         STATE     READ WRITE CKSUM
	RAID1        DEGRADED     0     0     0
	  mirror-0   DEGRADED     0     0     0
	    sdb      ONLINE       0     0     0
	    spare-1  DEGRADED     0     0     0
	      sdc    FAULTED      0     0     0  external device fault
	      sde    ONLINE       0     0     0
	    sdd      ONLINE       0     0     0
	spares
	  sde        INUSE     currently in use

errors: No known data errors
~~~

A continuación se va a restaurar el disco fallido:
~~~
vagrant@nfs:~$ sudo zpool clear RAID1 /dev/sdc
~~~

Y en la comprobación se observa que ahora todos están ONLINE y el cuarto disco ha vuelto a la reserva:
~~~
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: ONLINE
  scan: resilvered 72K in 0 days 00:00:01 with 0 errors on Mon Jan 20 17:19:13 2020
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	spares
	  sde       AVAIL   

errors: No known data errors
~~~

##### Automaticación de RAID1
Para automatizar todos los procesos anteriores hay que modificar el parámetro autoreplace de la configuración de zfs que se encuentra en off:
~~~
vagrant@nfs:~$ sudo zpool get all RAID1 | grep autoreplace
RAID1  autoreplace                    off                            default
~~~

Este parámetro se modifica de la siguiente forma:
~~~
vagrant@nfs:~$ sudo zpool autoreplace=on RAID1
~~~

Y se comprueba el cambio:
~~~
vagrant@nfs:~$ sudo zpool get all RAID1 | grep autoreplace
RAID1  autoreplace                    on                             local
~~~

##### Restauración del RAID1 - checkpoint
Se va a crear un checkpoint para, en caso de error, se pueda restaurar el raid a la versión en la cual estaba cuando se hizo el checkpoint:
~~~
vagrant@nfs:~$ sudo zpool checkpoint RAID1
~~~

Ahora, el estado del checkpoint del RAID es activo:
~~~
vagrant@nfs:~$ sudo zpool get all RAID1 | grep checkpoint
RAID1  checkpoint                     136K                           -
RAID1  feature@zpool_checkpoint       active                         local
~~~

> Comprobación:
Se va a comprobar el funcionamiento del checkpoint ante fallos. En primer lugar, se hace que dos discos fallen:
~~~
vagrant@nfs:~$ sudo zpool offline -f RAID1 /dev/sdc /dev/sdd
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
	repaired.
  scan: resilvered 27K in 0 days 00:00:00 with 0 errors on Mon Jan 20 17:28:43 2020
checkpoint: created Mon Jan 20 17:31:39 2020, consumes 141K
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       DEGRADED     0     0     0
	  mirror-0  DEGRADED     0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     FAULTED      0     0     0  external device fault
	    sdd     FAULTED      0     0     0  external device fault
	spares
	  sde       AVAIL   

errors: No known data errors
~~~

Se exporta el raid y se importa el checkpoint:
~~~
vagrant@nfs:~$ sudo zpool export RAID1
vagrant@nfs:~$ sudo zpool status
no pools available
vagrant@nfs:~$ sudo zpool import --rewind-to-checkpoint RAID1
vagrant@nfs:~$ sudo zpool status
  pool: RAID1
 state: ONLINE
  scan: resilvered 27K in 0 days 00:00:00 with 0 errors on Mon Jan 20 17:28:43 2020
config:

	NAME        STATE     READ WRITE CKSUM
	RAID1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	spares
	  sde       AVAIL   

errors: No known data errors
~~~

##### Creación del RAIDZ
RAID Z implementa un esquema de redundancia paredico al RAID 5. Se pueden realizar 3 tipos de RAIDZ. A continuación, se muestra un ejemplo de cómo realizar un RAIDZ 2 que tiene 2bits de paridad, por lo tanto necesita 4 discos:
~~~
vagrant@nfs:~$ sudo zpool create -f RAIDZ raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde
vagrant@nfs:~$ sudo zpool status -v RAIDZ
  pool: RAIDZ
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	RAIDZ       ONLINE       0     0     0
	  raidz2-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	    sde     ONLINE       0     0     0

errors: No known data errors
~~~

Además, se añade un disco de reserva:
~~~
vagrant@nfs:~$ sudo zpool add RAIDZ spare sdf
vagrant@nfs:~$ zpool status
-bash: zpool: command not found
vagrant@nfs:~$ sudo zpool status
  pool: RAIDZ
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	RAIDZ       ONLINE       0     0     0
	  raidz2-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	    sde     ONLINE       0     0     0
	spares
	  sdf       AVAIL   

errors: No known data errors
~~~

Si uno de los discos falla, al tener 2bits de paridad no hay pérdida de datos.

#### Ventajas e incovenientes mdadm/ZFS
**Ventajas de mdadm frente a ZFS**
- Se encuentra incluida en el kernel Linux. Quizás la ventaja más importante frente a ZFS. Esto es gracias a la licencia GPL de mdadm. ZFS tiene una licencia DDL que no es compatible con el kernel Linux.

**Ventajas ZFS**
- ZFS se apoya en espacios de almacenamiento virtuales (storage pools) que permite dinámicamente ir agregando discos físicos. 
- La nueva información se escribe en discos diferentes lo que asegura que en caso de fallo en la sobreescritura los datos antiguos serán preservados. En mdadm los datos se pierden al ser sobreescritos.
- Uso del modelo transaccional **copy-on-write**. 
- Mayor tamaño máximo -> 256 cuatrillones de Zettabytes 

> Las ventajas de ZFS frente a mdadm son muchas pero el único inconveniente que nos encontramos, la imposibilidad de que ZFS se incluya en el kérnel Linux, hace que no se pueda sacar todo el potencial de este sistema de ficheros.


#### Realiza ejercicios con pruebas de funcionamiento de las principales funcionalidades: compresión, cow, deduplicación, cifrado, etc.

##### Compresión
Los ficheros comprimidos en ZFS, desde el punto de vista de la aplicación, parece no estar comprimido. En cambio, ZFS está comprimiento y descomprimiendo los datos en el disco sobre la marcha. 


El estado de la compresión en principio está apagado:
~~~
vagrant@nfs:~$ sudo zfs get compression RAID1
NAME   PROPERTY     VALUE     SOURCE
RAID1  compression  off       default
~~~

Se va a activar la compresión lz4:
~~~
vagrant@nfs:~$ sudo zfs set compression=lz4 RAID1
~~~

Y se comprueba que se ha cambiado el estado de la compresión:
~~~
vagrant@nfs:~$ sudo zfs get compression RAID1
NAME   PROPERTY     VALUE     SOURCE
RAID1  compression  lz4       local
~~~

> Comprobación:
Se va a crear un agrupamiento de ficheros con tar de /home y /etc en un directorio previamente creado:
~~~
vagrant@nfs:/RAID1$ sudo mkdir compression
vagrant@nfs:/RAID1$ sudo tar -cf /RAID1/compression/prueba.tar /home/ /etc/
~~~

Se comprueba el tamaño:
~~~
vagrant@nfs:/RAID1$ ls -lh /RAID1/compression/prueba.tar 
-rw-r--r-- 1 root root 505M Jan 20 18:28 /RAID1/compression/prueba.tar
vagrant@nfs:/RAID1$ sudo zfs list /RAID1/compression
NAME    USED  AVAIL     REFER  MOUNTPOINT
RAID1   231M   601M      230M  /RAID1
vagrant@nfs:/RAID1$ sudo zfs get compressratio /RAID1/compression
NAME   PROPERTY       VALUE  SOURCE
RAID1  compressratio  2.18x  -
~~~


##### Deduplicación
La deduplicación es una técnica muy útil para optimizar el almacenamiento donde se elimina los datos redundantes de los datos almacenados.

Hay 3 formas de realizar la deduplicación de un fichero:
- Por fichero.
- Por bloque.
- Por byte.

La deduplicación por fichero es la más acertada. A través de la codificación de ficheros, si el hash del mismo coincide con otro, se hace referencia a este en lugar de almacenar el nuevo archivo en disco. Pero, una modificación en el fichero hace que haya que hacerse una copia de uno o varios bloques en el disco. 

Para habilitar la deduplicación hay que cambiar la propiedad debup a on:
~~~
vagrant@nfs:/RAID1$ sudo zfs set dedup=on RAID1
~~~

Se van a realizar 3 copias del fichero .tar que se creó anteriormente:
~~~
vagrant@nfs:/RAID1/compression$ sudo cp prueba.tar prueba1.tar
vagrant@nfs:/RAID1/compression$ sudo cp prueba.tar prueba2.tar
vagrant@nfs:/RAID1/compression$ sudo cp prueba.tar prueba3.tar
~~~

Se observa cuanto opcupa la suma de los megas del fichero y las copias no corresponden al volumen de megas usados:
~~~
vagrant@nfs:/RAID1/compression$ ls -lh
total 922M
-rw-r--r-- 1 root root 505M Jan 20 19:01 prueba1.tar
-rw-r--r-- 1 root root 505M Jan 20 19:02 prueba2.tar
-rw-r--r-- 1 root root 505M Jan 20 19:03 prueba3.tar
-rw-r--r-- 1 root root 505M Jan 20 18:28 prueba.tar
vagrant@nfs:/RAID1/compression$ sudo zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
RAID1   924M   368M      922M  /RAID1
~~~

##### CoW
La técnica de almacenamiento de datos Copy-on-write realiza una copia del bloque que se va a modificar en lugar de modificar el bloque de datos directamente. Toma instantáneas de los datos permitiendo el rescate en caso de fallo.























