# = NAME
# Dockerfile  - sshd + sudo (no root password) container image
#
# = SYNOPSIS
# == Build image
# $ ln -s ~/.ssh/id_rsa.pub id_rsa.pub
# $ tar -czh . | docker build -t c7-ssh -  # "tar -h" for symlink copy
#
# == Example after build c7-ssh
# $ docker run -d -P --name c7-ssh c7-ssh
# $ docker port c7-ssh
# 22/tcp -> 0.0.0.0:NNNNN
# $ ssh yariscross@localhost -p NNNNN   # NNNNN is printed above
#
# = DESCRIPTION
# Based on the reference (see "SEE ALSO" section), the followings are changed:
# * never touch the root password; rather use ${user} + sudo
# * use centos6 as base image just for my needs.
#
# = SEE ALSO
# https://docs.docker.jp/engine/examples/running_ssh_service.html
# https://stackoverflow.com/a/21618386/921859:: reason why pam_loginuid is skipped
# https://stackoverflow.com/a/36312151/921859:: xVISIBLE example can be skipped

FROM centos:7
MAINTAINER ido-gh@wtech.jp

COPY CentOS-Base.repo /etc/yum.repos.d/
COPY epel.repo        /etc/yum.repos.d/
  
# change to your name
ARG user="yariscross"
ARG uid="1000"
ARG gid="1000"

RUN yum install -y \
        sudo openssh-server tar which wget && \
    yum clean all && \
    rm -rf /var/cache/yum/ && \
    : && \
    echo 'create user "${user}"' && \
    groupadd -g ${gid} ${user} && useradd -u ${uid} -g ${gid} ${user} && \
    echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/${user} && \
    sed -i "s/\(^Defaults.*secure_path.*=\).*/\1 \/sbin:\/bin:\/usr\/sbin:\/usr\/bin:\/usr\/local\/bin/" /etc/sudoers && \
    : && \
    echo 'SSH login fix. Otherwise user is kicked off after login' && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    : && \
    echo 'need to create public key for sshd' && \
    ssh-keygen -t rsa -N "" -f /etc/ssh/ssh_host_rsa_key && \
    : && \
    mkdir /home/${user}/.ssh && \
    chown ${user}:${user} /home/${user}/.ssh && \
    chmod 700 /home/${user}/.ssh

COPY id_rsa.pub /home/${user}/.ssh/authorized_keys

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
