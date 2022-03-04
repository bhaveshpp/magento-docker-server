# Docker server for magento projects

## Steps to configure 

- Clone this ripository.
- Change data path for elastic search and mysql

`docker-server/services/elasticsearch/docker-compose.yml`

```yml

      - 9200:9200
      - 9300:9300
    volumes:
      - /Users/bhavesh/Documents/docker-server/services/elasticsearch/data:/usr/share/elasticsearch/data
    networks:
      - elastic
    
```

`docker-server/services/mysql/docker-compose.yml`

```yml

    environment:
      MYSQL_ROOT_PASSWORD: rootroot
    volumes:
      - /Users/bhavesh/Documents/docker-server/services/mysql/dump:/dump:nocopy,delegated
      - /Users/bhavesh/Documents/docker-server/services/mysql/data:/var/lib/mysql
      
```

- Now start elasticsearch and mysql services 

```bash

cd docker-server/
cd services/
cd elasticsearch/
docker-compose up -d
cd ../mysql/
docker-compose up -d

```

it will download and install elasticsearch,mysql image and start the container automatically.

## Add new project

- Add new node under `syncs`.

`docker-server/services/docker-sync.yml`

```yml 
  project-magento243p1-sync:
    notify_terminal: true
    src: '../projects/magento243p1'
    sync_strategy: 'unison'
    sync_excludes: [
      '.git',
      '.gitignore',
      'node_modules',
      'var/cache',
      'var/page_cache',
      'var/session',
      '.DS_Store'
    ]
    sync_userid: '33'
    max_attempt: 10
```
- Add volume under php and nginx

`docker-server/services/docker-compose.yml`

```yml

services:

  nginx:
    ...
    volumes:
      ...
      - project-magento243p1-sync:/magento243p1:nocopy,delegated
      ...
    ...

  phpfpm74:
    ...
    volumes:
      ...
      - project-magento243p1-sync:/magento243p1:nocopy,delegated
      ...

volumes:
  ...
  project-magento243p1-sync:
    external: true
  ...

```

- Add new host in `/etc/hosts` in local machin

```bash

sudo nano /etc/hosts
# press ctrl + x 
# press Y and enter to save file

sudo vim /etc/hosts
sudo vi /etc/hosts
# press esc to enable command mode 
# write :wq and enter to save file

```

use any editor for edit

```bash

127.0.0.1       magento243p1.local
127.0.0.1       magento23.local

```

- Test after add new host using `ping magento2443p1.local`

- Add virtual host in `nginx` conf file

`docker-server/services/nginx/conf.d/projects.conf`

```conf
upstream fastcgi_backend {
	server host.docker.internal:9000;
	server phpfpm:9000;
}

server {

    server_name magento243.local;
    index index.php;

    set $MAGE_ROOT /projects/magento243;
    set $MAGE_MODE production;

    root /projects/magento243;

    access_log /var/log/nginx/magento243-access.log;
    error_log /var/log/nginx/magento243-error.log;

    # include /projects/magento243/nginx.conf.sample;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass phpfpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    listen 80;
    listen [::]:80;
}

server {

    server_name magento243p1.local;
    index index.php;

    set $MAGE_ROOT /projects/magento243p1;
    set $MAGE_MODE production;

    root /projects/magento243p1;

    access_log /var/log/nginx/magento243p1-access.log;
    error_log /var/log/nginx/magento243p1-error.log;

    # include /projects/magento243p1/nginx.conf.sample;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass phpfpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    listen 80;
    listen [::]:80;
}

```

> Note: First start sync when you run compose up command for first time you will get this error.
> `external volume "" not found`
> if you want to compose up without start sync then set volume `external` value to `false` at here: `docker-server/services/docker-compose.yml`
> after start sync you can set this value to `true`

- Start sync and compose up to start nginx and php container

```bash

cd docker-server/
cd services
docker-sync start
docker-compose up -d

```

- Now login into `phpfpm` container and [download](https://getcomposer.org/download/) `composer.phar` and move it to `/use/local/bin/composer` to run installer

```bash

docker exec -it phpfpm bash
cd /projects/magento243

mv composer.phar /usr/local/bin/composer
composer

composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition:2.4.3 .

chmod -R 777 var/ generated/ pub/

```

- Configure nginx conf 

```conf

...

server {

    ...

    # root /projects/magento243;

    ...

    include /projects/magento243/nginx.conf.sample;
    ...
}
...

```

- restart `nginx` server.

```bash

docker exec -it nginx bash
service nginx restart

```

- Now check the browser http://magento243p1.local/setup/ 
- Create database and install magento.

```bash

docker exec -it mysql bash

mysql -u root -prootroot

create database magento243;

exit

exit

docker exec -it phpfpm bash

cd /projects/magento243p1/

php -d memory_limit=-1 bin/magento setup:install --base-url=http://magento243p1.local/ --db-host=host.docker.internal --db-name=magento243p1 --db-user=root --db-password=rootroot --admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com --admin-user=admin --admin-password=admin@123 --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1 --elasticsearch-host=host.docker.internal

bin/magento deploy:mode:set developer

chmod -R 777 var/ generated/ pub/

```

> you can create database using [adminer](https://www.adminer.org/)  http://localhost:8080.

## Add project with diffrent php version

- Add new `sync` magento234.

`docker-server/services/docker-sync.yml`
```yml 
  project-magento234-sync:
    notify_terminal: true
    src: '../projects/magento234'
    sync_strategy: 'unison'
    sync_excludes: [
      '.git',
      '.gitignore',
      'node_modules',
      'var/cache',
      'var/page_cache',
      'var/session',
      '.DS_Store'
    ]
    sync_userid: '33'
    max_attempt: 10
```
- Edit `phpfpm` container 

`docker-server/services/docker-compose.yml`
```yml
...
  phpfpm:
    ...

    image: php:7.3-fpm
    # image: php:7.4-fpm

    ...

    build:
      context: ./php/73/ 
      # context: ./php/74/ 
      dockerfile: Dockerfile

    ...
...
```

- Add new folder `projects/magento234` and add new volume information in compose file.

```yml
version: '3.8'

services:

  nginx:
    ...

    volumes:
      ...

      - project-magento234-sync:/projects/magento234:nocopy,delegated
      
      ...

    links:
      - phpfpm

  phpfpm:
    ...

    volumes:
      ...

      - project-magento234-sync:/projects/magento234:nocopy,delegated

volumes:
  ...

  project-magento234-sync:
    external: true

```

- Restart `sync` and rebuilt container.

```bash

cd docker-server/
cd services
docker-sync restart
docker-compose down
docker-compose up -d

```

> Note: When you rebuilt phpfpm then composer will automatically remove so reinstall it. `mv composer.phar /usr/local/bin/composer`

- Login to `phpfpm` and setup project

```bash
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition:2.3.4 . --ignore-platform-reqs


```

- Add new host in `/etc/hosts` file and add server detail in `nginx` server config file and restart server 

`docker-server/services/nginx/conf.d/projects.conf`
```conf
server {

    server_name magento234.local;
    index index.php;

    set $MAGE_ROOT /projects/magento234;
    set $MAGE_MODE production;

    # root /projects/magento234;

    access_log /var/log/nginx/magento234-access.log;
    error_log /var/log/nginx/magento234-error.log;

    include /projects/magento234/nginx.conf.sample;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass phpfpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    listen 80;
    listen [::]:80;
}
```

### Other 

```bash
cp /other/composer.phar /usr/local/bin/composer
ssh-keygen
cp /other/ssh/id_rsa* /root/.ssh/
ssh -T git@bitbucket.org
composer install --ignore-platform-reqs
apt install cron -y

```