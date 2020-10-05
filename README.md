# Coworker Local Development Docker Environment

Docker running Nginx, PHP-FPM, Composer, MySQL and PHPMyAdmin.

## Installion Overview

1. [Install prerequisites](#install-prerequisites)

    Before installing project make sure the following prerequisites have been met.

2. [Clone the Coworker.com git repository](#clone-the-coworkercom-git-repository)

    We’ll download the code from its repository on GitHub.

3. [Configure Nginx With SSL Certificates](#configure-nginx-with-ssl-certificates) [`Optional`]

    We'll generate and configure SSL certificate for nginx before running server.

4. [Configure Xdebug](#configure-xdebug) [`Optional`]

    We'll configure Xdebug for IDE (PHPStorm).

5. [Update your Hosts file](#update-your-hosts-file)

    We'll be setting your local development urls
    
6. [Import your database](#Import-your-database)

    We'll be importing your database    

7. [Run the application](#run-the-application)

    By this point we’ll have all the project pieces in place.


## Configuration Notes
1. [Configuration Notes](#configuration-notes)    
    
    Optional - But if you want to make configuration changes, check here first.

## Utility Commands
1. [Docker utility commands](#docker-utility-commands)

    When running, you can use docker commands for doing recurrent operations.
    


___
    
## Install prerequisites

1. For now, this project has been mainly created for Unix `(Linux/MacOS)`. Though we do our best to support Window 10 (and only Windows 10)

    All requisites should be available for your distribution. The most important are :

    * [Git](https://git-scm.com/downloads)
    * [Docker](https://docs.docker.com/engine/installation/)
    * [Docker Compose](https://docs.docker.com/compose/install/)

1. **Note: If you're using Windows OS, the following instructions when meant to be ran in `CGYWIN`**     
    
    1. Download and install Cgywin
        ```bash
        https://www.cygwin.com/install.html
        ```
        During the install, you will have the option to install packages
        Select the `git` package for installation
    2. Open Cgywin
    
    3. Run the following commands (ignore carriage returns)
        ```bash
        set -o igncr
        ```
    
    4. Mount the C: directory
        ```bash
        cd /
        mkdir c
        mount c: /c
        ```

1. Create a `coworker_project` folder (This is where we will store all of our repositories) 
    ```bash
    mkdir coworker_project
    
    cd coworker_project
    ```
    
1. Clone the `docker_local` repository
    
    ```bash
    git clone https://github.com/coworkerglobal/docker_local.git 
    ``` 
    
    Checkout development_fix branch
    
    ```bash
    cd docker_local
 
    git checkout master 
    ``` 
    
    Make data directories  
   
    ```bash
    mkdir -p data/db/dumps
    mkdir -p data/db/mysql
    ```

## Clone the Coworker.com git repository
 

1. Clone the Coworker.com git repository

    ```bash
    git clone https://github.com/coworkerglobal/coworker.com.git
    ```  

1. Rename root repository directory:

    ```bash
    mv coworker.com coworker
    ```
   

1. CD into the `docker_local` directory:

    ```bash
    cd docker_local
    ```

### Images we'll be using

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL](https://hub.docker.com/_/mysql/)
* [PHP-FPM](https://github.com/coworkerglobal/docker_local/blob/master/builds/php/Dockerfile-php5.6)
* [Composer](https://hub.docker.com/_/composer/)
* [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
* [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

For our dev environment, we'll use the following ports:

| Server     | Port |
|------------|------|
| MySQL      | 3306 |
| PHPMyAdmin | 8080 |
| Nginx      | 80   |
| Nginx SSL  | 443  |

___

### Project Tree

```bash
.
├── Makefile
├── README.md
├── builds
├── data
│   └── db
│       ├── dumps
│       └── mysql
├── doc
├── docker-compose.yml
├── etc
   ├── nginx
   │   ├── default.conf
   │   └── default.template.conf
   ├── php
   │   └── php.ini
   └── ssl

```

___

## Configure .env file

1. Create your `.env` file its template
    ```bash
    cp .env.template .env
    ```
    
  
2. Open your `.env` file and change the `EMAIL_DEFAULT` env var to your email address.

## Configure Nginx With SSL Certificates

1. Generate SSL certificates

    ```bash
    source .env && docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
    ```
    
2. [Install and Configure SSL for local Mac OSX](docs/configure-cacert-for-local-macosx.md)
___

## Configure Xdebug

If you use another IDE than [PHPStorm](https://www.jetbrains.com/phpstorm/) or [Netbeans](https://netbeans.org/), go to the [remote debugging](https://xdebug.org/docs/remote) section of Xdebug documentation.

For a better integration of Docker to PHPStorm, use the 

1. Get your own local IP address :

    ```bash
    ifconfig
    ```

1. Edit php file `etc/php/php.ini` and comment or uncomment the configuration as needed.

1. Set the `remote_host` parameter with your IP :

    ```bash
    xdebug.remote_host=192.168.0.1 # your IP
    ```
    
1. [Configure PhpStorm IDE](docs/phpstorm-macosx.md)
    
___

## Update your Hosts file

1. Open your `hosts` file on your host OS 
    
    Mac and Linux:
    ```bash
    vi /etc/hosts
    ``` 
    
    Windows:
    ```
    vi "/c:/Windows/System32/drivers/etc/hosts"
    ```

1. Add the following to your ```hosts``` file on your host machine OS (mac, windows or ubuntu):      
    ```
    0.0.0.0			coworker.dev
    127.0.0.1		coworker.dev
    0.0.0.0			phpmyadmin.dev
    127.0.0.1		phpmyadmin.dev  
    ``` 
___

## Import your database

1. Start/build the MySQL server:

    ```bash
    docker-compose up -d --build mysqldb
    ```

1. Download the database backup dump file from the link below and **make sure** to save in the following directory `docker_local/data/db/dumps`:
* [https://drive.google.com/drive/folders/166Wts9YTxUIhn-aRL6O8eBqmDU5Ec5RI?usp=sharing](https://drive.google.com/drive/folders/166Wts9YTxUIhn-aRL6O8eBqmDU5Ec5RI?usp=sharing) 

1. CD into root of docker_local repository
   
    ```bash
    cd docker_local
    ```
1. Create MySQL database users
    
    ```bash
    source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" -e "GRANT ALL PRIVILEGES ON *.* TO 'dev'@'%' IDENTIFIED BY 'dev' WITH GRANT OPTION; FLUSH PRIVILEGES;"
    source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" -e "GRANT ALL PRIVILEGES ON *.* TO 'staging_user'@'%' IDENTIFIED BY 'staging_user'; FLUSH PRIVILEGES;"
    source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" -e "GRANT ALL PRIVILEGES ON *.* TO 'cw_production'@'%' IDENTIFIED BY 'cw_production'; FLUSH PRIVILEGES;" # we need to create cw_production user, otherwise our db import will fail
    ```
1.  Create database
    
    ```bash
    source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" -e "CREATE DATABASE $DB_NAME"
    ```

1. Import database
    
    ```bash
    source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" cw_staging < "data/db/dumps/coworker.database.sql"
    ```
    

1. Shutdown the MySQL server:

    ```bash
    docker-compose down -v
    ```

___

## Run the application

1. Start the application :

    ```bash
    source .env && docker-compose up -d
    ```

    **Please wait this might take a several minutes...**

    ```bash
    docker-compose logs -f # Follow log output
    ```

1. Open your favorite browser :

    * [https://coworker.dev](https://coworker.dev) ([HTTPS](#configure-nginx-with-ssl-certificates) not configured by default)
    * [http://phpmyadmin.dev:8080](http://phpmyadmin.dev:8080) PHPMyAdmin (username: dev, password: dev)

1. Installtion complete!

1. To stop and clear services, run the following command:

    ```bash
    docker-compose down -v
    ```
___

## Configuration Notes

1. Configure NGINX (optional)

    Do not modify the `etc/nginx/default.conf` file, it is overwritten by `etc/nginx/default.template.conf`

    If you need to configure NGINX, make sure you edit the `etc/nginx/default.template.conf` file.

___

## Docker utility commands

### Installing a package with composer

```bash
# NOTE: cd into the root of coworker repository folder 
docker run --rm -v $(pwd):/app composer require symfony/intl

```

### Updating PHP dependencies with composer

```bash
# NOTE: cd into the root of coworker repository folder 
docker run --rm -v $(pwd):/app composer update
```

### Generating PHP API documentation

```bash
docker-compose exec -T php php -d memory_limit=256M -d xdebug.profiler_enable=0 /var/www/html/vendor/bin/apigen generate /var/www/html/application --destination /var/www/html/doc
```

### Testing PHP application with PHPUnit

```bash
docker-compose exec -T php /var/www/html/vendor/bin/phpunit --colors=always --configuration /var/www/html
```

### Fixing standard code with [PSR2](http://www.php-fig.org/psr/psr-2/)

```bash
docker-compose exec -T php /var/www/html/vendor/bin/phpcbf -v --standard=PSR2 /var/www/html/application
```

### Checking the standard code with [PSR2](http://www.php-fig.org/psr/psr-2/)

```bash
docker-compose exec -T php /var/www/html/vendor/bin/phpcs -v --standard=PSR2 /var/www/html/application
```

### Analyzing source code with [PHP Mess Detector](https://phpmd.org/)

```bash
docker-compose exec -T php /var/www/html/vendor/bin/phpmd /var/www/html/application text cleancode,codesize,controversial,design,naming,unusedcode
```

### Checking installed PHP extensions

```bash
docker-compose exec php php -m
```

### Handling database

#### MySQL shell access

```bash
docker exec -it mysql bash
```

and

```bash
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### Creating a backup of all databases

```bash
mkdir -p data/db/dumps
```

```bash
source .env && docker exec $(docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

#### Restoring a backup of all databases

```bash
source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

#### Creating a backup of single database

**`Notice:`** Replace "YOUR_DB_NAME" by your custom name.

```bash
source .env && docker exec $(docker-compose ps -q mysqldb) mysqldump -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" --databases YOUR_DB_NAME > "data/db/dumps/YOUR_DB_NAME_dump.sql"
```

#### Restoring a backup of single database

```bash
source .env && docker exec -i $(docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/YOUR_DB_NAME_dump.sql"
```

___
     
