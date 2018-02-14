# php72-fpm-docker

Use this together with e.g. nginx and database containers.

1) Add docker-compose.yml, nginx.conf and php.ini to one folder (example files below)
2) Add your codebase to a subfolder called "www". If your codebase doesn't use a public-folder, change ./www in docker-compose.yml to ./www/public.
3) Run "docker-compose up"

## Example file: docker-compose.yml

```
version: '2'
services:
 app:
   image: jensji/php72-fpm-docker
   working_dir: /var/www
   volumes:
     - ./www:/var/www
     - ./php.ini:/etc/php/7.2/fpm/conf.d/1000-myphp.ini
   links:
     - database
 web:
   image: nginx:latest
   ports:
     - "80:80"
   volumes:
     - ./www:/var/www
     - ./nginx.conf:/etc/nginx/conf.d/default.conf
   links:
     - app
     - database
 database:
   image: mysql:5.7
   volumes:
     - ./dbdata:/var/lib/mysql
   environment:
     - "MYSQL_DATABASE=homestead"
     - "MYSQL_USER=homestead"
     - "MYSQL_PASSWORD=secret"
     - "MYSQL_ROOT_PASSWORD=secret"
   ports:
       - "33061:3306"
```

## Example file: nginx.conf

```
server {
   listen 80;
   index index.php index.html;
   root /var/www/public;

   client_max_body_size 32M;

   location / {
       try_files $uri $uri/ /index.php$is_args$args;
   }

   location ~ \.php$ {
       fastcgi_split_path_info ^(.+\.php)(/.+)$;
       fastcgi_pass app:9000;
       fastcgi_index index.php;
       include fastcgi_params;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_param PATH_INFO $fastcgi_path_info;
       client_max_body_size 32M;
   }
}
```

# Example file: php.ini

```
cgi.fix_pathinfo = 0;
```
