ARG PHP_FPM_VERSION=7
FROM php:${PHP_FPM_VERSION}-fpm

ARG USER_ID
ARG GROUP_ID

RUN if [ ${USER_ID:-0} -ne 0 ] && [ ${GROUP_ID:-0} -ne 0 ]; then \
    userdel -f www-data &&\
    if getent group www-data ; then groupdel www-data; fi &&\
    groupadd -g ${GROUP_ID} www-data &&\
    useradd -l -u ${USER_ID} -g www-data www-data &&\
    install -d -m 0755 -o www-data -g www-data /home/www-data &&\
    chown --changes --silent --no-dereference --recursive \
          --from=33:33 ${USER_ID}:${GROUP_ID} \
        /home/www-data \
;fi

WORKDIR /var/www

RUN apt-get update -yqq \
  && apt-get install -y \
    libicu-dev \
    libzip-dev \
    zip \
    less \
    vim \
    iputils-ping \
  && docker-php-ext-configure intl \
  && docker-php-ext-install intl \
  && docker-php-ext-configure zip \
  && docker-php-ext-install zip \
  && docker-php-ext-install pdo_mysql \
  && docker-php-ext-install bcmath

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    printf "\n" | pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis \
;fi

###########################################################################
# PHP GD EXTENSION
###########################################################################

ARG INSTALL_GD=false

RUN if [ ${INSTALL_GD} = true ]; then \
  apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
;fi

ARG INSTALL_PCNTL=false

RUN if [ ${INSTALL_PCNTL} = true ]; then \
  docker-php-ext-install pcntl \
;fi

###########################################################################
# Composer:
###########################################################################

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

###########################################################################
# NodeJS & NPM:
###########################################################################

ARG INSTALL_NODE=false

RUN if [ ${INSTALL_NODE} = true ]; then \
  curl -O https://deb.nodesource.com/setup_12.x \
    && chmod +x setup_12.x \
    && ./setup_12.x \
    && apt-get install -y nodejs \
    && apt-get install -y build-essential \
    && apt-get install -y git \
    && rm setup_12.x \
;fi

# Lower TLS minimum protocol
RUN sed -i 's,^\(MinProtocol[ ]*=\).*,\1'TLSv1.0',g' /etc/ssl/openssl.cnf

USER www-data
