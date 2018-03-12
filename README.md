# docker-kubernetes-tutorial

## Pre-requisitos

Para llevar a cabo estos ejemplos se emplea Docker. Por tanto se requiere una de estas dos opciones:

- Docker instalado en local (windows / unix / mac)
- Play with Docker (https://labs.play-with-docker.com/)

## Ejercicio 1 - Link & Volumes

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
$ docker exec some-mariadb mysql -u root -pmy-secret-pw

... Entraremos a la consola mysql
```

*NOTA: Podemos usar el nombre del contenedor o bien el inicio del id. *

Escribimos algunos comandos SQL:
```SQL
> SHOW DATABASES;
// Muestra las BBDD, saldrán las 3 por defecto
```

Podríamos crear tablas y utilizarlas, pero mejor aún, vamos a desplegar un phpmyadmin para gestionar esta base de datos.



