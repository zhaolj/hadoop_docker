FROM ubuntu:latest

ARG user=hadoop
ARG passwd=hadoop

ENV TZ="Asia/Shanghai"
ENV LANG=C.UTF-8

RUN apt-get update && apt-get -y upgrade \
    && apt-get install -y openssh-server locales sudo \
    && locale-gen en_US.UTF-8 \
    && apt-get install -y wget ca-certificates \
    && apt-get install -y openjdk-11-jdk \
    && apt-get install -y vim net-tools zsh git \
    && apt-get install -y inetutils-ping iproute2 curl \
    && apt clean \
    && rm -rf /tmp/* /var/lib/apt/lists/* /var/tmp* \
    # && echo "PermitRootLogin yes" >> /etc/ssh/sshd_config \
    && mkdir -p /var/run/sshd

RUN echo 'root:pw123456' | chpasswd  
RUN useradd -m ${user} -s /bin/zsh \
    && adduser ${user} sudo \
    && echo "${user}:${passwd}" | chpasswd

RUN echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

#RUN usermod -u 1000 ${user} && usermod -G 1000 ${user}
RUN mkdir -p /home/${user}/.ssh \
    && ssh-keygen -q -t rsa -b 2048 -f /home/${user}/.ssh/id_rsa -N '' \
    && ssh-keygen -q -t ecdsa -f /home/${user}/.ssh/id_ecdsa -N '' \
    && ssh-keygen -t dsa -f /home/${user}/.ssh/id_ed25519 -N '' \
    && cat /home/${user}/.ssh/id_rsa.pub > /home/${user}/.ssh/authorized_keys \
    && chown -R ${user}:${user} /home/${user}/.ssh \
    && chmod 700 /home/${user}/.ssh

RUN cd /opt \
    && wget --tries=3 "https://archive.apache.org/dist/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz" \
    && tar -xzvf hadoop-3.3.4.tar.gz \
    && mv hadoop-3.3.4 hadoop \
    && chown -R ${user}:${user} /opt/hadoop \
    && rm /opt/hadoop-3.3.4.tar.gz

RUN sed -i '$a export HADOOP_HOME=/opt/hadoop' /etc/bash.bashrc \
    && sed -i '$a export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' /etc/bash.bashrc

RUN mkdir /home/${user}/build \
    && chown -R ${user}:${user} /home/${user}/build
#RUN cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
#RUN sed -i '$aPermitRootLogin yes' /etc/ssh/sshd_config
RUN sed -i 's/AcceptEnv LANG LC_\*/AcceptEnv no/' /etc/ssh/sshd_config
VOLUME /home/${user}/build
EXPOSE 22 80 443 9870
WORKDIR /home/${user}/build
USER ${user}

RUN touch ~/.zshrc
RUN sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)" \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    && sed -i '/^# HIST_STAMPS="mm\/dd\/yyyy"/a HIST_STAMPS="yyyy-mm-dd"' ~/.zshrc \
    && sed -i 's/^ZSH_THEME="robbyrussell"/# &/' ~/.zshrc \
    && sed -i '/^# ZSH_THEME="robbyrussell"/a ZSH_THEME="ys"' ~/.zshrc \
    && sed -i 's/^plugins=(.*$/# &/' ~/.zshrc \
    && sed -i '/^# plugins=(.*$/a plugins=(git z zsh-autosuggestions zsh-syntax-highlighting)' ~/.zshrc

RUN git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime \
    && sh ~/.vim_runtime/install_awesome_vimrc.sh

RUN sed -i '$a export HADOOP_HOME=/opt/hadoop' ~/.zshrc \
    && sed -i '$a export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' ~/.zshrc \
    && sed -i '$a export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' /opt/hadoop/etc/hadoop/hadoop-env.sh \
    && sed -i '$a export HADOOP_HOME=/opt/hadoop' /opt/hadoop/etc/hadoop/hadoop-env.sh

#ENTRYPOINT [ "/bin/bash", "-l", "-c"]
CMD ["sudo", "/usr/sbin/sshd", "-D"]