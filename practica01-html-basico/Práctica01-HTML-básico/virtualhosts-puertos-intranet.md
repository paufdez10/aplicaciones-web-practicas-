VirtualHosts por nombre y puerto: web e intranet
Índice

    Objetivo
    Crear las carpetas y las páginas web
    Crear el usuario de la intranet
    Configurar el puerto 9999
    Crear los VirtualHosts
    Activar la configuración
    Comprobar el funcionamiento
    Configurar el archivo hosts
    Conceptos importantes

1. Objetivo

Configurar Apache para que un mismo nombre de dominio muestre dos páginas diferentes según el puerto utilizado.
Dirección 	Resultado esperado
http://www.smr.com 	Página de bienvenida, puerto 80
http://www.smr.com:9999 	Intranet protegida con usuario y contraseña

Para ello utilizaremos dos VirtualHost, cada uno con su propio directorio, y configuraremos la autenticación de la intranet.

Todos los comandos se ejecutan en la máquina virtual Ubuntu Server, conectada por SSH, salvo cuando se indique lo contrario.
2. Crear las carpetas y las páginas web

Creamos dos directorios para separar el contenido de cada sitio:

sudo mkdir -p /var/www/smr/web /var/www/smr/intranet

    /var/www/smr/web/: contiene la página de bienvenida.
    /var/www/smr/intranet/: contiene la página privada.

Creamos la página principal de la web:

echo '<h1>Bienvenidos a SMR</h1>' | sudo tee /var/www/smr/web/index.html

Creamos la página de la intranet:

echo '<h1>Intranet de SMR</h1>' | sudo tee /var/www/smr/intranet/intranet.html

Utilizamos tee para escribir el contenido en los archivos con permisos de administrador.

Podemos comprobar que se han creado:

ls -l /var/www/smr/web/
ls -l /var/www/smr/intranet/

El primer directorio debe contener index.html y el segundo intranet.html.
3. Crear el usuario de la intranet

Instalamos apache2-utils, que incluye la herramienta htpasswd para gestionar usuarios y contraseñas de Apache:

sudo apt update
sudo apt install apache2-utils -y

Creamos el usuario alumno:

sudo htpasswd -c /etc/apache2/.htpasswd alumno

El sistema solicitará una contraseña y su confirmación.

Importante: la opción -c crea el archivo de contraseñas desde cero. Si ya existe /etc/apache2/.htpasswd de una práctica anterior, no la utilices, porque podrías borrar los usuarios existentes. En ese caso:

sudo htpasswd /etc/apache2/.htpasswd alumno

4. Configurar el puerto 9999

Apache escucha normalmente en el puerto 80 para HTTP. Para utilizar también el puerto 9999, editamos:

sudo nano /etc/apache2/ports.conf

Buscamos la línea Listen 80 y añadimos debajo:

Listen 80
Listen 9999

Guardamos con Ctrl + O, pulsamos Enter y salimos con Ctrl + X.

La directiva Listen indica en qué puertos debe aceptar conexiones Apache. Si el puerto no está configurado, el servicio no escuchará en él.
5. Crear los VirtualHosts

Creamos el archivo de configuración:

sudo nano /etc/apache2/sites-available/smr.conf

Escribimos la siguiente configuración:

<VirtualHost *:80>
    ServerName www.smr.com
    DocumentRoot /var/www/smr/web

    ErrorLog ${APACHE_LOG_DIR}/smr-web-error.log
    CustomLog ${APACHE_LOG_DIR}/smr-web-access.log combined
</VirtualHost>

<VirtualHost *:9999>
    ServerName www.smr.com
    DocumentRoot /var/www/smr/intranet

    <Directory /var/www/smr/intranet>
        DirectoryIndex intranet.html
        Options -Indexes
        AuthType Basic
        AuthName "Intranet SMR"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/smr-intranet-error.log
    CustomLog ${APACHE_LOG_DIR}/smr-intranet-access.log combined
</VirtualHost>

¿Qué significa cada directiva?

    VirtualHost *:80 y VirtualHost *:9999: definen los sitios que atienden en cada puerto.
    ServerName: establece el nombre del sitio. En ambos bloques utilizamos www.smr.com.
    DocumentRoot: indica dónde están los archivos de cada web.
    DirectoryIndex intranet.html: indica que la página principal de la intranet tiene ese nombre, en lugar de index.html.
    AuthType Basic: activa la autenticación básica.
    AuthName: define el mensaje que aparece al solicitar las credenciales.
    AuthUserFile: señala dónde se guardan los usuarios y sus contraseñas cifradas.
    Require valid-user: permite el acceso únicamente a usuarios válidos.
    Options -Indexes: evita que Apache muestre un listado de archivos si no encuentra una página de índice.
    ErrorLog y CustomLog: indican dónde se guardan los registros de errores y accesos.

Los bloques son independientes: aunque utilizan el mismo nombre de dominio, cada uno atiende en un puerto diferente.
6. Activar la configuración

Habilitamos el sitio:

sudo a2ensite smr.conf

Deshabilitamos el sitio predeterminado de Apache:

sudo a2dissite 000-default.conf

Si siguen habilitados los sitios de una práctica anterior, como sitio1.conf o sitio2.conf, conviene deshabilitarlos para evitar conflictos durante estas pruebas:

sudo a2dissite sitio1.conf sitio2.conf

Ejecutamos una comprobación de la sintaxis:

sudo apachectl configtest

El resultado esperado es:

Syntax OK

Si la configuración es correcta, reiniciamos Apache:

sudo systemctl restart apache2

El comando a2ensite habilita un sitio, a2dissite lo deshabilita y configtest comprueba que la configuración no contiene errores de sintaxis.
7. Comprobar el funcionamiento

Primero obtenemos la IP de la máquina virtual:

ip a

Buscamos la dirección de la interfaz Host-only, por ejemplo 192.168.56.10.
Prueba 1: página de bienvenida

Desde el navegador del equipo anfitrión, entramos en:

http://192.168.56.10

Debe aparecer:

Bienvenidos a SMR
Prueba 2: intranet protegida

Entramos en:

http://192.168.56.10:9999

Debe aparecer una ventana solicitando usuario y contraseña.

Introducimos las credenciales creadas anteriormente. Si son correctas, aparecerá:

Intranet de SMR

También podemos comprobarlo desde la terminal del anfitrión:

curl -i http://192.168.56.10

Para comprobar que la intranet exige autenticación:

curl -i http://192.168.56.10:9999

La respuesta debe incluir 401 Unauthorized, indicando que hace falta autenticarse.

Para probar con el usuario, ejecutamos:

curl -i -u alumno http://192.168.56.10:9999

curl solicitará la contraseña. Si es correcta, la respuesta debería incluir 200 OK y el contenido de la intranet.

Sustituimos 192.168.56.10 por la IP real de nuestra máquina virtual.
8. Configurar el archivo hosts

Para acceder mediante www.smr.com sin configurar un servidor DNS, debemos asociar ese nombre a la IP de la máquina virtual.

Si no tenemos permisos para modificar el archivo hosts del equipo anfitrión, podemos utilizar una segunda máquina virtual que esté conectada a la misma red Host-only.
Configurar la segunda máquina virtual

En VirtualBox, configuramos el adaptador Host-only de ambas máquinas para que utilicen la misma red, por ejemplo vboxnet0.

Cada máquina debe tener una IP distinta dentro del mismo rango:

    Primera VM: 192.168.56.10
    Segunda VM: 192.168.56.11

Comprobamos las direcciones con:

ip a

Desde la segunda VM, verificamos que podemos alcanzar la primera:

ping -c 3 192.168.56.10

Importante: ambas máquinas virtuales deben estar conectadas a la misma red Host-only para poder comunicarse. Además, cada una debe tener una dirección IP diferente dentro del mismo rango. Tener las dos máquinas encendidas no garantiza que puedan comunicarse entre sí.
Modificar el archivo hosts

En la segunda VM, ejecutamos:

sudo nano /etc/hosts

Añadimos esta línea, utilizando la IP de la primera VM:

192.168.56.10 www.smr.com

Guardamos los cambios.

El archivo hosts relaciona nombres con direcciones IP. Esta asociación solo afecta a la máquina en la que se modifica.

Desde el navegador de la segunda VM, probamos:

    http://www.smr.com
    http://www.smr.com:9999

La primera dirección debe mostrar la bienvenida y la segunda debe solicitar las credenciales de la intranet.
9. Conceptos importantes

    VirtualHost: permite configurar diferentes sitios o servicios en Apache.
    Puerto: identifica el servicio al que nos conectamos. HTTP utiliza habitualmente el puerto 80, pero también podemos configurar el 9999.
    DocumentRoot: indica el directorio que contiene los archivos de una web.
    DirectoryIndex: define qué archivo se sirve como página principal.
    Autenticación básica: solicita un usuario y una contraseña antes de permitir el acceso.
    Archivo hosts: permite asociar manualmente un nombre de dominio con una dirección IP sin utilizar DNS.

Conclusión: hemos configurado dos sitios con el mismo nombre, diferenciados por el puerto. La web del puerto 80 es pública, mientras que la del puerto 9999 está protegida mediante autenticación.
