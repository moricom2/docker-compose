FROM jenkins/jenkins:latest
COPY share /var/jenkins_home
USER root
COPY .m2 /root/.m2
RUN wget http://mirror.apache-kr.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
RUN tar xzvf apache-maven-3.6.3-bin.tar.gz -C /usr/share/
RUN update-alternatives --install /usr/bin/mvn mvn /usr/share/apache-maven-3.6.3/bin/mvn 9999
