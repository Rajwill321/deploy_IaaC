## My LAB setup for this Django deployment or Any **App deployment**

Kubernetes cluster setup:

```
[root@k8s-master deploy_IaaC]# kubectl get nodes
NAME                   STATUS   ROLES    AGE    VERSION
k8s-master.lab.net     Ready    master   5d9h   v1.16.3
k8s-worker-1.lab.net   Ready    <none>   16h    v1.16.3
k8s-worker-2.lab.net   Ready    <none>   5d8h   v1.16.3
[root@k8s-master deploy_IaaC]#
```

### I have containerized the Django App to a image and pushed in my custom docker repo

```
[root@docker-repo ~]# docker images | grep django
docker-repo.lab.net:5000/django-app   v1                  054f3f98e225        38 hours ago        965 MB
docker.io/django                      onbuild             cfe12b39b7e5        2 years ago         737 MB
docker.io/django                      latest              eb40dcf64078        2 years ago         436 MB
[root@docker-repo ~]#
```

### Now I have tested by deploying pods and designed a deployemt type yaml for final deployment

```
[root@k8s-master deploy_IaaC]# cat myapp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gopi-deploy
  labels:
    app: polls-app

spec:
  selector:
    matchLabels:
      app: polls-app
  template:
    metadata:
      labels:
        app: polls-app
    spec:
      containers:
      - image: docker-repo.lab.net:5000/django-app:v1
        name: polls-app
        ports:
        - containerPort: 8000
          name: polls-app

```

### Installed Metal Load Balancer in Kubernetes to test external load balancer and worked just fine

```
Every 2.0s: kubectl get all -o wide                                                                                                                                  Wed Dec  4 07:24:07 2019

NAME                               READY   STATUS    RESTARTS   AGE   IP          NODE                   NOMINATED NODE   READINESS GATES
pod/gopi-deploy-567b9cdf99-q5r65   1/1     Running   0          31m   10.36.0.1   k8s-worker-1.lab.net   <none>           <none>

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE    SELECTOR
service/gopi-mlb     LoadBalancer   10.99.136.98     192.168.134.240     8000:31229/TCP   9s     app=polls-app,tier=frontend
service/kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          5d9h   <none>

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                   SELECTOR
deployment.apps/gopi-deploy   1/1     1            1           31m   polls-app    docker-repo.lab.net:5000/django-app:v1   app=polls-app

NAME                                     DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                   SELECTOR
replicaset.apps/gopi-deploy-567b9cdf99   1         1         1       31m   polls-app    docker-repo.lab.net:5000/django-app:v1   app=polls-app,pod-template-hash=567b9cdf99

```

### With Kubernetes container orchestration we can easily scale up and down the docker image as and when it is needed with below command
Replica is how many docker instance it needs to spin

```
kubectl scale --replicas=2 deploy gopi-deploy
```

And portal works on each of their ports.



