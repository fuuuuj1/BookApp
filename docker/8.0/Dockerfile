FROM ubuntu:20.04

LABEL maintainer="Taylor Otwell"

ARG WWWGROUP

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C \
    && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.0-cli php8.0-dev \
       php8.0-pgsql php8.0-sqlite3 php8.0-gd \
       php8.0-curl php8.0-memcached \
       php8.0-imap php8.0-mysql php8.0-mbstring \
       php8.0-xml php8.0-zip php8.0-bcmath php8.0-soap \
       php8.0-intl php8.0-readline \
       php8.0-msgpack php8.0-igbinary php8.0-ldap \
       php8.0-redis \
    && apt-get install -y nginx php8.0-fpm \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn \
    && apt-get install -y mysql-client \
    && apt-get install -y postgresql-client \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN pecl channel-update https://pecl.php.net/channel.xml \
    && pecl install swoole xdebug \
    && pecl clear-cache \
    && rm -rf /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.0

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/8.0/cli/conf.d/99-sail.ini
COPY php.ini /etc/php/8.0/fpm/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

COPY xdebug.ini /etc/php/8.0/cli/conf.d/99-xdebug.ini
COPY xdebug.ini /etc/php/8.0/fpm/conf.d/99-xdebug.ini
COPY default /etc/nginx/sites-available/default

RUN cd /tmp \
    && mkdir cert \
    && cd ./cert \
    && openssl genrsa -des3 -passout pass:gsahdg -out snakeoil.pass.key 2048 \
    && openssl rsa -passin pass:gsahdg -in snakeoil.pass.key -out snakeoil.key \
    && openssl req -new -subj "/C=US/ST=California/L=San Francisco/O=Dis/CN=localhost" -key snakeoil.key -out snakeoil.csr \
    && openssl x509 -req -days 365 -in snakeoil.csr -signkey snakeoil.key -out snakeoil.pem \
    && mv ./snakeoil.key /etc/ssl/private/ssl-cert-snakeoil.key \
    && mv ./snakeoil.pem /etc/ssl/certs/ssl-cert-snakeoil.pem \
    && cd /tmp \
    && rm -rf /tmp/cert

EXPOSE 8000

ENTRYPOINT ["start-container"]
