FROM php:5.6-apache

ENV PHPIPAM_SOURCE https://github.com/phpipam/phpipam/archive/
ENV PHPIPAM_VERSION 1.2
ENV MYSQL_ROOT_PASSWORD somepassword

# Install required deb packages
RUN apt-get update && \ 
	apt-get install -y git php-pear php5-curl php5-mysql php5-json php5-gmp php5-mcrypt php5-ldap libgmp-dev cron nano && \
	rm -rf /var/lib/apt/lists/*

# Configure apache and required PHP modules 
RUN docker-php-ext-configure mysqli --with-mysqli=mysqlnd && \
	docker-php-ext-install mysqli && \
	docker-php-ext-install pdo_mysql && \
	docker-php-ext-install gettext && \ 
	docker-php-ext-install pcntl && \ 
	ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h && \
	docker-php-ext-configure gmp --with-gmp=/usr/include/x86_64-linux-gnu && \
	docker-php-ext-install gmp && \
	echo ". /etc/environment" >> /etc/apache2/envvars && \
	a2enmod rewrite

# Configure timezone and locale
RUN echo "Europe/Copenhagen" > /etc/timezone && \
	dpkg-reconfigure -f noninteractive tzdata

#RUN touch /usr/local/etc/php/php.ini
#&& echo "[Date]\ndate.timezone = "US/Central"" >> /usr/local/etc/php/php.ini
ADD php.ini /usr/local/etc/php/

# copy phpipam sources to web dir
ADD ${PHPIPAM_SOURCE}/${PHPIPAM_VERSION}.tar.gz /tmp/
RUN	tar -xzf /tmp/${PHPIPAM_VERSION}.tar.gz -C /var/www/html/ --strip-components=1 

# Use system environment variables into config.php

RUN mv /var/www/html/config.dist.php /var/www/html/config.php
RUN sed -i \
        -e "s/\['host'\] = \"localhost\"/\['host'\] = \"mysql\"/" \
    -e "s/\['user'\] = \"phpipam\"/\['user'\] = \"root\"/" \
    -e "s/\['pass'\] = \"phpipamadmin\"/\['pass'\] = getenv(\"MYSQL_ENV_MYSQL_ROOT_PASSWORD\")/" \
        /var/www/html/config.php

#set up cron to scan
#RUN crontab -l | { cat; echo "*/1 * * * *  /usr/bin/php /var/www/html/functions/scripts/pingCheck.php 1>>/dev/null 2>&1"; } | crontab -
#RUN crontab -l | { cat; echo "*/1 * * * * /usr/bin/php /var/www/html/functions/scripts/resolveIPaddresses.php 1>>/dev/null 2>&1"; } | crontab -
#RUN crontab -l | { cat; echo "*/1 * * * *  /usr/bin/php /var/www/html/functions/scripts/discoveryCheck.php  1>>/dev/null 2>&1"; } | crontab -
#CMD /usr/bin/crontab /etc/crontab

# Add crontab file in the cron directory
#ADD crontab /etc/cron.d/phpipam
 
# Give execution rights on the cron job
#RUN chmod 0644 /etc/cron.d/phpipam
 
# Create the log file to be able to run tail
#RUN touch /var/log/cron.log
 
#RUN echo "*/1 * * * * /usr/bin/php /var/www/html/functions/scripts/pingCheck.php >> /var/log/cron.log
#RUN echo "*/1 * * * * /usr/bin/php /var/www/html/functions/scripts/resolveIPaddresses.php >> /var/log/cron.log
#RUN echo "*/1 * * * * /usr/bin/php /var/www/html/functions/scripts/discoveryCheck.php >> /var/log/cron.log

EXPOSE 80

# Run the command on container startup
#RUN /usr/bin/crontab /etc/cron.d/phpipam

