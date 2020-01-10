# Instalar ZFS en Debian
En los repositorios, fichero **/etc/apt/source.list**, hay que tener la opción contrib:
~~~
deb http://ftp.es.debian.org/debian/ buster main contrib non-free
deb-src http://ftp.es.debian.org/debian/ buster main contrib non-free
~~~

## Paquetes necesarios:
Paquetes necesarios: 
~~~
paloma@coatlicue:~/zfs-0.8.2$ sudo apt install build-essential autoconf automake libtool gawk alien fakeroot ksh
paloma@coatlicue:~/zfs-0.8.2$ sudo apt install zlib1g-dev uuid-dev libattr1-dev libblkid-dev libselinux-dev libudev-dev
paloma@coatlicue:~/zfs-0.8.2$ sudo apt install libacl1-dev libaio-dev libdevmapper-dev libssl-dev libelf-dev
paloma@coatlicue:~/zfs-0.8.2$ sudo apt install linux-headers-$(uname -r)
~~~


## Descargar el codigo ZFS
Se descarga el paquete desde la siguiente [página](https://github.com/zfsonlinux/zfs/releases/download/zfs-0.8.2/zfs-0.8.2.tar.gz) y se descomprime:
~~~
paloma@coatlicue:~$ wget https://github.com/zfsonlinux/zfs/releases/download/zfs-0.8.2/zfs-0.8.2.tar.gz
paloma@coatlicue:~$ wget https://github.com/zfsonlinux/zfs/releases/download/zfs-0.8.2/zfs-0.8.2.sha256.asc
~~~

Se verifica la integridad del paquete:
~~~
paloma@coatlicue:~$ sudo grep 'tar.gz' zfs-0.8.2.sha256.asc | sha256sum -c -
[sudo] password for paloma: 
zfs-0.8.2.tar.gz: La suma coincide
~~~

Se extrae el paquete:
~~~
paloma@coatlicue:~$ tar axf zfs-0.8.2.tar.gz
paloma@coatlicue:~$ cd zfs-0.8.2/
paloma@coatlicue:~/zfs-0.8.2$ sudo chown -R root:root ./
paloma@coatlicue:~/zfs-0.8.2$ sudo chmod -R o-rwx ./
~~~

Se configura las opciones según las preferencias de cada sistema:
~~~
root@coatlicue:/home/paloma/zfs-0.8.2# ./configure \
>     --disable-systemd \
>     --enable-sysvinit \
>     --disable-debug \
>     --with-spec=generic \
>     --with-linux=$(ls -1dtr /usr/src/linux-headers-*.*.*-common | tail -n 1) \
>     --with-linux-obj=$(ls -1dtr /usr/src/linux-headers-*.*.*-amd64 | tail -n 1)
~~~

Se compila:
~~~
root@coatlicue:/home/paloma/zfs-0.8.2# make -j1 && make install
~~~

Y se instalan los scripts de inicio del servicio:
~~~
root@coatlicue:/home/paloma/zfs-0.8.2# cd /etc/init.d/
root@coatlicue:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-import /etc/init.d/
root@coatlicue:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-mount /etc/init.d/
root@coatlicue:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-share /etc/init.d/
root@coatlicue:/etc/init.d# ln -s /usr/local/etc/init.d/zfs-zed /etc/init.d/
~~~

https://www.linuxito.com/gnu-linux/nivel-alto/1256-compilar-zfs-en-linux

El paquete se instala por defecto en /usr/local. En este punto, se puede producir una condición de carrera. Si el sistema de archivos donde se aloja


