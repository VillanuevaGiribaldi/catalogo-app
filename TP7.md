# TP7        
## 1        

        kubectl create configmap catalogo-api-config --from-literal=DB_HOST=catalogo-db --from-literal=DB_PORT="27017" --from-literal=DB_NAME=catalogodb --from-literal=DB_USER=catalogo_user --from-literal=ROOT_PATH=/api --from-literal=SEED_DEMO=true --dry-run=client -o yaml > k8s/configmap-catalogo-api.yaml

## 2

        kubectl create configmap catalogo-frontend-config --from-literal=API_HOST=catalogo-api --from-literal=API_PORT="8000" --dry-run=client -o yaml > k8s/configmap-catalogo-frontend.yaml

## 3

        kubectl create secret generic catalogo-db-secret --from-literal=MONGO_INITDB_ROOT_USERNAME=catalogo_user --from-literal=MONGO_INITDB_ROOT_PASSWORD=catalogo_pass

Ejecutando el comando indicado y decodificando la contraseña se la puede obtener la contraseña

## 4
db (no tiene configmap, usa secret):

        kubectl apply -f k8s/deploy-catalogo-db.yaml

api:

        kubectl apply -f k8s/deploy-catalogo-api.yaml

        kubectl apply -f k8s/configmap-catalogo-api.yaml

frontend:

        kubectl apply -f k8s/configmap-catalogo-frontend.yaml

        kubectl apply -f k8s/deploy-catalogo-frontend.yaml





## 5
Se decidio tratar todos los elementos por separado y la contraseña como su única variable por una mejor legibilidad y simplicar el caso de que se desee una base de datos en la nube a la cual no se acceda mediante el servicio de resolución de k8s

## 6
Se crea el archivo del pvc y se aplican los cambios tanto de `catalogo-db-pvc.yaml` como de `deploy-catalogo-db.yaml`.
Al ejecutar `kubectl get pvc,pv -n catalogo-app` se obtiene:

                NAME                                    STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
                persistentvolumeclaim/catalogo-db-pvc   Pending                                      local-path     <unset>                 12h
Al ejecutar `kubectl exec deploy/catalogo-db -n catalogo-app -- ls -ld /data/db` se obtiene:
         
                drwxr-xr-x 5 mongodb mongodb 4096 Nov 13 12:51 /data/db

## 7

        Se observa que el producto creado persiste.

## 8

        echo "172.16.21.230 catalogo.local" | sudo tee -a /etc/hosts

de `kubectl get svc -n nginx-ingress nginx-ingress-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` obtenemos la ip de arriba.

Tras crear y aplicar el ingress podemos usar el comando `kubectl get ingress -n catalogo-app` para ver el estado del mismo:
                
                NAME                   CLASS   HOSTS            ADDRESS         PORTS   AGE
                catalogo-app-ingress   nginx   catalogo.local   172.16.21.230   80      8m22s

Usando curl, vemos que se recibe los productos de la api, además se puede acceder a la app mediante el dominio catalogo.local
## 9 
Se puede acceder
## 10 - los objetos
### pods:
- catalogo-db: Pod individual que utiliza la imagen de `mongo:7`. No tiene salida a internet y solo se comunica con la api
- catalogo-api: Pod que usa la imagen `camidock1/catalogo-api:v2`, recibe los pedidos y se comunica con catalogo-db
- catalogo-frontend: Pod que usa la imagen `camidock1/catalogo-frontend:v1`. Proxi de nginx que recibe todos los pedidos de los usuario y responde con html o redirecciona el request a la api
### services:
Definen servicios conectados o no a internet a los cuales se le pueden vincular pods mediante hostname
- svc-catalogo-db: define el servicio de la base de datos como un clusterIP, con un puerto al cual los elementos del cluster pueden llegar
- svc-catalogo-api: define el servicio de la api como un NodePort, con un puerto al cual se puede llegar desde internet
- svc-catalogo-frontend: define el servicio del frontend como un NodePort, con un puerto al cual se puede llegar desde internet
### deployments:
Los deploys se relacionan con un servicio y uno puede acceder a los mismos usando el nombre que definió  en el servicio.
- deploy-catalogo-db: deployment que crea un pod de mongo:7 siguiendo las configuraciones almacenadas en el secret definido
- deploy-catalogo-api: crea 2 pods de la imagen `camidock1/catalogo-api:v2` usando las configuraciones definidas en el configmap de la api y el secret de la bd
- deploy-catalogo-frontend: crea dos pods desde la imagen `camidock1/catalogo-frontend:v1` usando las configuraciones definidads en el configmap
### PVC
Es el volumen persistente al que van a acceder los pods de la base de datos, de esta forma todos los pods que se creen de la misma comparten el mismo almacenamiento y no se pierden los datos
### Ingress
Es un proxy por el que se recibe el tráfico de ingreso al cluster y redirecciona a los pods del frontend
