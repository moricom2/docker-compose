# docker-compose

CentOS7 기준 docker, docker-compose 설치방법

1. docker 커뮤니티에디션  리포지토리 연결 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

2. docker 커뮤니티에디션 패키지 설치 
yum -y install docker-ce

3. docker 서비스 시작/활성/상태확인 
systemctl start docker.service
systemctl enable docker.service
systemctl status docker.service

4 docker-compose 설치 
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
