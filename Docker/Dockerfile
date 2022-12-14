#
# Docker container-provisioning file for a container running a LAMP
# stack containing the XDMoD web application.
#
# Build
#
#   $ docker build --rm --tag local/xdmod:9.5.0 .
#
# Run
#
#   $ docker run --detach --restart unless-stopped --name XDMoD-Caviness \
#     --volume "$(pwd)/ingest-queue:/var/lib/XDMoD-ingest-queue:rw" \
#     --volume "$(pwd)/database:/var/lib/mysql:rw" \
#     --env CLUSTER_NAME="<cluster_name_for_xdmod-shredder>" \
#     --env RESOURCE_LOG_FORMAT="<resource_log_format_for_xdmod-shredder" \
#     --publish 8080:8080 \
#     local/xdmod:9.5.0
#

FROM    centos:7
LABEL   version="9.5.0" \
        description="XDMoD bundle containing database and web server" \
        maintainer="carlos@encs.concordia.ca"

#
# Install tools, libraries:
#
RUN     yum -y update && \
        yum -y install  epel-release && \
        yum -y install  sudo wget gmp-devel cronie logrotate \
                        coreutils shadow-utils tar xz bzip2 gzip zip jq \
                        libreoffice chromium-headless librsvg2-tools \
                        openssh-server openssh-clients 
RUN     set -o pipefail && echo "root:${ROOT_PASSWORD}" | chpasswd

#
# Install Apache and PHP:
#
RUN     yum -y install  httpd php php-cli php-gd php-gmp \
                        php-pdo php-xml php-pear-MDB2 php-pear-MDB2-Driver-mysql \
                        php-mbstring php-pecl-apcu perl-Image-ExifTool

#
# Install MySQL:
#
RUN     yum -y install  mariadb-server mariadb

#
# Install XDMoD:
#
RUN     yum -y install https://github.com/ubccr/xdmod/releases/download/v9.5.0/xdmod-9.5.0-1.0.el7.noarch.rpm

#
# Install Munge:
#
RUN     groupadd -g 966 munge
RUN     useradd  -m -c "MUNGE Uid 'N' Gid Emporium" -d /var/lib/munge -u 966 -g munge  -s /sbin/nologin munge
RUN     yum -y install munge munge-libs munge-devel

#
# XDMoD uses an Apache virtual host on port 8080:
#
EXPOSE  8080/tcp

#
# Ensure a mountpoint/directory exists for the data ingest pipeline:
#
RUN     mkdir --parents --mode=0755 /var/lib/XDMoD-ingest-queue/in
RUN     mkdir --parents --mode=0755 /var/lib/XDMoD-ingest-queue/out
RUN     mkdir --parents --mode=0755 /var/lib/XDMoD-ingest-queue/error

#
# Command to execute when container is started:
#
RUN     chown -R munge:munge /etc/munge/ /var/log/munge/
RUN     chmod 0700 /etc/munge/ /var/log/munge/
RUN     /usr/sbin/munged
COPY    --chown=0:0 XDMoD-start /usr/sbin/XDMoD-start
RUN     chmod 0755 /usr/sbin/XDMoD-start
CMD     /usr/sbin/XDMoD-start
