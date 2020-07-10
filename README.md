# CentOS7 기준 docker, docker-compose 설치방법

### 1. nameserver 확인 (8.8.8.8 없을 시 vi로 추가)
> cat /etc/resolv.conf  
```diff
    nameserver 8.8.8.8
```
### 2. timezone 설정
> timedatectl set-timezone Asia/Seoul  

### 3. docker 커뮤니티에디션  리포지토리 연결 
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo  

### 4. docker 커뮤니티에디션 패키지 설치 
> yum -y install docker-ce  

### 5. docker 서비스 시작/활성/상태확인 
> systemctl start docker.service  
> systemctl enable docker.service  
> systemctl status docker.service  

### 6. docker-compose 설치 
> sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  
>
> sudo chmod +x /usr/local/bin/docker-compose  


# docker-compose 로 DevOps/CICD 도구(gitlab, jenkins, nexus, sonarqube, registry) 설치하기

### 1. git 설치
> yum -y install git  

### 2. docker-compose.yml 샘플코드 저장소 복제
> mkdir /app && cd /app  
> git clone https://github.com/moricom2/docker-compose.git  

### 3. devtools 디렉토리 이동
> cd docker-compose/devtools  

### 4. docker-compose.yml 수정
> vi docker-compose.yml
```diff
version: '3'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    #restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
+       external_url 'http://192.168.99.100';registry_external_url 'http://192.168.99.100:5005'
        # Add any other gitlab.rb configuration here, each on its own line
      GITLAB_ROOT_PASSWORD: "root_git"
      GITLAB_TIMEZONE: "Asia/Seoul"  
    ports:
      - '80:80'
      - '5005:5005'
    volumes:
      - '/data/gitlab/config:/etc/gitlab'
      - '/data/gitlab/logs:/var/log/gitlab'
      - '/data/gitlab/data:/var/opt/gitlab'    
  jenkins:
    #image: 'jenkins:latest'
    image: 'moricom/jenkins:latest'
    user: root
    ports:
      - '8080:8080'
    volumes:
      - '/data/jenkins/jenkins_home:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/local/bin/docker:/usr/bin/docker'
  sonarqube:
    image: 'sonarqube:latest'
    ports:
      - '9000:9000'
    volumes:
      - '/data/sonarqube/conf:/opt/sonarqube/conf'
      - '/data/sonarqube/data:/opt/sonarqube/data'
      - '/data/sonarqube/logs:/opt/sonarqube/logs'
      - '/data/sonarqube/extensions:/opt/sonarqube/extensions'
  nexus:
    image: 'sonatype/nexus:oss'
    ports:
      - '8081:8081'
    volumes:
!     - '/data/nexus/data:/sonatype-work'      
  registry:
    image: 'registry:2.0'
    ports:
      - '5000:5000'
  eclipse-che:
    image: 'eclipse/che:6.19.0'
    volumes:
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/data/eclipse:/data'
    command:
      - start
    environment:
      CHE_PORT: '8082'
      CHE_MULTIUSER: 'true'
      CHE_IP: '192.168.99.100'
      CHE_HOST: '192.168.99.100'
```

### 5. nexus 컨테이너 실행시 volume 디렉토리 접근권한 오류 방지를 위한 설정
> mkdir -p /data/nexus/data && chown -R 200 /data/nexus/data  

### 6. eclipse-che 컨테이너 실행시 keycloak 관련 오류 방지를 위한 이미지 pull & tag 작업
> docker pull florentbenoit/jboss-keycloak-openshift:3.4.3.Final  
> docker tag florentbenoit/jboss-keycloak-openshift:3.4.3.Final jboss/keycloak-openshift:3.4.3.Final  

### 7. docker-compose 실행
> docker-compose up -d  

### 7-1. docker-compose로 이미지 받기(parallel) && docker-compose 실행
> docker-compose pull --parallel && docker-compose up -d

### 8. 결과 확인
> docker-compose ps  
```diff
                 Name                       Command                  State                                  Ports
    ------------------------------------------------------------------------------------------------------------------------------------
!   devtools_eclipse-che_1   /scripts/entrypoint.sh start     Exit 0
    devtools_gitlab_1        /assets/wrapper                  Up (healthy)   22/tcp, 443/tcp, 0.0.0.0:5005->5005/tcp, 0.0.0.0:80->80/tcp
    devtools_jenkins_1       /bin/tini -- /usr/local/bi ...   Up             50000/tcp, 0.0.0.0:8080->8080/tcp
    devtools_nexus_1         /bin/sh -c java   -Dnexus- ...   Up             0.0.0.0:8081->8081/tcp
    devtools_registry_1      registry cmd/registry/conf ...   Up             0.0.0.0:5000->5000/tcp
    devtools_sonarqube_1     bin/run.sh bin/sonar.sh          Up             0.0.0.0:9000->9000/tcp
```  

> docker ps  
```diff
 CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                    PORTS
                               NAMES
 3b76faf2f821        eclipse/che-server:6.19.0              "/entrypoint.sh"         16 minutes ago      Up 16 minutes             8000/tcp, 8080/tcp, 0.0.0.0:8082->8082/tcp
!                                  che-8082
 0c8d6cf52ea9        jboss/keycloak-openshift:3.4.3.Final   "start-keycloak.sh -…"   16 minutes ago      Up 16 minutes (healthy)   0.0.0.0:5050->8080/tcp
!                                  che8082_keycloak_1
 5cff8c7c936f        centos/postgresql-96-centos7:9.6       "container-entrypoin…"   16 minutes ago      Up 16 minutes (healthy)   5432/tcp
!                                  che8082_postgres_1
 e6e33dfd1bd4        sonatype/nexus:oss                     "/bin/sh -c 'java   …"   21 minutes ago      Up 21 minutes             0.0.0.0:8081->8081/tcp
+                                  devtools_nexus_1
 362e26958539        moricom/jenkins:latest                 "/bin/tini -- /usr/l…"   21 minutes ago      Up 21 minutes             0.0.0.0:8080->8080/tcp, 50000/tcp
+                                  devtools_jenkins_1
 802ac4d52b16        sonarqube:latest                       "bin/run.sh bin/sona…"   21 minutes ago      Up 21 minutes             0.0.0.0:9000->9000/tcp
+                                  devtools_sonarqube_1
 b4215ca7521e        registry:2.0                           "registry cmd/regist…"   21 minutes ago      Up 21 minutes             0.0.0.0:5000->5000/tcp
+                                  devtools_registry_1
 3b8c9cb93a7e        gitlab/gitlab-ce:latest                "/assets/wrapper"        21 minutes ago      Up 21 minutes (healthy)   22/tcp, 0.0.0.0:80->80/tcp,443/tcp, 0.0.0.0:5005->5005/tcp
+                                  devtools_gitlab_1
```  

### 9. 브라우저 테스트
 1) 소나큐브 : http://192.168.99.100:9000/  
 2) 젠킨스: http://192.168.99.100:8080/  
 3) gitlab: http://192.168.99.100/ > 로딩이 오래 걸림  
 4) 넥서스: http://192.168.99.100:8081/nexus  
 5) docker registry: http://192.168.99.100:5000/v2/  
  => "{}" 값 확인 하면 됨.  
 6) eclipse-che: http://192.168.99.100:8082  
