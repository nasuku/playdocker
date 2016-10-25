### Installation and Start VM
Install minikube on MAC with homebrew (along with virtualbox, vagrant and kubernetes-cli)

``` minikube start ``` starts the minikube VM inside virtualbox with all the kubernetes contents. It also sets ``` ~/.kube/config ``` so that we can use ```kubectl``` from macos terminal.

### To start and delete a simple pod from commandline
```
kubectl run web --image=nginx
```
This creats a deployment called 'web'.

```
$ kubectl get deploy
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
web       1         1         1            1           40s
$ kubectl get replicasets
NAME            DESIRED   CURRENT   READY     AGE
web-174036596   1         1         1         57s
$ kubectl get pods
NAME                  READY     STATUS    RESTARTS   AGE
web-174036596-5izyv   1/1       Running   0          30s
```


To delete these
```
$ kubectl delete deploy web
deployment "web" deleted
```

### To use a config yml file to create a service

Lets take the content from nginx-pod.yml
```
kubectl create -f nginx-pod.yml
```
This just created a pod and nothing else.
```
$ kubectl get pods -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP           NODE
nginx     1/1       Running   0          1m        172.17.0.3   minikube
```

Please note that the containers are actually running inside a VM.
`minikube ip` can give the IP address of the VM. But not
sure of the credentials to login. `minikube ssh` does ssh to the VM from where the IP address is accessible.

```
$ minikube ssh wget 172.17.0.3
Connecting to 172.17.0.3 (172.17.0.3:80)
index.html           100% |*******************************|   612   0:00:00 ETA

$ kubectl delete pod nginx
pod "nginx" deleted
```

### With replication controller
nginx-rc.yml has config for Replication controller. It makes
sure that atleast the specified number of PODs are running and
restarts if they go down.

```
$ kubectl create -f nginx-rc.yml
replicationcontroller "nginx-controller" created
$ kubectl get pods -a
NAME                     READY     STATUS    RESTARTS   AGE
nginx-controller-ghpff   1/1       Running   0          1m
nginx-controller-k1vog   1/1       Running   0          1m
$ kubectl get replicationcontrollers
NAME               DESIRED   CURRENT   READY     AGE
nginx-controller   2         2         2         1m
```

Lets delete a pod.
```
$ kubectl delete pod nginx-controller-ghpff
pod "nginx-controller-ghpff" deleted
```

We can see that a new POD is started.

```
$ kubectl get pods -a
NAME                     READY     STATUS    RESTARTS   AGE
nginx-controller-k1vog   1/1       Running   0          2m
nginx-controller-ugprk   1/1       Running   0          4s

$ kubectl delete -f nginx-rc.yml
replicationcontroller "nginx-controller" deleted
```


### Services
Service acts as a proxy and/or load balancer between the front
end and the backend. This enables us to remove/add PODs without
any distruption to the user services.

```
$ kubectl create -f nginx-service.yml
service "web-service" created
$ kubectl get services
NAME          CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes    10.0.0.1     <none>        443/TCP   7d
web-service   10.0.0.11    <none>        80/TCP    3s
$ minikube ssh 'curl -qa 10.0.0.11'
.... snip ... the payload..
```

The web-service is started first without any pods.
When replicationcontrollers are created by user, kubernetes services get stiched.

Notice that 10.0.0.11 is a virtual IP thats managed entirely by kubernetes and load balances to both the backend pods.
