---
layout: post
title:  LAMP en Manjaro
categories: [Linux, Manjaro, ES, Apache, MariaDB, PHP]
excerpt: Como instalar a día de hoy el stack LAMP (Apache + MariaDB + PHP) en Manjaro con las configuraciones más básicas. Debería ser suficiente, por ejemplo, para poder usar WordPress en local.
---

LAMP: Linux + Apache + MariaDB + PHP

## Consideraciones iniciales

Cambiaremos a usuario root para que sea más cómodo, ya que casi todos los comandos necesitan permisos de root. Si no quieres hacer este paso, añade `sudo` a todos los comandos que se nombran después:
```
sudo su -
```

Actualiza el sistema:
```
systemctl -Syu
```
(Reicinia si fuese necesario)

Cuando se pida editar un fichero, se puede usar un editor de texto en línea de comandos como `nano` o `vim`.

También sería posible usar un editor de texto gráfico. Si, por ejemplo, usas Plasma puedes usar el editor `kate`, pero tendrías que abrirlo desde una sesión de tu propio usuario (y sin `sudo`). Plasma no permite abrirlo como root, pero a cambio te pregunta tu contraseña si necesita más permisos para guardar el fichero.

En Gnome creo que se usa `gedit`.

## Apache

Instalar servidor web
```
pacman -S apache
```

Ajustar los permisos de la carpeta pública `/srv/http`:
```
setfacl -m u:http:rwx /srv/http
setfacl -d -m u:http:rwx /srv/http
```

Conviene darle a tu propio usuario permisos sobre la misma carpeta:
```
setfacl -m u:usuario:rwx /srv/http
setfacl -d -m u:usuario:rwx /srv/http
```
(siendo `usuario` tu propio usuario)

Edita el fichero `/etc/httpd/conf/httpd.conf` y modifica las siguientes líneas de la siguiente manera:
```
Listen 127.0.0.1:80
#LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
# Al final de la lista de "LoadModule" añadir:
LoadModule php_module modules/libphp.so
AddHandler php-script .php
<Directory "/srv/http">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
# Al final del fichero
Include conf/extra/php_module.conf
```


## MariaDB

Instala el servidor de MariaDB y realiza las configuraciones iniciales:
```
pacman -S mariadb
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
systemctl enable --now mariadb
mariadb-secure-installation
```
Responder lo siguiente a las preguntas:
- Root password: None (Solo pulsar **Enter**)
- Switch to unix_socket authentication: Yes (**Enter**)
- Change the root password? No (**n**)
- Remove anonymous users? Yes (**Enter**)
- Disallow root login remotely Yes (**Enter**)
- Remove test database and access to it? Yes (**Enter**)
- Reload privilege tables now? Yes (**Enter**)

Editar el fichero `/etc/my.cnf.d/server.cnf` y justo debajo de la línea `[mariadb]` añadir:
```
bind-address = localhost
skip-networking
```
Grabar, salir y ejecutar:
```
systemctl restart mariadb
```

## PHP

Instalar paquetes:
```
pacman -S php php-gd php-fpm php-imagick libiconv
```

Editar el fichero `/etc/php/conf.d/local.ini` y añade lo siguiente:
```
[Date]
date.timezone = Europe/Madrid

[PHP]
error_reporting = E_ALL
display_errors = On
log_errors = On
;extension=curl
extension=exif
extension=imagick
extension=intl
extension=mysqli
;extension=zip
# Opcionales:
extension=gd
extension=iconv
extension=shmop
```
Si tu zona geográfica es otra, usa tu zona. Por ejemplo: `America/Bogota`. La documentación de PHP incluye una [lista de zonas de husos horarios](https://www.php.net/manual/en/timezones.php).


## Finalización

Habilitar y arrancar el servidor web:
```
systemctl enable --now httpd
```

Ya puedes salir de la sesión de root con `exit`.

Ahora podrías abrir tu web. Para probar, crea un fichero de prueba:
```
echo "<?php phpinfo(); ?>" > /srv/http/test.php
```
Y ábrelo en tu navegador desde la dirección: [http://localhost/test.php](http://localhost/test.php).

Crea lo que quieras en la ruta `/srv/http/` y prueba a abrirlo desde tu navegador.