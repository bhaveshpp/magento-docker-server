FROM php:7.3-fpm
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin
RUN chmod +x /usr/local/bin/install-php-extensions
RUN install-php-extensions mysqli mcrypt bcmath curl intl json mbstring pdo_mysql readline session soap zip gd
RUN install-php-extensions xsl sockets
RUN apt-get update && apt-get upgrade -y
RUN apt install git -y
RUN echo 'export PS1="[phpfpm73 \w] \$ "' >> ~/.bashrc