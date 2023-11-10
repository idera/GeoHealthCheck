# GeoHealthCheck
Repositorio para levantar el servicio de GeoHealthCheck

## Contenido

Se encuentra un archivo `docker-compose.yml` que despliega un entorno Docker para ejecutar GeoHealthCheck (GHC), una herramienta de monitoreo de servicios geoespaciales, con PostgreSQL como base de datos backend. 

A continuación, un desglose de cada sección:
```
version: "3"
```
Establece la versión de la sintaxis de Docker Compose que se va a utilizar.
```
services:
  ghc_web:
    image: geopython/geohealthcheck:latest
    container_name: ghc_web
    restart: unless-stopped
    env_file:
      - ghc.env
    links:
      - postgis_ghc
    depends_on:
      - postgis_ghc
    ports:
      - 8083:80
```

Define un servicio llamado ghc_web que utiliza la imagen `geopython/geohealthcheck:latest`. Este servicio se reiniciará automáticamente a menos que se detenga explícitamente. Se carga la configuración del entorno desde dos archivos .env. Se vincula con el servicio postgis_ghc, dependiendo de él. El contenedor escuchará en el puerto 8083 del host y se mapeará al puerto 80 del contenedor.
```
  ghc_runner:
    image: geopython/geohealthcheck:latest
    container_name: ghc_runner
    restart: unless-stopped
    env_file:
      - ghc.env
    links:
      - postgis_ghc
    depends_on:
      - postgis_ghc
    entrypoint:
      - /run-runner.sh
```

Define otro servicio llamado ghc_runner con configuración similar a ghc_web. Sin embargo, este servicio utiliza un script de entrada (entrypoint) llamado /run-runner.sh.
```
  postgis_ghc:
    image: mdillon/postgis:10-alpine
    container_name: postgis_ghc
    restart: unless-stopped
    env_file:
      - ghc.env
    volumes:
      - ghc_pgdb:/var/lib/postgresql/data
    expose:
      - 5432
```


Configura el servicio de base de datos PostgreSQL llamado postgis_ghc. Utiliza la imagen `mdillon/postgis:10-alpine`. Se reinicia automáticamente a menos que se detenga explícitamente. Carga la configuración del entorno desde un archivo `.ghc.env`. También especifica un volumen para almacenar los datos de la base de datos PostgreSQL. Expone el puerto 5432 para que otros servicios puedan acceder a la base de datos.

## Instalación

Para proceder a la instalación, necesitaremos generar un archivo llamado `ghc.env` con la estructura similar a la del archivo que se encuentra en el repositorio, llamado `ghc.env.example`

Podríamos ejecutar
```
cp ghc.env.example ghc.env
```

luego de esto, podemos proceder a levantar los servicios de GHC y Postgresql mediante el uso de docker compose

`docker compose up -d`

## Logs

No se especifica directorio para logs. se pueden ver los logs ejecutando `docker compose logs -f [--tail=xxx]`

## Backups y persistencia

Se utilizan el volumen `ghc_pgdb`. Que por defecto, la ubicación de este, se encuentra en `/var/lib/docker/volumes`

## Acceso
El servicio por defecto, se levanta en el puerto 8083. en caso de cambiarlo, ver linea 27 del archivo docker-compose.yml

## Reseteo
Para reiniciar los servicios, podemos ejecutar `docker compose restart`