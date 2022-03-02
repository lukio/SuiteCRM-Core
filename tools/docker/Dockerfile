# start from base
FROM debian:bullseye-slim
MAINTAINER Luciano Rossi <lukio@gcoop.coop>

WORKDIR "/tmp"

SHELL ["/bin/bash", "-c"]
# Instalamos paquetes necesarios
RUN apt-get -yqq update && \
    apt-get -yqq install php7.4 \
    php7.4-bcmath \
    php7.4-cli \
    php7.4-common \
    php7.4-curl \
    php7.4-gd \
    php7.4-gmp \
    php7.4-imap \
    php7.4-intl \
    php7.4-json \
    php7.4-mbstring \
    php7.4-mysql \
    php7.4-opcache \
    php7.4-readline \
    php7.4-soap \
    php7.4-xml \
    php7.4-zip \
    php-xdebug \
    procps \
    curl \
    build-essential \
    sudo \
    telnet \
    iproute2 \
    ssh \
    python \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

# Instalo última versión de composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === '$(wget -q -O - https://composer.github.io/installer.sig)') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php --2
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer


#Cambio TimeZone
RUN ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime

RUN sed -i "s|;date.timezone =|date.timezone=/America/Argentina/Buenos_Aires|g" /etc/php/7.4/cli/php.ini

RUN sed -i "s|;date.timezone =|/date.timezone=/America/Argentina/Buenos_Aires|g" /etc/php/7.4/apache2/php.ini

RUN echo "xdebug.var_display_max_depth = 10" >> /etc/php/7.4/cli/conf.d/20-xdebug.ini
RUN echo "xdebug.var_display_max_children = 256" >> /etc/php/7.4/cli/conf.d/20-xdebug.ini
RUN echo "xdebug.var_display_max_data = 1024" >> /etc/php/7.4/cli/conf.d/20-xdebug.ini

# Authorizo el host donde corre esta imagen
RUN mkdir -p /root/.ssh && \
    chmod 0700 /root/.ssh

# Seteamos un nuevo PATH
RUN echo "export PATH=\$PATH:\$GCA_DOCKER_DIRECTORY/vendor/bin" >> /root/.bashrc

# Creo el script para arrancar el servicio
RUN echo "#!/bin/bash -e" >> /root/run.sh
RUN echo "HOST_USER=\$1" >> /root/run.sh
RUN echo "IP_HOST=\$(ip route | grep default | awk '{print \$3}')" >> /root/run.sh
RUN echo "export XDEBUG_CONFIG=\"remote_host=\"\$IP_HOST\" remote_port=9000 remote_enable=on\"" >> /root/run.sh
RUN echo "rm -f /root/.ssh/config" >> /root/run.sh
RUN echo "rm -f /root/.ssh/known_hosts" >> /root/run.sh
RUN echo "echo 'Host elhost' >> /root/.ssh/config" >> /root/run.sh
RUN echo "echo '     IdentitiesOnly yes' >> /root/.ssh/config" >> /root/run.sh
RUN echo "echo '     IdentityFile ~/.ssh/id_rsa' >> /root/.ssh/config" >> /root/run.sh
RUN echo "echo '     Port 22' >> /root/.ssh/config" >> /root/run.sh
RUN echo "echo \"     User \$HOST_USER\" >> /root/.ssh/config" >> /root/run.sh
RUN echo "echo \"     Hostname \"\$IP_HOST\"\" >> /root/.ssh/config" >> /root/run.sh
RUN echo "ssh-keyscan \"\$IP_HOST\" > /root/.ssh/known_hosts" >> /root/run.sh
RUN echo "" >> /root/run.sh
RUN echo "" >> /root/run.sh
RUN echo "" >> /root/run.sh
RUN echo "" >> /root/run.sh
RUN echo "cd \$GCA_DOCKER_DIRECTORY" >> /root/run.sh
RUN echo "composer install" >> /root/run.sh
RUN echo "php -S 0.0.0.0:\$GCA_PORT_IN -t \$GCA_DOCKER_DIRECTORY/public" >> /root/run.sh
RUN chmod +x /root/run.sh

EXPOSE $GCA_PORT_IN
RUN mkdir "\$GCA_DOCKER_DIRECTORY"
STOPSIGNAL SIGTERM
CMD /root/run.sh $HOST_USER
