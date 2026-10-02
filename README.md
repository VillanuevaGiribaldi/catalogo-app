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

| Imagen | Version | Tamaño | Reduccion | Razonamiento |
| :--- | :---: | ---: | ---: | ---: |
| backend | ingenua |  1.71GB | - | simple de implementar y directo, no necesita conocimientos extensos de docker ni node |
| backend | definitiva |  258MB | 85.3 % | Mucho mas pequeña, separar en una etapa de compilación y otra de servidor reduce overhead inecesario para un entorno de porducción | 
| frontend | definitiva | Texto C | 73.9MB | basandonos en lo anterior es conveniente usar un build de dos etapas para evitar tener cosas inecesarias (codigo, compiladores y node) |
