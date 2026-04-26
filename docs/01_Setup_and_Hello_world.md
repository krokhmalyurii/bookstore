# Installing and "Hello, world"

## Installing Symfony
1. Initialize the project with the command `composer create-project symfony/skeleton:"8.0.*" my_project_directory`
2. Add the service directory of PhpStorm to .gitignore 
    ```gitignore
    .idea/*
    ```

## Add a WorldController with the hello action

1. Create class `App\Controller\WorldController`
    ```php
    <?php

    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class WorldController
    {
        public function hello(): Response
        {
            return new Response('<html><body><h1><b>Hello,</b> <i>world</i>!</h1></body></html>');
        }
    }
    ```
2. In the `config/routes.yaml` file, add the endpoint description
    ```yaml
    hello_world:
        path: /world/hello
        controller: App\Controller\WorldController::hello
    ```
3. Add domain bookstore.local to local hosts
4. Execute the command `symfony serve` or `docker compose up -d`
5. Go to the address `http://bookstore.local:7777`, we see the Symfony welcome page
6. Go to the address `http://bookstore.local:7777/world/hello`, we see the result of our controller’s work

## Add Docker

1. Create `docker-compose.yml` file
    ```yaml
    services:
    
      php-fpm:
        build: docker
        container_name: 'php'
        ports:
          - '9000:9000'
        volumes:
          - ./:/app
        working_dir: /app
    
      nginx:
        image: nginx
        container_name: 'nginx'
        working_dir: /app
        ports:
          - '7777:80'
        volumes:
          - ./:/app
          - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf
    ```
2. Create `docker\nginx.conf` file
    ```
    server {
        listen 80;
     
        server_name localhost;
        error_log  /var/log/nginx/error.log;
        access_log /var/log/nginx/access.log;
        root /app/public;
     
        rewrite ^/index\.php/?(.*)$ /$1 permanent;
     
        try_files $uri @rewriteapp;
     
        location @rewriteapp {
            rewrite ^(.*)$ /index.php/$1 last;
        }
     
        # Deny all . files
        location ~ /\. {
            deny all;
        }
     
        location ~ ^/index\.php(/|$) {
            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
            fastcgi_index index.php;
            send_timeout 1800;
            fastcgi_read_timeout 1800;
            fastcgi_pass php:9000;
        }
    }
    ```
3. Create `docker\Dockerfile` file
    ```dockerfile
    FROM php:8.5-fpm-alpine
    
    # Install dev dependencies
    RUN apk update \
        && apk upgrade --available \
        && apk add --virtual build-deps \
            autoconf \
            build-base \
            icu-dev \
            libevent-dev \
            openssl-dev \
            zlib-dev \
            libzip \
            libzip-dev \
            zlib \
            zlib-dev \
            bzip2 \
            git \
            libpng \
            libpng-dev \
            libjpeg \
            libjpeg-turbo-dev \
            libwebp-dev \
            freetype \
            freetype-dev \
            postgresql-dev \
            linux-headers \
            curl \
            wget \
            bash
    
    # Install Composer
    RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer
    
    # Install PHP extensions
    RUN docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/
    RUN docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) \
        intl \
        gd \
        bcmath \
        pdo_pgsql \
        sockets \
        zip
    
    # Install symfony CLI
    RUN curl -sS https://get.symfony.com/cli/installer | bash
   
    RUN pecl channel-update pecl.php.net \
        && pecl install -o -f \
            redis \
            event \
        && rm -rf /tmp/pear \
        && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini \
        && echo "extension=event.so" > /usr/local/etc/php/conf.d/event.ini
     ```
4. Launch containers with the command`docker-compose up -d`
5. Go to the address `http://bookstore.local:7777/world/hello`, we see the result of our controller’s work
