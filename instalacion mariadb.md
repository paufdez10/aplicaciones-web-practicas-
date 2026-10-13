1. Actualiza la lista de paquetes y comprueba que Apache sigue funcionando: sudo apt update, y systemctl is-active apache2
2. Para instalar mariadb: sudo apt install mariadb-server -y
3. Para comprobar la versión instalada: mariadb --version
4. ¿Qué diferencia hay entre el paquete mariadb-server y el paquete mariadb-client? ¿Cuál de los dos
has instalado? Mariadb-server es el motor de la base de datos que almacena y gestiona los datos, mientras que mariadb-client es la herramienta que te permite conectarte y enviar comandos a ese servidor.
5. Comprobación del funcionamiento: sudo systemctl status mariadb y systemctl is-enabled mariadb.
6. Entramos en la base de datos: sudo mariadb, si todo va bien, verás un mensaje de bienvenida y el prompt cambiará, porque ahora estás dentro de MariaDB.
7. ¿Qué diferencia hay entre active (running) y enabled? ¿Qué pasaría con la web de WordPress si MariaDB estuviera active pero no enabled y se reiniciara el servidor? La diferencia principal radica en el estado actual del servicio versus su comportamiento tras un reinicio del sistema.
