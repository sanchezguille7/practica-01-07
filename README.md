# Practica-01-07
En esta práctica vamos a realizar la administración de un sitio **WordPress** desde el terminal con la utilidad **WP-CLI**
Con **WP-CLI** podemos realizar las mismas tareas que se pueden hacer desde el panel de administración web de **WordPress**, pero desde la línea de comandos.
En esta práctica tendremos 4 directorios:

 - - scripts
   		- .env
   		-  install_lamp.sh
   		-  setup_letsencrypt_https.sh
   		- deploy_wordpress_with_wpcli.sh
   	- conf
   		- 000-default.conf
   	- php
   		- index.php
   	- htaccess
   		- .htaccess

## scripts
Aquí automatizaremos los procesos de instalación y configuración de **WordPress** y **WP**.
### install_lamp.sh
Muestra todos los comandos que se van ejecutando

    set -ex

Actualizamos los repositorios

    apt update

Actualizamos los paquetes 

    apt upgrade -y

Instalamos el servidor web **Apache**

    apt install apache2 -y

Copiamos la configuración predeterminada del servidor

    cp ../conf/000-default.conf /etc/apache2/sites-available/000-default.conf

Instalar  el sistema gestor de datos **MySQL**

    apt install mysql-server -y
Descomentar las siguientes líneas y asignar valores a DB_USER y DB_PASSWD
#DB_USER=usuario
#DB_PASSWD=contraseña

    mysql -u $DB_USER -p$DB_PASSWD < ../sql/database.sql

Instalamos **PHPMyAdmin**

    sudo apt install php libapache2-mod-php php-mysql -y

Reiniciamos servicio

    systemctl restart apache2

Modificamos el propietario

    chown -R www-data:www-data /var/www/html

### setup_letsencrypt_https.sh
Muestra todos los comandos que se van ejecutando

    set  -ex

  

Actualizamos los repositorios

    apt  update

Actualizamos los paquetes

    apt upgrade -y
 

Ponemos las variables del archivo *.env*

    source  .env

  

Instalamos y actualizamos **Snap**

    snap  install  core

    snap  refresh  core

  

Eliminamos cualquier instalación previa de **Certbot** con **apt**

    apt  remove  certbot

  

Instalamos la aplicación **Certbot**

    snap  install  --classic  certbot

  

Creamos una alias para el comando **Certbot**.

    ln  -fs  /snap/bin/certbot  /usr/bin/certbot

  

Obtenemos el certificado y configuramos el servidor web **Apache**

Ejecutamos el comando **Certbot**

    certbot  --apache  -m  $CERTIFICATE_EMAIL  --agree-tos  --no-eff-email  -d  $CERTIFICATE_DOMAIN  --non-interactive


### deploy_wordpress_with_wpcli.sh
Muestra todos los comandos que se van ejecutando

    set -ex

Actualizamos los repositorios

    apt update

Actualizamos los paquetes 

    apt upgrade -y

Ponemos las variables del archivo *.env*

    source .env

Borramos instalaciones previas de **wp-cli**

    rm -rf /tmp/wp-cli.phar

Descargamos el archivo *wp-cli.phar* del repositorio oficial de **WP-CLI.** 

    wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar 

Le asignamos permisos de ejecución al archivo *wp-cli.phar*.

    chmod +x wp-cli.phar

Movemos el archivo *wp-cli.phar* al directorio */usr/local/bin/* con el nombre **wp** para poder utilizarlo sin necesidad de escribir la ruta completa donde se encuentra.

    mv wp-cli.phar /usr/local/bin/wp

Eliminamos instalaciones revias de **WordPress**

    rm -rf /var/www/html/*

Descargamos el código fuente de **WordPress** en */var/www/html*

    wp core download --locale=es_ES --path=/var/www/html --allow-root 

Creamos la base de datos y el usuario de la base de datos

    mysql -u root <<< "DROP DATABASE IF EXISTS $WORDPRESS_DB_NAME"
    mysql -u root <<< "CREATE DATABASE $WORDPRESS_DB_NAME"
    mysql -u root <<< "DROP USER IF EXISTS $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"
    mysql -u root <<< "CREATE USER $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL IDENTIFIED BY '$WORDPRESS_DB_PASSWORD'"
    mysql -u root <<< "GRANT ALL PRIVILEGES ON $WORDPRESS_DB_NAME.* TO $WORDPRESS_DB_USER@$IP_CLIENTE_MYSQL"

Creamos el archivo *wp-config*

    wp config create \
      --dbname=$WORDPRESS_DB_NAME \
      --dbuser=$WORDPRESS_DB_USER \
      --dbpass=$WORDPRESS_DB_PASSWORD \
      --path=/var/www/html \
      --allow-root

Instalamos el **WordPress** con variables personalizadas

      wp core install \
      --url=$CERTIFICATE_DOMAIN \
      --title="$WORDPRESS_TITTLE" \
      --admin_user=$WORDPRESS_ADMIN_USER \
      --admin_password=$WORDPRESS_ADMIN_PASS \
      --admin_email=$WORDPRESS_ADMIN_EMAIL \
      --path=/var/www/html \
      --allow-root

Instalar un tema con **wp**

    wp theme install Twenty Twenty-One

Instalar un plugin con **wp**

    wp plugin install akismet

Y para activar el plugin 

    wp plugin activate akismet
