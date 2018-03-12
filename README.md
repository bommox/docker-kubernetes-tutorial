# docker-kubernetes-tutorial

## Pre-requisitos

Para llevar a cabo estos ejemplos se emplea Docker. Por tanto se requiere una de estas dos opciones:

- Docker instalado en local (windows / unix / mac)
- Play with Docker (https://labs.play-with-docker.com/)

## Ejercicio 1 - Link & Volumes

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
docker run --name myadmin -d --link some-mariadb -p 8080:80 -e PMA_HOST=some-mariadb  phpmyadmin/phpmyadmin
```

Con este ```run``` estamos indicando lo siguiente:

- --name myadmin: nombre del contenedor
- -d: modo detached
- --link some-mariadb con esto indicamos que este contenedor puede acceder al contenedor some-mariadb por el puerto 3306, que debe estar expuesto.
- -e PMA_HOST=some-mariadb : Establece la variable de entorno PMA_HOST que debe apuntar al contenedor de la base de datos.
- -p 8080:80: vinculamos el puerto 8080 de nuestro host anfitrión al puerto 80 del contenedor. Al tratarse de una aplicación web, debemos poder acceder por el navegador.

### Persistencia

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

Si vamos a ver los datos de nuestra tabla, veremos que sigue existiendo.
