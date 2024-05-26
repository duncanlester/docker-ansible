FROM centos:8
LABEL maintainer="Jeff Geerling"
ENV container=docker

ENV pip_packages "ansible yamllint ansible-lint flake8 testinfra molecule docker"

# Install systemd -- See https://hub.docker.com/_/centos/
RUN yum -y update; yum clean all; \
  (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
  rm -f /lib/systemd/system/multi-user.target.wants/*;\
  rm -f /etc/systemd/system/*.wants/*;\
  rm -f /lib/systemd/system/local-fs.target.wants/*; \
  rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
  rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
  rm -f /lib/systemd/system/basic.target.wants/*;\
  rm -f /lib/systemd/system/anaconda.target.wants/*;

RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

# Install requirements.
RUN yum makecache --timer \
  && yum -y install epel-release initscripts \
  && yum -y update \
  && yum -y install \
  sudo \
  which \
  hostname \
  wget \
  python3-pip \
  python3-devel \
  @development \
  && yum clean all

# # # # # # pyenv setup # # # # # # # #
RUN curl https://pyenv.run | bash
RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
RUN echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc
RUN exec "$SHELL"
RUN yum install -y gcc make patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
RUN $HOME/.pyenv/bin/pyenv install 3.9
RUN $HOME/.pyenv/bin/pyenv global 3.9.19
RUN exec "$SHELL"

######### venv ##############
ENV VIRTUAL_ENV=/home/redis-venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

######### docker ###############
RUN yum -y install yum-utils
RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
RUN yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Disable requiretty.
RUN sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \1/'  /etc/sudoers

# Install Ansible inventory file.
RUN mkdir -p /etc/ansible
RUN echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts

VOLUME ["/sys/fs/cgroup"]
CMD ["/usr/lib/systemd/systemd"]

# # ENV PYTHONHOME "${PYTHONHOME}:/your/custom/path"
# # yum install gcc make patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
# # RUN yum -y install \
# #   python3-pip \
# #   python3-devel \
# #   && yum clean all

# RUN curl https://pyenv.run | bash
# RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
# RUN echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
# RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# RUN $HOME/.pyenv/bin/pyenv install 3.9
# RUN $HOME/.pyenv/bin/pyenv global 3.9.19

# # Install Ansible via Pip.
# # RUN pip3 install --user $pip_packages3

# # Disable requiretty.
# RUN sed -i -e 's/^\(Defaults\s*requiretty\)/#--- \\/'  /etc/sudoers

# # Install Ansible inventory file.
# RUN mkdir -p /etc/ansible
# RUN echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts

# # RUN echo $PATH
# # RUN echo "export PATH=$PATH:~/.local/bin" >> ~/.bashrc
# # RUN cat ~/.bashrc
# # RUN source ~/.bashrc
# # RUN echo $PATH
# ln -s $PYTHONHOME/lib64/python3.10.11/lib-dynload/ $PYTHONHOME/lib/python3.10.11/lib-dynload


# RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# RUN yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# VOLUME ["/sys/fs/cgroup"]
# CMD ["/usr/lib/systemd/systemd"]
