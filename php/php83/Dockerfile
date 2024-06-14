FROM php:8.2-fpm-alphine

ARG WWWGROUP
ENV TZ=UTC
ARG NODE_VERSION=20
ARG POSTGRES_VERSION=15
ARG MYSQL_CLIENT="default-mysql-client"
ENV SUPERVISOR_PHP_USER="sail"
WORKDIR /app


# Surpresses debconf complaints of trying to install apt packages interactively
# https://github.com/moby/moby/issues/4032#issuecomment-192327844

ARG DEBIAN_FRONTEND=noninteractive

# Update package lists
RUN apt-get update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*

# Create directory for GPG keys
RUN mkdir -p /etc/apt/keyrings

RUN apt-get update -y 

RUN apt-get -y --no-install-recommends install nano wget gnupg gosu ca-certificates zip unzip supervisor sqlite3 libcap2-bin dnsutils librsvg2-bin fswatch ffmpeg
RUN apt-get -y --no-install-recommends install dialog libsqlite3-dev libsqlite3-0 zlib1g-dev libzip-dev libicu-dev
RUN apt-get -y --no-install-recommends install --fix-missing apt-utils build-essential git curl libonig-dev && \
    apt-get install -y iputils-ping && \
    apt-get -y --no-install-recommends install --fix-missing libcurl4 \
    libcurl4-openssl-dev \
    openssl 
RUN rm -rf /var/lib/apt/lists/* && \
    apt-get update -y 



# Install Composer
RUN curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

# Import NodeSource repository key
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

# Add NodeSource repository to sources list
RUN echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" > /etc/apt/sources.list.d/nodesource.list

# Update package lists (again)
RUN apt-get update

# Install Node.js
RUN apt-get install -y nodejs


# Install npm package managers
RUN npm install -g npm
RUN npm install -g pnpm
RUN npm install -g bun

# Import Yarn repository key
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /etc/apt/keyrings/yarn.gpg >/dev/null

# Add Yarn repository to sources list
RUN echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list

# Import PostgreSQL repository key
RUN curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /etc/apt/keyrings/pgdg.gpg >/dev/null

# Add PostgreSQL repository to sources list
RUN echo "deb [signed-by=/etc/apt/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list

# Update package lists (again)
RUN apt-get update


# Install MySQL client (assuming environment variable MYSQL_CLIENT is set)
RUN apt-get -y --no-install-recommends install  $MYSQL_CLIENT

# Install PostgreSQL client (assuming environment variable POSTGRES_VERSION is set)
RUN apt-get install -y postgresql-client-$POSTGRES_VERSION

RUN apt-get install -y libssl-dev pkg-config
# Install PHP extensions
RUN pecl install mongodb \
    && docker-php-ext-enable mongodb

# Install xdebug 3.3.2
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug && \
    mkdir /var/log/xdebug

RUN docker-php-ext-install pdo_mysql && \
    docker-php-ext-install pdo_sqlite && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install curl && \
    docker-php-ext-install zip && \
    docker-php-ext-install -j$(nproc) intl && \
    docker-php-ext-install mbstring && \
    docker-php-ext-install gettext && \
    docker-php-ext-install calendar && \
    docker-php-ext-install exif

RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev \
libjpeg62-turbo-dev \
libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail
    
COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod +x /usr/local/bin/start-container
EXPOSE 9000
RUN mkdir -p /var/www/html/logs
ENV PHP_FPM_ERROR_LOG=/var/www/html/logs/php-fpm.log




# Clean up temporary files
RUN apt-get -y autoremove
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# Start the PHP-FPM server
CMD ["php-fpm"]