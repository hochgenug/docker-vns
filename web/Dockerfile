FROM php:7.2-apache

LABEL maintainer="hochgenug@gmail.com"

# Install some extensions
RUN apt-get update \
    && apt-get install -y \
        autoconf \
        curl \
        git \
        libicu-dev \
        libfreetype6-dev \
        libjpeg-dev \
        libmcrypt-dev \
        libpng-dev \
        libtidy-dev \
        libxml2-dev \
        wget \
        zip \
        awscli \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) \
        gd \
        intl \
        opcache \
        pdo_mysql \
        soap \
        tidy \
        zip \
    && yes | pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && rm -rf /var/lib/apt/lists/*


# Add Composer globally
RUN curl -s https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer global require "fxp/composer-asset-plugin:~1.2"

# Enable custom virtual hosts
COPY vhosts /etc/apache2/sites-available
RUN echo 'ServerName localhost' >> /etc/apache2/apache2.conf \
    && a2ensite $(find /etc/apache2/sites-available/ -type f -name '*.conf' -not -name '*default*' -printf '%f\n') \
    && a2enmod rewrite \
    && usermod -u 1000 www-data \
    && ln -sf /etc/apache2/mods-available/headers.load /etc/apache2/mods-enabled/headers.load \
    && ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

# Set up sendmail config, see http://linux.die.net/man/5/ssmtp.conf for options
RUN mkdir /etc/ssmtp
RUN echo "hostname=localhost.localdomain" > /etc/ssmtp/ssmtp.conf
RUN echo "root=your@gmail.com" >> /etc/ssmtp/ssmtp.conf
RUN echo "mailhub=maildev" >> /etc/ssmtp/ssmtp.con

RUN echo "localhost localhost.localdomain" >> /etc/hosts

# Add custom PHP configuration
COPY custom.ini /usr/local/etc/php/conf.d/

# Add the certificat
COPY cacert.pem /etc/ssl/certs
