# Practica_1.1
Repositorio para la practica 1.1 de IAW

## Instalación Apache, PHP y MySQL
### Instalación de Apache
Para la instalación de **Apache** lo primero que haremos en nuestro archivo de creación de la **LAMP** sera un `apt update` y `apt upgrade` para tener los paquetes actualizados y para automatizarlo tendremos que poner *-y* seguido de *upgrade*, después realizamos la instalación del servicio apache `apt install apache2 -y`, habilitamos el modulo rewrite para mejorar la estética `a2enmod rewrite`.  
#### Configuración archivo apache
Para configurar el archivo de apache, lo primero que haremos será crear un archivo llamado `000-default.conf`el cual tendrá el siguiente contenido: 
><VirtualHost *:80>
>    #ServerName www.example.com
>    ServerAdmin webmaster@localhost
>    DocumentRoot /var/www/html/
>
>    DirectoryIndex index.php index.html
>
>    ErrorLog ${APACHE_LOG_DIR}/error.log
>    CustomLog ${APACHE_LOG_DIR}/access.log combined
> </VirtualHost>  

Copiamos el archivo a *sites-available* con el comando `cp ../conf/000-default.conf /etc/apache2/sites-available`.  
### Instalación php
Utilizando el comando `sudo apt install php libapache2-mod-php php-mysql -y` realizamos la instalación de algunos módulos que nos vendrán bien como el relacionado con apache y con mysql.  
Ademas crearemos otro archivo .sh para automatizar la instalación de php, en la cual tendremos que escribir el siguiente código:  
>echo "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2" | debconf-set-selections
>echo "phpmyadmin phpmyadmin/dbconfig-install boolean true" | debconf-set-selections
>echo "phpmyadmin phpmyadmin/mysql/app-pass password $PHPMYADMIN_APP_PASSWORD" | debconf-set-selections
>echo "phpmyadmin phpmyadmin/app-password-confirm password $PHPMYADMIN_APP_PASSWORD" | debconf-set-selections
  
Para automatizar las preguntas de instalación.  
A continuación realizamos la instalación de phpmyadmin `apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl -y`
Realizamos un `systemctl restart apache2` para reiniciar el servicio apache y que se integre la configuración realizada. Creamos un archivo de prueba php y lo copiamos en *html* `cp ../php/index.php /var/www/html`.
Comprobamos con la ip que el servidor apache y php funcionan  
![pagina php](/imagenes/1.png)  
Para mayor seguridad cambiamos el propietario y el grupo `chown -R www-data:www-data /var/www/html`
### Instalación de MySQL
Para la instalación de MySQL solo sera necesario el siguiente script `sudo apt install mysql-server -y`.  
## Instalación de adminer
A continuacion realizaremos los pasos para instalar adminer en el servidor.  
### Paso 1. Creamos un directorio para Adminer
Creamos una carpeta para adminer.  
>mkdir -p /var/www/html/adminer  
### Paso 2. Descargamos Adminer
Nos descargamos adminer.  
>wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-mysql.php -P /var/www/html/adminer
### Paso 3. Renombramos el nombre del script de Adminer
Para facilitarnos la entrada a adminer le cambiaremos el nombre.  
>mv /var/www/html/adminer/adminer-4.8.1-mysql.php /var/www/html/adminer/index.php
### Paso 4. Modificamos el propietario y el grupo del archivo
Para mayor seguridad cambiamos el usuario y grupo de adminer.
>chown -R www-data:www-data /var/www/html/adminer  

Comprábamos que funciona la pagina con adminer.  
![pagina adminer](/imagenes/2.png)  
## Instalación de GoAccess
Vamos a realizar la instalación de GoAccess para ver de forma más cómoda las estadísticas de de la pagina.
### Paso 1. Instalación GoAcces
Para instalar goAccess implementamos el siguiente script dentro del archivo tool que hemos creado previamente.
>apt install goaccess -y 
### Paso 2. Creamos un directorio para los informes estadísticos
Creamos un directorio para guardar los informes estadísticos que se generen.
>mkdir -p /var/www/html/stats
### Paso 3. Ejecutamos GoAccess en background 
Para que se actualice constantemente en tiempo real realizamos la siguiente configuración:
>goaccess /var/log/apache2/access.log -o /var/www/html/stats/index.html --log-format=COMBINED --real-time-html --daemonize  
Comprobamos que funciona el GoAccess.  
![pagina goaccess](/imagenes/3.png)
## Control de acceso con .htaccess
Vamos a realizar un control para el stats, para esto creamos un archivo de configuración que nombraremos como `000-default-htaccess.conf` y le introduciremos el siguiente código:
><VirtualHost *:80>
>  #ServerName www.example.com
>  ServerAdmin webmaster@localhost
>  DocumentRoot /var/www/html
>
>  DirectoryIndex index.php index.html
>
>  <Directory "/var/www/html/stats">
>    AllowOverride All
>  </Directory>
>
>  ErrorLog ${APACHE_LOG_DIR}/error.log
>  CustomLog ${APACHE_LOG_DIR}/access.log combined
> </VirtualHost>  

Aparte crearemos un archivo de configuración llamado `.htaccess` con el siguiente codigo:
> AuthType Basic
> AuthName "Acceso restringido"
> AuthBasicProvider file
> AuthUserFile "/etc/apache2/.htpasswd"
> Require valid-user  

Lo copiamos en sites-available `cp ../conf/000-default-htaccess.conf /etc/apache2/sites-available`, deshabilitamos el virtualhost que hay por defecto utilizando `a2dissite 000-default.conf`, habilitamos el nuevo virtualhost `a2ensite 000-default-htaccess.conf` y recargamos la configuración de apache `systemctl reload apache2`. A continuacion copiamos el archivo .htaccess `cp ../conf/.htaccess /var/www/html/stats`.  
La contraseña y usuario esta dentro de un archivo .env el cual esta oculto por protección. Comprobamos que al intentar acceder a stats nos solicita un usuario y una contraseña.
![pagina seguridad](/imagenes/4.png)
