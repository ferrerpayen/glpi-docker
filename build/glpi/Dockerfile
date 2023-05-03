FROM php:8.1-apache-bullseye

RUN a2enmod rewrite

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions \
        && install-php-extensions gd xdebug mysqli intl

RUN install-php-extensions @composer

# Ajoutez des extensions ici
RUN install-php-extensions zip bz2 ldap exif opcache

# Changez la version de glpi ici
ENV VERSION=10.0.7

ADD https://github.com/glpi-project/glpi/releases/download/$VERSION/glpi-$VERSION.tgz .

ADD https://github.com/glpi-project/glpi/archive/refs/tags/$VERSION.zip /tmp/

RUN tar xvzf glpi-$VERSION.tgz -C /var/www/html  && rm glpi-$VERSION.tgz

WORKDIR /var/www/html/glpi

RUN mkdir /var/log/glpi \
        && chgrp -R www-data ./files ./config /var/log/glpi \
        && chmod -R g+rwx ./files ./config /var/log/glpi \
        && mv ./config /etc/glpi && mv ./files /var/lib/glpi \
        && rm -rf ./vendor

RUN unzip /tmp/$VERSION.zip \
        && rm  /tmp/$VERSION.zip \
        && mv glpi-$VERSION/composer.json . \
        && rm -rf glpi-$VERSION \
        && composer require apereo/phpCAS \
        && composer install

COPY ./vh.conf /etc/apache2/sites-available/000-default.conf

COPY ./glpi.ini $PHP_INI_DIR/php.ini

COPY ./downstream.php /var/www/html/glpi/inc/downstream.php

COPY ./local_define.php /etc/glpi/local_define.php

RUN chown -R www-data /var/www/html/glpi
