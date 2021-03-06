# this is a docker image for my thesis
FROM ubuntu
MAINTAINER Luis Tascon Gutierrez <luis.tascon@student.uclouvain.be>

# set the working directory inside the container
WORKDIR /usr/src/

# Install requirements
RUN apt-get update
RUN apt-get install -y python python-dev libffi-dev git libpcre3 libpcre3-dev libpcre++-dev libjpeg-dev tcpdump libcap2-bin build-essential libssl-dev 
RUN apt-get install -y python-pip python-pil python-bson python-gridfs python-chardet python-dpkt python-jinja2 python-magic python-pymongo python-libvirt python-bottle python-pefile ssdeep
RUN apt-get update
RUN apt-get install -y python-virtualenv python-setuptools zlib1g-dev

# Requirements for PostgreSQL as database:
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata
RUN apt-get install -y postgresql libpq-dev

# Requirements for Cuckoo monitor and adding hooks
RUN apt-get install -y mingw-w64 nasm
RUN pip install sphinx docutils pyyaml

# configuring tcpdump
RUN setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

# install pydeep from pip
RUN apt-get install -y libfuzzy-dev
RUN pip install pydeep

# install yara
WORKDIR /opt
RUN apt-get install -y autoconf libtool libjansson-dev libmagic-dev -y wget libssl1.0-dev
RUN wget https://github.com/plusvic/yara/archive/v3.4.0.tar.gz -O yara-3.4.0.tar.gz
RUN tar -zxf yara-3.4.0.tar.gz
WORKDIR /opt/yara-3.4.0
RUN ./bootstrap.sh
RUN ./configure --with-crypto --enable-cuckoo --enable-magic
RUN make
RUN make install
WORKDIR /opt/yara-3.4.0/yara-python
RUN python setup.py build
RUN python setup.py install

# installing volatility
WORKDIR /opt
RUN pip install openpyxl ujson pycrypto distorm3 pytz
RUN git clone https://github.com/volatilityfoundation/volatility.git
WORKDIR /opt/volatility
RUN python setup.py build
RUN python setup.py install

# installing M2Crypto
RUN apt-get install -y swig m2crypto

# installing guacd
RUN apt-get update
RUN apt-get install -y guacd

# Install VirtualBox
RUN apt-get install -y linux-headers-generic linux-headers-$(uname -r)
RUN apt-get update && apt-get install -y curl libcurl4 libvpx5 procps dbus
RUN rm -rf /var/lib/apt/lists/*
RUN buildDeps=' ca-certificates curl gnupg '
RUN set -x && mkdir -p /etc/xdg/QtProject
RUN apt-get update && apt-get install -y $buildDeps
RUN rm -rf /var/lib/apt/lists/*
RUN curl -sSL https://www.virtualbox.org/download/oracle_vbox_2016.asc | apt-key add -
RUN echo "deb http://download.virtualbox.org/virtualbox/debian buster contrib" >> /etc/apt/sources.list.d/virtualbox.list
RUN apt-get update && apt-get install -y virtualbox-5.2 --no-install-recommends
#RUN apt-get install -y sudo

# Install Virtualbox Extension Pack
# !!! Not able to install it as the driver is not mounted!! you have to install it once the container is runned and the drivers are mounted
WORKDIR /home/cuckoo
RUN VBOX_VERSION=`dpkg -s virtualbox-5.2 | grep '^Version: ' | sed -e 's/Version: \([0-9\.]*\)\-.*/\1/'` ; \
    wget http://download.virtualbox.org/virtualbox/${VBOX_VERSION}/Oracle_VM_VirtualBox_Extension_Pack-${VBOX_VERSION}.vbox-extpack ;
#    VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-${VBOX_VERSION}.vbox-extpack -y; \
#    rm Oracle_VM_VirtualBox_Extension_Pack-${VBOX_VERSION}.vbox-extpack

# install vmware
#WORKDIR /home
#RUN wget --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0" https://www.vmware.com/go/getplayer-linux 
#RUN chmod +x /home/getplayer-linux
#RUN /home/getplayer-linux --help	
#RUN /home/getplayer-linux --console --eulas-agreed --deferred-gtk 
#--set-setting vmware-player-app softwareUpdateEnabled no || /bin/true

# cuckoo
RUN adduser cuckoo --disabled-password --disabled-login
RUN usermod -G cuckoo -a cuckoo
RUN usermod -a -G vboxusers cuckoo
RUN apt-get install -y --no-install-recommends apt-utils && apt-get update
RUN apt-get install -y virt-manager
RUN addgroup libvirtd
RUN usermod -G libvirtd cuckoo
RUN pip install -U pip setuptools
RUN pip install --ignore-installed -U cuckoo 
RUN wget https://github.com/cuckoosandbox/community/archive/master.tar.gz

# install the cuckoo monitor 
# not needed if you only want to use last monitor version just use the command "cuckoo community"
WORKDIR /home/cuckoo
RUN git clone https://github.com/cuckoosandbox/monitor.git
WORKDIR /home/cuckoo/monitor
# if we don't run the line below there is a problem in compilation, the error has been corrected in a pull request but still not merged:
# https://github.com/cuckoosandbox/monitor/pull/71
RUN sed -i 's/last == insn\["funcname"\]/last == insn["module"] + insn["funcname"]/g' utils/process.py
RUN make

# In order to dump the network activity, we need a network sniffer -> tcpdump
RUN apt-get install -y tcpdump
RUN groupadd pcap
RUN usermod -a -G pcap cuckoo
RUN chgrp pcap /usr/sbin/tcpdump
RUN setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

# todo on the host because network is the one of the host
#RUN aa-disable /usr/sbin/tcpdump

WORKDIR /home/cuckoo
RUN mkdir malwares vms
RUN chown cuckoo:cuckoo malwares vms



# CMD is the instruction to run when a container starts; note that the previously-set environment variables can be used.

# if I want to expose a port number of the container: not really needed if network=host
# EXPOSE port_num

# to copy a file from my systeme in my image:
# COPY /path/to/filename/in/host /path/to/filename/in/image

# to define an entry point (first script launching at startup)
# ENTRYPOINT /path/to/script/in/image

# to set an environment variable in the container:
# ENV key value
# ENV CURRENT_DIR "/usr/src"
