# docker-kubernetes-tutorial

## Pre-requisitos

Para llevar a cabo estos ejemplos se emplea Docker. Por tanto se requiere una de estas dos opciones:

- Docker instalado en local (windows / unix / mac)
- Play with Docker (https://labs.play-with-docker.com/)

## Ejercicio 1 - Link & Volumes

### Objetivos

- Link entre contenedores
- Persistencia en volúmenes: bind-mount vs volume

### Contenedor DDBB MariaDB

Creamos una instancia de **mariadb**, con lo que tendríamos una base de datos:

```
docker run --name some-mariadb -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mariadb
```
https://hub.docker.com/_/mariadb/

Repasamos los flags:

- -name: nombre de la imagen docker, some-mariadb
- -e: setea la variable de entorno MYSQL_ROOT_PASSWORD a "my-secret-pw"
- -d: modo detached, para que la terminal no quede bloqueada por la ejecución
- mariadb: es el nombre de la imagen docker. Sin tag indica la versión latest.

Podemos ejecutar un ```docker ps```para confirmar que está iniciado.

```sh
$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
7815685c51cc        mariadb             "docker-entrypoint.s…"   About a minute ago   UpAbout a minute   3306/tcp            some-mariadb
```

¿Por qué sabe que el puerto es el 3306 si no lo hemos especificado en el docker run? Porque está definido así en su dockerfile.
El argumento del --port en docker run lo que hace es vincular un puerto del anfitrión a uno expuesto por el contenedor.

Podemos acceder a la base de datos para crear una tabla y un registro.

```sh
$ docker exec -ti some-mariadb mysql -u root -pmy-secret-pw

... Entraremos a la consola mysql
```
*NOTA: Podemos usar el nombre del contenedor o bien el inicio del id. *

Sobre los flags del exec:

- -ti: habilita el modo terminal interactivo
- some-mariadb: es el nombre del contenedor.
- mysql -u root -pmysecret-pw : es el comando a ejecutar en el contenedor

Escribimos algunos comandos SQL:
```SQL
> SHOW DATABASES;
// Muestra las BBDD, saldrán las 3 por defecto
```

Salimos con ```Ctrl+d``` o escribiendo ```EXIT```

Podríamos crear tablas y utilizarlas, pero mejor aún, vamos a desplegar un phpmyadmin para gestionar esta base de datos.

### Contenedor Aplicación phpmyadmin

Imagen: https://hub.docker.com/r/phpmyadmin/phpmyadmin/

```
docker run --name myadmin -d -p 8080:80 phpmyadmin/phpmyadmin
```
Con esto hemos levantado un phpmyadmin en el puerto 8080 de nuestro host anfitrión. Pero veremos que no podemos acceder pues no conecta con la base de datos.

Para ello debemos crear una red y conectar ambos contenedores:

```
docker network create tuto-net
docker network connect tuto-net some-mariadb
docker network connect tuto-net myadmin
```

Ahora los contenedores pueden verse. Por ejemplo:

```
docker exec -ti myadmin ping some-mariadb
```

Pero seguimos sin poder entrar. Hemos de añadir variables de entorno al contenedor de PhpMyAdmin.
```
docker rm -f myadmin
docker run --name myadmin -d --network tuto-net -p 8080:80 -e PMA_HOST=some-mariadb  phpmyadmin/phpmyadmin
```

Con este ```run``` estamos indicando lo siguiente:
- --network tuto-net: añade el contenedor a la red
- -e PMA_HOST=some-mariadb : Establece la variable de entorno PMA_HOST que debe apuntar al contenedor de la base de datos.

### Persistencia

![docker persistence](https://docs.docker.com/storage/images/types-of-mounts.png)

Probemos a crear una tabla en la base de datos del contenedor **some-mariadb** en un namespace cualquiera. E insertamos unos datos en esa tabla.

Ahora realizemos las siguientes pruebas:
```
docker stop some-mariadb
```

Con esto paramos el contenedor, veremos que phpmyadmin ya no puede conectar.

Lo volvemos a levantar:
```
docker start some-mariadb
```

Si vamos a ver los datos de nuestra tabla, veremos que sigue existiendo. Pero, ¿por qué?.

Es cierto que los contenedores Docker no tienen persistencia, por lo que vamos a analizar el Dockerfile de la base de datos.
https://github.com/docker-library/mariadb/blob/0ab4688d0e6d3a81d7cd19e04db44fb499c49d6a/10.3/Dockerfile

Ahí veremos que hay definido un VOLUME a la carpeta ```/var/lib/mysql``` que es donde se almacenan los datos en una base de datos.

Docker nos puede mostrar los volúmenes de la siguiente forma:
```
docker volume ls
```
Y con ```docker volume inspect <vol-name>``` podemos ver la ruta física que usa para ello.

Podemos hacer un ls de esa ruta dentro y fuera del contenedor para ver que el contenido es el mismo.

En este caso, el volumen venía definido en el dockerfile, por lo que se ha creado automáticamente al hacer el "docker run".

### Volumen manual con aplicación web

Levantamos un contenedor apache:
```
docker run -d -it --name webapp -p 8888:80 httpd:2.4
```

Ahora si vamos a la url por el puerto 8888 veremos el texto **It works!** que hay por defecto en el contenedor.

Vamos a modificar el contenido del index.html:
```
$ docker exec -ti webapp bash
# (webapp) echo "<h1>Mola</h1>" > htdocs/index.html
# exit
```

Al refrescar en el navegador, veremos el cambio.

Pero ahora probamos a recrear el contenedor
```
docker rm -f webapp
docker run -d --name webapp -p 8888:80 httpd:2.4
```

Vemos que ahora no existen los datos, ha vuelto al origen.

### Bind mount

```
docker rm -f webapp
docker run -d --name webapp -v ${PWD}/www:/usr/local/apache2/htdocs/ -p 8888:80 httpd:2.4
```
Ahora si volvemos a repetir el proceso, sí se guardará la información.


### Volume

La forma aconsejada por Docker es crear un Volume, y utilizarlo entre aplicaciones. No depender de las rutas del host anfitrión.

Para crear un volumen cualquiera:
```
docker volume create www-volume
```

Y luego se utilizar con el flag ```-v``` de la misma forma que antes

Y lo podríamos reutilizar entre diferentes aplicaciones:
```
docker rm -f webapp
docker run -d --name webapp -v www-volume:/usr/local/apache2/htdocs/ -p 8888:80 httpd:2.4
```

*NOTA: al usar un volume, si este estaba vacío o no existía, se le copian los datos del contenedor. En un bind mount no ocurre así*

Podemos levantar otra aplicación que también haga uso del mismo voluen
```
docker run -d -v www-volume:/srv -p 8889:80 hacdias/filebrowser
```

