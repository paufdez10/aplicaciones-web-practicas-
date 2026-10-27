Índice
1.Comprobar el punto de partida
2.Crear la base de datos y su usuario
3.Descargar WordPress y colocarlo en su carpeta
4.Propietario y permisos
5.Crear el virtualhost de WordPress
6.Instalación desde el navegador
7.¿Dónde ha guardado WordPress cada cosa?
8.Primera entrada e imagen
9.Comprobación final
10.Conclusiones y problemas encontrados

WordPress es un gestor de contenidos (CMS) hecho en PHP. Permite crear y mantener una web (entradas, páginas, imágenes, usuarios) desde un panel de administración sin programar cada página a mano.

El programa WordPress, temas, plugins e imágenes subidas	Ficheros en una carpeta de Apache	/var/www/wordpress/wp-content
Entradas, páginas, usuarios, comentarios y ajustes	Base de datos (MariaDB)	tabla wp_posts
Datos de conexión a la base de datos	Fichero de configuración	wp-config.php

Flujo de una petición:

Navegador --> Apache --> PHP (WordPress) --> MariaDB
(visitante)  (sirve)    (genera la página)   (guarda los datos)

WordPress va en su propio virtualhost en el puerto 8080, igual que el puerto 9999 de la práctica anterior, para no tocar los sitios que ya existen.

Apartado 1. Comprobar el punto de partida
bash
systemctl is-active apache2 mariadb
php -v
ls /etc/apache2/sites-enabled/
grep Listen /etc/apache2/ports.conf

Salida relevante:

active
active
PHP 8.3.x ...
000-default.conf  (y los sitios de la práctica anterior)
Listen 80
Listen 9999



Explicación: systemctl is-active comprueba que Apache y MariaDB están en marcha (deben decir active). php -v muestra la versión de PHP. ls sites-enabled lista los sitios de Apache habilitados y grep Listen muestra los puertos en los que escucha Apache.

Pregunta: ¿Qué sitios tienes habilitados y en qué puertos escucha Apache? ¿Qué puerto vas a usar para WordPress y por qué no el 80?

Tengo habilitados el sitio por defecto y los de las prácticas anteriores (pon aquí los nombres de los tuyos). Apache escucha en el 80 y en el 9999. Para WordPress usaré el 8080. No uso el 80 porque ya lo ocupa otro sitio y cada virtualhost de este tipo necesita su propio puerto; así no interfiero con los sitios existentes. (Si ya existiera Listen 8080, habría que usar el 8081.)

Apartado 2. Crear la base de datos y su usuario
bash
sudo mariadb
sql
CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '********';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
Orden	Qué hace
CREATE DATABASE	Crea la base de datos wordpress. utf8mb4 permite tildes, ñ y emojis.
CREATE USER	Crea wp_user, que solo se puede conectar desde la propia máquina (localhost).
GRANT ALL PRIVILEGES ON wordpress.*	Le da todos los permisos, pero solo sobre la base wordpress.
FLUSH PRIVILEGES	Aplica los cambios de permisos.

Comprobación (la contraseña no se ve al escribirla):

bash
mariadb -u wp_user -p wordpress
sql
SHOW TABLES;
EXIT;

Salida: Empty set (la base existe, el usuario entra y aún no hay tablas).

📸 img/02-comprobacion-usuario.png (sin contraseña visible)

Pregunta: ¿Por qué creamos wp_user en lugar de usar root? ¿Qué significa wordpress.* en la orden GRANT?

Se crea un usuario propio por seguridad (mínimo privilegio): si WordPress o su contraseña se vieran comprometidos, el atacante solo podría tocar la base wordpress y no todo el servidor de bases de datos, como ocurriría con root. wordpress.* significa "la base de datos wordpress y todas sus tablas" (base.tablas).

Apartado 3. Descargar WordPress y colocarlo en su carpeta
bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
ls wordpress

wget descarga el fichero comprimido y tar -xzf lo descomprime (x extraer, z gzip, f fichero).

bash
sudo mkdir -p /var/www/wordpress
sudo cp -a /tmp/wordpress/. /var/www/wordpress/
ls /var/www/wordpress

El . final de /tmp/wordpress/. copia el contenido de la carpeta (incluidos ocultos) y no la carpeta en sí. -a conserva permisos y estructura.

Carpeta	Para qué sirve
wp-admin	Panel de administración.
wp-includes	Núcleo de WordPress. No se toca.
wp-content	Temas, plugins e imágenes subidas. Es lo único que se personaliza.

📸 img/03-contenido-wordpress.png

Pregunta: ¿Qué carpeta guarda tus temas, plugins e imágenes subidas? ¿Por qué colocamos WordPress en su propia carpeta y no dentro de la de otro de tus sitios?

La carpeta es wp-content (themes, plugins y uploads). WordPress va en su propia carpeta para mantener cada sitio aislado: así no se mezclan ficheros, no se rompen los otros sitios y cada virtualhost tiene su propio DocumentRoot.

Apartado 4. Propietario y permisos
bash
sudo chown -R www-data:www-data /var/www/wordpress
sudo find /var/www/wordpress -type d -exec chmod 755 {} \;
sudo find /var/www/wordpress -type f -exec chmod 644 {} \;
ls -l /var/www/wordpress
Permiso	Significado
755 en carpetas	El propietario lee, escribe y entra; los demás solo leen y entran.
644 en ficheros	El propietario lee y escribe; los demás solo leen.

Apache se ejecuta como www-data, por lo que debe ser propietario para que WordPress pueda escribir (crear su configuración y guardar las imágenes subidas).

📸 img/04-permisos.png

Pregunta: ¿Qué pasaría si el propietario fuera root? ¿Por qué no se usa nunca 777?

Si el propietario fuera root, www-data solo podría leer: WordPress no podría crear wp-config.php, subir imágenes ni instalar plugins o actualizaciones. 777 da lectura, escritura y ejecución a todo el mundo: cualquier usuario o proceso del sistema podría modificar o sustituir ficheros de la web (por ejemplo, subir código malicioso), así que es un grave riesgo de seguridad.

Apartado 5. Crear el virtualhost de WordPress
5.1. Abrir el puerto 8080
bash
sudo nano /etc/apache2/ports.conf
apache
Listen 80
Listen 9999
Listen 8080

Se añade Listen 8080 sin borrar las líneas existentes.

5.2. Crear el fichero del sitio
bash
sudo nano /etc/apache2/sites-available/wordpress.conf
apache
<VirtualHost *:8080>
    ServerName wordpress.local
    DocumentRoot /var/www/wordpress
    <Directory /var/www/wordpress>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
Directiva	Qué hace
<VirtualHost *:8080>	Atiende las peticiones que llegan al puerto 8080.
DocumentRoot	Carpeta donde está WordPress.
AllowOverride All	Permite usar .htaccess (necesario para los enlaces permanentes).
Require all granted	Permite el acceso a cualquier visitante.
ErrorLog / CustomLog	Registros propios del sitio para diagnosticar fallos.
5.3. Activar el sitio
bash
sudo a2enmod rewrite
sudo a2ensite wordpress.conf
sudo apachectl configtest
sudo systemctl restart apache2
ls /etc/apache2/sites-enabled/
a2enmod rewrite: activa el módulo que permite los enlaces permanentes.
a2ensite: habilita el nuevo sitio (crea un enlace en sites-enabled).
configtest: debe responder Syntax OK.
No se ejecuta a2dissite: los sitios anteriores siguen habilitados.
Si el firewall está activo: sudo ufw allow 8080/tcp.

📸 img/05-ports-conf.png, img/05-wordpress-conf.png, img/05-sites-enabled.png

Pregunta: ¿Por qué no ha hecho falta tocar ninguno de los sitios anteriores? ¿Qué dos cosas distinguen a este sitio de los demás?

Cada sitio de Apache tiene su propio fichero de configuración y Apache decide cuál responde según el puerto/nombre de la petición, así que añadir uno nuevo no modifica los demás. Este sitio se distingue por su puerto (8080) y por su DocumentRoot (/var/www/wordpress).

Apartado 6. Instalación desde el navegador

Se obtiene la IP de la máquina virtual con ip a y se entra desde el navegador del anfitrión:

http://IP_DE_TU_VM:8080

Importante: WordPress guarda en su base de datos la dirección con la que se instala y siempre redirige a ella. Si se instala por IP, hay que entrar siempre por esa IP.

Pasos del asistente: idioma Español → bienvenida → datos de la base de datos → iniciar instalación → datos del sitio → éxito e inicio de sesión.

Datos de la base de datos

Campo	Valor	Para qué sirve
Nombre de la base de datos	wordpress	Cuál de las bases de MariaDB usar.
Nombre de usuario	wp_user	Usuario con el que WordPress entra en MariaDB (no es el administrador de la web).
Contraseña	********	La del apartado 2.
Servidor de la base de datos	localhost	MariaDB está en esta misma máquina.
Prefijo de tablas	wp_	Texto con el que empiezan las tablas.

Datos del sitio

Campo	Valor
Título del sitio	(el que elijas)
Nombre de usuario	(el tuyo, no admin)
Contraseña	********
Correo electrónico	alumno@example.com
Visibilidad en buscadores	Marcado "Disuadir a los motores de búsqueda" (servidor de pruebas)

📸 img/06-datos-bd.png, img/06-datos-sitio.png, img/06-escritorio.png

Si aparece "Error al establecer una conexión con la base de datos": revisar (1) systemctl is-active mariadb, (2) usuario y contraseña con mariadb -u wp_user -p, (3) conector PHP con php -m | grep mysqli.

Pregunta: ¿Qué diferencia hay entre wp_user y el usuario administrador de WordPress? ¿Dónde vive cada uno?

wp_user es el usuario de MariaDB: lo usa WordPress para conectarse a la base de datos y vive en MariaDB. El administrador es el usuario de la web: se usa para entrar al panel de WordPress y vive en la tabla wp_users de la base de datos wordpress.

Apartado 7. ¿Dónde ha guardado WordPress cada cosa?
7.1. El fichero de configuración
bash
sudo grep -E "DB_NAME|DB_USER|DB_HOST" /var/www/wordpress/wp-config.php
sudo chmod 640 /var/www/wordpress/wp-config.php
ls -l /var/www/wordpress/wp-config.php

640: propietario lee y escribe, el grupo solo lee y el resto nada. Se restringe porque el fichero contiene la contraseña de la base de datos. (No se usa cat para no mostrarla.)

7.2. Las tablas de la base de datos
bash
mariadb -u wp_user -p wordpress
sql
SHOW TABLES;
SELECT option_name, option_value FROM wp_options WHERE option_name IN ('siteurl','blogname');
SELECT user_login FROM wp_users;
EXIT;

Aparecen unas doce tablas creadas por el asistente. wp_options guarda los ajustes del sitio y wp_users sus usuarios.

📸 img/07-tablas.png, img/07-consultas.png

Pregunta: ¿Qué valor tiene siteurl? Relaciónalo con el aviso del apartado 6. ¿En qué tabla está el administrador que creaste?

siteurl vale http://IP_DE_TU_VM:8080 (pon tu IP real), que es la dirección con la que instalé WordPress. Coincide con el aviso: WordPress guarda esa dirección en la base de datos y siempre redirige a ella, por lo que si entro con otra dirección me redirigirá a esta. El administrador está en la tabla wp_users.

Apartado 8. Primera entrada e imagen: ficheros frente a base de datos

En el Escritorio de WordPress: se crea la entrada "Hola desde mi WordPress", se publica, y en Medios se sube una imagen de menos de 2 MB.

bash
mariadb -u wp_user -p wordpress
sql
SELECT ID, post_title, post_type, post_status FROM wp_posts ORDER BY ID DESC LIMIT 5;
EXIT;
bash
ls -R /var/www/wordpress/wp-content/uploads

La entrada aparece como fila en wp_posts. La imagen está como fichero en wp-content/uploads y además como fila de tipo attachment.

📸 img/08-consulta-posts.png, img/08-uploads.png

Pregunta: ¿Qué se guarda en la base de datos y qué en ficheros? Si tuvieras que hacer una copia de seguridad completa, ¿qué dos cosas copiarías?

En la base de datos van los contenidos y ajustes: entradas, páginas, usuarios, comentarios y configuración. En ficheros van el programa, los temas, los plugins y las imágenes subidas. Para una copia completa copiaría: (1) un volcado de la base de datos (por ejemplo con mariadb-dump) y (2) la carpeta /var/www/wordpress (sobre todo wp-content y wp-config.php).

Apartado 9. Comprobación final
bash
systemctl is-active apache2 mariadb
ls /etc/apache2/sites-enabled/

Desde el navegador del anfitrión se comprueba que los sitios anteriores funcionan como antes.

📸 img/09-comprobacion.png, img/09-sitio-anterior.png

Elemento	Dónde	Estado
Base de datos wordpress	MariaDB	Creada
Usuario wp_user	MariaDB	Creado, con permisos solo sobre wordpress
Ficheros de WordPress	/var/www/wordpress	Colocados, propietario www-data
Sitio de Apache	wordpress.conf, puerto 8080	Activo
WordPress	http://IP_DE_TU_VM:8080	Instalado y accesible
Sitios anteriores	Sus puertos de siempre	Siguen funcionando

Pregunta: ¿Por qué no se ha estropeado ninguno de los sitios anteriores al añadir WordPress?

Porque WordPress tiene su propio virtualhost (fichero, puerto 8080 y carpeta propios) y no se ha modificado ni desactivado ningún sitio anterior: solo se han añadido un Listen y un .conf nuevos.

Al terminar se hace una instantánea (snapshot) de la máquina virtual como punto de partida seguro para la siguiente práctica.
