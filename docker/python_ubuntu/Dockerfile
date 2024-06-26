FROM python:3
USER root

RUN apt-get update
RUN apt-get -y install locales-all && apt-get -y install locales nano vim less sudo default-jdk

RUN pip install --upgrade pip \
  && pip install --upgrade setuptools

RUN git clone https://github.com/edenhill/librdkafka.git
WORKDIR librdkafka
RUN ./configure --prefix /usr --install-deps
RUN make
RUN make install

# Add user
ARG USERNAME=pyuser
ARG GROUPNAME=pyuser
ARG PASSWORD=pyuser
ARG UID=1001
ARG GID=1001
RUN groupadd -g $GID $GROUPNAME && \
  useradd -m -s /bin/bash -u $UID -g $GID -G sudo $USERNAME && \
  echo $USERNAME:$PASSWORD | chpasswd && \
  echo "$USERNAME   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Install SSH server
RUN apt-get update
RUN apt-get install -y --no-install-recommends openssh-server
RUN rm -rf /var/lib/apt/lists/*
RUN sed -i "s/#PermitRootLogin prohibit-password/PermitRootLogin yes/" /etc/ssh/sshd_config
RUN ssh-keygen -A
RUN service ssh start


USER $USERNAME
WORKDIR /home/$USERNAME
RUN mkdir /home/$USERNAME/.ssh
COPY authorized_keys /home/$USERNAME/.ssh/
RUN sudo chmod 700 /home/$USERNAME/.ssh
RUN sudo chmod 644 /home/$USERNAME/.ssh/authorized_keys

RUN mkdir /home/$USERNAME/src
COPY init.sh /home/$USERNAME/
COPY init_bashrc /home/$USERNAME/
COPY requirements.txt /home/$USERNAME/src/

RUN echo "sh ~/init.sh" >> /home/$USERNAME/.profile

RUN wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz
RUN tar xvf kafka_2.13-3.7.0.tgz
RUN sudo mv kafka_2.13-3.7.0 /usr/local/kafka

EXPOSE 22
CMD ["sudo", "/usr/sbin/sshd", "-D"]