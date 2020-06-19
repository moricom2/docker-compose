# CentOS7 기준 docker, docker-compose 설치방법

### 1. docker 커뮤니티에디션  리포지토리 연결 
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo  

### 2. docker 커뮤니티에디션 패키지 설치 
> yum -y install docker-ce  

### 3. docker 서비스 시작/활성/상태확인 
> systemctl start docker.service  
> systemctl enable docker.service  
> systemctl status docker.service  

### 4. docker-compose 설치 
> sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  
>
> sudo chmod +x /usr/local/bin/docker-compose  


# docker-compose 로 DevOps/CICD 도구(gitlab, jenkins, nexus, sonarqube, registry) 설치하기

### 1. git 설치
> yum -y install git  

### 2. docker-compose.yml 샘플코드 저장소 복제
> git clone https://github.com/moricom2/docker-compose.git  

### 3. devtools 디렉토리 이동
> cd docker-compose/devtools  

### 4. docker-compose 실행
> docker-compose up -d  

### 5. 결과 확인
> docker-compose ps  

> docker-compose ps  

        Name                      Command                  State                                  Ports
----------------------------------------------------------------------------------------------------------------------------------
devtools_gitlab_1      /assets/wrapper                  Up (healthy)   22/tcp, 443/tcp, 0.0.0.0:5005->5005/tcp, 0.0.0.0:80->80/tcp
devtools_jenkins_1     /bin/tini -- /usr/local/bi ...   Up             50000/tcp, 0.0.0.0:8080->8080/tcp
devtools_nexus_1       /bin/sh -c java   -Dnexus- ...   Up             0.0.0.0:8081->8081/tcp
devtools_registry_1    registry cmd/registry/conf ...   Up             0.0.0.0:5000->5000/tcp
devtools_sonarqube_1   bin/run.sh bin/sonar.sh          Up             0.0.0.0:9000->9000/tcp
>  

>  

          Name                      Command                  State                                  Ports
  ----------------------------------------------------------------------------------------------------------------------------------
  devtools_gitlab_1      /assets/wrapper                  Up (healthy)   22/tcp, 443/tcp, 0.0.0.0:5005->5005/tcp, 0.0.0.0:80->80/tcp
  devtools_jenkins_1     /bin/tini -- /usr/local/bi ...   Up             50000/tcp, 0.0.0.0:8080->8080/tcp
  devtools_nexus_1       /bin/sh -c java   -Dnexus- ...   Up             0.0.0.0:8081->8081/tcp
  devtools_registry_1    registry cmd/registry/conf ...   Up             0.0.0.0:5000->5000/tcp
  devtools_sonarqube_1   bin/run.sh bin/sonar.sh          Up             0.0.0.0:9000->9000/tcp
