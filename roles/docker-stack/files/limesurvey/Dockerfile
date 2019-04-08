FROM php:7-apache

ARG version

# Download, unzip and chown of LimeSurvey
ADD limesurvey$version.zip /tmp/lime.zip

# Install OS dependencies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends -y \
    libldap2-dev \
    zlib1g-dev \
    libc-client-dev \
    libkrb5-dev \
    libpng-dev \
    libpq-dev \
    libzip-dev \
    apt-utils \
    zip \
    unzip \
    make \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    netcat \
    iputils-ping \
    net-tools \
    telnet \
    && apt-get autoclean; apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*
 
# Link LDAP library for PHP ldap extension
RUN ln -fs /usr/lib/x86_64-linux-gnu/libldap.so /usr/lib/

# Configure PHP imap and gd plugins
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

# Install PHP Plugins
RUN docker-php-ext-install -j5 \
    gd \
    imap \
    ldap \
    mbstring \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    zip \
    iconv
    
# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
      echo 'opcache.memory_consumption=128'; \
      echo 'opcache.interned_strings_buffer=8'; \
      echo 'opcache.max_accelerated_files=4000'; \
      echo 'opcache.revalidate_freq=2'; \
      echo 'opcache.fast_shutdown=1'; \
      echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini
    
#Set PHP defaults for Limesurvey (allow bigger uploads)
RUN { \
      echo 'memory_limit=256M'; \
      echo 'upload_max_filesize=128M'; \
      echo 'post_max_size=128M'; \
      echo 'max_execution_time=120'; \
      echo 'max_input_vars=10000'; \
      echo 'date.timezone=UTC'; \
    } > /usr/local/etc/php/conf.d/uploads.ini


RUN unzip /tmp/lime.zip -d /tmp/ && \
    mv -f /tmp/limesurvey /tmp/html && \
    mv -f /tmp/html /var/www/ && \
    chown -R www-data:www-data /var/www/html


COPY entrypoint.sh /var/www/html/entrypoint.sh

RUN chmod +rwx /var/www/html/entrypoint.sh

ENTRYPOINT ["/var/www/html/entrypoint.sh"]
CMD ["apache2-foreground"]
