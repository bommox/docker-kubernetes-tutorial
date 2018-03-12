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

