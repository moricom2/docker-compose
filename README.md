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

### 4. docker-compose.yml 수정
> vi docker-compose.yml
>> 

    version: '3'
    services:
      gitlab:
        image: 'gitlab/gitlab-ce:latest'
        #restart: always
        hostname: 'gitlab.example.com'
        environment:
          GITLAB_OMNIBUS_CONFIG: |
            external_url 'http://192.168.99.100';registry_external_url 'http://192.168.99.100:5005'
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
          - '/data/nexus/data:/sonatype-work'      
      registry:
        image: 'registry:2.0'
        ports:
          - '5000:5000'  

### 4. nexus 컨테이너 실행시 volume 디렉토리 접근권한 오류 방지를 위한 설정
> mkdir -p /data/nexus/data && chown -R 200 /data/nexus/data  

### 5. docker-compose 실행
> docker-compose up -d  

### 6. 결과 확인
> docker-compose ps  
>> 

            Name                      Command                  State                                  Ports
    ----------------------------------------------------------------------------------------------------------------------------------  
    devtools_gitlab_1      /assets/wrapper                  Up (healthy)   22/tcp, 443/tcp, 0.0.0.0:5005->5005/tcp, 0.0.0.0:80->80/tcp  
    devtools_jenkins_1     /bin/tini -- /usr/local/bi ...   Up             50000/tcp, 0.0.0.0:8080->8080/tcp  
    devtools_nexus_1       /bin/sh -c java   -Dnexus- ...   Up             0.0.0.0:8081->8081/tcp  
    devtools_registry_1    registry cmd/registry/conf ...   Up             0.0.0.0:5000->5000/tcp  
    devtools_sonarqube_1   bin/run.sh bin/sonar.sh          Up             0.0.0.0:9000->9000/tcp  
