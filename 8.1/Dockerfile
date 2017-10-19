# This image provides a base for building and running WildFly applications.
# It builds using maven and runs the resulting artifacts on WildFly 8.1.

FROM openshift/base-centos7

MAINTAINER Ben Parees <bparees@redhat.com>

EXPOSE 8080 9990

ENV WILDFLY_VERSION=8.1 \
    WILDFLY_VERSION_BIN=8.1.0.Final \
    MAVEN_VERSION=3.3.9

LABEL io.k8s.description="Platform for building and running JEE applications on WildFly 8.1" \
      io.k8s.display-name="WildFly 8.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,wildfly,wildfly81" \
      io.openshift.s2i.destination="/opt/s2i/destination"

# Install Maven, Wildfly 8
RUN INSTALL_PKGS="tar ant unzip bc which lsof java-1.8.0-openjdk java-1.8.0-openjdk-devel glibc-devel.i686 ncurses-libs.i686 libxml2-devel.i686 file emacs" && \
    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    (curl -v https://www.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    ln -sf /usr/local/apache-maven-$MAVEN_VERSION/bin/mvn /usr/local/bin/mvn
RUN    mkdir -p /wildfly && \
    wget -O /tmp/wildfly-$WILDFLY_VERSION_BIN.tar.gz https://download.jboss.org/wildfly/$WILDFLY_VERSION_BIN/wildfly-$WILDFLY_VERSION_BIN.tar.gz 
RUN     gunzip /tmp/wildfly-$WILDFLY_VERSION_BIN.tar.gz && \
    tar -xvf  /tmp/wildfly-$WILDFLY_VERSION_BIN.tar -C /wildfly && \
    rm /tmp/wildfly-$WILDFLY_VERSION_BIN.tar && \
    mv /wildfly/wildfly-$WILDFLY_VERSION_BIN/* /wildfly/ && \
    rm -rf /tmp/wildfly-$WILDFLY_VERSION_BIN && \
    mkdir -p /opt/s2i/destination

RUN wget https://github.com/openshift/origin/releases/download/v3.6.0/openshift-origin-client-tools-v3.6.0-c4dd4cf-linux-64bit.tar.gz  && \
    tar -xf openshift-origin-client-tools-v3.6.0-c4dd4cf-linux-64bit.tar.gz

ENV PATH="${HOME}/openshift-origin-client-tools-v3.6.0-c4dd4cf-linux-64bit/:${PATH}"

# Add s2i wildfly customizations
#ADD ./contrib/wfmodules/ /wildfly/modules/
#ADD ./contrib/wfbin/standalone.conf /wildfly/bin/standalone.conf
#ADD ./contrib/wfcfg/standalone.xml /wildfly/standalone/configuration/standalone.xml
COPY customizations/undertow-servlet-1.0.15.Final.jar /wildfly/modules/system/layers/base/io/undertow/servlet/main/

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH

COPY ./s2i/bin/ $STI_SCRIPTS_PATH

#lets add our postgres driver
COPY ./postgres/postgresql-9.4.1208.jre6.jar /tmp/
RUN  /bin/sh -c "/wildfly/bin/standalone.sh &" && \
    sleep 10 && \
    /wildfly/bin/jboss-cli.sh --connect  --command="module add --name=org.postgres --resources=/tmp/postgresql-9.4.1208.jre6.jar --dependencies=javax.api,javax.transaction.api" && \
    /wildfly/bin/jboss-cli.sh --connect	 --command=:shutdown && \
    rm -rf /tmp/postgresql-9.4.1208.jre6.jar



RUN chown -R 1001:0 /wildfly  && \
    chown -R 1001:0 $HOME/.pki && \
    chmod -R a+rwx /wildfly && \
    mkdir -p /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/hummingbird/data/ && \
    chmod -R g+rw /opt/s2i/destination

RUN /wildfly/bin/add-user.sh --silent=true test-admin '^NtEEzYULRt?dJ7?X3$w83FtrbMk'

USER 1001

CMD $STI_SCRIPTS_PATH/usage
