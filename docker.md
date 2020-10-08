# Docker
Docker cloud implementation in digital ocean. Simple and clear commmand. First download ubuntu image from docker hub, 
run the image with name n6. Check if docker process and login to bash. 
```sh
docker pull ubuntu
# -d run as a daemon or backgroun process, -t to allocate psedo tty
docker run -d -t --name n6 ubuntu
docker ps
docker exec -it n6 bash
```

if you wan to mapping docker port(8080) to host(80) just add -p option
```sh
docker run -d -t -p 80:8080 --name n6 ubuntu
```

to Stop and start docker and look at docker resources
```sh
docker stop n6
docker start n6
docker stats
docker rm n6
```

you may change image to different version using tag lates. or you may use your own image in dockerhub by using username/image:tag format.
ps : if you add container, have other command like bash
```sh
docker container run -d -t ubuntu:lates
```
So you must start deploy your apps on your docker container, and put it on the dockerhub.

## Digital Ocean Docker
Setting for digital ocean provider. FIrst thing to do is generate personal access token.
In the API navigation, Personal Access Tokens, Generate new token.

```sh
#create droplet
docker-machine create --digitalocean-size "s-2vcpu-4gb" --driver digitalocean --digitalocean-access-token PERSONAL_ACCESS_TOKEN dropletname
docker-machine ls
#connect to droplet
eval $(docker-machine env dropletname)
# run docker image with -e to set environment variable
docker run -d -p 80:8080 -e "MB_WORKERS=2" -e "MB_KEY=$MB_KEY" machinebox/facebox
#get the droplet ip address
docker-machine ip dropletname
```

## Build docker image
For example flask application, default 2 file is app.py and requirements.txt.
The key is you must create new file name it Dockerfile, and write is down inside Dockerfile :
```sh
FROM python:3-alpine

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```
so this is mean, you using alpine docker image as base image. change direktori to /app in the image and copy and run pip install.
open port 5000 in container, and always tun python app.py for first time. then built the app image and run it:
```sh
# -t means using tag
docker build -t flask_demo:v0 .
docker run -p 80:5000 flask_demo:v0
```
to push your image to repository:
```sh
docker login --username=awangga
docker images
docker tag a4b95e11a188 awangga/ok:demo
docker push awangga/ok
```

https://www.digitalocean.com/community/meetup_kits/getting-started-with-containers-and-kubernetes-a-digitalocean-workshop-kit

## Ubuntu Server Setting
tips: getting internet in ubuntu host for docker:
nano /etc/ufw/sysctl.conf
```sh
#net.ipv4.ip_forward=1 # please uncomment this
ufw allow http
ufw allow https
ufw disable && ufw enable
docker pull romeoz/docker-nginx-php:7.3
docker run -d -t -p 6060:80 --name n6 romeoz/docker-nginx-php:7.3
docker run -d -t -p 7070:80 --name n7 romeoz/docker-nginx-php:7.3
docker run -d -t -p 8080:80 --name n8 romeoz/docker-nginx-php:7.3
docker run -d -t -p 9090:80 --name n9 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1010:80 --name n10 romeoz/docker-nginx-php:7.3

docker exec -it n6 bash

docker run -d -t -p 1212:80 --name n12 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1313:80 --name n13 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1414:80 --name n14 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1515:80 --name n15 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1616:80 --name n16 romeoz/docker-nginx-php:7.3
docker run -d -t -p 1717:80 --name n17 romeoz/docker-nginx-php:7.3

docker pull nats-streaming
docker run -p 4219:4223 -p 8219:8223 --name n19 nats-streaming -p 4223 -m 8223
docker run -p 4220:4223 -p 8220:8223 --name n20 nats-streaming
docker run -p 4220:4223 -p 8220:8223 --name n21 nats-streaming

docker pull minio/minio
docker run -p 9000:9000 --name n22 \
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  minio/minio server /data
```


## Kubernetes
For automatic deployment you must use k8s with yaml configuration. This benefit for load balance or high availaibility apps.
kubctl is all we need, just download it and place on your system execution path. afterthat download or create yaml file and 
add to your OS env variable this is for accessing the kubernetes:

```sh
export KUBECONFIG=filename.yaml
kubectl get nodes
kubectl cluster-info
```

A pods consisting a container, you must create pod to put your container. Explain the task in yaml file and apply it. 
```sh
kubectl run podname --image=ubuntu:latest --port=80
kubectl get pods
kubectl describe pods
kubectl delete pods podname
#creating newdeployment for multiply HA.
kubectl apply -f newdeployment.yaml
kubectl get deployments
kubectl edit newdeployment
kubectl get pods -o wide
#create yaml service and apply it
kubectl apply -f newservice.yaml
kubectl get services
kubectl describe service servicenew
```