<h1 align="center">
Deploying Argo CD in Kubernetes
</h1>

![alt text](https://github.com/junoteam/demo-argocd/blob/main/pics/argocd-welcome.png?raw=true)

1. Prepare local/remote Kubernetes cluster with Minikube, k0s or k3s
   - enable/install ingress controller 

2. Install helm and add argocd repo
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```
3. Install ArgoCD from helm
```bash
kubectl create namespace argocd
helm pull argo/argo-cd --version 3.12.1
tar xvf argo-cd-3.12.1.tgz
helm install argo-cd -n argocd ./argo-cd
```

4. Check your deployment
```bash
kubectl get all -n argocd
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/argo-cd-argocd-application-controller-5fcd7df5dc-dgb2g   1/1     Running   0          10m
pod/argo-cd-argocd-dex-server-5b464cd55b-d4g4k               1/1     Running   0          10m
pod/argo-cd-argocd-redis-5bcb54dbf7-g6k94                    1/1     Running   0          10m
pod/argo-cd-argocd-repo-server-c479c56d6-fpmmf               1/1     Running   0          10m
pod/argo-cd-argocd-server-778cfb5c5c-46krp                   1/1     Running   0          10m

NAME                                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/argo-cd-argocd-application-controller   ClusterIP   10.107.235.156   <none>        8082/TCP            10m
service/argo-cd-argocd-dex-server               ClusterIP   10.109.217.120   <none>        5556/TCP,5557/TCP   10m
service/argo-cd-argocd-redis                    ClusterIP   10.99.38.19      <none>        6379/TCP            10m
service/argo-cd-argocd-repo-server              ClusterIP   10.102.3.79      <none>        8081/TCP            10m
service/argo-cd-argocd-server                   ClusterIP   10.97.217.181    <none>        80/TCP,443/TCP      10m

NAME                                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argo-cd-argocd-application-controller   1/1     1            1           10m
deployment.apps/argo-cd-argocd-dex-server               1/1     1            1           10m
deployment.apps/argo-cd-argocd-redis                    1/1     1            1           10m
deployment.apps/argo-cd-argocd-repo-server              1/1     1            1           10m
deployment.apps/argo-cd-argocd-server                   1/1     1            1           10m

NAME                                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/argo-cd-argocd-application-controller-5fcd7df5dc   1         1         1       10m
replicaset.apps/argo-cd-argocd-dex-server-5b464cd55b               1         1         1       10m
replicaset.apps/argo-cd-argocd-redis-5bcb54dbf7                    1         1         1       10m
replicaset.apps/argo-cd-argocd-repo-server-c479c56d6               1         1         1       10m
replicaset.apps/argo-cd-argocd-server-778cfb5c5c                   1         1         1       10m
```

5. Check your ingress resource
```bash
kubectl get ingress -A                                                                                                                                
NAMESPACE   NAME                    CLASS    HOSTS                ADDRESS          PORTS   AGE
argocd      argo-cd-argocd-server   <none>   argocd.berber        192.168.99.100   80      12m
default     hello-world-ingress     <none>   hello-world.berber   192.168.99.100   80      60m
```
5.1 In case you don't want to use ingress
```bash
kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443
```
5.2 To make ingress work locally please setup  
- Add the annotation for ssl passthrough
- Add the `--insecure` flag to `server.extraArgs`

In helm chart
```bash
ingress:
    enabled: true
    annotations: 
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/force-ssl-redirect: true
      nginx.ingress.kubernetes.io/ssl-passthrough: true
    labels: {}
    ingressClassName: ""

    ## Argo Ingress.
    ## Hostnames must be provided if Ingress is enabled.
    ## Secrets must be manually created in the namespace
    ##
    hosts:
      - argocd.berber
    paths:
      - /
    pathType: Prefix
...
...
## Additional command line arguments to pass to argocd-server
##
extraArgs:
 - --insecure
```

6. Get the password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
