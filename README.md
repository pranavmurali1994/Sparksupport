**1. A) Install Docker 19.03 and enable docker swarm**
 A) Docker 19.03 Installation
#Update the apt package index and install packages to allow apt to use a repository over HTTPS:

sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
#Add Docker’s official GPG key:    

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

#set up the repository.

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
 sudo apt update
  
 # Install Docker engine 19.03
 sudo apt-get install docker-ce=5:19.03.15~3-0~ubuntu-focal docker-ce-cli=5:19.03.15~3-0~ubuntu-focal containerd.io  

**#Adding docker to SUDO Group to run docker command without sudo**
sudo usermod -aG docker ${USER}
sudo su - ${USER}



**1. A) Enabling Docker Swarm**

sudo docker swarm init --advertise-addr < IP of manager node >


docker node ls ( to check the number of node )
docker swarm info  ( information master node and worker node )
docker swarm leave ( leave a node from manager list )
docker swarm rm < node id > (to remove node from list)
docker swarm join-token worker ( to get the tocken to join new worker nodes )
docker service create --name pranav --replicas 3 --publish 80:80 httpd ( create  replicas of microservices )

**#Final output Screenshot
https://user-images.githubusercontent.com/92468658/159152380-50eab8ae-2c98-47ad-b4c5-fc8d35bb74a8.jpg
**
                    **************************************************************************************************************

 **1. B) Run docker registry container with port 5000 and mount volume to /opt/registry

created two EC2 insatce one is Docker Registry Macine another one is client machine

#Perform inside Docker Registry Machine

create a folder inside /opt/registry (spark-registry)
create one more inside this directory volume
create docker compose adding UI service also
---
version: '3'
 
services:
   spark-registry:
      image: registry:2
      container_name: docker-registry
      ports:
          - 5000:5000
      restart: always
      volumes:
          - ./volume:/opt/registry

   spark-registry-ui:
      image: konradkleine/docker-registry-frontend:v2
      container_name: spark-registry-ui
      ports:
         - 8080:80
      environment:
         ENV_DOCKER_REGISTRY_HOST: spark-registry
         ENV_DOCKER_REGISTRY_PORT: 5000

Run the docker-compose up 

#Update the hostfile client node
sudo nano /etc/hosts
<ip>  spark.local.com
#Enable authentication in client node
sudo nano /etc/docker/daemon.json
"insecure-registries": ["spark.local.com:5000"]

#Restart docker service
systemctl stop docker
systemctl start docker

#pull one image from docker hub repo for test
docker pull alpine

once it create image of alpine tag it
docker tag alpine:latest spark.local.com:5000/spark/alpine-spark:v1

push it to Private registry

docker push spark.local.com:5000/spark/alpine-spark:v1

the pushed image we can see on the UI
  
**#Final out put screenshot **
  
https://user-images.githubusercontent.com/92468658/159152612-91d9164f-1411-41a6-acc3-3dcf3d8b81ea.jpg

https://user-images.githubusercontent.com/92468658/159152617-10c1761f-93b3-4f82-8573-f66f21953783.jpg

  ***********************************************************************************************************************************************

**1. C) Build wordpress docker image with Dockerfile and push it to the previously created registry.**
 
#create a folder in machine
sudo mkdir Dockerfile

# inside Dockerfile directory create a file Dockerfile
sudo nano Dockerfile

#inside Dockerfile 
FROM wordpress:latest (this will build the images file from docker hub)

save it

# run the docker file 
docker build Dockerfile

#once it build completely we can see the images file
 docker images
 
 #from the images add tag to wordpress images
docker tag wordpress:latest spark.local.com:5000/pranav/wordpress:v1

# then push it to Docker private registry
docker push spark.local.com:5000/pranav/wordpress:v1

final out put screenshot
https://user-images.githubusercontent.com/92468658/159152795-010aab46-0550-4320-a687-e28e5e497175.jpg

**1.d) Create a stack file for wordpress with a previously built image and deploy.**
 
 stack file services include wordpress,mysql,Nginx 
 
 same dockerfile folder create a docker-compose.yml
 
contents in compose file given below 

version: '3'

networks:
   frontend:
   backend:

volumes:
     db_data: {}
     wordpress_data: {}

services:
    db:
      image: mysql:5.7
      volumes:
        - db_data:/var/lib/mysql
      environment:
        MYSQL_RANDOM_ROOT_PASSWORD: '1'
        MYSQL_DATABASE: admin
        MYSQL_USER: admin
        MYSQL_PASSWORD: admin
      networks:
        - backend

    wordpress:
      depends_on:
        - db
      build: .
      volumes:
        - wordpress_data:/var/www/html/wp-content
      environment:
        WORDPRESS_DB_HOST: db:3306
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpressPASS
        WORDPRESS_DB_NAME: wordpress
      networks:
        - frontend
        - backend

    nginx:
      depends_on:
        - wordpress
        - db
      image: nginx:latest
      volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
        - /etc/letsencrypt/:/etc/letsencrypt/
      ports:
        - 8000:80
        - 443:443
      networks:
       - frontend
       
       
 Final out put screenshot

https://user-images.githubusercontent.com/92468658/159152882-c30e926e-c49c-4265-b42e-d831f0d3a0ba.jpg
  
  *************************************************************************************************************************************************
  
  2. a) Create a custom bridge network with  the name "mynet"
  
  #to create a new network in Docker
    docker network create <name>

#to check network list
    docker network ls
  
  
   2. b) Create two docker containers with the above custom network with the alpine image in detached mode.
  container Name container1 and container2
  
  docker run -itd --name node1 e9adb5357e84 /bin/sh
  docker run -itd --name node2 e9adb5357e84 /bin/sh

  attch network mynet 

  docker network connect mynet container1
  docker network connect mynet container2
  
  2. c) Ping the containers to each other by using the hostname.
  
 #login to container1 & container2

docker exec -it <container id>

#check the connectivity of both container

ping container1 & ping container2 

Attached final output Screenshot 2a,2b,2c

[Updated]  https://user-images.githubusercontent.com/92468658/159484749-51a243a7-8a51-4840-8d66-273e265917e7.png


 
**************************************************************************************************************************************************
  
 **ANSIBLE** 
  
  2) Ansible create role and ansible playbook to create wordpress 
#Ansible Lab set-up
Launch two Ec2 instance one is Ansible control node and another one manage node
Setup hostname in control nodes
setup one user account called ansadmin with directory (sudo useradd -m -d /home/ansadmin ansadmin)
added to root user group (sudo nano /etc/sudoers)
switch to ansadmin user (sudo su - ansadmin)
Generate SSH key. (SSH-Keygen)
enabled password based authentication (sudo nano /etc/ssh/sshd_config)
install python3 and pip
install ansible
setup manged node done with above mentioned all the steps except SSH-key generate 
add managed node ip in control node inventory (sudo nano /etc/ansible/hosts)
create Ansible role (ansible-galaxy wordpress-setup)
screeshot link
  https://user-images.githubusercontent.com/92468658/159153164-831a6da8-a7ac-4a31-94c7-a3494a84a350.jpg


created two ansible roles for pulling image & Up the container from the image
 

sudo ansible-galaxy init setup-wordpress
sudo ansible-galaxy init setup-container

Image pull ansible-playbook
---
- name : Pull wordpress image
  hosts: all
  become: true
  roles:
   - setup-wordpress

Create container from image
---
- name : container wordpress
  hosts: all
  become: true
  roles: 
    - setup-container
 ![Play1]  https://user-images.githubusercontent.com/92468658/159538219-b384d7f5-0e36-439b-be49-d88071837f2b.png
 ![Play2]  https://user-images.githubusercontent.com/92468658/159538268-cfb211d2-6ec8-4c9d-b02a-c7c62aebac55.png
  ![play3] https://user-images.githubusercontent.com/92468658/159538305-c9abed3e-6556-4993-8a8b-0cd729d838a5.png

  

  **************************************************************************************************************************************************
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

