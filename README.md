# Prerequisitos

Como vamos a servir la aplicación utilizando el servidor web Apache, debemos instalarlo previamente:

```
sudo yum install epel-release
sudo yum install python-pip httpd httpd-devel mod_wsgi
```

# INSTALAR MYSQL

## Agregar repo e instalar el server

Descargar repo desde https://dev.mysql.com/downloads/repo/yum/

```

# Agregar el repo: 

sudo rpm –ivh mysql80-community-release-el7-3.noarch.rpm

# Instalar el server: 

sudo yum install mysql-server

# Iniciar el servicio: 

sudo systemctl start mysqld`
```

## Configurar mysql

Buscar el password por defecto que genera: 

`sudo grep ‘temporary password’ /var/log/mysqld.log`

Para configurar la instancia, debemos ejecutar: 

`sudo mysql_secure_installation`

## Configurar la base

```sql
create database django_project character set = latin1;

create user panel@localhost identified by 'Panel.1234';

use django_project;

grant all privileges on django_project.* to 'panel'@'localhost';

flush privileges;
```

# INSTALAR PYTHON3 DJANGO, APACHE Y OTROS...

## Instalar python y virtualenv

```
sudo yum install python3.x86_64

pip install virtualenv
```

## Configurar entorno virtual

Crear directorio donde vamos a alojar el proyecto

`mkdir /var/www/proyecto`

Crear el entorno virtual

```
cd /var/www/proyecto

virtualenv -p python3.6 env36
```

Activar el entorno virtual

`source env36/bin/activate`

## Instalar django y dependencias del proyecto

Para la aplicación vamos a utilizar como motor de base de datos MySQL, por lo tanto debemos instalar el paquete mysqlclient que nos permite trabajar con dicho motor desde python.

```
sudo yum install python3-devel mysql-devel

pip install mysqlclient
```

Ahora si, instalamos el framework. Para esto creamos el archivo *requirements.txt* y vamos a incluir todas las dependencias a instalar:

```
django
djangorestframework
markdown
django-filter
django-guardian

```
Para instalar sólo debemos ejecutar:

`pip install -r requirements.txt`

# Crear la app

Lo primero a realizar es crear el proyecto. Para esto utilizamos los comandos del framework:

`django-admin.py startproject panel .`

Luego editamos el archivo panel/settings.py para indicar:

1. el directorio con el contenido estático a servir.
1. la configuración de base de datos a utilizar.

```python

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_project',
        'USER': 'panel',
        'PASSWORD': 'Panel.1234',
        'HOST': '',
        'PORT': '',
    }
}

# Agregar al final del archivo
STATIC_ROOT = os.path.join(BASE_DIR, "static/")

```

Ahora, ejecutamos el comando *migrate* para crear las estructuras básicas en la base de datos.

`python manage.py migrate`

## Servir la app con Apache

Antes de editar la configuración debemos instalar el módulo *mod-wsgi* de python:

`pip install mod-wsgi`

Luego, editar el archivo */etc/httpd/conf/httpd.conf* agregando las siguientes líneas:

```apache

LoadModule wsgi_module /var/www/proyecto/env36/lib/python3.6/site-packages/mod_wsgi/server/mod_wsgi-py36.cpython-36m-x86_64-linux-gnu.so

WSGIPythonHome /var/www/proyecto/env36/lib
WSGIRestrictEmbedded On
```

Crear el archivo */etc/httpd/conf.d/panel.conf* con el siguiente contenido:

```apache

Alias /static /var/www/proyecto/static

<Directory /var/www/proyecto/static>
    Require all granted
</Directory>

<Directory /var/www/proyecto>
    Require all granted
    <Files wsgi.py>
        Require all granted
    </Files>
</Directory>

WSGIDaemonProcess panel python-home=/var/www/proyecto/env36 python-path=/var/www/proyecto
WSGIProcessGroup panel
WSGIScriptAlias / /var/www/proyecto/panel/wsgi.py

```

Verificar que el usuario apache tenga permisos sobre el directorio */var/www/proyecto*

# Fuentes:

* https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-centos-7
* https://github.com/GrahamDumpleton/mod_wsgi/issues/336
* https://stackoverflow.com/questions/17773963/vagrant-synced-folder-permission-issue-with-apache
