# Docker-ce

# jenkins image build & push
docker build -t moricom/jenkins .\
docker push moricom/jenkins

# jenkins default run 
docker run -d -p 8080:8080 --name jenkins moricom/jenkins

# ★ jenkins docker in docker
> which docker  
> docker run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker --name jenkins moricom/jenkins  
or    
> docker run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/usr/bin/docker --name jenkins moricom/jenkins

# jenkins volume share example
docker run -d -p 8080:8080 -v /workspace/jenkins/share:/var/jenkins_home --name jenkins moricom/jenkins

# jenkins docker in docker && volume share example
docker run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -v /workspace/jenkins/share:/var/jenkins_home --name jenkins moricom/jenkins

default ID: admin  
default PW: admin
