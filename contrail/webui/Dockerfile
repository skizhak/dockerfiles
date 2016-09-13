FROM ubuntu:14.04

RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# Install Required dependencies
RUN apt-get update && apt-get install -y \
  build-essential \
  wget \
  python-lxml \
  redis-server

# Install NVM
RUN wget -O - https://raw.githubusercontent.com/creationix/nvm/v0.16.1/install.sh | sh
RUN source $HOME/.profile 

# Install Nodejs
RUN nvm install 0.10.35
RUN nvm use 0.10.35

# Create a new Rails app under /src/my-app
RUN mkdir -p /sandbox && cd /sandbox 
#WORKDIR /src/my-app

ARG branch=master
#Clone Juniper repos
RUN git clone -b $branch https://github.com/Juniper/contrail-web-core.git
RUN git clone -b $branch https://github.com/Juniper/contrail-webui-third-party.git
RUN git clone -b $branch https://github.com/Juniper/contrail-web-controller.git
RUN git clone -b $branch https://github.com/Juniper/contrail-web-server-manager.git
RUN git clone -b $branch  https://github.com/Juniper/contrail-web-storage.git

#Start building the web repo
ARG env=prod
RUN cd contrail-web-core
RUN make fetch-pkgs-$env

ARG contrail_ip='127.0.0.1'
ARG config_file=config/config.global.js
# Update the config
RUN sed -i -e s/\(config.networkManager.ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.imageManager.ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.computeManager.ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.identityManager.ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.storageManager.ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.cnfg.server_ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.analytics.server_ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.vcenter.server_ip = \)[^;]*;/\1 ${contrail_ip};/g \
              s/\(config.cassandra.server_ips = \)[^;]*;/\1 [${contrail_ip}];/g \
              $config_file

RUN sed -i -e "s/\(config.featurePkg.webController.path = \)[^;]*;/\1'\/sandbox\/contrail-web-controller';/" \
           -e /config.featurePkg.webController.enable = true;/a \n\rconfig.featurePkg.serverManager = {};\n\r \
           config.featurePkg.serverManager.path = \'\/sandbox\/contrail-web-servermanager\';\n\r \
           config.featurePkg.serverManager.enable = true;\n\r \
           config.featurePkg.webStorage = {};\n\r \
           config.featurePkg.webStorage.path = \'\/sandbox\/contrail-web-storage\';\n\r \
           config.featurePkg.webStorage.enable = true;\n\r \
           $config_file

# Default command is to run a rails server on port 3000
#CMD ["rails", "server", "--binding", "0.0.0.0", "--port", "3000"]
#EXPOSE 8080
