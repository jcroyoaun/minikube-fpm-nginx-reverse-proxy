# How to create a reverse proxy to connect fpm container to an nginx container
The other day I was playing around with nginx and php and found that for the insanely popular combination of php-fpm and nginx to work in Kubernetes there might be a quite rather interesting (or at least not as straight forward) setup needed.

## Initial issue
At first I just tried generating a custom php-fpm where I would copy the index.php file into the php-fpm container. The container would just die immediately.

I figured that it was dying because there's nothing being executed on the php-fpm side and it needs an nginx server to hold the php file for it to work.

I then started an nginx container and linked it to the custom php-fpm. nginx worked fine but the php-fpm would still die.

Then I figured that, for both to work together, there needs to be a way that nginx can reference the php-fpm docker container, so there was a reverse proxy config to be added, and a nice way to do it would be to modify default.conf from nginx :

Since the php-fpm container is being named "fpm" I figured this would be a good solution :
```
server {
    listen  80;

    # this path MUST be exactly as docker-compose.fpm.volumes,
    # even if it doesn't exist in this dock.
    root /complex/path/to/files;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/.+\.php(/|$) {
        fastcgi_pass fpm:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
```

Then I would create a custom nginx image with this overwritten default.conf file :
```
FROM nginx:latest
COPY ./default.conf /etc/nginx/conf.d/
```

With this configuration in place (running custom nginx image with modified default.conf and a base php-fpm image) both containers would now keep running. Now it was time to figure a way to make them access the index.php file, and I did [that through mounting a volume](https://github.com/jcroyoaun/fpm-nginx-compose-example/blob/master/docker-compose.yml). 

In the steps below, will show you how I did it on minikube : 

After everything worked,
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
