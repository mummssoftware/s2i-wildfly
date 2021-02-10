# This image provides a base for building and running WildFly applications.
# It builds using maven and runs the resulting artifacts on WildFly 9.0

FROM openshift/base-centos7

MAINTAINER Ben Parees <bparees@redhat.com>

EXPOSE 8080 9990

ENV WILDFLY_VERSION=9.0.2 \
    WILDFLY_VERSION_BIN=9.0.2.Final \
    MAVEN_VERSION=3.3.9

LABEL io.k8s.description="Platform for building and running JEE applications on WildFly 9.0" \
      io.k8s.display-name="WildFly 9.0" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,wildfly,wildfly90" \
      io.openshift.s2i.assemble-input-files="/wildfly/standalone/deployments;/wildfly/standalone/configuration;/wildfly/provided_modules" \
      io.openshift.s2i.destination="/opt/s2i/destination" \
      com.redhat.deployments-dir="/wildfly/standalone/deployments" \
      maintainer="Ben Parees <bparees@redhat.com"

# Install Maven, Wildfly 9
RUN INSTALL_PKGS="tar ant unzip bc which lsof java-1.8.0-openjdk java-1.8.0-openjdk-devel glibc-devel.i686 ncurses-libs.i686 libxml2-devel.i686 file" && \
    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    (curl -v https://www.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    ln -sf /usr/local/apache-maven-$MAVEN_VERSION/bin/mvn /usr/local/bin/mvn && \
    mkdir -p $HOME/.m2 && \
    mkdir -p /wildfly && \
    gunzip /tmp/wildfly-$WILDFLY_VERSION_BIN.tar.gz && \
    tar -xvf  /tmp/wildfly-$WILDFLY_VERSION_BIN.tar -C /wildfly && \
    rm /tmp/wildfly-$WILDFLY_VERSION_BIN.tar && \
    mv /wildfly/wildfly-$WILDFLY_VERSION_BIN/* /wildfly/ && \
    rm -rf /tmp/wildfly-$WILDFLY_VERSION_BIN && \
    mkdir -p /opt/s2i/destination

# Add s2i wildfly customizations
ADD ./contrib/wfmodules/ /wildfly/modules/
ADD ./contrib/wfbin/standalone.conf /wildfly/bin/standalone.conf
ADD ./contrib/wfcfg/standalone.xml /wildfly/standalone/configuration/standalone.xml
ADD ./contrib/settings.xml $HOME/.m2/

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH
  
COPY ./postgres/postgresql-9.4.1208.jre6.jar /tmp/
RUN /wildfly/bin/add-user.sh test-admin ^NtEEzYULRt?dJ7?X3$w83FtrbMk -s && \
   /bin/sh -c "/wildfly/bin/standalone.sh &" && \
   sleep 10 && \
   /wildfly/bin/jboss-cli.sh --connect --user=test-admin --password=^NtEEzYULRt?dJ7?X3$w83FtrbMk --command="module add --name=org.postgres --resources=/tmp/postgresql-9.4.1208.jre6.jar --dependencies=javax.api,javax.transaction.api" && \
   /wildfly/bin/jboss-cli.sh --connect        --user=test-admin --password=^NtEEzYULRt?dJ7?X3$w83FtrbMk --command=:shutdown && \
   rm /tmp/postgresql-9.4.1208.jre6.jar

RUN mkdir -p /opt/hummingbird/data/ && \
    chown -R 1001:0 /wildfly && chown -R 1001:0 $HOME && \
    chmod -R ug+rwX /wildfly && \
    mkdir -p /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/hummingbird/data/ && \
    chown -R 1001:0 /opt/hummingbird/data/ && \
    chmod -R ug+rwX /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/s2i/destination

USER 1001

CMD $STI_SCRIPTS_PATH/usage
