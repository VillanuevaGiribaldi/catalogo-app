# Trabajo Practico 1
## 3
### -a
    docker run -d --name catalogo-db mongo:7
Salida de `docker ps`:
    PS C:\Users\Cami\Desktop\Facultad\docker-curso\catalogo-app> docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS       NAMES
    aba3a4138625   mongo:7   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   27017/tcp   catalogo-db

### -b
ingreso a la consola del contenedor:
    docker exec -it catalogo-db bash
ingreso a la consola de mongo:
    mongosh
ejecuto el comando para ver las bases de datos de mongo:
    test> show databases
    admin   40.00 KiB
    config  12.00 KiB
    local   40.00 KiB
### -c
cualquier persona en la misma red del contenedor puede acceder. Si el contenedor tuviera acceso a internet se podría acceder a las bases de datos con permisos de admin

## 4
### a
uso el comando:
    docker run -d --name catalogo-db -e MONGO_INITDB_ROOT_USERNAME=catalogo_user -e MONGO_INITDB_ROOT_PASSWORD=catalogo_pass mongo:7
Y tras ejecutar
    docker logs -f catalogo-db
se puede ver:
{"t":{"$date":"2026-09-30T21:57:25.570+00:00"},"s":"I",  "c":"NETWORK",  "id":23016,   "ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}

## 5
`ping` no existe en el contenedor.En mongosh, ejecuto
    ping: "test"
Y la salida es
    "test"
Al usar `show databases` la salida es:
    MongoServerError[Unauthorized]: Command listDatabases requires authentication

## 6
### a
Ejecuto en el contenedor:
    mongosh -u catalogo_user -p catalogo_pass
Y al usar `show databases` da resultado
### b
Ejecuto:
    mongosh -u catalogo_user -p catalogo_pass ?authSource=admin
Y la salida es:
    Current Mongosh Log ID: 6abd8a59464a7461302d90b2
    Connecting to:          mongodb://<credentials>@127.0.0.1:27017/%3FauthSource%3Dadmin?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.10.0
    MongoServerError: Authentication failed.

Esto se debe a que el usuario no está asociado a ninguna base de datos específica, y al especificar authSource=admin intenta buscar un usuario de esa base de datos específica.

## 7
    docker inspect --format '{{json .Config.Env}}'
la salida es:
    ["MONGO_INITDB_ROOT_USERNAME=catalogo_user","MONGO_INITDB_ROOT_PASSWORD=catalogo_pass","PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","GOSU_VERSION=1.19","JSYAML_VERSION=3.13.1","JSYAML_CHECKSUM=662e32319bdd378e91f67578e56a34954b0a2e33aca11d70ab9f4826af24b941","MONGO_PACKAGE=mongodb-org","MONGO_REPO=repo.mongodb.org","MONGO_MAJOR=7.0","MONGO_VERSION=7.0.43","HOME=/data/db"]
Como se observa, puede leerse la contraseña como texto, esto es una gran falta de seguridad.

## 8
Crear el objeto:
    test> db.collection.insertOne({id:1,test: "hola mongo"})
    {
    acknowledged: true,
    insertedId: ObjectId('6abd8df6241e961fd43ef313')
    }
Recuperar los objetos:
    test> db.collection.find()
    [
    {
        _id: ObjectId('6abd8df6241e961fd43ef313'),
        id: 1,
        test: 'hola mongo'
    }
    ]
tras eliminar el contenedor y volverlo a crear, la salida es:
    test>  db.collection.find()

    test>