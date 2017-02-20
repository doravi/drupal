# from https://www.drupal.org/requirements/php#drupalversions
FROM php:7.0-apache

RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng12-dev libjpeg-dev libpq-dev git-all \
	&& rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
	&& docker-php-ext-install gd mbstring pdo pdo_mysql pdo_pgsql zip

# Make ssh dir
RUN mkdir -p /root/.ssh
RUN chown -R root:root /root/.ssh
RUN chmod 700 /root/.ssh


# Create known_hosts
RUN touch /root/.ssh/known_hosts
# Add github key
RUN ssh-keyscan -t rsa github.com > ~/.ssh/known_hosts

WORKDIR /var/www/html

# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 7.53
ENV DRUPAL_MD5 4230279ecca4f0cde652a219e10327e7

RUN curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz \
	&& echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c - \
	&& tar -xz --strip-components=1 -f drupal.tar.gz \
	&& rm drupal.tar.gz \
	&& chown -R www-data:www-data sites
	
# Clone the conf files into the docker container
RUN git clone https://github.com/doravi/drupal7config.git temp \
	&& cp -r temp/* /var/www/html \
	&& rm -rf temp
	
# Download latest stable release using the code below or browse to github.com/drush-ops/drush/releases.
RUN php -r "readfile('https://s3.amazonaws.com/files.drush.org/drush.phar');" > drush
# Make `drush` executable as a command from anywhere. Destination can be anywhere on $PATH.
RUN chmod +x drush
RUN mv drush /usr/local/bin
# Optional. Enrich the bash startup file with completion and aliases.
RUN drush init -y
#Drupal wizard installation
RUN cd /var/www/html
RUN drush site-install standard --account-name=admin --account-pass=Gigya123 --db-url=sqlite://sites/example.com/files/.ht.sqlite -y
RUN chmod 777 -R /var/www/html
