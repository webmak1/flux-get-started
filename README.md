# flux-get-started

<br/>

Step-by-step run-through on how to use Flux and Helm Operator [over
here](https://github.com/fluxcd/flux/blob/master/docs/tutorials/get-started-helm.md).

More advanced Helm setup, take a look at the
[`fluxcd/helm-operator-get-started` repository](https://github.com/fluxcd/helm-operator-get-started).


<br/>

    $ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -q -N ""

<br/>

    $ cat ~/.ssh/id_rsa.pub

<br/>

    // FluxCtl Installation
    $ curl -sL https://fluxcd.io/install | sh && chmod +x $HOME/.fluxcd/bin/fluxctl && sudo mv $HOME/.fluxcd/bin/fluxctl /usr/local/bin/

<br/>

    $ export PATH=$PATH:$HOME/.fluxcd/bin


<!--

<br/>

### WITH COMMAND LINE

    $ kubectl create ns flux
    
    $ export GHUSER=webmakaka
    $ export GHREPO=flux-get-started
    $ echo ${GHUSER}/${GHREPO}
    
    $ fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.gihub.com \
    --git-url=git@github.com:${GHUSER}/${GHREPO} \
    --namespace=flux | kubectl apply -f -
        
    $ fluxctl --k8s-fwd-ns=flux identity

GITHUB -> SETTINGS -> DELPLOY KEYS

+ allow write access

        $ kubectl -n flux logs deploy/flux

        $ kubectl -n flux get pods
        NAME                         READY   STATUS    RESTARTS   AGE
        flux-7d6c44f798-b8zbc        1/1     Running   0          8m38s
        memcached-5bd7849b84-4xzh8   1/1     Running   0          8m38s


        $ fluxctl --k8s-fwd-ns=flux sync
-->

<br/>

### With HELM3

    $ helm repo add fluxcd https://charts.fluxcd.io
    
    $ kubectl create namespace fluxcd

    $ kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml

<br/>
    
    $ helm upgrade -i flux fluxcd/flux --wait \
    --namespace fluxcd \
    --set git.url=git@github.com:webmak1/flux-get-started \
    --set git.timeout=3m \
    --set git.pollInterval=1m \
    --set resources.requests.cpu=500m \
    --set resources.requests.memory=500Mi \
    --set sync.timeout=3m \
    --set helm.versions=v3


<br/>

    $ helm upgrade -i helm-operator fluxcd/helm-operator --wait \
    --namespace fluxcd \
    --set git.ssh.secretName=flux-git-deploy \
    --set helm.versions=v3

<br/>

    $ fluxctl --k8s-fwd-ns fluxcd identity 
     
<br/>

GITHUB --> Project (flux-get-started) --> SETTINGS --> DELPLOY KEYS

    + allow write access
    
<br/>

    $ fluxctl --k8s-fwd-ns=fluxcd sync

<br/>

    $ kubectl logs -n fluxcd  deployment/flux -f

    $ kubectl logs -n fluxcd deploy/helm-operator -f

<br/>

### Cheks

    $ watch kubectl -n fluxcd get pods

<br/>

```
$ helm list --namespace demo
NAME 	NAMESPACE	REVISION	UPDATED                                STATUS  	CHART      	APP VERSION
ghost	demo     	1       	2021-01-16 17:27:59.501952901 +0000 UTCdeployed	ghost-9.0.4	3.1.1      

```

<br/>

```
$ kubectl get pods -n demo
NAME                       READY   STATUS              RESTARTS   AGE
ghost-55c87c6bd5-g9vs5     0/1     ContainerCreating   0          62s
ghost-mariadb-0            0/1     ContainerCreating   0          61s
podinfo-6fb4c8894d-9gp98   1/1     Running             0          78s
podinfo-6fb4c8894d-j9b8x   1/1     Running             0          62s

```

<br/>

```
$ kubectl get svc -n demo
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
ghost           ClusterIP   10.100.79.47     <none>        80/TCP     79s
ghost-mariadb   ClusterIP   10.103.249.225   <none>        3306/TCP   79s
podinfo         ClusterIP   10.105.27.5      <none>        9898/TCP   95s

```

<br/>

```
$ kubectl get deployments -n demo
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
ghost     0/1     1            0           2m5s
podinfo   2/2     2            2           2m21s

```



<br/>

```
$ kubectl describe -n demo deployment/podinfo | grep Image
    Image:      alpine:3.10
    Image:       stefanprodan/podinfo:3.1.5

```


<br/>

### Test case

<br/>

https://github.com/webmak1/flux-get-started/edit/master/releases/mongodb.yaml

<br/>

set

<br/>

tag: 4.0.14

<br/>

vait for 5 minutes.

<br/>

    $ kubectl -n fluxcd logs deployment/flux -f
    
<br/>

```    
$ kubectl describe -n demo deployment/mongodb | grep Image
    Image:      docker.io/bitnami/mongodb:4.0.14
```


<br/>

# Solution with error [git push delete context deadline exceeded]


```
$ fluxctl --k8s-fwd-ns=flux sync
Error: git repository ssh://git@github.com/webmakaka/flux-get-started is not ready to sync

Full error message: running git command: git [push --delete git@github.com:webmakaka/flux-get-started tag flux-write-check-1b50d03dc8]: context deadline exceeded
```

To solve this issue use

    --set git.timeout=3m
    

---

<strong>Marley</strong>

<a href="https://webmakaka.com"><strong>WebMakaka</strong></a>
