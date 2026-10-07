# Trabajo Practico 2
## 1
Dockerfile

    FROM python:3.12
    WORKDIR /app
    COPY . .
    EXPOSE 8000
    RUN ["pip","install","-r","requirements.txt"]

El comando de docker es:

    docker build -t catalogo-backend ./backend

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
## 9
La salida es:

    IMAGE          CREATED             CREATED BY                                      SIZE      COMMENT
    a73fa9047629   About an hour ago   CMD ["uvicorn" "main:app" "--host" "0.0.0.0"…   0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   EXPOSE [8000/tcp]                               0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   USER appuser                                    0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   ENV APP_VERSION=v1                              0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   ARG APP_VERSION=v1                              0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   ENV DB_HOST=localhost DB_PORT=27017 DB_NAME=…   0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   ENV PYTHONUNBUFFERED=1                          0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   ENV PATH=/home/appuser/.local/bin:/usr/local…   0B        buildkit.dockerfile.v0
    <missing>      About an hour ago   COPY . . # buildkit                             553kB     buildkit.dockerfile.v0
    <missing>      About an hour ago   WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
    <missing>      About an hour ago   COPY /root/.local /home/appuser/.local # bui…   53MB      buildkit.dockerfile.v0
    <missing>      2 hours ago         RUN /bin/sh -c useradd --create-home appuser…   69.6kB    buildkit.dockerfile.v0
    <missing>      17 hours ago        CMD ["python3"]                                 0B        buildkit.dockerfile.v0
    <missing>      17 hours ago        RUN /bin/sh -c set -eux;  for src in idle3 p…   16.4kB    buildkit.dockerfile.v0
    <missing>      17 hours ago        RUN /bin/sh -c set -eux;   savedAptMark="$(a…   41.4MB    buildkit.dockerfile.v0
    <missing>      17 hours ago        ENV PYTHON_SHA256=c2c4321961fab0fb999d66e0ce…   0B        buildkit.dockerfile.v0
    <missing>      17 hours ago        ENV PYTHON_VERSION=3.12.15                      0B        buildkit.dockerfile.v0
    <missing>      17 hours ago        ENV GPG_KEY=7169605F62C751356D054A26A821E680…   0B        buildkit.dockerfile.v0
    <missing>      17 hours ago        RUN /bin/sh -c set -eux;  apt-get update;  a…   13.2MB    buildkit.dockerfile.v0
    <missing>      17 hours ago        ENV LANG=C.UTF-8                                0B        buildkit.dockerfile.v0
    <missing>      17 hours ago        ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
    <missing>      2 weeks ago         # debian.sh --arch 'amd64' out/ 'trixie' '@1…   87.6MB    debuerreotype 0.17

Se observa que los dos elementos más pesados son el SO y el código compilado y como todas las llamadas de ENV no aumentan el tamaño.
## 10
Realizo la sgte modificación en el archivo main.py:

    class ProductoIn(BaseModel):
    sku: int = Field(ge=1, description="Código único del producto, identificador del sistema")

Ejecuto build de vuelta. Se observa:

    => CACHED [stage-1 2/5] RUN useradd --create-home appuser                                                                                             0.0s
    => CACHED [builder 2/4] COPY ./requirements.txt .                                                                                                     0.0s
    => CACHED [builder 3/4] RUN python -m venv .venv                                                                                                      0.0s
    => CACHED [builder 4/4] RUN pip install --user --no-cache-dir -r requirements.txt                                                                     0.0s
    => CACHED [stage-1 3/5] COPY --from=builder /root/.local /home/appuser/.local                                                                         0.0s
    => CACHED [stage-1 4/5] WORKDIR /app                                                                                                                  0.0s
    => [stage-1 5/5] COPY . .                                                                                                                             0.0s                                                                                                                              0.1s

Todos los pasos estaban cacheados a excepcion del último que es la copia de los archivos modificados.
Al agregar la librería "panda" en requirements.txt, se observa que el unico paso cacheado es la primera copia de la imagen de python y creación del usuario. Esto nos muestra que el cache es consciente de dependencias y sabe que crear el usuario no depende de lo que hizo el builder.

    => CACHED [builder 1/4] FROM docker.io/library/python:3.12-slim@sha256:dddfd7e07f9d15aeeca61529320492139d21cac7f0070c00609243e51e4e0016               0.0s
    => => resolve docker.io/library/python:3.12-slim@sha256:dddfd7e07f9d15aeeca61529320492139d21cac7f0070c00609243e51e4e0016                              0.0s
    => [builder 2/4] COPY ./requirements.txt .                                                                                                            0.0s
    => [builder 3/4] RUN python -m venv .venv                                                                                                             3.2s
    => [builder 4/4] RUN pip install --user --no-cache-dir -r requirements.txt                                                                           13.1s
    => CACHED [stage-1 2/5] RUN useradd --create-home appuser                                                                                             0.0s
    => [stage-1 3/5] COPY --from=builder /root/.local /home/appuser/.local                                                                                0.5s

## 11
Imagen del frontend:

    https://hub.docker.com/repository/docker/camidock1/catalogo-frontend

Imagen del backend:

    https://hub.docker.com/repository/docker/camidock1/catalogo-backend