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
### Docker compose
#### heathchecks
1. catalogo-db: Este healthchek usa la operación `ping` de mongo, este comando puede ejecutarse sin autenticación. Tiene la ventaja de que no expone credenciales inecesariamente pero no verifica la creación del usuario que utiliza la api.
La dependencia de `catalogo-api` con `catalogo-db` y existencia de este healthcheck se debe a que si no usaramos la condición de `service-healthy` la api podría iniciarse e intentar conectarse a mongo antes de que este acepte conexiones.

1. catalogo-api: utiliza la biblioteca urllib la cual es incluida en python por defecto. No se utiliza otras herramientas como curl debido a que estas no están incluidas en la versión reducida de linux usada e instalarlas significaría aumentar el tamaño de la imagen de forma inecesaria. En este caso, nginx necesita que exista el contenedor para poder resolver la dirección del contenedor usando su nombre.


### Orden de arranque - sin compose


1. Crear la red `catalogo-net`

1. Crear el volumen para la base de datos `catalogo-db-data`

1. Usar Docker build para las imagenes.

1. Iniciar el contenedor de la base de datos mongo con el comando:        

        docker run --network catalogo-net --name catalogo-db -e MONGO_INITDB_ROOT_USERNAME=catalogo_user -e MONGO_INITDB_ROOT_PASSWORD=catalogo_pass -e MONGO_INITDB_DATABASE=catalogodb -v catalogo-db-data:/data/db mongo:7
2. Crear la imagen e Iniciar el contenedor de la api. La api requiere que la base de datos esté encendida, realiza 10 intentos de conxion y si no lo logra se termina la ejecución del contenedor. Comando de ejecución:
        docker build --no-cache -t catalogo-api:v1 .\backend\  
    Luego:
        

        docker run --name catalogo-api --network catalogo-net -e  DB_HOST=catalogo-db -e DB_PORT=27017 -e DB_NAME=catalogodb -e DB_USER=catalogo_user -e DB_PASSWORD=catalogo_pass -e APP_VERSION=v1 -e ROOT_PATH=/api -e SEED_DEMO=true -p 8000:8000    catalogo-api:v1

3. Crear la imagen a partir del Dockerfile y lugo inicar el contenedor de nginx, el frontend. Si no se inició antes el contenedor de la api, cuando nginx  intente resolver la dirección de la api, docker no podrá darle una respuesta, causando que finalice la ejecición del contenedor. Comando:

        docker build -t catalogo-frontend:v1 .\frontend
    Luego:

    docker run --name catalogo-frontend -e API_HOST=catalogo-api -e API_PORT=8000 -p 3000:8080 --network catalogo-net catalogo-frontend:v1

## Comparación de metodotodos de creación de imagenes

| Imagen | Version | Tamaño | Reduccion | Razonamiento |
| :--- | :---: | ---: | ---: | ---: |
| backend | ingenua |  1.71GB | - | simple de implementar y directo, no necesita conocimientos extensos de docker ni node |
| backend | definitiva |  258MB | 85.3 % | Mucho mas pequeña, separar en una etapa de compilación y otra de servidor reduce overhead inecesario para un entorno de porducción | 
| frontend | definitiva | Texto C | 73.9MB | basandonos en lo anterior es conveniente usar un build de dos etapas para evitar tener cosas inecesarias (codigo, compiladores y node) |

### TP4 - Ej10 - Revisión de seguridad
#### a - Usuarios
- catalogo-db: root
- catalogo-api: appuser
- catalogo-frontend: nginx
#### b - Puertos
- catalogo-db:
    2701:null (Usado internamente, no publicado)
- catalogo-api: 
    8000:8000 (para depuración)
- catalogo-frontend: 
    3000:8080 (Para acceso público)
#### c -Vulnerabilidades

    docker scout cves
En nginx se encontraron las siguientes vulnerabilidades:
- libavif 1.0.4-r0: 13 vulnerabilidades en 3 paquetes de alpine.
2 no tienen una versión solucionada 
- openssl 3.3.3-r0: 10 se solucion en la versión 3.3.7-r2
- expat 2.7.0-r0: 1 crítica es solucionada en la versión 2.7.5-r0
### TP4 - Ej 11 - Capabilities mínimas
catalogo-db es el único contenedor que requiere capabilities especificas, se requieren las siguientes: [CHOWN, SETUID, SETGID, DAC_OVERRIDE, FOWNER]
### TP4 - Ej 12 - Comparación entre docker compose y comandos previos
En lugar de seguir los pasos en la sección de [Orden de arranque](#orden-de-arranque---sin-compose), uno puede simplemente usar el comando `docker compose up -d` el cual crea todos los componentes necesarios para el funcionamiento del stack de la aplicación, como si se siguiera la guía indicada. Además de la maypr simplicidad y legibilidad, tiene elementos extras de seguridad.

### TP5 - Ej 10

| base | variaciones | descripcion |
|-----------|-----------|-----------|
| kubectl get pods   |   -o wide, -w        | Muestra los pods y sus estados          |
| kubectl run    | catalogo-frontend --image=camidock1/catalogo-frontend:v2 --dry-run=client -o yaml > k8s/pod-catalogo-frontend.yaml          |   Genera un yml base para crear un pod        |
| kubectl apply    |   -f k8s/pod-catalogo-api.yaml        |  Aplica los cambios realizados al yml indiado y crea el pod         |
| kubectl delete pod   | catalogo-api, catalogo-db           | elimina un pod deteniendo su ejecucion          |
| kubectl dete pods    |  -n catalogo-app --all         |         elimina todos los pods de un namespace  |
|  kubectl port-forward    |  pod/catalogo-api 8000:8000         | Expone los puertos indicados del pod indicado en el host que se ejecuta el comando, usado para depuración          |

### TP5 - Ej 11
Se tiene: 
- Nombres básicos
- Los contenedores corriendo

Falta:
- resolución de nombres de host en funcion de los nombre de pods
- healthchecks y orden de arranque automático
- cap_drop