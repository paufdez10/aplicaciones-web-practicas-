1. Actualiza la lista de paquetes y comprueba que Apache sigue funcionando: sudo apt update, y systemctl is-active apache2
2. Para instalar mariadb: sudo apt install mariadb-server -y
3. Para comprobar la versión instalada: mariadb --version
4. ¿Qué diferencia hay entre el paquete mariadb-server y el paquete mariadb-client? ¿Cuál de los dos
has instalado? Mariadb-server es el motor de la base de datos que almacena y gestiona los datos, mientras que mariadb-client es la herramienta que te permite conectarte y enviar comandos a ese servidor.
5. Comprobación del funcionamiento: sudo systemctl status mariadb y systemctl is-enabled mariadb.
6. Entramos en la base de datos: sudo mariadb, si todo va bien, verás un mensaje de bienvenida y el prompt cambiará, porque ahora estás dentro de MariaDB.
7. ¿Qué diferencia hay entre active (running) y enabled? ¿Qué pasaría con la web de WordPress si MariaDB estuviera active pero no enabled y se reiniciara el servidor? La diferencia principal radica en el estado actual del servicio versus su comportamiento tras un reinicio del sistema.
8. Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos de este apartado usarías
para saber si el problema es que MariaDB está parado? Para comprobar de forma rápida si MariaDB está parado cuando WordPress no puede conectar con la base de datos, utiliza estos dos comandos: sudo systemctl status mariadb y sudo systemctl is-active mariadb


9. Instalación de PHP

WordPress está hecho en PHP, por lo que necesitamos instalarlo para que Apache pueda ejecutar su código. También instalaremos varios módulos que necesitará WordPress para funcionar correctamente.

Ejecutamos el siguiente comando:

sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y

Cada paquete sirve para una cosa distinta:

    php: instala PHP.
    libapache2-mod-php: permite que Apache ejecute PHP.
    php-mysql: permite que PHP se conecte con MariaDB.
    php-curl: permite realizar conexiones con otros servidores.
    php-gd: sirve para trabajar con imágenes.
    php-mbstring: permite trabajar con textos y caracteres especiales.
    php-xml: sirve para trabajar con archivos XML.
    php-zip: permite trabajar con archivos ZIP.
    php-intl: permite trabajar con idiomas y formatos regionales.

5.1. Comprobamos la versión de PHP

Ejecutamos:

php -v

Debería aparecer la versión de PHP instalada. En Ubuntu Server 24.04 normalmente será PHP 8.3.
5.2. Reiniciamos Apache

Para que Apache cargue los cambios relacionados con PHP, lo reiniciamos:

sudo systemctl restart apache2

Ahora comprobamos que ha cargado los módulos correspondientes:

sudo apache2ctl -M | grep -E 'mpm_prefork|php'

Deberían aparecer mpm_prefork_module y php_module si estamos utilizando el módulo de PHP para Apache.
5.3. Comprobamos que PHP puede conectarse a MariaDB

Ejecutamos:

php -m | grep -i mysql

Deberían aparecer estos módulos:

mysqli
mysqlnd
pdo_mysql

Estos módulos permiten que PHP se comunique con MariaDB para consultar y guardar datos.
5.4. Comprobamos que Apache ejecuta PHP

Vamos a crear un archivo de prueba para comprobar que PHP funciona desde el navegador.

Ejecutamos:

echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/info.php

Este comando crea el archivo info.php dentro de la carpeta principal de Apache.

Ahora entramos desde el navegador en:

http://IP_DE_TU_SERVIDOR/info.php

Sustituimos IP_DE_TU_SERVIDOR por la IP de nuestra máquina virtual.

Si todo funciona correctamente, aparecerá una página con información sobre PHP, su versión y los módulos instalados.

Importante: cuando terminemos la comprobación, debemos eliminar este archivo, ya que muestra información sobre la configuración del servidor.

sudo rm /var/www/html/info.php

Comprobamos que se ha eliminado:

ls /var/www/html

¿Por qué es peligroso dejar info.php en un servidor real?

Porque muestra información sobre la configuración, la versión y los módulos de PHP. Alguien podría utilizar esos datos para buscar posibles vulnerabilidades del servidor.
¿Para qué sirve el paquete php-mysql?

Sirve para que PHP pueda conectarse a MariaDB o MySQL. Sin él, WordPress no tendría los controladores necesarios para comunicarse con la base de datos.
6. Comprobación final

Para terminar, comprobamos que Apache y MariaDB siguen funcionando:

systemctl is-active apache2 mariadb

Debería aparecer active para los dos servicios.

También comprobamos la versión de PHP:

php -v

Resumen del estado del servidor
Pieza 	Programa 	Estado
Linux 	Ubuntu Server 24.04 	Instalado
Apache 	Apache2 	Funcionando
MariaDB 	MariaDB 	Funcionando
PHP 	PHP 8.3 	Instalado y preparado para MariaDB

Por último, creamos una instantánea (snapshot) de la máquina virtual desde VirtualBox. Así tendremos una copia del estado actual del servidor antes de empezar a instalar WordPress.
7. Conclusión

En esta práctica hemos instalado MariaDB y comprobado que funciona correctamente. También hemos aprendido a consultar el estado de un servicio, detenerlo y volver a iniciarlo.

Después hemos instalado PHP y los módulos necesarios para que Apache pueda ejecutarlo y para que PHP pueda conectarse con MariaDB.

Con esto, el servidor ya tiene instalados los componentes principales de LAMP y está preparado para continuar con la instalación de WordPress.
