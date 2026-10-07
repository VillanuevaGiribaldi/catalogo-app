FROM node:20-alpine AS builder
WORKDIR /frontend
COPY . .
RUN npm ci
RUN npm run build

FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY --from=builder /frontend/dist/. ./app-catalogo/
COPY nginx/templates/default.conf.template /etc/nginx/templates/default.conf.template
ARG APP_VERSION=v1
ENV APP_VERSION=${APP_VERSION}
ENV API_HOST=catalogo-api \
    API_PORT=8000 \
    NGINX_ENVSUBST_FILTER=^API_
EXPOSE 8080 