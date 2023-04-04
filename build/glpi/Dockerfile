FROM php:8.1-apache-bullseye

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions \
	&& install-php-extensions gd xdebug mysqli intl
	
# Ajoutez des extensions ici
RUN install-php-extensions zip bz2

# Changez la version de glpi ici
ENV VERSION=10.0.6

ADD https://github.com/glpi-project/glpi/releases/download/$VERSION/glpi-$VERSION.tgz .

RUN tar xvzf glpi-$VERSION.tgz && rm glpi-$VERSION.tgz

WORKDIR /var/www/html/glpi

RUN mkdir /var/log/glpi \
	&& chgrp -R www-data ./files ./config /var/log/glpi \
	&& chmod -R g+rwx ./files ./config /var/log/glpi \
	&& mv ./config /etc/glpi && mv ./files /var/lib/glpi

COPY ./glpi.ini $PHP_INI_DIR/php.ini

COPY ./downstream.php /var/www/html/glpi/inc/downstream.php

COPY ./local_define.php /etc/glpi/local_define.php

RUN chown -R www-data /var/www/html/glpi
