FROM php:8.0-apache

ARG OPENDCIM_VERSION=23.04

# ------------------------------------------------------------
# 1) OS packages + PHP extensions
# ------------------------------------------------------------
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    curl ca-certificates \
    libzip-dev \
    libicu-dev \
    libpng-dev libjpeg-dev libfreetype6-dev \
    ipmitool \
    freeipmi-tools \
    snmp \
    snmpd \
  ; \
\
  docker-php-ext-configure gd --with-freetype --with-jpeg; \
  docker-php-ext-install \
    mysqli pdo_mysql zip gd intl gettext \
  ; \
  a2enmod rewrite headers; \
  rm -rf /var/lib/apt/lists/*

# ------------------------------------------------------------
# 2) Apache: allow .htaccess
# ------------------------------------------------------------
RUN set -eux; \
  sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

WORKDIR /var/www/html

# ------------------------------------------------------------
# 3) Download and install openDCIM
#    - Add retry & timeouts to survive opendcim.org intermittent network issues
# ------------------------------------------------------------
RUN set -eux; \
  curl -fL \
    --retry 6 \
    --retry-all-errors \
    --connect-timeout 10 \
    --max-time 180 \
    -o /tmp/opendcim.tar.gz \
    "https://opendcim.org/packages/openDCIM-${OPENDCIM_VERSION}.tar.gz"; \
  tar -xzf /tmp/opendcim.tar.gz -C /tmp; \
  rm -rf /var/www/html/*; \
  mv "/tmp/openDCIM-${OPENDCIM_VERSION}/"* /var/www/html/; \
  chown -R www-data:www-data /var/www/html; \
  rm -rf /tmp/opendcim.tar.gz "/tmp/openDCIM-${OPENDCIM_VERSION}"

# ------------------------------------------------------------
# 4) Compatibility patch:
#    configuration.php: DateTime(null, $z) -> DateTime('now', $z)
# ------------------------------------------------------------
RUN set -eux; \
  if grep -q 'new DateTime(null, $z)' /var/www/html/configuration.php; then \
    sed -i 's/new DateTime(null, $z)/new DateTime('\''now'\'', $z)/' /var/www/html/configuration.php; \
  fi

# ------------------------------------------------------------
# 5) PHP runtime tuning
# ------------------------------------------------------------
RUN set -eux; \
  { \
    echo "display_errors=Off"; \
    echo "log_errors=On"; \
    echo "error_reporting=E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED"; \
  } > /usr/local/etc/php/conf.d/opendcim-runtime.ini

COPY opendcim-auth.conf /etc/apache2/sites-available/opendcim-auth.conf

RUN set -eux; \
  a2dissite 000-default.conf; \
  a2ensite opendcim-auth.conf; \
  apache2ctl -t
