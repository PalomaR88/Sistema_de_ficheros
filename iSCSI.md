Configura un sistema que exporte algunos targets por iSCSI y los conecte a diversos clientes, explicando con detalle la forma de trabajar.
La creación del escenario se va a realizar con vagrant, la configuración se encuentra en el [siguiente enlace](aqui la URL). Se ha creado una máquina debian con 3 volúmenes asociados y dos máquinas clientes, una debian y otra windows. Todas tienen dos interfaces de red, un enlace a internet y una red interna. Lo correcto sería que el servidor no tuviera conexión con internet, pero para la configuración es necesaria. 

- Crea un target con una LUN y conéctala a un cliente GNU/Linux. Explica cómo escaneas desde el cliente buscando los targets disponibles y utiliza la unidad lógica proporcionada, formateándola si es necesario y montándola.

- Utiliza systemd mount para que el target se monte automáticamente al arrancar el cliente

- Crea un target con 2 LUN y autenticación por CHAP y conéctala a un cliente windows. Explica cómo se escanea la red en windows y cómo se utilizan las unidades nuevas (formateándolas con NTFS)

