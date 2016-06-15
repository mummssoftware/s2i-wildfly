# This image provides a base for building and running WildFly applications.
# It builds using maven and runs the resulting artifacts on WildFly 8.1.

FROM openshift/base-centos7

MAINTAINER Ben Parees <bparees@redhat.com>

EXPOSE 8080 9990

ENV WILDFLY_VERSION=8.1 \
    WILDFLY_VERSION_BIN=8.1.0.Final \
    MAVEN_VERSION=3.3.3

LABEL io.k8s.description="Platform for building and running JEE applications on WildFly 8.1" \
      io.k8s.display-name="WildFly 8.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,wildfly,wildfly81" \
      io.openshift.s2i.destination="/opt/s2i/destination"

# Install Maven, Wildfly 8
RUN INSTALL_PKGS="tar unzip bc which lsof java-1.8.0-openjdk java-1.8.0-openjdk-devel" && \
    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    (curl -v https://www.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    ln -sf /usr/local/apache-maven-$MAVEN_VERSION/bin/mvn /usr/local/bin/mvn && \
    mkdir -p /wildfly && \
    (curl -v https://download.jboss.org/wildfly/$WILDFLY_VERSION_BIN/wildfly-$WILDFLY_VERSION_BIN.tar.gz | tar -zx --strip-components=1 -C /wildfly) && \
    mkdir -p /opt/s2i/destination

# Add s2i wildfly customizations
#ADD ./contrib/wfmodules/ /wildfly/modules/
#ADD ./contrib/wfbin/standalone.conf /wildfly/bin/standalone.conf
#ADD ./contrib/wfcfg/standalone.xml /wildfly/standalone/configuration/standalone.xml

RUN yum update && yum install -y ant && yum install -y maven && yum clean all -y

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH

COPY ./s2i/bin/ $STI_SCRIPTS_PATH

#lets add our postgres driver
COPY ./postgres/postgresql-9.4.1208.jre6.jar /tmp/
RUN /wildfly/bin/add-user.sh test-admin ^NtEEzYULRt?dJ7?X3$w83FtrbMk -s && \
    /bin/sh -c "/wildfly/bin/standalone.sh &" && \
    sleep 10 && \
    /wildfly/bin/jboss-cli.sh --connect --user=test-admin --password=^NtEEzYULRt?dJ7?X3$w83FtrbMk --command="module add --name=org.postgres --resources=/tmp/postgresql-9.4.1208.jre6.jar --dependencies=javax.api,javax.transaction.api" && \
    /wildfly/bin/jboss-cli.sh --connect	--user=test-admin --password=^NtEEzYULRt?dJ7?X3$w83FtrbMk --command=:shutdown && \
    rm /tmp/*.sh /tmp/*.jar


RUN chown -R 1001:0 /wildfly  && \
    chown -R 1001:0 $HOME/.pki && \
    chmod -R ug+rw /wildfly && \
    mkdir -p /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/s2i/destination


USER 1001

CMD $STI_SCRIPTS_PATH/usage
