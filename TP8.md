Parte 1 y metricas:
estudiante@curso-docker:~/Descargas/catalogo-app$ helm version
version.BuildInfo{Version:"v4.3.0", GitCommit:"bec5b06ed841fe5269972d864d5177944fd5970f", GitTreeState:"clean", GoVersion:"go1.27.1", KubeClientVersion:"v1.37"}
estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl get deployment metrics-server -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           6d2h
estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl top nodes
NAME          CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
cvillanueva   67m          3%       1310Mi          69%
estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl top pods
NAME                                CPU(cores)   MEMORY(bytes)
catalogo-api-68dcc8f6-ls9qq         2m           41Mi
catalogo-api-68dcc8f6-sptv9         2m           41Mi
catalogo-db-df65b9459-d8jmz         4m           120Mi
catalogo-frontend-d79576569-qm52n   0m           2Mi
catalogo-frontend-d79576569-xxbrh   0m           2Mi
estudiante@curso-docker:~/Descargas/

## 1
        kubectl set resources deployment catalogo-api  \
        --requests=cpu=100m,memory=64Mi --limits=cpu=250m,memory=128Mi

## 2


        estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl set resources deployment catalogo-api  \
        --requests=cpu=100m,memory=64Mi --limits=cpu=250m,memory=128Mi
        deployment.apps/catalogo-api resource requirements updated
        estudiante@curso-docker:~/Descargas/catalogo-app$ kubectl autoscale deployment catalogo-api --cpu-percent=50 --min=2 --max=6
        Flag --cpu-percent has been deprecated, Use --cpu with percentage or resource quantity format (e.g., '70%' for utilization or '500m' for milliCPU).
        horizontalpodautoscaler.autoscaling/catalogo-api autoscaled



## 3
        kubectl run carga --rm -it --restart=Never --image=busybox -- \
        /bin/sh -c "while true; do wget -q -O- http://catalogo:8000/productos; done"


         kubectl get hpa -w

        NAME               REFERENCE                 TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 2%/50%   2         6         2          7m16s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 1%/50%   2         6         2          7m45s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 2%/50%   2         6         2          8m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 70%/50%   2         6         2          8m30s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 52%/50%   2         6         3          8m45s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 15%/50%   2         6         3          9m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 88%/50%   2         6         3          9m15s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 98%/50%   2         6         6          9m30s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 33%/50%   2         6         6          9m45s
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 10%/50%   2         6         6          10m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 60%/50%   2         6         6          10m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 66%/50%   2         6         6          10m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 45%/50%   2         6         6          10m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 1%/50%    2         6         6          11m
        ...
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 1%/50%    2         6         6          15m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 1%/50%    2         6         6          15m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 1%/50%    2         6         2          16m
        catalogo-api-hpa   Deployment/catalogo-api   cpu: 2%/50%    2         6         2          16m


## 6
se creael template y luego se verifica la existencia de los 12 objetos

## 7 

                estudiante@curso-docker:~/Descargas/catalogo-app$ helm history catalogo-app
                REVISION        UPDATED                         STATUS          CHART               APP VERSION     DESCRIPTION                                             
                1               Fri Nov 20 14:09:35 2026        superseded      catalogo-app-0.1.0  1.16.0          Install complete                                        
                2               Fri Nov 20 14:47:24 2026        failed          catalogo-app-0.1.0  1.16.0          Upgrade "catalogo-app" failed: server-side apply failed for object catalogo-app/catalogo-api-hpa autoscaling/v2, Kind=HorizontalPodAutoscaler: failed to create typed patch object (catalogo-app/catalogo-api-hpa; autoscaling/v2, Kind=HorizontalPodAutoscaler): erro...
                3               Fri Nov 20 14:48:16 2026        superseded      catalogo-app-0.1.0  1.16.0          Upgrade complete                                        
                4               Fri Nov 20 14:54:44 2026        failed          catalogo-app-0.1.0  1.16.0          Upgrade "catalogo-app" failed: conflict occurred while applying object catalogo-app/catalogo-api apps/v1, Kind=Deployment: Apply failed with 1 conflict: conflict with "kube-controller-manager" with subresource "scale" using apps/v1: .spec.replicas
                5               Fri Nov 20 14:55:12 2026        failed          catalogo-app-0.1.0  1.16.0          Upgrade "catalogo-app" failed: conflict occurred while applying object catalogo-app/catalogo-api apps/v1, Kind=Deployment: Apply failed with 1 conflict: conflict with "kube-controller-manager" with subresource "scale" using apps/v1: .spec.replicas
                6               Fri Nov 20 14:56:04 2026        failed          catalogo-app-0.1.0  1.16.0          Upgrade "catalogo-app" failed: conflict occurred while applying object catalogo-app/catalogo-api apps/v1, Kind=Deployment: Apply failed with 1 conflict: conflict with "kube-controller-manager" with subresource "scale" using apps/v1: .spec.replicas
                7               Fri Nov 20 14:57:45 2026        deployed        catalogo-app-0.1.0  1.16.0          Upgrade complete                                        