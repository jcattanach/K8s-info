# Kubernetes Deployment #

### Build first docker image ###
docker build -t project .

### Push to docker hub ###
docker tag project username/online-repo

docker push username/online-repo

### Create Kubernetes deployment yaml ###

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: project-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: project
    spec:
      containers:
      - name: project
        image: username/online-repo
        ports:
        - name: nodejs-port
          containerPort: 8080
```

### Apply the yaml file to Kubernetes ###
kubectl create -f deployments/deployment.yaml

### Check that it worked correctly ###
kubectl get pods
```
NAME                                  READY   STATUS    RESTARTS   AGE
project-deployment-6b5998c565-54wtq   1/1     Running   0          6s
project-deployment-6b5998c565-qnkd6   1/1     Running   0          6s
project-deployment-6b5998c565-rvsdj   1/1     Running   0          6s
```
kubectl get deployments
```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
project-deployment   3/3     3            3           14s
```
kubectl get rs
```
NAME                            DESIRED   CURRENT   READY   AGE
project-deployment-6b5998c565   3         3         3       21s
```

### Expose the port, so you can access it in your browser ###
_Note this only works on Minikube_
_otherwise it is handled in a separate yaml file or on your AWS or Google dashboard_


kubectl expose deployment project-deployment --type=NodePort
```
service/project-deployment exposed
```
kubectl get service
```
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
project-deployment   NodePort    10.104.238.185   <none>        8080:32425/TCP   11s
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          26h
```
kubectl describe service project-deployment
```
Name:                     project-deployment
Namespace:                default
Labels:                   app=project
Annotations:              <none>
Selector:                 app=project
Type:                     NodePort
IP:                       10.104.238.185
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32425/TCP
Endpoints:                172.17.0.4:8080,172.17.0.5:8080,172.17.0.6:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Get the url, so you can check it is working with curl or in your browser ###
minikube service project-deployment --url
```
http://192.168.99.101:32425
```
curl http://192.168.99.101:32425
```
Hello Kubernetes **V1**
```

### Build and push a version 2 to docker hub ###
_I just changed V1 in my index.js file to v2_

docker build -t project .

docker tag project username/online-repo:2
docker push username/online-repo:2

### Update Kuberenetes deployment to version 2 ###
kubectl set image deployment/project-deployment project=username/online-repo:2
```
deployment.extensions/project-deployment image updated
```

### Check status of rollout ###
kubectl rollout status deployments/project-deployment
```
 deployment "project-deployment" successfully rolled out
 ```

 curl http://192.168.99.101:32425
 ```
 Hello Kubernetes **V2**
 ```

kubectl rollout history deployment/project-deployment

```
 deployment.extensions/project-deployment
 REVISION  CHANGE-CAUSE
 1         <none>
 2         <none>
 ```

### Go back to previous version ###

kubectl rollout undo deployment/project-deployment
```
deployment.extensions/project-deployment rolled back
```
curl http://192.168.99.101:32425
```
Hello Kubernetes **V1**
```

_To go back to a specific rollout you can do "kubectl rollout undo deployment/project-deployment --to-revision=1"_
