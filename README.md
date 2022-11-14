# How to create a reverse proxy to connect fpm container to an nginx container

## pre-requisites
Have minikube installed
modify the

PersistentVolume-fpm.yaml

so that 
path: 
includes your local working directory (absolute path!!)


## PRE Config steps
1. Create nginx custom image
```
docker build -t nginx-fpm .
```

2. Push the custom image to your private repo (OPTIONAL)
```
docker push nginx-fpm <yourdockerhub>/nginx-fpm
```

OPTION B ) If you don't want to push the repo to your docker hub repo, you can modify imagePullPolicy to IfNotPresent


3. Mount your current working directory (where all the yaml templates and Dockerfile are) so that you will allow minikube to resolve hostPath value from the PersistentVolume-fpm.yaml
export DIR=$(pwd)
minikube mount $DIR:$DIR
```


## Once you have pre-configured the environment
just run 

```
kubectl apply -f .
```

then 

```
kubectl port-forward service/nginx 8080:80
```

then on your browser test by going on 

```
localhost:8080
```
