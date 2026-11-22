### 7
El comando:
`docker compose down -v`
Además de eliminar los contenedores elimina los volumenes usadas por el  compose
### 10 - Revisión de seguridad
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
### 11
Al quitar permisos a la base de datos se obtiene el error:

    find: '/data/db/journal': Permission denied
    find: '/data/db/diagnostic.data': Permission denied
    find: '/data/db/.mongodb': Permission denied

Por lo que se agregan los siguientes permisos:

    [CHOWN, SETUID, SETGID, DAC_OVERRIDE, FOWNER]