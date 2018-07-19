FROM payara/server-full

USER root

#install maven
ENV MAVEN_VERSION=3.3.9
ENV M2_HOME=/opt/apache-maven-$MAVEN_VERSION
ENV PATH=$M2_HOME/bin:$PATH
RUN \
    mkdir ${M2_HOME} && \
    wget -qO- "http://apache.ip-connect.vn.ua/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" | tar -zx --strip-components=1 -C /opt/apache-maven-$MAVEN_VERSION/ 

#install gradle
ENV GRADLE_HOME=/opt/gradle-4.8.1
ENV PATH=$GRADLE_HOME/bin:$PATH
RUN \
    wget https://downloads.gradle.org/distributions/gradle-4.8.1-all.zip -O /tmp/temp.zip && unzip /tmp/temp.zip -d /opt/ && rm /tmp/temp.zip && \
    rm -rf ${GRADLE_HOME}/samples

WORKDIR /projects
