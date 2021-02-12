# This image provides a base for building and running WildFly applications.
# It builds using ant and runs the resulting artifacts on WildFly 13.0.0 Final
# It is extends the Wildfly 13.0.0 Final s2i image replacing maven with ant
# and adding the require postgres driver

FROM openshift/wildfly-130-centos7:latest

MAINTAINER Brad Durrow <brad@nolab.org>

EXPOSE 8080 9990
USER 0

# Install ant, I believe this includes ivy
RUN INSTALL_PKGS="ant" && \
    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y

#We need to replace the assemble script with one that works with Ivy
#and prepare /opt/hummingbird for artifacts from the build
COPY s2i_bin.patch /tmp/
RUN patch -d $STI_SCRIPTS_PATH -p2 < /tmp/s2i_bin.patch && \
    mkdir -p /opt/hummingbird/data/ && \
    chown -R 1001:0 /opt/hummingbird/data/ && \
    chmod -R ug+rwX /opt/hummingbird/data

USER 1001

CMD $STI_SCRIPTS_PATH/usage
