FROM centos

RUN yum update-minimal --security -y && \

  yum install -y \
  nginx \
  arpwatch \
  bind-utils \
  curl \
  dropwatch \
  iperf3 \
  mtr \
  nano \
  net-tools \
  nmap \
  strace \
  sysstat \
  tcpdump \
  tcptraceroute \
  telnet \
  tmux \
  traceroute \
  vim \
  wireshark \
  wget \
  openssh-clients \
  && yum clean all && rm -rf /var/cache/yum /tmp/* /var/tmp/*

RUN  sysctl -w net.ipv4.ip_forward=1

RUN touch /run/nginx.pid &&chgrp -R 0 /run/* && chmod 770 -R /run/* && ls -al /run/nginx.pid 

RUN curl -s https://raw.githubusercontent.com/jedrecord/sysinfo/master/sysinfo \
  -o /usr/bin/sysinfo && chmod 755 /usr/bin/sysinfo

COPY nginx.conf /etc/nginx/nginx.conf

CMD [ "nginx", "-g", "daemon off; " ]