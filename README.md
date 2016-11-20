## <a name="contents-link"></a>Table Of Contents

* [Project Description](#description-link)
* [Requirements](#requirements-link)
* [Quick Install (via Composer)](#quickinstall-link)
  * [Important informations and warnings](#importantinformations-link)
  * [The Docker .env file](#theenvfile-link)
  * [Data Directory](#datadirectory-link)
* [Start](#start-link)
* [Interacting with projects](#interact-link)
* [Hostnames](#hostnames-link)
* [Troubleshooting](#troubleshooting-link)
* [Actual Issues](#actualissues-link)

## <a name="description-link"></a>PROJECT DESCRIPTION

Get your **PHP/MySQL** project up and running within minutes with the power of Docker and Composer!
With few commands, you'll have your development, staging and production infrastructures ready-to-go taking advantage of **Docker** containers.
Within this project, you'll find a container for the following dependencies:
- apache
- php-fpm
- mysql
- mongodb
- couchdb

Moreover, database data and sessions are managed with a specific container, and a last container is provided as the workspace (a special container you can use to run CLI commands).  
Thanks to Composer, this project can be easily integrated and encapsulated into you existing webapp, permitting you to deploy it faster on development machines, staging servers and production servers.

## <a name="requirements-link"></a>REQUIREMENTS

- docker-compose, min version 1.8
- docker, min version 1.10
- composer
- php, min version 5.6


## <a name="quickinstall-link"></a>QUICK INSTALL via Composer

- Install phpdocker running  

```
composer global require lombax85/phpdocker
```
- If you want to update it instead, run

```
composer global update lombax85/phpdocker
```

This command will install phpdocker globally for your user.

```
NOTE: Be sure to add your composer bin directory to your PATH variable. 
On OSX and Linux, this is usually in $HOME/.composer/vendor/bin
```

Now, go into your project's root folder and type

```
phpdocker dockerize
```

Running this command creates a `docker` directory inside your project's root directory, and you are almost ready to go.


#### <a name="importantinformations-link">IMPORTANT INFORMATIONS</a>
- A `docker_data` directory will appear when you start you containers the first time. It is recomended to add `/docker_data` in your `.gitignore` file, but read carefully the [Data Directory](#datadirectory-link) section.
- Inside the `docker` directory, you will find all your docker infrastructure configuration and a file named `.env`. The `.env` file is pre-configured and containes your docker configuration, go to [The Docker .env file](#theenvfile-link) section for more informations.



#### <a name="theenvfile-link">THE Docker .env FILE</a>
- The configuration (ports to bind, modules to enable in the containers) is stored in a file named `.env` inside the `docker` directory. Inside this file, you can enable or disable specific options and modules, for example you can switch from PHP 7 to PHP 5.6, or set the default password for MySQL.

#### <a name="datadirectory-link">DATA DIRECTORY</a>

The `./docker_data` directory contains all data of **databases and sessions**.
If you use this setup in a production environment, **don't forget to backup all data** with the appropriate tools (example: mysqldump for mysql).   
The `./docker_data` directory is shared among containers using directory binding and is kept between container rebuilds.   
For this reason, **when you rebuild - for example - your mysql container, the data are not lost**. 
However, pay attention because if you change your mysql engine to somethings not compatible with the content of your data directory, the content itself can become corrupted.

By default, the data directory is configured to be inside `./docker_data`.

The directory is created when you start your containers the first time. If you want to change the default path for the `docker_data` directory, look at the `.env` file.

 

## <a name="start-link"></a>START


- go inside the `docker` directory (`cd docker`)
- execute this commands (if you don't need a specific engine, omit it in the "up" command)

```
docker-compose build apache2 mysql workspace mongo php-fpm couchdb
docker-compose up -d apache2 mysql mongo couchdb
```

- After these commands you'll have your containers up and running, use `docker ps` to see them
- Now do some post-install things:
	- MongoDB: Unlike MySQL, MongoDB doesn't allow to set default username and password prior to installation. For this reason, you must set them with a post-run script. To set default user and password for mongodb, type
	```
	docker-compose exec mongo sh /mongo.sh user password
	```

## <a name="interact-link"></a>INTERACTING with projects

The "workspace" container should be used for all cli commands (composer install/update, artisan)

```
docker-compose exec workspace bash
```

will give you a shell inside the www directory.
If you prefer, you can send your command directly without using the shell. For example, to send a "php artisan migrate", simply do

```
docker-compose exec workspace php artisan migrate
```
 

## <a name="hostnames-link"></a> HOSTNAMES:

Docker creates a virtual private and isolated network for all containers of the same project (it uses the root directory name as a prefix).  
To reach one container from another (for example for reaching mysql container from php-fpm) simply use the hostname.
The hostname is the name of the container in the docker-compose.tml file.  
Don't use the private ip because it can change at any time.  

So, when you have to configure your mysql server hostname in your web app's config file, simply type "mysql"

> MYSQL_HOST=mysql

If you bash into a container you'll see 

```
root@7aa4b96361fb:/var/www# ping mysql
PING mysql (172.19.0.4) 56(84) bytes of data.
64 bytes from mongo_mysql_1.mongo_default (172.19.0.4): icmp_seq=1 ttl=64 time=0.148 ms
```

In this project, these containers/hostname exists

workspace
mysql
php-fpm
apache2
mongo
couchdb

## <a name="troubleshooting-link"></a>ADDITIONAL SETUP and Troubleshooting
- on mac: if the root folder of your project is not already shared, enable file sharing on `./docker_data` and `./docker` folders.


## <a name="actualissues-link"></a>ACTUAL ISSUES


- If you stop (ctrl+c) during "docker-compose up" during the first container startup, the content of /docker/data can became corrupt or not correctly initialized. In this case, for example, you won't be able to connect to MySQL.
To solve:

```
docker-compose stop
rm -Rf ./docker/data/mysql/*
```
NOTE: if you wipe MongoDB Data, don't forget to re-add the default user
