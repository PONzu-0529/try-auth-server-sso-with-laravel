FROM php:7.4-apache

# define
ARG DOCUMENT_ROOT
ARG USERNAME
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# install and main setup
RUN apt -y update && \
    apt -y upgrade && \
    curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && \
    apt -y install vim sudo tzdata git wget nodejs python3-pip libzip-dev cron && \
    npm install -g yarn && \
    yarn global add @vue/cli

# setup composer
WORKDIR /var
RUN wget -O composer-setup.php https://getcomposer.org/installer && \
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
    composer self-update

# setup DocumentRoot, php.ini, extentions
RUN sed -ri -e "s!/var/www/html!${DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf && \
    sed -ri -e "s!/var/www/!${DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf && \
    mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini" && \
    sed -ri -e 's/;extension=mysqli/extension=mysqli/g' $PHP_INI_DIR/php.ini && \
    docker-php-ext-install mysqli pdo_mysql zip

# # Setup AWS CLI
# RUN wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh && \
#     yes yes | bash Anaconda3-2022.05-Linux-x86_64.sh
# ENV PATH $PATH:$PATH:/home/${USERNAME}/bin:/home/${USERNAME}/.local/bin

# # setup python
# COPY requirements.txt /var/requirements.txt
# RUN pip install --no-cache-dir -r /var/requirements.txt

# add user
RUN useradd --uid ${USER_UID} --create-home --shell /bin/bash -G sudo,root ${USERNAME} && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# setting setup.sh
COPY setup.sh /var/setup.sh
RUN chmod 700 /var/setup.sh && \
    chown ${USERNAME}:${USERNAME} /var/setup.sh

# # Setting Cron
# COPY crontab /var/spool/cron/crontabs/root
# RUN crontab /var/spool/cron/crontabs/root

# User Settings
USER 1000
ENV PATH $PATH:/home/${USERNAME}/.composer/vendor/bin
RUN composer global require "laravel/installer=^4.0"

CMD sh /var/setup.sh && sudo sudo apache2-foreground
