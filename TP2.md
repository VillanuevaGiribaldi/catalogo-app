# Trabajo Practico 2
## 1
Dockerfile
    FROM python:3.12
    WORKDIR /app
    COPY . .
    EXPOSE 8000
    RUN ["pip","install","-r","requirements.txt"]
El comando de docker es:
    docker build -t catalogo-backend .
Salida:
Se ven todos las capas, tarda bastante en las etapas de cargar la imagen de python y hacer el pip install 
Resultado:
    IMAGE                       ID             DISK USAGE   CONTENT SIZE   EXTRA
    catalogo-backend:latest     d8ff9f659881       1.71GB          445MB
## 2
El nuevo dockerfile tiene dos etapas.
Se ve como al ejecutar el build algunos comandos se ejecutan mientras no finalizó la etapa del builder.
    => [builder 2/3] COPY ./requirements.txt .                            0.4s
    => [stage-1 2/4] RUN useradd --create-home appuser                    0.5s
    => [builder 3/3] RUN ["pip","install","--user" ,"--no-cache-dir","-r  9.0s
    => [stage-1 3/4] COPY --from=builder /root/.local /home/appuser/.loc  4.1s
El resultado es:
    IMAGE                       ID             DISK USAGE   CONTENT SIZE   EXTRA
    catalogo-api:latest         1bdd8be13318        258MB         61.7MB
Se observa que es mucho mas pequeña. Al iniciar el contenedor se ve que está buscando conección con la base de datos.
## 4
v1:
    docker run --rm --entrypoint sh catalogo-api:latest -c 'echo $APP_VERSION'
devuelve:
    v1
v2:
    docker run --rm --entrypoint sh catalogo-api:v2 -c 'echo $APP_VERSION'
devuelve:
    v2
## 5
Dockerfile creado:
    FROM node:20-alpine AS builder
    WORKDIR /frontend
    COPY . .
    RUN npm ci
    RUN npm run build

    FROM nginxinc/nginx-unprivileged:1.27-alpine
    COPY --from=builder /frontend/dist/. .
    ENV API_HOST=catalogo-api \
        API_PORT=8000 \
        NGINX_ENVSUBST_FILTER=^API_
    EXPOSE 8000
Al iniciar el servidor salta advertencia de seguridad. Problema del servidor
## 6
### a
    nginx
### b
    /docker-entrypoint.sh: exec: line 47: node: not found
### c
    IMAGE                      ID             DISK USAGE   CONTENT SIZE   EXTRA
    catalogo-frontend:v1       ac5d38cd2af8       73.9MB           21MB