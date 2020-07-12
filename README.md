# CentOS7 기준 docker, docker-compose, git 설치

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

### 7. git 설치
> yum -y install git  

# docker-compose 로 CICD 도구 설치

### 1. docker-compose.yml 샘플코드 저장소 복제
> mkdir /app && cd /app  
> git clone https://github.com/moricom2/docker-compose.git  

### 2. cicd 디렉토리 이동
> cd docker-compose/cicd  

### 3. docker-compose.yml 수정 (IP 셋팅 등)
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
+       external_url 'http://192.168.63.180';registry_external_url 'http://192.168.63.180:5005'
        # Add any other gitlab.rb configuration here, each on its own line
      GITLAB_ROOT_PASSWORD: 'root_git'
      GITLAB_TIMEZONE: 'Asia/Seoul'
      TZ: 'Asia/Seoul'
    ports:
      - '80:80'
      - '5005:5005'
    volumes:
      - '/data/gitlab/config:/etc/gitlab'
      - '/data/gitlab/logs:/var/log/gitlab'
      - '/data/gitlab/data:/var/opt/gitlab'   
  jenkins:
!   image: 'jenkins/jenkins:latest'
    user: root
    ports:
      - '8080:8080'
    volumes:
!     - '/data/jenkins/jenkins_home:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/local/bin/docker:/usr/bin/docker'
    environment:
      TZ: 'Asia/Seoul'
  sonarqube:
    image: 'sonarqube:latest'
    ports:
      - '9000:9000'
    volumes:
      - '/data/sonarqube/conf:/opt/sonarqube/conf'
      - '/data/sonarqube/data:/opt/sonarqube/data'
      - '/data/sonarqube/logs:/opt/sonarqube/logs'
      - '/data/sonarqube/extensions:/opt/sonarqube/extensions'
    environment:
      TZ: 'Asia/Seoul'
  nexus:
    image: 'sonatype/nexus:oss'
    ports:
      - '8081:8081'
    volumes:
!     - '/data/nexus/data:/sonatype-work'      
    environment:
      TZ: 'Asia/Seoul'
  registry:
    image: 'registry:2.0'
    ports:
      - '5000:5000'
    environment:
      TZ: 'Asia/Seoul'
```
### 4.  maven 플러그인 copy + jenkins 컨테이너 이미지 빌드 && volume 디렉토리에 디폴트 플러그인 copy
> cp -r /app/docker-compose/dockerfiles/.m2 /app/docker-compose/dockerfiles/jenkins/  
> docker build -t jenkins/jenkins:latest /app/docker-compose/dockerfiles/jenkins/  
> mkdir -p /data/jenkins && cp -r /app/docker-compose/dockerfiles/jenkins/share /data/jenkins/jenkins_home  

### 5. nexus 컨테이너 실행시 volume 디렉토리 접근권한 오류 방지를 위한 설정
> mkdir -p /data/nexus/data && chown -R 200 /data/nexus/data  

### 6. docker-compose로 이미지 받기(parallel) && 실행 && 로그보기
> docker-compose pull --parallel  
> docker-compose up -d  
> docker-compose logs -f

### 7. 결과 확인
> docker-compose ps  
```diff
                 Name                       Command                  State                                  Ports
    ------------------------------------------------------------------------------------------------------------------------------------
    cicd_gitlab_1        /assets/wrapper                  Up (healthy)   22/tcp, 443/tcp, 0.0.0.0:5005->5005/tcp, 0.0.0.0:80->80/tcp
    cicd_jenkins_1       /bin/tini -- /usr/local/bi ...   Up             50000/tcp, 0.0.0.0:8080->8080/tcp
    cicd_nexus_1         /bin/sh -c java   -Dnexus- ...   Up             0.0.0.0:8081->8081/tcp
    cicd_registry_1      registry cmd/registry/conf ...   Up             0.0.0.0:5000->5000/tcp
    cicd_sonarqube_1     bin/run.sh bin/sonar.sh          Up             0.0.0.0:9000->9000/tcp
```  

> docker ps  
```diff
 CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                    PORTS
                               NAMES
 e6e33dfd1bd4        sonatype/nexus:oss                     "/bin/sh -c 'java   …"   21 minutes ago      Up 21 minutes             0.0.0.0:8081->8081/tcp
+                                  cicd_nexus_1
 362e26958539        jenkins/jenkins:latest                 "/bin/tini -- /usr/l…"   21 minutes ago      Up 21 minutes             0.0.0.0:8080->8080/tcp, 50000/tcp
+                                  cicd_jenkins_1
 802ac4d52b16        sonarqube:latest                       "bin/run.sh bin/sona…"   21 minutes ago      Up 21 minutes             0.0.0.0:9000->9000/tcp
+                                  cicd_sonarqube_1
 b4215ca7521e        registry:2.0                           "registry cmd/regist…"   21 minutes ago      Up 21 minutes             0.0.0.0:5000->5000/tcp
+                                  cicd_registry_1
 3b8c9cb93a7e        gitlab/gitlab-ce:latest                "/assets/wrapper"        21 minutes ago      Up 21 minutes (healthy)   22/tcp, 0.0.0.0:80->80/tcp,443/tcp, 0.0.0.0:5005->5005/tcp
+                                  cicd_gitlab_1
```  

### 8. 브라우저 테스트
 1) gitlab: http://192.168.63.180/ > 로딩이 오래 걸림  
 2) 젠킨스: http://192.168.63.180:8080/   
 3) 소나큐브 : http://192.168.63.180:9000/   
 4) 넥서스: http://192.168.63.180:8081/nexus  
 5) docker registry: http://192.168.63.180:5000/v2/  
  => "{}" 값 확인 하면 됨. 

# docker-compose 로 개발도구 설치

### 1. docker-compose.yml 샘플코드 저장소 복제
> mkdir /app && cd /app    
> git clone https://github.com/moricom2/docker-compose.git  

### 2. devtools 디렉토리 이동
> cd docker-compose/devtools  

### 3. docker-compose.yml 수정 (IP 셋팅 등)
> vi docker-compose.yml
```diff
version: '3'
services:
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
+     CHE_IP: '192.168.63.186'
+     CHE_HOST: '192.168.63.186'
      TZ: 'Asia/Seoul'
```

### 4. eclipse-che 컨테이너 실행시 keycloak 관련 오류 방지를 위한 이미지 pull & tag 작업
> docker pull florentbenoit/jboss-keycloak-openshift:3.4.3.Final  
> docker tag florentbenoit/jboss-keycloak-openshift:3.4.3.Final jboss/keycloak-openshift:3.4.3.Final  

### 5. docker-compose로 이미지 받기(parallel) && 실행 && 로그보기
> docker-compose pull --parallel  
> docker-compose up -d  
> docker-compose logs -f

### 6. 결과 확인
> docker-compose ps  
```diff
                 Name                       Command                  State                                  Ports
    ------------------------------------------------------------------------------------------------------------------------------------
+   devtools_eclipse-che_1   /scripts/entrypoint.sh start     Exit 0

```  

> docker ps  
```diff
 CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                    PORTS
                               NAMES
 3b76faf2f821        eclipse/che-server:6.19.0              "/entrypoint.sh"         16 minutes ago      Up 16 minutes             8000/tcp, 8080/tcp, 0.0.0.0:8082->8082/tcp
+                                  che-8082
 0c8d6cf52ea9        jboss/keycloak-openshift:3.4.3.Final   "start-keycloak.sh -…"   16 minutes ago      Up 16 minutes (healthy)   0.0.0.0:5050->8080/tcp
+                                  che8082_keycloak_1
 5cff8c7c936f        centos/postgresql-96-centos7:9.6       "container-entrypoin…"   16 minutes ago      Up 16 minutes (healthy)   5432/tcp
+                                  che8082_postgres_1
```  

### 7. 브라우저 테스트
 1) che-8082: http://192.168.63.186:8082  
 2) che8082_keycloak: http://192.168.63.186:5050
 
<s> ### 8. maven 플러그인 copy + dev-machine 컨테이너 이미지 빌드</s>
> cp -r /app/docker-compose/dockerfiles/.m2 /app/docker-compose/dockerfiles/ubuntu_jdk8/  
> docker build -t eclipse/ubuntu_jdk8:latest /app/docker-compose/dockerfiles/ubuntu_jdk8/  


```diff
docker cp /app/docker-compose/dockerfiles/.m2/repository 컨테이너명:/home/user/.m2/
docker exec 컨테이너명 sudo chown -R user:user /home/user/.m2/repository
```

```diff
cp ${current.project.path}/target/*.war $TOMCAT_HOME/webapps/ROOT.war
$TOMCAT_HOME/bin/catalina.sh run 2>&1
```
