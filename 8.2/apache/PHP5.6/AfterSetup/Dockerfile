FROM php:5.6-apache

RUN a2enmod rewrite

MAINTAINER Reload A/S <kontakt@reload.dk>

ENV PATH /root/.composer/vendor/bin:$PATH
ENV PHP_VERSION 5.6

RUN \
  apt-get update && \
  # Install packages needed to enable an extra repository.
  DEBIAN_FRONTEND=noninteractive apt-get -y install python-software-properties && \
  # Add a repo that contains php ${PHP_VERSION}
  LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php && \
  # Do the remaining installation of packages.
  apt-get update && \
  DEBIAN_FRONTEND=noninteractive \
    apt-get -y install \
      apache2 \
      # Drush needs this to work
      mysql-client \
      libapache2-mod-php${PHP_VERSION} \
      # Default extensions.
      php${PHP_VERSION}-curl \
      php${PHP_VERSION}-gd \
      php${PHP_VERSION}-mysql \
      php${PHP_VERSION}-xml \
      php${PHP_VERSION}-xdebug \
      php${PHP_VERSION}-xhprof \
      php${PHP_VERSION}-mbstring \
      php${PHP_VERSION}-mcrypt \
      php${PHP_VERSION}-soap \
      # Added for installing composer.
      php${PHP_VERSION}-zip \
      # Other extensions
      php${PHP_VERSION}-intl \
      php-memcache \
      php-memcached \
      # For default snakeoil certificates which SSL is configuered to use
      # per default in Apache.
      ssl-cert \
  && \
  a2enmod rewrite && \
  a2enmod ssl && \
  a2ensite default-ssl && \
  a2enconf drupal && \
  phpenmod drupal-recommended && \
  phpenmod xdebug && \
  # Drush 8 is the current stable that supports Drupal version 6, 7 and 8.
  curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer && \
  composer global require drush/drush:8.* && \
  apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng12-dev libjpeg-dev libpq-dev git-all wget curl php5-cli git \
	&& rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
	&& docker-php-ext-install gd mbstring opcache pdo pdo_mysql pdo_pgsql zip
	
# Make ssh dir
RUN mkdir -p /root/.ssh
RUN chown -R root:root /root/.ssh
RUN chmod 700 /root/.ssh


# Create known_hosts
RUN touch /root/.ssh/known_hosts
# Add github key
RUN ssh-keyscan -t rsa github.com > ~/.ssh/known_hosts

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

WORKDIR /var/www/html

# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 8.2.5
ENV DRUPAL_MD5 4bfd35c34ce1b4ba5bac2d18e0d6b5c9

RUN curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz \
	&& echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c - \
	&& tar -xz --strip-components=1 -f drupal.tar.gz \
	&& rm drupal.tar.gz \
	&& chown -R www-data:www-data sites modules themes

# Clone the conf files into the docker container
RUN git clone https://github.com/doravi/Drupal8AfterWizard.git temp \
	&& cp -r temp/* /var/www/html \
	&& rm -rf temp
	
RUN chmod 777 -R /var/www/html

ENV PHP_DEFAULT_EXTENSIONS calendar ctype curl dom exif fileinfo ftp gd gettext iconv json mcrypt mysql mysqli mysqlnd opcache pdo pdo_mysql phar posix readline shmop simplexml soap sockets sysvmsg sysvsem sysvshm tokenizer wddx xdebug xhprof xml xmlreader xmlwriter xsl mbstring zip

EXPOSE 80 443
