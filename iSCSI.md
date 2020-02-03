**Configura un sistema que exporte algunos targets por iSCSI y los conecte a diversos clientes, explicando con detalle la forma de trabajar.**
La creación del escenario se va a realizar con vagrant, la configuración se encuentra en el [siguiente enlace](https://github.com/PalomaR88/Sistema_de_ficheros/blob/master/iSCSI/Vagrantfile). Se ha creado una máquina debian con 3 volúmenes asociados y dos máquinas clientes, una debian y otra windows. Todas tienen dos interfaces de red, un enlace a internet y una red interna. Lo correcto sería que el servidor no tuviera conexión con internet, pero para la configuración es necesaria. 

**- Crea un target con una LUN y conéctala a un cliente GNU/Linux. Explica cómo escaneas desde el cliente buscando los targets disponibles y utiliza la unidad lógica proporcionada, formateándola si es necesario y montándola.**

### Configuración desde el servidor
Los volúmenes que aparecen en el servidor son:
~~~
vagrant@Servidor1:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0    1G  0 disk 
sdc      8:32   0    1G  0 disk 
sdd      8:48   0    1G  0 disk 
~~~

Para el manejo de los volúmenes se descarga el paquete LVM:
~~~
vagrant@Servidor1:~$ sudo apt install lvm2
~~~

Se inicia el volumen físico para el uso de LVM:
~~~
vagrant@Servidor1:~$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
~~~

Y el grupo de volumenes:
~~~
vagrant@Servidor1:~$ sudo vgcreate vg1 /dev/sdb
  Volume group "vg1" successfully created
~~~

Por último, se crea el volumen:
~~~
vagrant@Servidor1:~$ sudo lvcreate -L 700M -n vol1 vg1
  Logical volume "vol1" created.
~~~

Y así se encuentra el volumen:
~~~
vagrant@Servidor1:~$ sudo lvs
  LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vol1 vg1 -wi-a----- 700.00m                                                    
~~~

Se instala el paquete tgt:
~~~
vagrant@Servidor1:~$ sudo apt install tgt
~~~

Para crear el target se configura el fichero **/etc/tgt/targets.conf** con la siguiente configuración:
~~~
<target iqn.2020-01.com:tg1>
    backing-store /dev/vg1/vol1
</target>
~~~

Se reinicia el servicio:
~~~
vagrant@Servidor1:~$ sudo systemctl restart tgt.service 
~~~

Y se comprueba que el target se ha creado correctamente:
~~~
vagrant@Servidor1:~$ sudo tgtadm --mode target --op show
Target 1: iqn.2020-01.com:tg1
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00010000
            SCSI SN: beaf10
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags: 
        LUN: 1
            Type: disk
            SCSI ID: IET     00010001
            SCSI SN: beaf11
            Size: 734 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/vg1/vol1
            Backing store flags: 
    Account information:
    ACL information:
        ALL
~~~


### Configuración en el cliente

Para iniciar iSCSI en el cliente hay que instalar el paquete open-iscsi:
~~~
vagrant@Cliente1:~$ sudo apt install open-iscsi 
~~~

A continuación se edita el fichero **/etc/iscsi/iscsid.conf** y se indica que se iniciará de forma automática:
~~~
iscsid.startup = automatic   
~~~

Y se reinicia el servicio:
~~~
vagrant@Cliente1:~$ sudo systemctl restart open-iscsi.service 
~~~

Para conectarse a los iSCSI del servidor creado:
~~~
vagrant@Cliente1:~$ sudo iscsiadm -m discovery -t sendtargets -p 192.168.100.1
192.168.100.1:3260,1 iqn.2020-01.com:tg1
~~~

Y se comprueba que se ha conectado:
~~~
vagrant@Cliente1:~$ sudo iscsiadm -m node
192.168.100.1:3260,1 iqn.2020-01.com:tg1
~~~

Para conectarse:
~~~
vagrant@Cliente1:~$ sudo iscsiadm -m node --targetname "iqn.2020-01.com:tg1" --portal "192.168.100.1" --login
Logging in to [iface: default, target: iqn.2020-01.com:tg1, portal: 192.168.100.1,3260] (multiple)
Login to [iface: default, target: iqn.2020-01.com:tg1, portal: 192.168.100.1,3260] successful.
~~~

> Para desconectarse de un target:
~~~
iscsiadm -m node --targetname "<nombreDelTarget>" --portal "<direcciónIPDelServidor>" --logout
~~~

Y de esta forma, aparece el nuevo disco remoto en la máquina cliente:
~~~
vagrant@Cliente1:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0 19.8G  0 disk 
├─sda1   8:1    0 18.8G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0 1021M  0 part [SWAP]
sdb      8:16   0  700M  0 disk 
~~~

Para formatear el disco se va a utilizar el comando **fdisk**:
~~~
vagrant@Cliente1:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xabf8043d.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p 
Partition number (1-4, default 1): 
First sector (2048-1433599, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1433599, default 1433599): 

Created a new partition 1 of type 'Linux' and of size 699 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
~~~

Y, por último, se formatea y se monta:
~~~
vagrant@Cliente1:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 178944 4k blocks and 44736 inodes
Filesystem UUID: e8a6336e-9823-4181-b557-b7a674ec11df
Superblock backups stored on blocks: 
	32768, 98304, 163840

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@Cliente1:~$ sudo mount /dev/sdb1 /mnt
~~~

El resultado es el siguiente:
~~~
vagrant@Cliente1:~$ lsblk -f
NAME FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                   
├─sda1
│    ext4         b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5   16.4G     5% /
├─sda2
│                                                                     
└─sda5
     swap         f8f6d279-1b63-4310-a668-cb468c9091d8                [SWAP]
sdb                                                                   
└─sdb1
     ext4         e8a6336e-9823-4181-b557-b7a674ec11df  621.7M     0% /mnt
~~~

Para que al iniciarse la máquina cliente, el disco aparezca de forma automática, sin tener que logearse, se debe introducir el siguiente comando:
~~~
vagrant@Cliente1:~$ sudo iscsiadm --mode node -T "iqn.2020-01.com:tg1" -o update -n node.startup -v automatic
~~~


**- Utiliza systemd mount para que el target se monte automáticamente al arrancar el cliente.**
Para montar el disco con systemd hay que crear un fichero en **/etc/systemd/system/** con la extensión .mount para que systemd entienda que es un fichero de configuración donde se van a establecer los parámetros para el montaje. El nombre del fichero debe coincidir con el punto del montaje, es decir, en nuestro caso se va a montar en /mnt, luego el fichero va a ser **mnt.mount**:
~~~
[Unit]
Description=Disco Remoto MNT
[Mount]
What=/dev/sdb1
Where=/mnt
Type=ext4
Options=_netdev
[Install]
WantedBy=multi-user.target
~~~

Se inicia el servicio:
~~~
vagrant@Cliente1:~$ sudo systemctl start mnt.mount
~~~

Y se habilita para que el disco se monte autimáticamente al iniciar la máquina:
~~~
vagrant@Cliente1:~$ sudo systemctl enable mnt.mount 
Created symlink /etc/systemd/system/multi-user.target.wants/mnt.mount → /etc/systemd/system/mnt.mount.
~~~


**- Crea un target con 2 LUN y autenticación por CHAP y conéctala a un cliente windows. Explica cómo se escanea la red en windows y cómo se utilizan las unidades nuevas (formateándolas con NTFS)**

Se inician los dos disco como volumen físico:
~~~
vagrant@Servidor1:~$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
vagrant@Servidor1:~$ sudo pvcreate /dev/sdd
  Physical volume "/dev/sdd" successfully created.
~~~

Se crea el grupo de volúmenes y se añaden los dos volúmenes anteriores:
~~~
vagrant@Servidor1:~$ sudo vgcreate vg2 /dev/sdc /dev/sdd
  Volume group "vg2" successfully created
~~~

Y se crea el volumen:
~~~
vagrant@Servidor1:~$ sudo lvcreate -L 700M -n vol1 vg2
  Logical volume "vol1" created.
vagrant@Servidor1:~$ sudo lvcreate -L 700M -n vol2 vg2
  Logical volume "vol2" created.
~~~

Y se comprueba el estado:
~~~
vagrant@Servidor1:~$ sudo lvs
  LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vol1 vg1 -wi-ao---- 700.00m                                                    
  vol1 vg2 -wi-a----- 700.00m                                                    
  vol2 vg2 -wi-a----- 700.00m    
~~~

Se configura el fichero **/etc/tgt/targets.conf**, añadiendo la configuración de autenticación CHAP. Hay que introducir el usuario y la contraseña (entre 12 y 16 caracteres):
~~~
<target iqn.2020-02.com:tg2>
    backing-store /dev/vg2/vol1
    backing-store /dev/vg2/vol2
    incominguser windows1 WindowsUsuario1
</target>
~~~

Se reinicia el servicio:
~~~
vagrant@Servidor1:~$ sudo systemctl restart tgt.service 
~~~

Y se comprueba que el target se ha creado correctamente:
~~~
vagrant@Servidor1:~$ sudo tgtadm --mode target --op show
Target 1: iqn.2020-01.com:tg1

...

Target 2: iqn.2020-02.com:tg2
    System information:
        Driver: iscsi
        State: ready
    I_T nexus information:
    LUN information:
        LUN: 0
            Type: controller
            SCSI ID: IET     00020000
            SCSI SN: beaf20
            Size: 0 MB, Block size: 1
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: null
            Backing store path: None
            Backing store flags: 
        LUN: 1
            Type: disk
            SCSI ID: IET     00020001
            SCSI SN: beaf21
            Size: 734 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/vg2/vol1
            Backing store flags: 
        LUN: 2
            Type: disk
            SCSI ID: IET     00020002
            SCSI SN: beaf22
            Size: 734 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/vg2/vol2
            Backing store flags: 
    Account information:
        windows1
    ACL information:
        ALL
~~~

### Configuración del cliente
Se abre la ventana del Inicador iSCSA y en la pestaña de detección se pulsa en **Detectar portal...** Aquí se indica la IP de la máquina servidor.
![imagen](https://github.com/PalomaR88/Sistema_de_ficheros/blob/master/iSCSI/Captura%20de%20pantalla%20de%202020-02-02%2023-11-20.png)

Pestaña **Destinos**, se pulsa el target al que se quiere conectar.
![imagen](iSCSI/Captura%de%pantalla%de%2020-02-02%23-12-48.png)

Aparece una ventana donde hay que seleccionar **Opciones avanzadas**. Se selecciona la opción **Habilitar inicio de sesión CHAP**, donde se introduce el usuario y la contraseña que se indicó en el servidor.
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-14-02.png)

De esta forma, aparece el target cuya conexión se ha configurado con el estado Conectado:
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-16-37.png)

A Continuación, se inicializa y se formatea con NFTS en la controladora **Administración de discos**. 
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-17-22.png)

Se seleccionan ambos discos para comenzar el proceso:
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-17-41.png)

Se va formatear con NTFS ambos volúmenes:
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-18-20.png)
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-18-49.png)

Y así aparecerán en las unidades del sistema como el disco E: y F:
![imagen](iSCSI/Captura\ de\ pantalla\ de\ 2020-02-02\ 23-23-07.png)















