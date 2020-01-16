# NAS (Network Attached Storage)
Dos máquinas conextadas por red comparten fichero o directorios (no volúmenes).

En NAS los permisos son rwx. Aunque la mayoría de los casos son rw-. Esto implica que necesita autenticación si tiene escritura. 

Protocolos más extendidios para NAS que son: NFS (de UNIX) y SMB (Windows, cliente samba) y CIFS (Windows, cliente samba).

### NFS
**NFS3** muy fácil de montar. No es modo lectura y escritura y no tiene autenticación propiamente dicho. 

**NFS4** versión con autenticación con un mecanismo muy complejo. Tan complejo que se sigue usando NFS3.e
