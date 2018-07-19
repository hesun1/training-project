FROM openjdk:8-jdk

RUN \
 apt-get update && \
 apt-get install -y unzip && \
 apt-get install -y libmysql-java && \
 rm /etc/localtime; ln -sf /usr/share/zoneinfo/Japan /etc/localtime

ENV ADMIN_USER admin

ENV PAYARA_PATH /opt/payara5

RUN \
 mkdir -p ${PAYARA_PATH}/deployments && \
 useradd -b /opt -m -s /bin/bash -d ${PAYARA_PATH} payara && echo payara:payara | chpasswd

# specify Payara version to download
ENV PAYARA_PKG https://search.maven.org/remotecontent?filepath=fish/payara/distributions/payara/5.182/payara-5.182.zip
ENV PAYARA_VERSION 182

ENV PKG_FILE_NAME payara-full-${PAYARA_VERSION}.zip

# Download Payara Server, install, then remove downloaded file
RUN wget --quiet -O /opt/${PKG_FILE_NAME} ${PAYARA_PKG}
RUN \
 unzip -qq /opt/${PKG_FILE_NAME} -d /opt && \
 chown -R payara:payara /opt && \
 rm /opt/${PKG_FILE_NAME} && \
 ln -s ${PAYARA_PATH} /opt/payara

USER payara
WORKDIR ${PAYARA_PATH}

# set credentials to admin/admin

ENV ADMIN_PASSWORD admin

RUN echo 'AS_ADMIN_PASSWORD=\n\
AS_ADMIN_NEWPASSWORD='${ADMIN_PASSWORD}'\n\
EOF\n'\
>> /opt/tmpfile

RUN echo 'AS_ADMIN_PASSWORD='${ADMIN_PASSWORD}'\n\
EOF\n'\
>> /opt/pwdfile

 # domain1
RUN ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/tmpfile change-admin-password && \
 ${PAYARA_PATH}/bin/asadmin start-domain domain1 && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile enable-secure-admin && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile add-library /usr/share/java/mysql.jar && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile create-jdbc-connection-pool --datasourceclassname "com.mysql.jdbc.jdbc2.optional.MysqlDataSource" --restype "javax.sql.DataSource" --property "user=muit:password=password:DatabaseName=training:ServerName=db:port=3306" mysql-pool && \
 ${PAYARA_PATH}/bin/asadmin stop-domain domain1 && \
 rm -rf ${PAYARA_PATH}/glassfish/domains/domain1/osgi-cache
 # production
RUN \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/tmpfile change-admin-password --domain_name=production && \
 ${PAYARA_PATH}/bin/asadmin start-domain production && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile enable-secure-admin && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile add-library /usr/share/java/mysql.jar && \
 ${PAYARA_PATH}/bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile create-jdbc-connection-pool --datasourceclassname "com.mysql.jdbc.jdbc2.optional.MysqlDataSource" --restype "javax.sql.DataSource" --property "user=muit:password=password:DatabaseName=training:ServerName=db:port=3306" mysql-pool && \
 ${PAYARA_PATH}/bin/asadmin stop-domain production && \
 rm -rf ${PAYARA_PATH}/glassfish/domains/production/osgi-cache

# cleanup
RUN rm /opt/tmpfile

ENV PAYARA_DOMAIN domain1
ENV DEPLOY_DIR ${PAYARA_PATH}/deployments
ENV AUTODEPLOY_DIR ${PAYARA_PATH}/glassfish/domains/${PAYARA_DOMAIN}/autodeploy

# Default payara ports to expose
EXPOSE 4848 8009 8080 8181

ENV POSTBOOT_COMMANDS ${PAYARA_PATH}/post-boot-commands.asadmin

COPY generate_deploy_commands.sh ${PAYARA_PATH}/generate_deploy_commands.sh
COPY bin/startInForeground.sh ${PAYARA_PATH}/bin/startInForeground.sh

USER root
RUN \
 chown -R payara:payara ${PAYARA_PATH}/generate_deploy_commands.sh && \
 chmod a+x ${PAYARA_PATH}/generate_deploy_commands.sh && \
 chmod a+x ${PAYARA_PATH}/bin/startInForeground.sh
USER payara

ENTRYPOINT ${PAYARA_PATH}/generate_deploy_commands.sh && ${PAYARA_PATH}/bin/startInForeground.sh --passwordfile=/opt/pwdfile --postbootcommandfile ${POSTBOOT_COMMANDS} ${PAYARA_DOMAIN}