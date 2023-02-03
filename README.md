
https://hackmamba.io/blog/2022/04/running-docker-in-a-jenkins-container/
https://davelms.medium.com/run-jenkins-in-a-docker-container-part-1-docker-in-docker-7ca75262619d

docker build -t myjenkins-blueocean:2.332.3-1 .

--privileged !!!!!!!! -> selinux
-u root !!!!  --> git

connect to docker socket:

docker run -u root --privileged --name jenkins-blueocean --restart=on-failure --detach   --network jenkins  --publish 8080:8080 --publish 50000:50000   --volume jenkins-data:/var/jenkins_home:z --volume /var/run/docker.sock:/var/run/docker.sock:z myjenkins-blueocean:2.332.3-1

and use unix:///var/run/docker.sock in jenkins docker config


RUN apt-get update && apt-get install -y docker-ce vim
RUN usermod -aG docker jenkins
RUN groupmod -g 968 docker

use socat to forward to docker socket:

docker run -u root --privileged --name jenkins-blueocean --restart=on-failure --detach   --network jenkins --env DOCKER_HOST=tcp://<IPAddress>:2375   --env DOCKER_CERT_PATH="" --env DOCKER_TLS_VERIFY=""   --publish 8080:8080 --publish 50000:50000   --volume jenkins-data:/var/jenkins_home:z  myjenkins-blueocean:2.332.3-1


docker run -d --restart=always -p 127.0.0.1:2375:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock:z alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock

docker inspect <container_id> | grep IPAddress
and use tcp://<IPAddress>:2375 in jenkins docker config

if any selinux issues -> sudo setenforce 0
