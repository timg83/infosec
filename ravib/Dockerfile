# Use an official PHP image with Apache
FROM php:7.4-apache

# Install necessary packages, PHP extensions, and cron
RUN apt-get update && apt-get install -y \
    default-libmysqlclient-dev \
    libxml2-dev \
    libxslt1-dev \
    libfreetype6-dev \             
    libjpeg62-turbo-dev \          
    libpng-dev \                   
    git \
    wget \
    unzip \
    cron \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd mysqli pdo pdo_mysql json xml xsl gd

# Set timezone in php.ini
RUN echo "date.timezone=Europe/Amsterdam" > /usr/local/etc/php/php.ini

# Clone the Banshee repository
RUN git clone https://gitlab.com/hsleisink/ravib.git /var/www/html

# Enable Apache mod_rewrite and set up configuration
RUN a2enmod rewrite

# Set permissions
RUN chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html

# Configure ENCRYPT_DATA, DEBUG_MODE, WEBSITE_DOMAIN, and USE_AUTHENTICATOR in banshee.conf
ARG ENCRYPT_DATA=false
RUN sed -i "s/^ENCRYPT_DATA=.*/ENCRYPT_DATA=${ENCRYPT_DATA}/" settings/banshee.conf && \
    sed -i "s/^DEBUG_MODE=.*/DEBUG_MODE=no/" settings/banshee.conf && \
    sed -i "s/^WEBSITE_DOMAIN=.*/WEBSITE_DOMAIN=yourdomain.com/" settings/banshee.conf && \
    sed -i "s/^USE_AUTHENTICATOR=.*/USE_AUTHENTICATOR=yes/" settings/banshee.conf

# Copy cron settings
COPY ./ravib-cron /etc/cron.d/ravib-cron

# Apply cron job and give execution rights
RUN chmod 0644 /etc/cron.d/ravib-cron && crontab /etc/cron.d/ravib-cron

# Install MySQL, set environment variables, and create a database
RUN apt-get install -y mariadb-server
ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_DATABASE=ravib
RUN service mariadb start

# Expose Apache and MySQL ports
EXPOSE 80 3306

# Start cron, Apache, and MySQL
CMD cron && service mariadb start && apache2-foreground
