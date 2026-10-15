Instalación y configuración de Apache2 en Ubuntu Server
Índice

    Fundamentos del servidor web
    Instalación de Apache2
    Estructura de directorios y comandos de gestión
    Creación del primer sitio virtual
    Creación del segundo sitio virtual
    Configuración de la seguridad del servidor
    Autenticación mediante usuario y contraseña
    Código en el servidor y código en el cliente
    Instalación y gestión de módulos
    Configuración de HTTPS
    Ejercicio autónomo: intranet

1. Fundamentos del servidor web
¿Qué es un servidor web?

Un servidor web es un programa que escucha las peticiones que realizan los clientes y les devuelve una respuesta mediante el protocolo HTTP.

Por ejemplo, cuando escribimos una dirección web en el navegador, el navegador solicita una página al servidor y este devuelve el contenido solicitado.

La comunicación sigue este esquema:

Cliente → Petición HTTP → Servidor web → Respuesta HTTP → Cliente

Los puertos utilizados habitualmente son:

    Puerto 80: HTTP, comunicación web sin cifrar.
    Puerto 443: HTTPS, comunicación web cifrada mediante TLS.

HTTP es un protocolo sin estado (stateless). Esto significa que cada petición se trata de forma independiente y el protocolo no recuerda automáticamente las peticiones anteriores del mismo cliente.
1.1. Observar una petición HTTP

Comando ejecutado:

curl -v https://example.com

¿Qué hace?

El comando curl permite realizar peticiones a direcciones web desde la terminal. La opción -v (verbose) muestra información detallada sobre la comunicación entre el cliente y el servidor.

¿Qué debemos identificar en la salida?

    Petición HTTP: indica el método utilizado, la ruta solicitada y la versión del protocolo. Por ejemplo, GET / HTTP/1.1.
    Cabeceras HTTP: proporcionan información adicional sobre la petición o la respuesta. Por ejemplo, Host identifica el servidor solicitado y User-Agent identifica el cliente.
    Código de estado: indica el resultado de la petición.

Los códigos más importantes son:

    200 OK: la petición se ha realizado correctamente.
    301 Moved Permanently y 302 Found: indican una redirección.
    404 Not Found: el recurso solicitado no existe o no se encuentra.

2. Instalar Apache
2.1. Actualizar la información de los paquetes

Comando ejecutado:

sudo apt update

¿Qué hace?

Actualiza la información de los paquetes disponibles en los repositorios configurados en Ubuntu. No actualiza por sí solo todos los programas instalados.

    sudo: ejecuta el comando con permisos de administrador.
    apt: gestiona los paquetes de software.
    update: actualiza los índices de paquetes disponibles.

2.2. Instalar Apache2

Comando ejecutado:

sudo apt install apache2 -y

¿Qué hace?

Instala Apache2, el servidor web que permitirá atender las peticiones HTTP de los clientes.

    install: instala el paquete indicado.
    apache2: nombre del paquete del servidor web.
    -y: acepta automáticamente las confirmaciones de la instalación.

2.3. Comprobar el estado del servicio

Comando ejecutado:

sudo systemctl status apache2

¿Qué hace?

Muestra el estado del servicio Apache2, incluyendo si está iniciado, si se está ejecutando y algunos mensajes recientes.

Resultado esperado:

Active: active (running)

Esto significa que el servicio está activo y ejecutándose.

Otros estados posibles son inactive (inactivo) o failed (ha fallado).
2.4. Permitir el acceso mediante el cortafuegos

Comando ejecutado:

sudo ufw allow 'Apache'

¿Qué hace?

Permite el tráfico de red correspondiente al perfil de aplicación Apache de UFW, el cortafuegos de Ubuntu.

Este perfil permite el tráfico HTTP por el puerto 80.

El comando configura una regla del cortafuegos, pero no garantiza por sí solo que UFW esté activado ni que la máquina virtual sea accesible desde el equipo cliente.
2.5. Comprobar el funcionamiento desde el navegador

Desde el equipo cliente, abrimos un navegador y escribimos:

http://IP_DEL_SERVIDOR

Debemos sustituir IP_DEL_SERVIDOR por la dirección IP real de nuestra máquina virtual.

Resultado esperado: aparece la página predeterminada de Apache2 en Ubuntu, conocida como Apache2 Ubuntu Default Page.

Esta página confirma que el servidor responde a las peticiones HTTP y sirve contenido web.

Conclusión: Apache2 está instalado y funcionando cuando el servicio aparece como active (running) y podemos acceder a su página predeterminada desde el cliente.

    500 Internal Server Error: se ha producido un error interno en el servidor.

Conclusión: curl -v permite observar cómo se comunican un cliente y un servidor web mediante HTTP, incluyendo las cabeceras y los códigos de respuesta.
3. Directorios y archivos importantes
3.1. Directorios y archivos importantes
Ruta 	Función
/etc/apache2/apache2.conf 	Archivo principal de configuración de Apache2.
/etc/apache2/sites-available/ 	Contiene las configuraciones de los sitios virtuales disponibles.
/etc/apache2/sites-enabled/ 	Contiene los enlaces simbólicos de los sitios virtuales activados.
/etc/apache2/mods-available/ 	Contiene los módulos disponibles para Apache2.
/etc/apache2/mods-enabled/ 	Contiene los enlaces simbólicos de los módulos activados.
/var/www/html/ 	Directorio raíz de la página web predeterminada.
/var/log/apache2/ 	Directorio donde se guardan los registros del servidor.
3.2. Archivos de registro

Los dos registros principales son:

    access.log: registra las peticiones recibidas por el servidor.
    error.log: registra errores y otros mensajes relacionados con el funcionamiento de Apache2.

3.3. Comandos de gestión

Comprobar la sintaxis de la configuración:

sudo apachectl configtest

Comprueba que los archivos de configuración tienen una sintaxis válida.

Resultado esperado:

Syntax OK

Es recomendable ejecutar esta comprobación antes de recargar o reiniciar Apache2.

Recargar la configuración:

sudo systemctl reload apache2

Hace que Apache2 vuelva a leer la configuración sin realizar un reinicio completo del servicio.

Se utiliza habitualmente después de modificar sitios virtuales o ajustes de configuración.

Reiniciar el servicio:

sudo systemctl restart apache2

Detiene y vuelve a iniciar Apache2. Puede ser necesario cuando los cambios requieren un reinicio del servicio.

Consultar los registros en tiempo real:

sudo journalctl -u apache2 -f

Muestra los mensajes del servicio Apache2 y continúa mostrando los nuevos mensajes que aparezcan.

    -u apache2: filtra los mensajes correspondientes al servicio.
    -f: sigue los mensajes en tiempo real.

Conclusión: conocer las rutas de configuración, los directorios web y los registros facilita administrar Apache2 y detectar errores.
4. Creación del primer sitio virtual
4.1. ¿Qué es un sitio virtual?

Un sitio virtual (Virtual Host o vhost) permite alojar varias páginas web en un mismo servidor Apache, incluso utilizando la misma dirección IP y el mismo puerto.

En este apartado vamos a crear un sitio llamado sitio1.local, independiente de la página predeterminada de Apache.

Para ello, crearemos un directorio propio para sus archivos y un archivo de configuración específico.
4.2. Copiar la configuración predeterminada

Comando ejecutado:

sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/sitio1.conf

¿Qué hace?

Copia el archivo de configuración del sitio predeterminado de Apache y crea una copia llamada sitio1.conf.

    sudo: ejecuta el comando con permisos de administrador.
    cp: copia archivos.
    000-default.conf: archivo de configuración del sitio predeterminado.
    sitio1.conf: nombre del nuevo archivo de configuración.

La copia nos permite partir de una configuración que ya existe y modificarla para nuestro sitio.
4.3. Crear el directorio del sitio web

Comando ejecutado:

sudo mkdir -p /var/www/sitio1

¿Qué hace?

Crea el directorio /var/www/sitio1, donde guardaremos los archivos de nuestra página web.

    mkdir: crea directorios.
    -p: permite crear los directorios necesarios de la ruta si todavía no existen y evita errores si el directorio ya existe.

Este directorio será el DocumentRoot del nuevo sitio.
4.4. Crear la página principal

Comando ejecutado:

echo '<h1>sitio1.local</h1>' | sudo tee /var/www/sitio1/index.html

¿Qué hace?

Crea el archivo index.html con un encabezado HTML que muestra el texto sitio1.local.

    echo: muestra el texto indicado.
    |: envía la salida de un comando al siguiente mediante una tubería (pipe).
    sudo tee: escribe el texto recibido en el archivo con permisos de administrador.
    index.html: archivo HTML que Apache servirá como página principal del directorio.

Usamos sudo tee porque escribir sudo echo ... > archivo no eleva los permisos de la redirección > y puede producir un error de permisos.
4.5. Configurar el sitio virtual

Archivo que debemos editar:

/etc/apache2/sites-available/sitio1.conf

Podemos abrirlo con el editor de terminal Nano:

sudo nano /etc/apache2/sites-available/sitio1.conf

Dentro del bloque <VirtualHost *:80> ... </VirtualHost>, debemos configurar las siguientes directivas:

<VirtualHost *:80>
    ServerName sitio1.local
    DocumentRoot /var/www/sitio1

    ErrorLog ${APACHE_LOG_DIR}/sitio1_error.log
    CustomLog ${APACHE_LOG_DIR}/sitio1_access.log combined
</VirtualHost>

Explicación de las directivas:

    <VirtualHost *:80>: define un sitio virtual que atiende en el puerto 80. El asterisco indica que puede atender en las direcciones IP correspondientes al servidor.
    ServerName sitio1.local: establece el nombre que identifica al sitio virtual.
    DocumentRoot /var/www/sitio1: indica el directorio del que Apache obtiene los archivos de la página.
    ErrorLog: establece el archivo donde se registran los errores específicos del sitio.
    CustomLog: establece el archivo donde se registran las peticiones del sitio.
    combined: indica el formato de registro utilizado, que incluye información como la dirección del cliente, la petición y el código de estado HTTP.

Importante: si hemos copiado 000-default.conf, debemos revisar el archivo y sustituir o eliminar las directivas que entren en conflicto con esta configuración. No debemos añadir un segundo bloque idéntico sin revisar el contenido original.

Para guardar en Nano, pulsamos Ctrl + O, confirmamos con Enter y salimos con Ctrl + X.
4.6. Activar el sitio virtual

Comando ejecutado:

sudo a2ensite sitio1.conf

¿Qué hace?

Activa la configuración del sitio virtual.

En Ubuntu, a2ensite habilita un sitio creando un enlace simbólico desde sites-enabled/ hacia el archivo correspondiente de sites-available/.

    sites-available/: contiene los sitios configurados disponibles.
    sites-enabled/: contiene los sitios que Apache tiene habilitados mediante sus enlaces simbólicos.

4.7. Desactivar el sitio predeterminado

Comando ejecutado:

sudo a2dissite 000-default.conf

¿Qué hace?

Desactiva la configuración del sitio predeterminado de Apache.

El objetivo de esta práctica es que el sitio sitio1.local sustituya a la página predeterminada. Desactivar el sitio por defecto evita que siga siendo el sitio virtual seleccionado cuando no coincide otro nombre.
4.8. Comprobar la configuración y aplicar los cambios

Primero validamos la sintaxis:

sudo apachectl configtest

El resultado esperado es:

Syntax OK

Después recargamos Apache:

sudo systemctl reload apache2

La recarga aplica los cambios sin realizar un reinicio completo del servicio.
4.9. Comprobar el resultado

Podemos comprobar el estado de Apache:

sudo systemctl status apache2

Y comprobar la respuesta HTTP desde la propia máquina virtual:

curl -i -H "Host: sitio1.local" http://localhost/

¿Qué hace el segundo comando?

    curl: realiza una petición HTTP.
    -i: muestra las cabeceras HTTP de la respuesta junto con el contenido.
    -H "Host: sitio1.local": envía la cabecera Host con el nombre del sitio virtual.
    http://localhost/: realiza la petición al propio servidor mediante la interfaz local.

Resultado esperado: una respuesta HTTP satisfactoria, normalmente 200 OK, y el contenido HTML que contiene el encabezado sitio1.local.

La cabecera Host permite comprobar el sitio virtual sin necesidad de configurar un DNS público.

Conclusión: hemos creado un directorio para el sitio, una página HTML, un archivo de configuración propio y hemos habilitado el sitio virtual en Apache.
5. Creación de un segundo sitio virtual
5.1. Objetivo

Vamos a crear un segundo sitio web llamado sitio2.local, independiente de sitio1.local.

Los dos sitios utilizarán la misma dirección IP y el mismo puerto HTTP (80), pero tendrán directorios y archivos de configuración diferentes.

Apache decidirá qué sitio debe mostrar según el nombre indicado en la cabecera HTTP Host que envía el cliente.
5.2. Crear el directorio y la página principal

Comandos ejecutados:

sudo mkdir -p /var/www/sitio2

Crea el directorio donde se almacenarán los archivos del segundo sitio.

echo '<h1>sitio2.local</h1>' | sudo tee /var/www/sitio2/index.html

Crea el archivo index.html con un encabezado que identifica al segundo sitio.
5.3. Crear el archivo de configuración

Comando ejecutado:

sudo cp /etc/apache2/sites-available/sitio1.conf /etc/apache2/sites-available/sitio2.conf

Copia la configuración del primer sitio para utilizarla como base del segundo.

A continuación, editamos el archivo:

sudo nano /etc/apache2/sites-available/sitio2.conf

Debemos cambiar el contenido para que las directivas principales sean las siguientes:

<VirtualHost *:80>
    ServerName sitio2.local
    DocumentRoot /var/www/sitio2

    ErrorLog ${APACHE_LOG_DIR}/sitio2_error.log
    CustomLog ${APACHE_LOG_DIR}/sitio2_access.log combined
</VirtualHost>

Explicación:

    ServerName sitio2.local: identifica el segundo sitio.
    DocumentRoot /var/www/sitio2: indica dónde están sus archivos.
    ErrorLog: guarda los errores de este sitio.
    CustomLog: registra las peticiones recibidas por este sitio.

Es importante modificar tanto ServerName como DocumentRoot y los nombres de los registros. Si dejamos los valores de sitio1, Apache podría dirigir las peticiones al sitio equivocado o mezclar los registros.
5.4. Activar el segundo sitio

Comando ejecutado:

sudo a2ensite sitio2.conf

Activa la configuración de sitio2.local, creando el enlace simbólico correspondiente en sites-enabled/.

Comprobamos la sintaxis:

sudo apachectl configtest

El resultado esperado es:

Syntax OK

Aplicamos los cambios:

sudo systemctl reload apache2

5.5. Comprobar la discriminación de sitios mediante curl

Primera petición:

curl -H "Host: sitio1.local" http://localhost/

Segunda petición:

curl -H "Host: sitio2.local" http://localhost/

¿Qué estamos comprobando?

Las dos peticiones se envían a la misma dirección local y al mismo puerto. La diferencia es el valor de la cabecera Host.

En la primera petición, Apache debe mostrar el contenido de sitio1.local. En la segunda, debe mostrar el contenido de sitio2.local.

Este mecanismo se denomina alojamiento virtual basado en nombre (name-based virtual hosting).
5.6. Acceder desde el navegador del equipo cliente

Para utilizar estos nombres desde otro equipo, debemos asociarlos a la dirección IP de la máquina virtual.

En Windows, el archivo hosts suele encontrarse en:

C:\Windows\System32\drivers\etc\hosts

En Linux, el archivo es:

/etc/hosts

Añadimos dos entradas, sustituyendo la dirección de ejemplo por la IP real del servidor:

192.168.1.100 sitio1.local
192.168.1.100 sitio2.local

Estas líneas son un ejemplo, no hay que copiar esa IP sin comprobar cuál tiene vuestra máquina virtual.

El archivo hosts permite asociar nombres con direcciones IP sin configurar un servidor DNS.

Ahora podemos probar desde el navegador:

http://sitio1.local
http://sitio2.local

Resultado esperado: cada dirección muestra la página correspondiente.

Conclusión: Apache puede alojar varios sitios virtuales con la misma IP y el mismo puerto. La cabecera Host permite identificar qué sitio debe responder.
6. Configuración de la seguridad del servidor
6.1. Ocultar información sobre la versión de Apache

Archivo que debemos editar:

sudo nano /etc/apache2/conf-available/security.conf

Buscamos las directivas ServerTokens y ServerSignature. Si ya existen, modificamos sus valores en lugar de duplicarlas.

La configuración debe incluir:

ServerTokens Prod
ServerSignature Off

Explicación:

    ServerTokens Prod: reduce la información de identificación que Apache proporciona en la cabecera HTTP Server. Normalmente, el valor mostrado se limita a identificar el producto, sin detallar la versión.
    ServerSignature Off: desactiva la firma de Apache que puede aparecer en determinadas páginas de error generadas por el servidor.

Estas opciones reducen la información que se revela sobre el software del servidor, aunque no sustituyen a las actualizaciones ni a otras medidas de seguridad.
6.2. Establecer los permisos correctos

Comando ejecutado:

sudo chown -R www-data:www-data /var/www/sitio1 /var/www/sitio2

¿Qué hace?

Cambia el propietario y el grupo de los archivos y directorios de ambos sitios a www-data, una cuenta utilizada habitualmente por Apache en Ubuntu.

    chown: cambia el propietario y el grupo.
    -R: aplica el cambio de forma recursiva a los archivos y subdirectorios.
    www-data:www-data: establece www-data como propietario y grupo.

Establecer permisos en los directorios:

sudo find /var/www/sitio1 -type d -exec chmod 755 {} \;
sudo find /var/www/sitio2 -type d -exec chmod 755 {} \;

Estos comandos buscan todos los directorios de ambos sitios y les asignan permisos 755.

Establecer permisos en los archivos:

sudo find /var/www/sitio1 -type f -exec chmod 644 {} \;
sudo find /var/www/sitio2 -type f -exec chmod 644 {} \;

Estos comandos buscan todos los archivos de ambos sitios y les asignan permisos 644.

¿Qué significan estos permisos?
Permiso 	Significado
755 	El propietario puede leer, escribir y acceder al directorio. El grupo y los demás usuarios pueden leer y acceder.
644 	El propietario puede leer y escribir el archivo. El grupo y los demás usuarios pueden leerlo.

En los directorios, el permiso de ejecución permite acceder a ellos y recorrerlos. En los archivos, permite ejecutarlos como programas, por lo que normalmente no es necesario para páginas HTML.

Estos permisos son adecuados para archivos web estáticos habituales. Si más adelante necesitamos que una aplicación escriba archivos, tendremos que estudiar qué permisos específicos necesita.
6.3. Evitar el listado de directorios

Si un directorio no contiene un archivo de inicio reconocido, Apache podría mostrar un listado de los archivos que contiene si la opción Indexes está habilitada.

Para evitarlo, podemos añadir un bloque como este dentro de la configuración de cada sitio virtual:

<Directory /var/www/sitio1>
    Options -Indexes
</Directory>

Para el segundo sitio, utilizamos su propia ruta:

<Directory /var/www/sitio2>
    Options -Indexes
</Directory>

Explicación:

    <Directory ...>: establece opciones para una ruta del sistema de archivos.
    Options -Indexes: desactiva la generación automática de listados de directorios.

Debemos colocar cada bloque dentro de la configuración correspondiente y comprobar que no contradiga otras directivas existentes.
6.4. Comprobar la configuración y aplicar los cambios

Validamos la sintaxis:

sudo apachectl configtest

Resultado esperado:

Syntax OK

Recargamos Apache:

sudo systemctl reload apache2

6.5. Comprobar las cabeceras HTTP

Comando ejecutado:

curl -I -H "Host: sitio1.local" http://localhost/

¿Qué hace?

    -I: solicita las cabeceras de la respuesta sin mostrar el cuerpo de la página.
    -H "Host: sitio1.local": selecciona el sitio virtual que queremos comprobar.

En la salida buscamos una línea similar a:

HTTP/1.1 200 OK
Server: Apache

El resultado exacto puede variar. Lo importante es que la cabecera Server no revele la versión concreta de Apache.

Si queremos comprobar que no se genera un listado de directorios, debemos solicitar una ruta sin archivo de inicio y que contenga archivos, por ejemplo, una carpeta de prueba sin index.html. Si Indexes está desactivado, no se mostrará automáticamente la lista de su contenido.

Conclusión: hemos reducido la información expuesta por Apache, configurado los permisos de los sitios web y desactivado el listado automático de directorios.
7. Comprobación del acceso de los usuarios
7.1. Objetivo

Vamos a proteger una carpeta del sitio sitio1.local para que solamente puedan acceder los usuarios autorizados.

Utilizaremos la autenticación básica HTTP (Basic Authentication), que permite solicitar un nombre de usuario y una contraseña al acceder a una ruta protegida.
7.2. Crear la carpeta privada

Comando ejecutado:

sudo mkdir -p /var/www/sitio1/privado

¿Qué hace?

Crea el directorio privado dentro de la raíz del primer sitio web.

Esta carpeta contendrá los archivos que queremos proteger.
7.3. Crear una página dentro de la carpeta privada

Comando ejecutado:

echo 'zona privada' | sudo tee /var/www/sitio1/privado/index.html

¿Qué hace?

Crea el archivo index.html con el texto zona privada.

Este archivo nos permitirá comprobar si Apache sirve correctamente el contenido después de superar la autenticación.
7.4. Instalar las herramientas de autenticación

Comando ejecutado:

sudo apt install apache2-utils -y

¿Qué hace?

Instala el paquete apache2-utils, que incluye herramientas auxiliares para administrar Apache.

Entre ellas se encuentra htpasswd, que permite crear y gestionar archivos con credenciales para la autenticación HTTP.
7.5. Crear el archivo de usuarios

Comando ejecutado:

sudo htpasswd -c /etc/apache2/.htpasswd alumno

¿Qué hace?

Crea el archivo de contraseñas /etc/apache2/.htpasswd y añade el usuario alumno.

El comando solicitará que introduzcamos una contraseña y que la confirmemos.

    htpasswd: herramienta para administrar usuarios de autenticación básica.
    -c: crea un archivo nuevo.
    /etc/apache2/.htpasswd: ubicación del archivo de credenciales.
    alumno: nombre del usuario que vamos a crear.

La contraseña no se guarda como texto legible, sino mediante un hash adecuado para este uso.

Importante: la opción -c crea un archivo nuevo y puede sobrescribir uno existente. Se utiliza al crear el archivo por primera vez, no al añadir usuarios adicionales.
7.6. Configurar la protección de la carpeta

Archivo que debemos editar:

sudo nano /etc/apache2/sites-available/sitio1.conf

Dentro de la configuración del sitio, añadimos el siguiente bloque:

<Directory /var/www/sitio1/privado>
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>

Explicación de las directivas:

    <Directory /var/www/sitio1/privado>: establece las reglas de acceso para el directorio privado.
    AuthType Basic: activa la autenticación básica HTTP.
    AuthName "Zona restringida": define el nombre que identifica la zona protegida en la solicitud de autenticación.
    AuthUserFile: indica dónde está el archivo que contiene los usuarios y sus credenciales.
    Require valid-user: permite acceder a cualquier usuario que se haya autenticado correctamente.

La protección se aplica al directorio indicado y a su contenido.
7.7. Comprobar la sintaxis y recargar Apache

Primero ejecutamos:

sudo apachectl configtest

El resultado esperado es:

Syntax OK

Después aplicamos los cambios:

sudo systemctl reload apache2

7.8. Comprobar el acceso sin credenciales

Comando ejecutado:

curl -i -H "Host: sitio1.local" http://localhost/privado/

¿Qué hace?

Realiza una petición HTTP a la carpeta privada sin proporcionar usuario ni contraseña.

    -i: muestra las cabeceras de la respuesta junto con el contenido.
    -H "Host: sitio1.local": indica qué sitio virtual queremos consultar.
    /privado/: ruta protegida.

Resultado esperado:

HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="Zona restringida"

El código 401 Unauthorized indica que la petición necesita autenticación. La cabecera WWW-Authenticate informa al cliente del mecanismo de autenticación solicitado.
7.9. Comprobar el acceso con credenciales correctas

Comando ejecutado:

curl -i -u alumno http://localhost/privado/ -H "Host: sitio1.local"

curl solicitará la contraseña del usuario alumno.

La opción -u permite proporcionar las credenciales de autenticación. Si se escribe solamente el nombre de usuario, curl solicita la contraseña de forma interactiva.

Resultado esperado:

HTTP/1.1 200 OK

En el cuerpo de la respuesta debe aparecer:

zona privada

7.10. Seguridad de la autenticación básica

La autenticación básica no cifra por sí misma las credenciales. Sin HTTPS, estas pueden recuperarse a partir del tráfico de red.

Por eso, en un entorno real, debe utilizarse junto con HTTPS para proteger las comunicaciones entre el cliente y el servidor.

Conclusión: hemos protegido un directorio mediante autenticación básica. Sin credenciales válidas, Apache responde con 401 Unauthorized; después de autenticarse correctamente, devuelve 200 OK y el contenido de la página.
8. Código en el servidor frente a código en el cliente
8.1. Objetivo

Vamos a comparar dos tipos de páginas web:

    PHP: se ejecuta en el servidor antes de enviar la respuesta al navegador.
    HTML y JavaScript: el HTML estructura la página y JavaScript puede ejecutar instrucciones directamente en el navegador del cliente.

Esta diferencia es importante porque el código PHP no se envía al navegador como código fuente PHP cuando el servidor lo procesa correctamente.
8.2. Instalar el módulo PHP para Apache

Comando ejecutado:

sudo apt install libapache2-mod-php -y

¿Qué hace?

Instala el módulo que permite integrar PHP con Apache mediante mod_php.

Esto permite que Apache procese archivos PHP y devuelva al cliente el resultado generado.
8.3. Reiniciar Apache

Comando ejecutado:

sudo systemctl restart apache2

Reinicia el servicio para aplicar la integración con PHP.

Comprobamos que el servicio sigue funcionando:

sudo systemctl status apache2

El resultado esperado es active (running).
8.4. Crear una página PHP

Comando ejecutado:

sudo nano /var/www/sitio1/fecha.php

Escribimos el siguiente contenido:

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Fecha del servidor</title>
</head>
<body>
    <h1>Fecha y hora del servidor</h1>
    <p><?php echo date("d/m/Y H:i:s"); ?></p>
</body>
</html>

¿Qué hace el código?

    <?php ... ?>: delimita el código PHP.
    echo: muestra el resultado de una expresión.
    date("d/m/Y H:i:s"): obtiene la fecha y hora con el formato día/mes/año, horas, minutos y segundos.
    d/m/Y: representa la fecha.
    H:i:s: representa la hora en formato de 24 horas.

Cuando el navegador solicita fecha.php, el servidor ejecuta PHP y envía el HTML resultante.

La fecha y la hora corresponden al reloj y a la configuración horaria del servidor, que pueden ser diferentes de los del cliente.
8.5. Crear una página HTML con JavaScript

Comando ejecutado:

sudo nano /var/www/sitio1/reloj.html

Escribimos el siguiente contenido:

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Reloj del cliente</title>
</head>
<body>
    <h1>Hora del navegador</h1>
    <p id="reloj"></p>

    <script>
        function actualizarReloj() {
            const ahora = new Date();
            document.getElementById("reloj").textContent =
                ahora.toLocaleTimeString();
        }

        actualizarReloj();
        setInterval(actualizarReloj, 1000);
    </script>
</body>
</html>

¿Qué hace el código?

    new Date(): obtiene la fecha y hora actuales del entorno donde se ejecuta JavaScript, en este caso el navegador.
    document.getElementById("reloj"): selecciona el elemento HTML cuyo identificador es reloj.
    textContent: modifica el texto que aparece dentro del elemento.
    setInterval(actualizarReloj, 1000): ejecuta la función cada 1.000 milisegundos, es decir, cada segundo.

El reloj se actualiza en el navegador sin necesidad de volver a solicitar la página al servidor.
8.6. Probar las dos páginas

Desde el navegador, accedemos a:

http://sitio1.local/fecha.php
http://sitio1.local/reloj.html

Si los nombres locales no están configurados, podemos utilizar la IP del servidor y la cabecera Host con curl para realizar las comprobaciones correspondientes.

Diferencia al consultar el código fuente con Ctrl + U:

    En fecha.php, el navegador recibe el HTML generado por PHP, no las instrucciones PHP originales.
    En reloj.html, el navegador recibe el JavaScript completo porque es el encargado de ejecutarlo.

Conclusión: PHP se procesa en el servidor antes de enviar la respuesta. JavaScript puede ejecutarse en el navegador, modificando la página sin necesidad de recargarla.
9. Instalación y gestión de módulos de Apache
9.1. ¿Qué es un módulo?

Un módulo es un componente que amplía las funciones de Apache. Por ejemplo, puede permitir la reescritura de direcciones, la gestión de cabeceras HTTP o la integración con PHP.

En Ubuntu, los módulos se administran habitualmente mediante los comandos a2enmod y a2dismod.
9.2. Listar los módulos activos

Comando ejecutado:

sudo apache2ctl -M

¿Qué hace?

Muestra los módulos que Apache tiene cargados.

Entre los módulos que podemos encontrar están:

    rewrite_module: permite reescribir o redirigir direcciones mediante reglas de reescritura.
    headers_module: permite modificar y controlar cabeceras HTTP.
    ssl_module: permite configurar conexiones HTTPS mediante TLS.
    php_module: integra PHP con Apache cuando se utiliza mod_php.

La lista exacta depende de la instalación y de los módulos que estén activados.
9.3. Activar el módulo rewrite

Comando ejecutado:

sudo a2enmod rewrite

Activa el módulo rewrite, que permite utilizar reglas de reescritura de direcciones.
9.4. Activar el módulo headers

Comando ejecutado:

sudo a2enmod headers

Activa el módulo headers, que permite modificar determinadas cabeceras HTTP mediante la configuración de Apache.
9.5. Desactivar el módulo rewrite

Comando ejecutado:

sudo a2dismod rewrite

Desactiva el módulo rewrite.

Si queremos volver a utilizarlo, podemos ejecutar de nuevo sudo a2enmod rewrite.
9.6. Aplicar los cambios

Después de modificar los módulos, comprobamos la configuración:

sudo apachectl configtest

Si la configuración es correcta, aplicamos los cambios:

sudo systemctl reload apache2

La recarga permite que Apache aplique la configuración actualizada. Si el módulo es incompatible con otros ajustes o falta alguna dependencia, pueden aparecer errores que tendremos que resolver.
9.7. Comprobar los enlaces simbólicos

Comando ejecutado:

ls -la /etc/apache2/mods-enabled/ | grep rewrite

¿Qué hace?

    ls -la: muestra los archivos, incluidos los ocultos, junto con información detallada.
    /etc/apache2/mods-enabled/: directorio que contiene los módulos habilitados.
    |: envía la salida al siguiente comando.
    grep rewrite: filtra las líneas que contienen la palabra rewrite.

Cuando el módulo está habilitado, normalmente veremos enlaces como rewrite.load y, según la configuración del módulo, rewrite.conf.

Si no aparece ninguna línea, puede que el módulo esté deshabilitado o que no exista un archivo que coincida con el filtro.
9.8. Comprobar la integración con PHP

Comando ejecutado:

sudo apache2ctl -M | grep php

Si utilizamos la integración de PHP mediante mod_php, esperamos encontrar una línea similar a:

php_module (shared)

El nombre y la disponibilidad del módulo dependen de la versión de PHP y del tipo de integración utilizado.

Conclusión: los módulos permiten ampliar las capacidades de Apache. a2enmod habilita un módulo y a2dismod lo deshabilita. Después debemos comprobar la configuración y aplicar los cambios.
10. Mecanismos para asegurar las comunicaciones cliente-servidor
10.1. Diferencia entre HTTP y HTTPS

HTTP es el protocolo que permite intercambiar peticiones y respuestas web. HTTPS utiliza HTTP protegido mediante TLS para cifrar la comunicación entre el cliente y el servidor.

Los puertos habituales son:

    HTTP: puerto 80.
    HTTPS: puerto 443.

Para configurar HTTPS necesitamos que Apache tenga habilitado el soporte SSL/TLS y disponga de un certificado y una clave privada.

En esta práctica utilizaremos un certificado autofirmado, creado por nosotros mismos, porque estamos trabajando con un dominio local y no con un dominio público validado.
10.2. Activar el módulo SSL

Comando ejecutado:

sudo a2enmod ssl

¿Qué hace?

Habilita el módulo ssl, que proporciona a Apache las funciones necesarias para configurar conexiones HTTPS mediante TLS.
10.3. Crear el directorio para los certificados

Comando ejecutado:

sudo mkdir -p /etc/apache2/ssl

Crea el directorio donde almacenaremos el certificado y la clave privada del sitio.

    mkdir: crea un directorio.
    -p: permite crear los directorios necesarios si no existen.

10.4. Generar el certificado autofirmado y la clave privada

Comando ejecutado:

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/sitio1.key -out /etc/apache2/ssl/sitio1.crt -subj "/CN=sitio1.local"

¿Qué hace?

Utiliza OpenSSL para generar una clave privada y un certificado autofirmado para sitio1.local.

Explicación de los parámetros:

    openssl: herramienta para trabajar con certificados y operaciones criptográficas.
    req: permite generar solicitudes de certificados y realizar operaciones relacionadas.
    -x509: genera un certificado X.509 autofirmado en lugar de una solicitud de certificado convencional.
    -nodes: evita cifrar la clave privada con una contraseña, para que Apache pueda utilizarla sin tener que introducirla al iniciar.
    -days 365: establece una validez de 365 días para el certificado.
    -newkey rsa:2048: genera una clave RSA de 2048 bits.
    -keyout: indica dónde guardar la clave privada.
    -out: indica dónde guardar el certificado.
    -subj "/CN=sitio1.local": establece el nombre común (Common Name) del certificado.

Los archivos generados son:
Archivo 	Función
/etc/apache2/ssl/sitio1.key 	Contiene la clave privada del servidor.
/etc/apache2/ssl/sitio1.crt 	Contiene el certificado que presenta el servidor al cliente.

Importante: la clave privada debe mantenerse protegida y no debe compartirse. En un entorno real también hay que configurar correctamente los permisos de acceso a ella.
10.5. Configurar el sitio virtual HTTPS

Archivo que debemos editar:

sudo nano /etc/apache2/sites-available/sitio1.conf

Añadimos un segundo bloque de sitio virtual para el puerto 443, separado del bloque HTTP existente:

<VirtualHost *:443>
    ServerName sitio1.local
    DocumentRoot /var/www/sitio1

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/sitio1.crt
    SSLCertificateKeyFile /etc/apache2/ssl/sitio1.key

    ErrorLog ${APACHE_LOG_DIR}/sitio1_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/sitio1_ssl_access.log combined
</VirtualHost>

Explicación de las directivas:

    <VirtualHost *:443>: define el sitio virtual que atenderá las conexiones por el puerto HTTPS 443.
    ServerName sitio1.local: identifica el sitio.
    DocumentRoot /var/www/sitio1: indica dónde se encuentran los archivos de la página.
    SSLEngine on: habilita TLS para ese sitio virtual.
    SSLCertificateFile: indica la ruta del certificado.
    SSLCertificateKeyFile: indica la ruta de la clave privada.
    ErrorLog y CustomLog: establecen los archivos de registro de errores y peticiones HTTPS.

El bloque del puerto 443 es independiente del bloque del puerto 80. Ambos pueden servir los mismos archivos, pero utilizan protocolos y puertos diferentes.
10.6. Permitir el tráfico HTTPS en el cortafuegos

Comando ejecutado:

sudo ufw allow 443/tcp

Permite el tráfico TCP entrante por el puerto 443 a través de UFW.

Esta regla es necesaria si el cortafuegos está activo y no existe otra regla que permita ese tráfico.
10.7. Validar la configuración y recargar Apache

Primero comprobamos la sintaxis:

sudo apachectl configtest

Resultado esperado:

Syntax OK

Después recargamos Apache:

sudo systemctl reload apache2

Si aparece un error, debemos revisar el bloque del puerto 443, las rutas del certificado y de la clave privada, y que el módulo SSL esté habilitado.
10.8. Comprobar el funcionamiento de HTTPS

Si sitio1.local ya está asociado a la IP correcta en el archivo hosts, podemos ejecutar:

curl -k -I https://sitio1.local

¿Qué hace?

    curl: realiza una petición al servidor.
    -k: permite continuar aunque el certificado no sea de confianza para el cliente.
    -I: solicita las cabeceras de respuesta sin mostrar el cuerpo de la página.
    https://sitio1.local: realiza la conexión mediante HTTPS.

Resultado esperado:

HTTP/1.1 200 OK

La versión exacta del protocolo que aparezca en la respuesta puede variar.

La opción -k se utiliza aquí para las pruebas con el certificado autofirmado. No convierte el certificado en confiable ni es una opción recomendable para navegar habitualmente por sitios reales.
10.9. Diferencia entre un certificado autofirmado y uno de una entidad reconocida

Certificado autofirmado:

    Lo firma el propio titular del certificado.
    No cuenta automáticamente con la confianza de los navegadores.
    Es útil para prácticas, laboratorios y pruebas locales.
    Normalmente provoca un aviso de seguridad en el navegador.

Certificado emitido por una entidad de confianza:

    Está firmado por una autoridad certificadora reconocida por el cliente.
    Permite al cliente comprobar la cadena de confianza del certificado.
    En un sitio público debe incluir los nombres de dominio correctos y cumplir los requisitos de validez correspondientes.

HTTPS protege la comunicación mediante TLS, pero la confianza en la identidad del servidor depende también de la validación del certificado.

Conclusión: hemos habilitado SSL/TLS, generado un certificado autofirmado y configurado un sitio virtual en el puerto 443. La respuesta 200 OK obtenida mediante HTTPS demuestra que el servidor responde correctamente a la petición, aunque el certificado autofirmado no sea reconocido automáticamente como confiable.
