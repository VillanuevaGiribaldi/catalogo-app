# catalogo-app
repositorio hecho para el cursado de el curso de docker y  kubernetes de la FACET UNT
## Arquitectura
navegador ──► catalogo-frontend ──────► catalogo-api ──────► catalogo-db
React + Vite Python + FastAPI MongoDB 7
nginx :8080 :8000 :27017
(host 3000) (host 8000, solo (sin puerto
depuración) publicado)

| Capa | Imagen | Puerto interno | Puerto publicado |
| --- | --- | --- | --- |
| `catalogo-frontend` | React + Vite servido por nginx | 8080 | 3000 |

IMPORTANTE: La variable de entorno

    NGINX_ENVSUBST_FILTER=^API_

debe tener el valor indicado, siin ella, las variables propias de nginx serán vaciadas y se genera una configuración invalida.
| `catalogo-api` | Python + FastAPI | 8000 | 8000 (solo depuración) |
| `catalogo-db` | MongoDB 7 | 27017 | — |
El frontend es el único punto de entrada: nadie le habla a la base
directamente, y a la API le habla el frontend.

### Orden de arranque

1. Iniciar el contenedor de la base de datos mongo con el comando:

    docker run --network catalogo-net --name catalogo-db -e MONGO_INITDB_ROOT_USERNAME=catalogo_user -e MONGO_INITDB_ROOT_PASSWORD=catalogo_pass -v catalogo-db-data:/data/db mongo:7
2. Iniciar el contenedor de la api. La api requiere que la base de datos esté encendida, realiza 10 intentos de conxion y si no lo logra se termina la ejecución del contenedor. Comando de ejecución:

    docker run --name catalogo-api --network catalogo-net -e  DB_HOST=catalogo-db -e DB_PORT=27017 -e DB_NAME=catalogodb -eDB_USER=catalogo_user -e DB_PASSWORD=catalogo_pass -e APP_VERSION=v1 -e ROOT_PATH=/api -e SEED_DEMO=true -p 8000:8000    catalogo-api:v1

3. Inicar el contenedor de nginx, el frontend. Si no se inició antes el contenedor de la api, cuando nginx  intente resolver la dirección de la api, docker no podrá darle una respuesta, causando que finalice la ejecición del contenedor. Comando:

    docker run --name catalogo-frontend -e API_HOST=catalogo-api -e API_PORT=8000 -p 3000:8080 --network catalogo-net catalogo-frontend:v1

## Comparación de metodotodos de creación de imagenes

| Imagen | Version | Tamaño | Reduccion | Razonamiento |
| :--- | :---: | ---: | ---: | ---: |
| backend | ingenua |  1.71GB | - | simple de implementar y directo, no necesita conocimientos extensos de docker ni node |
| backend | definitiva |  258MB | 85.3 % | Mucho mas pequeña, separar en una etapa de compilación y otra de servidor reduce overhead inecesario para un entorno de porducción | 
| frontend | definitiva | Texto C | 73.9MB | basandonos en lo anterior es conveniente usar un build de dos etapas para evitar tener cosas inecesarias (codigo, compiladores y node) |

