FROM ubuntu:18.04
LABEL maintainer="Jeff Geerling"

ARG DEBIAN_FRONTEND=noninteractive

# Fix for https://github.com/pypa/pip/issues/10219
ARG LANG="en_US.UTF-8"
ARG LC_ALL="en_US.UTF-8"
ENV PYTHON_VERSION 3.12.3
ENV VENV_NAME venv-${PYTHON_VERSION}
ENV pip_packages "ansible yamllint ansible-lint flake8 molecule-docker"

# Install dependencies.
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  apt-utils \
  build-essential \
  locales \
  libffi-dev \
  libssl-dev \
  libyaml-dev \
  python3-dev \
  python3-setuptools \
  python3-pip \
  python3-yaml \
  software-properties-common \
  rsyslog systemd systemd-cron sudo iproute2 \
  && apt-get clean \
  && rm -Rf /var/lib/apt/lists/* \
  && rm -Rf /usr/share/doc && rm -Rf /usr/share/man
RUN sed -i 's/^\($ModLoad imklog\)/#\1/' /etc/rsyslog.conf

#Set of all dependencies needed for pyenv to work on Ubuntu
RUN apt-get update \
  && apt-get install -y --no-install-recommends make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget ca-certificates curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev mecab-ipadic-utf8 git

# Set-up necessary Env vars for PyEnv
ENV PYENV_ROOT /root/.pyenv
ENV PATH $PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH

# Install pyenv
RUN set -ex \
  && curl https://pyenv.run | bash \
  && echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc \
  && echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc \
  && echo 'eval "$(pyenv init -)"' >> ~/.bashrc \
  && echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc \
  && pyenv update \
  && pyenv install $PYTHON_VERSION \
  && pyenv global $PYTHON_VERSION \
  && pyenv virtualenv 3.12.3 ${VENV_NAME} \
  && exec bash \
  # && $PYENV_ROOT/bin/pyenv activate ${VENV_NAME} \
  # && pip install --upgrade pip \
  # && pip install $pip_packages \
  # && source deactivate \
  && pyenv rehash
# RUN 'eval "$(pyenv init -)"' && $HOME/.pyenv/bin/pyenv activate ${container}-${pythonversion}

# Upgrade pip to latest version.
# RUN pip3 install --upgrade pip

# Fix potential UTF-8 errors with ansible-test.
RUN locale-gen en_US.UTF-8

# Install Ansible via Pip.
# RUN pip3 install $pip_packages

COPY initctl_faker .
RUN chmod +x initctl_faker && rm -fr /sbin/initctl && ln -s /initctl_faker /sbin/initctl

# Install Ansible inventory file.
RUN mkdir -p /etc/ansible
RUN echo "[local]\nlocalhost ansible_connection=local" > /etc/ansible/hosts

# Remove unnecessary getty and udev targets that result in high CPU usage when using
# multiple containers with Molecule (https://github.com/ansible/molecule/issues/1104)
RUN rm -f /lib/systemd/system/systemd*udev* \
  && rm -f /lib/systemd/system/getty.target

######### docker ###############
RUN apt-get update
RUN apt install -y docker.io

VOLUME ["/sys/fs/cgroup", "/tmp", "/run"]
CMD ["/lib/systemd/systemd"]

# ENV pip_packages "ansible yamllint ansible-lint flake8 testinfra molecule"

# FROM debian:bullseye
# LABEL maintainer="Jeff Geerling"

# ARG DEBIAN_FRONTEND=noninteractive
# ENV PYTHON_VERSION=3.12.3
# ENV pip_packages "ansible cryptography yamllint ansible-lint flake8 pytest testinfra molecule molecule-docker"

# # Install dependencies.
# RUN apt-get update \
#   && apt-get install -y --no-install-recommends \
#   sudo systemd systemd-sysv \
#   build-essential wget libffi-dev libssl-dev procps \
#   python3-pip python3-dev python3-setuptools python3-wheel python3-apt \
#   iproute2 \
#   && rm -rf /var/lib/apt/lists/* \
#   && rm -Rf /usr/share/doc && rm -Rf /usr/share/man \
#   && apt-get clean

# # # # # # # pyenv setup # # # # # # # #
# RUN curl https://pyenv.run | bash
# RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
# RUN echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
# RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc
# RUN echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
# RUN yum install -y gcc make patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
# RUN $HOME/.pyenv/bin/pyenv install ${pythonversion}
# RUN $HOME/.pyenv/bin/pyenv global ${pythonversion}
# RUN $HOME/.pyenv/bin/pyenv virtualenv 3.12.3 ${container}-${pythonversion}
# # RUN 'eval "$(pyenv init -)"' && $HOME/.pyenv/bin/pyenv activate ${container}-${pythonversion}

# # Install Ansible via pip.
# RUN pip3 install $pip_packages

# COPY initctl_faker .
# RUN chmod +x initctl_faker && rm -fr /sbin/initctl && ln -s /initctl_faker /sbin/initctl

# # ######### docker ###############
# # RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# # RUN yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# # Install Ansible inventory file.
# RUN mkdir -p /etc/ansible
# RUN echo "[local]\nlocalhost ansible_connection=local" > /etc/ansible/hosts

# # Make sure systemd doesn't start agettys on tty[1-6].
# RUN rm -f /lib/systemd/system/multi-user.target.wants/getty.target

# VOLUME ["/sys/fs/cgroup"]
# CMD ["/lib/systemd/systemd"]

# FROM centos:8
# LABEL maintainer="Jeff Geerling"
# ENV container=docker
# ENV pythonversion=3.12.3
# ENV pip_packages "ansible yamllint ansible-lint flake8 pytest-testinfra pytest testinfra molecule molecule-docker"

# # Install systemd -- See https://hub.docker.com/_/centos/
# RUN yum -y update; yum clean all; \
#   (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
#   rm -f /lib/systemd/system/multi-user.target.wants/*;\
#   rm -f /etc/systemd/system/*.wants/*;\
#   rm -f /lib/systemd/system/local-fs.target.wants/*; \
#   rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
#   rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
#   rm -f /lib/systemd/system/basic.target.wants/*;\
#   rm -f /lib/systemd/system/anaconda.target.wants/*;

# RUN cd /etc/yum.repos.d/
# RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
# RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

# # Install requirements.
# RUN yum makecache --timer \
#   && yum -y install epel-release initscripts \
#   && yum -y update \
#   && yum -y install \
#   yum-utils \
#   sudo \
#   which \
#   hostname \
#   @development \
#   && yum clean all

# # # # # # # pyenv setup # # # # # # # #
# RUN curl https://pyenv.run | bash
# RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
# RUN echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
# RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc
# RUN echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
# RUN yum install -y gcc make patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
# RUN $HOME/.pyenv/bin/pyenv install ${pythonversion}
# RUN $HOME/.pyenv/bin/pyenv global ${pythonversion}
# RUN $HOME/.pyenv/bin/pyenv virtualenv 3.12.3 ${container}-${pythonversion}
# # RUN 'eval "$(pyenv init -)"' && $HOME/.pyenv/bin/pyenv activate ${container}-${pythonversion}
# # pip install $pip_packages

# ######### docker ###############
# RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# RUN yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# # Disable requiretty.
# RUN sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers

# # Install Ansible inventory file.
# RUN mkdir -p /etc/ansible
# RUN echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts

# VOLUME ["/sys/fs/cgroup"]
# CMD ["/usr/lib/systemd/systemd"]
