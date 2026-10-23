
## 2
Contexto:

        estudiante@curso-docker:~$ kubectl config current-context
        cvillanueva


Namespace:
        estudiante@curso-docker:~$ kubectl get ns
        NAME              STATUS   AGE
        catalogo-app      Active   127m
        default           Active   9d
        kube-flannel      Active   9d
        kube-node-lease   Active   9d
        kube-public       Active   9d
        kube-system       Active   9d
        metallb-system    Active   9d
        nginx-ingress     Active   9d

pods:

        estudiante@curso-docker:~$ kubectl get pods -A
        NAMESPACE        NAME                                        READY   STATUS    RESTARTS        AGE
        kube-flannel     kube-flannel-ds-l75xk                       1/1     Running   0               9d
        kube-system      coredns-559f6c778d-8bzcd                    1/1     Running   0               9d
        kube-system      coredns-559f6c778d-gjv4z                    1/1     Running   0               9d
        kube-system      etcd-cvillanueva                            1/1     Running   0               9d
        kube-system      kube-apiserver-cvillanueva                  1/1     Running   1 (3d22h ago)   9d
        kube-system      kube-controller-manager-cvillanueva         1/1     Running   2 (3d21h ago)   9d
        kube-system      kube-proxy-hr54r                            1/1     Running   0               9d
        kube-system      kube-scheduler-cvillanueva                  1/1     Running   1 (3d22h ago)   9d
        metallb-system   controller-76f4cf76b7-jc2m5                 1/1     Running   0               9d
        metallb-system   speaker-mcb6p                               1/1     Running   0               9d
        nginx-ingress    nginx-ingress-controller-78fc5b8845-hv84t   1/1     Running   0               9d

## 3
Creacion del pod `catalogo-db`

                kubectl run catalogo-db --image=mongo:7 --dry-run=client -o yaml > k8s/pod-catalogo-db.yaml

en el .yml:


## 4
### catalogo-db
Aplicando los cambios de `catalogo-db`:
       
        kubectl apply -f k8s/pod-catalogo-db.yaml
        pod/catalogo-db created

`kubectl get pods -o wide`

        NAME          READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
        catalogo-db   1/1     Running   0          14m   10.244.0.6   cvillanueva   <none>           <none>

En `kubectl get pods -A` aparece:

        NAMESPACE        NAME                                        READY   STATUS    RESTARTS        AGE
        catalogo-app     catalogo-db                                 1/1     Running   0               3m44s

### catalogo-api
Creación de `catalogo-api`

                 kubectl run catalogo-api --image=camidock1/catalogo-api:v1 --dry-run=client -o yaml > k8s/pod-catalogo-api.yaml

aplicar:

        kubectl apply -f k8s/pod-catalogo-api.yaml
En la lista:

    NAME                READY   STATUS              RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
    catalogo-api        1/1     Running             0          95s   10.244.0.7   cvillanueva   <none>           <none>
### catalogo-frontend
Creación de `catalogo-frontend`

                 kubectl run catalogo-frontend --image=camidock1/catalogo-frontend:v1 --dry-run=client -o yaml > k8s/pod-catalogo-frontend.yaml


aplicar:

        kubectl apply -f k8s/pod-catalogo-frontend.yaml
en la lista:


### estado de los pods:
el comando `kubectl get pods -o wide` nos mustra la tabla:

        NAME                READY   STATUS    RESTARTS      AGE     IP            NODE          NOMINATED NODE   READINESS GATES
        catalogo-api        1/1     Running   2 (23s ago)   6m4s    10.244.0.12   cvillanueva   <none>           <none>
        catalogo-db         1/1     Running   0             5m23s   10.244.0.13   cvillanueva   <none>           <none>
        catalogo-frontend   0/1     Error     3 (44s ago)   4m50s   10.244.0.14   cvillanueva   <none>           <none>

En donde se ven los 3 contenedores con sus ips. adenás, durante la creación se podía leer `ContainerCreating` mientras se descargaba la imagen 

## 5
Debido a error de mi parte tuve que usar la imagen con tag :v2, ya que publique la imagen equivocada en docker hub

En catalogo-api se veía el error:

        [2026-10-23T03:17:15+0000] WARNING intento 1/10 de conexión falló: catalogo-db:27017: [Errno -2] Name or service not known (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms), Timeout: 3.0s, Topology Description: <TopologyDescription id: 6adad1b743a632571c86b874, topology_type: Unknown, servers: [<ServerDescription ('catalogo-db', 27017) server_type: Unknown, rtt: None, error=AutoReconnect('catalogo-db:27017: [Errno -2] Name or service not known (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>]>

## 6
Remplazando el hostname de mongo por su ip la api notifica conexión con mongo.
Al recrear el pod de mongo se ve que la ip pasó de 10.244.0.13 a 10.244.0.16.
## 7

        /docker-entrypoint.sh: Configuration complete; ready for start up
        2026/10/23 03:41:27 [emerg] 1#1: host not found in upstream "catalogo-api" in /etc/nginx/conf.d/default.conf:24
        nginx: [emerg] host not found in upstream "catalogo-api" in /etc/nginx/conf.d/default.conf:24
## 8
ejecuto: 

        kubectl port-forward pod/catalogo-api 8000:8000
El cual ocupa la consola, desde otro terminal ejecuto `curl -s localhost:8000/health` curl y obtenemos

    {"status":"ok","db":"ok","version":"v1"}

con `curl -s localhost:8000/productos` se obtienen los productos

## 9

        estudiante@curso-docker:~/Descargas/catalogo-app$ $ kubectdelete pod catalogo-api
        pod "catalogo-api" deleted from catalogo-app namespace
        estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl get pods -o wide
        NAME                READY   STATUS             RESTARTS        AGE   IP            NODE          NOMINATED NODE   READINESS GATES
        catalogo-db         1/1     Running            0               18m   10.244.0.16   cvillanueva   <none>           <none>
        catalogo-frontend   0/1     CrashLoopBackOff   9 (2m14s ago)   27m   10.244.0.14   cvillanueva   <none>           <none>
        estudiante@curso-docker:~/Descargas/catalogo-app$

## 10

| base | variaciones | descripcion |
|-----------|-----------|-----------|
| kubectl get pods   |   -o wide, -w        | Muestra los pods y sus estados          |
| kubectl run    | catalogo-frontend --image=camidock1/catalogo-frontend:v2 --dry-run=client -o yaml > k8s/pod-catalogo-frontend.yaml          |   Genera un yml base para crear un pod        |
| kubectl apply    |   -f k8s/pod-catalogo-api.yaml        |  Aplica los cambios realizados al yml indiado y crea el pod         |
| kubectl delete pod   | catalogo-api, catalogo-db           | elimina un pod deteniendo su ejecucion          |
| kubectl dete pods    |  -n catalogo-app --all         |         elimina todos los pods de un namespace  |
|  kubectl port-forward    |  pod/catalogo-api 8000:8000         | Expone los puertos indicados del pod indicado en el host que se ejecuta el comando, usado para depuración          |

## 11
Se tiene: 
- Nombres básicos
- Los contenedores corriendo

Falta:
- resolución de nombres de host en funcion de los nombre de pods
- healthchecks y orden de arranque automático
- cap_drop