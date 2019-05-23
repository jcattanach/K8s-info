# Kubernetes Deployment #

#### Build first docker image ####
docker build -t project .

#### Push todocker hub ####
docker tag project username/online-repo
docker push username/online-repo

#### Create Kubernetes deployment yaml ####

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

#### Apply the yaml file to Kubernetes ####
kubectl create -f deployments/deployment.yaml

#### Check that it worked correctly ####
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
#### Build and push a version 2 to docker hub ####
docker build -t project .

docker tag project username/online-repo:2
docker push username/online-repo:2
