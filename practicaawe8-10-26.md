# Guía sencilla: un nombre, dos webs.

1. Conectas el ssh de tu pc a la maquina virtual de esta manera: ssh pau@192.168.56.10 y pones la contraseña y ya.
2. Luego pones este comando: sudo mkdir -p /var/www/smr/web /var/www/smr/intranet y la contraseña.
3. Ahora pones esto, echo "Bienvenidos a SMR</h1 | sudo tee /var/www/smr/web/index.html y debajo esto,echo h1 Intranet de SMR h1 | sudo tee /var/www/smr/intranet/intranet.html
4. Ahora procedemos a crear el usuario de la intranet con estos comandos, sudo apt install apache2-utils -y y sudo htpasswd -c /etc/apache2/.htpasswd alumno
5. Os pedirá la contraseña dos veces. Ese usuario y esa contraseña serán los que pida la intranet.
6. Para decirle a Apache que escuche en el puerto 9999 hacemos esto, sudo nano /etc/apache2/ports.conf, debajo de la línea Listen 80 añadid una línea nueva:Listen 9999
7. Para crear el VirtualHost, sudo nano /etc/apache2/sites -available/smr.conf
8. Dentro del archivo escribes esto
9.  <VirtualHost *:80>
        ServerName www.smr.com
        DocumentRoot /var/www/smr/web
    </VirtualHost>
    
    <VirtualHost *:9999>
        ServerName www.smr.com
        DocumentRoot /var/www/smr/intranet
        DirectoryIndex intranet.html
    
     <Directory /var/www/smr/intranet>
         AuthType Basic
         AuthName "Intranet SMR"
         AuthUserFile /etc/apache2/.htpasswd
         Require valid-user
     </Directory>
  </VirtualHost> 
10. Para Activar el sitio y reiniciar Apache,
11. sudo a2ensite smr.conf , sudo a2dissite 000-default.conf , sudo apachectl configtest , sudo systemctl restart apache2
12. Y ahora escribes en la maquina real tus ips y ya te debería dejar.
13. Y en un windows de maquina virtual escribes, www.smr.com y www.smr.com:9999 ya ya esta, sino te funciona tienes que ir a la carpeta de hosts y darle permisos al usuario y ya estaría.
  
     
