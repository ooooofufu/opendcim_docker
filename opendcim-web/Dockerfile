FROM php:8.2-apache

ARG OPENDCIM_VERSION=23.04

# OS packages and PHP extensions
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates \
    libzip-dev \
    libicu-dev \
    libpng-dev libjpeg-dev libfreetype6-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install \
      mysqli pdo_mysql zip gd intl gettext \
  && a2enmod rewrite headers \
  && rm -rf /var/lib/apt/lists/*

# allowe .htaccess（openDCIM used oftenly）
RUN sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

WORKDIR /var/www/html

# download openDCIM officially release to web root
RUN curl -fsSL -o /tmp/opendcim.tar.gz "https://opendcim.org/packages/openDCIM-${OPENDCIM_VERSION}.tar.gz" \
  && tar -xzf /tmp/opendcim.tar.gz -C /tmp \
  && rm -rf /var/www/html/* \
  && mv "/tmp/openDCIM-${OPENDCIM_VERSION}/"* /var/www/html/ \
  && chown -R www-data:www-data /var/www/html \
  && rm -rf /tmp/opendcim.tar.gz "/tmp/openDCIM-${OPENDCIM_VERSION}"
