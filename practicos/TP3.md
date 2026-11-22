docker run -v demo-db-data:/var/lib/postgresql/data  -e POSTGRES_USER=demo_user -e POSTGRES_PASSWORD=demo_pass --network demo-net --name demo-db postgres:16

# Trabajo Práctico 1
## 1
Ejecutando el comando:
`docker network create catalogo-net`
Obtenemos el id de la red:
`af3e196f401e416f6c8657a71c599689d8fc2ab7e0949a4d16c60da293e59e8e`

Usando el comando para ver las redes:
`docker network ls`
Obtenemos las redes creadas:

    NETWORK ID     NAME                     DRIVER    SCOPE
    0a1474640ac2   bridge                   bridge    local
    af3e196f401e   catalogo-net             bridge    local
    2a8213f22b5a   docker-test_default      bridge    local
    fc24239fb846   host                     host      local
    3cf30525c60e   ldb_default              bridge    local
    ace77780b341   next-intro_default       bridge    local
    2f4c41e89a25   next-tuto-dock_default   bridge    local
    e571423820b9   none                     null      local
Se observan las 3 redes por defecto (`bridge, host y none`) y otras que se crearon anteriormente al usar docker compose. También está la recién creada los primeros caracteres del id

Ejecutando el comando:
`docker volume create catalogo-db-data`
Obtenemos el nombre del volumen:
`catalogo-db-data`

Usando el comando para ver los volumenes:
`docker volume ls`
Obtenemos los volumenes creados:

    DRIVER    VOLUME NAME
    local     2b6d0aaab610f8249d1ef81ead6357f4c6a0642e9b2ab803501d56c34fa3537d
    local     3dc268af2a112eee425629243be9074c6bafa08d620b9d37ee717b62831eff17
    local     31e3ce428affe67dbe59409c9a3327e42f3df8d67cf35b5f9d2776d4306618c7
    local     7515ca17d3f416eb6b76393913d5faffbe223af5625c3b2b12ca413c6449c956
    local     862980dac0855ef229483cd3f880d2730fcfd4608197db01a9baeb0f31b0ab4e
    local     61158858ff5e4d27a1736b9a3c7e6f9bd800f20d5f5ad0ce2c3bdfbf8f72a013
    local     65619892f3fb3c73e9d8f24d3ae89f261903677283393ec211d3b59c17dc1ed9
    local     afbea9a37c27dc5e9bb69437543adbec5ac4023a30f427f4aa2653881d0f5f77
    local     bded7166540fdf2e42ab73fbf16ff8919eab334028efc0b3911c7793cafb3509
    local     catalogo-db-data
    local     demo-db-data
    local     e097646f51ae1af1d1e8ec8ef10544ff991293172a5359ccc091fc6b7b34b3c9
    local     node_mongo-data

Se ven varios volumenes creados automaticamente al crear contenedores y se ve el volumen `catalogo-db-data` recién creado.

## 2
Comando con todo lo necesario:

    docker run --network catalogo-net --name catalogo-db -e MONGO_INITDB_ROOT_USERNAME=catalogo_user -e MONGO_INITDB_ROOT_PASSWORD=catalogo_pass -v catalogo-db-data:/data/db mongo:7

Destacar que no se expone ningún puerto de el contenedor debido a que el único que debe ser capaz de conectarse a la base de datos es catalogo-api.
Se obtienen varias salidas en formato JSON, la indicativa del funcionamiento de la base de datos es:

    {"t":{"$date":"2026-10-04T20:25:59.968+00:00"},"s":"I",  "c":"NETWORK",  "id":23016,   "ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}

## 3
Creacion del contenedor catalogo-api:

    docker run --rm --name catalogo-api --network catalogo-net -e  DB_HOST=catalogo-db -e DB_PORT=27017 -e DB_NAME=catalogodb -eDB_USER=catalogo_user -e DB_PASSWORD=catalogo_pass -e APP_VERSION=v1 -e ROOT_PATH=/api -e SEED_DEMO=true -p 8000:8000    catalogo-api:v1

### a
    PS C:\Users\Cami\Desktop\Facultad\docker-curso\catalogo-app> docker exec catalogo-api getent hosts catalogo-db
    172.22.0.2      catalogo-db

### b
Como estoy ejecutando docker en mi maquina sin linux ni curl, accedo a la dirección web y se ve el sgte. JSON:

    {
    "status":"ok",
    "db":"ok",
    "version":"v1"
    }

### c
accediendo a `localhost:8000/productos` se observan los JSON que devuelva la api, se ven 12 productos en formato JSON.

## 4
### a
usando:

    DB_HOST=catalogo-inexiste

Se ven errores en los logs de la api

    atalogo-inexistente:27017: [Errno -2] Name or service not known (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms), Timeout: 3.0s, Topology Description: <TopologyDescription id: 6ac2bd9667e3097ce7a49ce2, topology_type: Unknown, servers: [<ServerDescription ('catalogo-inexistente', 27017) server_type: Unknown, rtt: None, error=AutoReconnect('catalogo-inexistente:27017: [Errno -2] Name or service not known (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>]>

### b
Sin ?authSource=admin mongo no sabe a que base de datos te intentas conectar y salta error

### c 
En el primero docker no conoce un contenedor con nombre "catalogo-inexiste" por lo que no puede darle la ip a catalogo-api cuando este intenta comunicarse.

### 5
Creamos el contenedor con el comando:

    docker run --name catalogo-frontend -e API_HOST=catalogo-api -e API_PORT=8000 -p 3000:8080 catalogo-frontend:v1  

Al ejecutar `docker ps -a` se observa:

    CONTAINER ID   IMAGE                          COMMAND                  CREATED          STATUS                      PORTS     NAMES
    2f5c57f0b0df   catalogo-frontend:v1           "/docker-entrypoint.…"   2 minutes ago   Exited (1) 2 minutes ago              catalogo-frontend

La salida de `` es:
    /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
    /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    20-envsubst-on-templates.sh: Running envsubst on /etc/nginx/templates/default.conf.template to /etc/nginx/conf.d/default.conf
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    /docker-entrypoint.sh: Configuration complete; ready for start up
    2026/10/07 11:09:29 [emerg] 1#1: host not found in upstream "catalogo-api" in /etc/nginx/conf.d/default.conf:24
    nginx: [emerg] host not found in upstream "catalogo-api" in /etc/nginx/conf.d/default.conf:24

El proceso termina al no encontrar al host "catalogo-api" según está indicado en /etc/nginx/conf.d/default

## 6
Comando correcto

    docker run --name catalogo-frontend -e API_HOST=catalogo-api -e API_PORT=8000 -p 3000:8080 --network catalogo-net catalogo-frontend:v1 

Al intentar acceder surgieron problemas, tuve que modificar el dockerfile:
`COPY --from=builder /frontend/dist/. .`
fue modificado a:
`COPY --from=builder /frontend/dist/. ./app-catalogo/`
Y en la config de nginx, el root es: `/app-catalogo`

Al crear un producto se observan las sgtes entradas en el apartado de red de las herramientas de desarrollador:

    POST http://localhost:3000/api/productos
    GET http://localhost:3000/api/productos
    GET http://localhost:3000/api/categorias

Como se observa, realiza los pedidos a `localhost:3000`, el cual corresponde al contenedor de nginx

## 7
### a
Ambos JSON recibidos son iguales

### b
con `ROOT_PATH=/api` al hacer run me devuelve:
    
    servers	
        0	
            url	"/api
### c
Con `ROOT_PATH=""` no se ve ninguna salida
### d
Sin la variable dirije a la pagina de error:

    Unable to render this definition

Con la variable nos lleva al docs del backend.
## 8
Tras eliminar el contenedor de la base de datos y volverlo a iniciar sigue el producto creado