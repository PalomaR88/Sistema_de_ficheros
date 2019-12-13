# Sistema de ficheros (filesystem)
Un sistema de ficheros es una capa entre los ficheros y un sistema de bloques donde se indica la ruta, los directorios y el ugoa (permisos). Pero además, también aporta el índice, la posición y dispoción de los ficheros, etc. 

## Sistemas de ficheros simples
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
Funciona más rápido que ext3 y son mejores los límites de tamaño.

#### 