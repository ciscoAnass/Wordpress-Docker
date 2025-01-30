# Wordpress Docker



### 1- Creamos directorio de trabajo : 

```bash
mkdir wordpress-docker
cd wordpress-docker
```


### 2- Creamos una red para Docker : 

```bash
docker network create wp_network
```

### 3- Creamos el fichero de Docker Compose (Contiene Docker,Nginx,Mysql )

```bash
nano  docker-compose.yml
```
Y agregamos esta configuracion al fichero **Docker-Compose** :


```yaml
version: '3'

services:
  # MySQL Database
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: your_root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress_user
      MYSQL_PASSWORD: wordpress_password
    networks:
      - wordpress_network

  # WordPress
  wordpress:
    depends_on:
      - db
    image: wordpress:php8.1-fpm
    volumes:
      - wordpress_data:/var/www/html
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress_user
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_DB_NAME: wordpress
    networks:
      - wordpress_network

  # Nginx
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
      - "8443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - wordpress_data:/var/www/html
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - wordpress
    networks:
      - wordpress_network

networks:
  wordpress_network:
    driver: bridge

volumes:
  db_data:
  wordpress_data:

```


### 4- Creamos el fichero de configuracion Nginx para habilitar el SSL : 


```bash
nano nginx.conf
```

Y agregamos esta configuracion al fichero de configuracion **Nginx** :

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    
    # Redirect HTTP to HTTPS
    return 301 https://$host:8443$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name localhost;

    # SSL configuration
    ssl_certificate /etc/nginx/ssl/localhost.crt;
    ssl_certificate_key /etc/nginx/ssl/localhost.key;
    
    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/html;
    index index.php;

    # Better handling of requests
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # Improved PHP-FPM configuration
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_read_timeout 600;
    }

    # Deny access to sensitive files
    location ~ /\. {
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        log_not_found off;
        access_log off;
        allow all;
    }

    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
}
```

### 5- Creamos un Carpeta SSL donde generamos nuestro ceritificado SSL : 



```bash
mkdir ssl
cd ssl
```

- Ahora generamos el certificado SSL :


```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt -subj "/CN=localhost"
```

- Volvemos al directorio anterior donde tenemos el fichero **Docker Compose** : 


```bash
cd ..
```

- Lanzamos nuestro Fichero **Docker Compose** : 

```bash
docker compose up -d
```
- Comprobamos :

```bash
docker ps
```
