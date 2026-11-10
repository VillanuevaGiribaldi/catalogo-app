## 1
### Deploys
`catalogo-db`

        kubectl create deployment catalogo-db --image=mongo:7 --dry-run=client -o yaml > k8s/deploy-catalogo-db.yaml

`catalogo-api`

        kubectl create deployment catalogo-api --image=camidock1/catalogo-api:v2 --replicas=2  --dry-run=client -o yaml > k8s/deploy-catalogo-api.yaml

`catalogo-frontend`

        kubectl create deployment catalogo-frontend --image=camidock1/catalogo-frontend:v2 --replicas=2 --dry-run=client -o yaml > k8s/deploy-catalogo-frontend.yaml

### Services
`catalogo-db`

        kubectl create service clusterip catalogo-db --tcp=27017:27017  --dry-run=client -o yaml > k8s/svc-catalogo-db.yaml

`catalogo-api`

        kubectl create service clusterip catalogo-api --tcp=8000:8000  --dry-run=client -o yaml > k8s/svc-catalogo-api.yaml

`catalogo-frontend`

        kubectl create service nodeport catalogo-frontend --tcp=30302:8080 --node-port=30302 --dry-run=client -o yaml > k8s/svc-catalogo-frontend.yaml

## 2

        kubectl apply -f k8s/svc-catalogo-db.yaml
        kubectl apply -f k8s/svc-catalogo-api.yaml
        kubectl apply -f k8s/svc-catalogo-frontend.yaml
        kubectl apply -f k8s/deploy-catalogo-db.yaml
        kubectl apply -f k8s/deploy-catalogo-api.yaml
        kubectl apply -f k8s/deploy-catalogo-frontend.yaml

## 3

        kubectl get endpoints -n catalogo-app

Obtuve:

        estudiante@curso-docker:~$ kubectl get endpoints -n catalogo-app
        Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
        NAME                ENDPOINTS           AGE
        catalogo-api        <none>              4m40s
        catalogo-db         10.244.0.20:27017   6m35s
        catalogo-frontend   10.244.0.21:8080    4m34s

No tenía el nombre de host de la base de datos por lo que la api inició con error, tras corregirlo obtuve:

                NAME                ENDPOINTS                           AGE
                catalogo-api        10.244.0.25:8000,10.244.0.26:8000   17m
                catalogo-db         10.244.0.20:27017                   19m
                catalogo-frontend   10.244.0.21:8080,10.244.0.23:8080   17m

## 4
Obtuve la ip del nodo usando:

                kubectl get nodes -o wide
Y luego accedo a `HTTP://172.16.21.23:30302` y veo la página web.

## 5

        kubectl create service nodeport catalogo-api-debug --tcp=30502:8000 --node-port=30502 --dry-run=client -o yaml > k8s/svc-catalogo-api-debug.yaml

Hay que modificar el selector para que se vincule con `catalogo-api`

                  selector:
                        app: catalogo-api

## 6
A la salida de `kubectl get pods -w` se ve:

                NAME                                 READY   STATUS    RESTARTS   AGE
                catalogo-api-6d7b5b6c9b-59sdm        1/1     Running   0          41m
                catalogo-api-6d7b5b6c9b-fw579        1/1     Running   0          41m
                catalogo-db-7f5bbfd8cb-smhf5         1/1     Running   0          55m
                catalogo-frontend-78dfb7d4b5-pfkpm   1/1     Running   0          30m
                catalogo-frontend-78dfb7d4b5-qbs9c   1/1     Running   0          30m
                catalogo-api-6d7b5b6c9b-59sdm        1/1     Terminating   0          41m
                catalogo-api-6d7b5b6c9b-59sdm        1/1     Terminating   0          41m
                catalogo-api-6d7b5b6c9b-5mjpz        0/1     Pending       0          1s
                catalogo-api-6d7b5b6c9b-5mjpz        0/1     Pending       0          3s
                catalogo-api-6d7b5b6c9b-5mjpz        0/1     ContainerCreating   0          7s
                catalogo-api-6d7b5b6c9b-59sdm        0/1     Completed           0          41m
                catalogo-api-6d7b5b6c9b-59sdm        0/1     Completed           0          41m
                catalogo-api-6d7b5b6c9b-59sdm        0/1     Completed           0          41m
                catalogo-api-6d7b5b6c9b-5mjpz        0/1     ContainerCreating   0          15s
                catalogo-api-6d7b5b6c9b-5mjpz        1/1     Running   

En donde se observa como se termina un contenedor y se crea otro automáticamente.

## 7

        kubectl set image deployment/catalogo-api        catalogo-api=camidock1/catalogo-api:v3 -n catalogo-app