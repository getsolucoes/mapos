FROM php:8.3-fpm

# PHP extensions installer
COPY --from=ghcr.io/mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Install extension
RUN apt update && apt upgrade -y \
    && apt install -y \
    g++ \
    libbz2-dev \
    libc-client-dev \
    libcurl4-gnutls-dev \
    libedit-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libkrb5-dev \
    libldap2-dev \
    libldb-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libmemcached-dev \
    libpng-dev \
    libpq-dev \
    libsqlite3-dev \
    libssl-dev \
    libreadline-dev \
    libxslt1-dev \
    libzip-dev \
    memcached \
    wget \
    unzip \
    zlib1g-dev \
    && install-php-extensions bcmath \
    bz2 \
    calendar \
    exif \
    gettext \
    mysqli \
    opcache \
    pdo_mysql \
    xsl \
    gd \
    imap \
    intl \
    zip \
    sockets \
    xmlrpc \
    memcached \
    redis \
    imagick \
    && docker-php-source delete \
    && apt remove -y g++ wget \
    && apt autoremove --purge -y && apt autoclean -y && apt clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* /var/tmp/*

# Permission for PHP to write in volumes
RUN usermod -u 1000 www-data

# Install supervisor and cron
RUN apt update && apt install supervisor cron -y

# Setup cron jobs
RUN touch /var/log/cron.log
COPY cron-jobs /etc/cron.d/cron-jobs

# Give execution rights on the cron job
RUN chmod 0644 /etc/cron.d/cron-jobs

# Apply cron job
RUN crontab /etc/cron.d/cron-jobs

# Supervisor logs directory
RUN mkdir -p /var/log/supervisor

# Supervisor config file
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord"]
