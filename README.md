# How to create a reverse proxy to connect fpm container to an nginx container

## pre-requisites
Have minikube installed
modify the

PersistentVolume-fpm.yaml

so that 
path: 
includes your local working directory (absolute path!!)


## Steps followed
You have to first use minikube mount on a terminal to reference your working directory. This will allow minikube to resolve hostPath from the PersistentVolume-fpm.yaml file. 


export DIR=($pwd)	
minikube mount $DIR:$DIR


## Once you have pre-configured the environment
just run 

kubectl apply -f .


then 

kubectl port-forward service/nginx 8080:80

then

localhost:8080

