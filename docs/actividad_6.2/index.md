# Práctica 6.2 - Despliegue de una aplicación PHP con Nginx y MySQL usando Docker y Docker-Compose

## Proceso de dockerización de Nginx+PHP+MySQL

### 1. Estructura de directorios

Como vamos a necesitar que la estructura de directorios quede de cierta manera, empezaremos creándolos con los siguientes comandos:

![Foto](../assets/images/practica%206.2/1.png)
![Foto](../assets/images/practica%206.2/2.png)
![Foto](../assets/images/practica%206.2/3.png)

### 2. Creación de un Contenedor Nginx

Primero empezaremos creando un contenedor para Nginx, que en sí servirá para la aplicación PHP.

Debemos comenzar editando el archivo `docker-compose.yml`:

```
nginx:
  image: nginx:latest
  container_name: nginx-container
  ports:
    - 80:80
```

![Foto](../assets/images/practica%206.2/4.png)

¡OJO! Yo he puesto el puerto 8081 para que no diese problemas con el Nginx que tenía instalado de antes en la máquina virtual.

Luego, iniciamos el contenedor y comprobamos que funciona bien todo:

![Foto](../assets/images/practica%206.2/5.png)

Con el comando `sudo docker ps` siempre podemos ver si el contenedor está corriendo.

El archivo en sí que acabamos de crear se descargará la última versión de la imagen de Nginx, creando un contenedor con ella y publicando o escuchando en el puerto 80, y en mi caso corresponder al puerto 8081.

Si luego probamos abrir `http://IP_Maq_Virtual:80` desde el navegador de nuestra máquina anfitriona, deberíamos ver lo siguiente:

![Foto](../assets/images/practica%206.2/6.png)

### 3. Creación de un contenedor PHP

Como ya tenemos el archivo, abrimos el `index.php` y añadimos el siguiente contenido:

```
<!DOCTYPE html>
<head>
  <title>¡Hola mundo!</title>
</head>

<body>
  <h1>¡Hola mundo!</h1>
  <p><?php echo 'Estamos corriendo PHP, version: ' . phpversion(); ?></p>
</body>
```

![Foto](../assets/images/practica%206.2/7.png)

Una vez configurado ese, se modificará `default.conf`, el de la carpeta `nginx`, con el siguiente contenido:

```
server {

     listen 80 default_server;
     root /var/www/html;
     index index.html index.php;

     charset utf-8;

     location / {
      try_files $uri $uri/ /index.php?$query_string;
     }

     location = /favicon.ico { access_log off; log_not_found off; }
     location = /robots.txt { access_log off; log_not_found off; }

     access_log off;
     error_log /var/log/nginx/error.log error;

     sendfile off;

     client_max_body_size 100m;

     location ~ .php$ {
      fastcgi_split_path_info ^(.+.php)(/.+)$;
      fastcgi_pass php:9000;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_intercept_errors off;
      fastcgi_buffer_size 16k;
      fastcgi_buffers 4 16k;
    }

     location ~ /.ht {
      deny all;
     }
    }
```

![Foto](../assets/images/practica%206.2/8.png)

Y a continuación se modifica el `Dockerfile` del directorio de `Nginx` con lo siguiente:

```
FROM nginx:latest
COPY ./default.conf /etc/nginx/conf.d/default.conf
```

![Foto](../assets/images/practica%206.2/9.png)

Y por último, se modifica `docker-compose.yml` para añadir lo siguiente:

```
services:
  nginx:
    build: ./nginx/
    container_name: nginx-container
    ports:
      - 80:80
    links:
      - php
    volumes:
      - ./www/html/:/var/www/html/

  php:
    image: php:7.0-fpm
    container_name: php-container
    expose:
      - 9000
    volumes:
      - ./www/html/:/var/www/html/
```

![Foto](../assets/images/practica%206.2/10.png)

En mi caso, se cambia de nuevo lo del puerto para ponerlo como `8081:80`.

Ahora, creamos el contenedor para el PHP en el puerto 9000, enlazado con el anterior contenedor Nginx.

![Foto](../assets/images/practica%206.2/11.png)

Si ahora probamos la dirección `http://IP_Maq_Virtual:80`, la pantalla habrá cambiado:

![Foto](../assets/images/practica%206.2/12.png)

### 4. Creación de un Contenedor para Datos

Empezamos modificando el archivo de `docker-compose.yml` con el siguiente contenido:

```
nginx:
  build: ./nginx/
  container_name: nginx-container
  ports:
    - 80:80
  links:
    - php
  volumes_from:
    - app-data

php:
  image: php:7.0-fpm
  container_name: php-container
  expose:
    - 9000
  volumes_from:
    - app-data

app-data:
  image: php:7.0-fpm
  container_name: app-data-container
  volumes:
    - ./www/html/:/var/www/html/
  command: "true"
```

![Foto](../assets/images/practica%206.2/13.png)

Luego comprobamos que los conteneores funcionan bien:

![Foto](../assets/images/practica%206.2/14.png)

### 5. Creación de un Contenedor MySQL

Para que la base de datos de PHP se pueda alojar, deberemos hacer un contenedor MySQL.

Esta vez, modificamos el archivo `Dockerfile` del directorio `php` con el siguiente contenido:

```
FROM php:7.0-fpm
RUN docker-php-ext-install pdo_mysql
```

![Foto](../assets/images/practica%206.2/15.png)

Como hemos hecho antes, modificamos el `docker-compose.yml` de nuevo, para actualizarlo con la siguiente información:

```
services:
  nginx:
    build: ./nginx/
    container_name: nginx-container
    ports:
      - 80:80
    links:
      - php
    volumes_from:
      - app-data
  php:
    build: ./php/
    container_name: php-container
    expose:
      - 9000
    links:
      - mysql
    volumes_from:
      - app-data

  app-data:
    image: php:7.0-fpm
    container_name: app-data-container
    volumes:
      - ./www/html/:/var/www/html/
    command: "true"

  mysql:
    image: mysql:5.7
    container_name: mysql-container
    volumes_from:
      - mysql-data
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: mydb
      MYSQL_USER: myuser
      MYSQL_PASSWORD: password

  mysql-data:
    image: mysql:5.7
    container_name: mysql-data-container
    volumes:
      - /var/lib/mysql
    command: "true"
```

![Foto](../assets/images/practica%206.2/16.png)

Y luego modificamos el archivo `index.php` para cambiar su contenido:

```
     <!DOCTYPE html>
     <head>
      <title>¡Hola mundo!</title>
     </head>

     <body>
      <h1>¡Hola mundo!</h1>
      <p><?php echo 'Estamos corriendo PHP, version: ' . phpversion(); ?></p>
      <?
       $database ="mydb";
       $user = "myuser";
       $password = "password";
       $host = "mysql";

       $connection = new PDO("mysql:host={$host};dbname={$database};charset=utf8", $user, $password);
       $query = $connection->query("SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE='BASE TABLE'");
       $tables = $query->fetchAll(PDO::FETCH_COLUMN);

        if (empty($tables)) {
          echo "<p>No hay tablas en la base de datos \"{$database}\".</p>";
        } else {
          echo "<p>La base de datos \"{$database}\" tiene las siguientes tablas:</p>";
          echo "<ul>";
            foreach ($tables as $table) {
              echo "<li>{$table}</li>";
            }
          echo "</ul>";
        }
        ?>
    </body>
</html>
```

![Foto](../assets/images/practica%206.2/17.png)

Probamos a continuación que el contenedor funciona:

![Foto](../assets/images/practica%206.2/18.png)
![Foto](../assets/images/practica%206.2/19.png)
![Foto](../assets/images/practica%206.2/20.png)

### 6. Verificación de conexión a la base de datos

Si ahora vamos a `http://IP_Maq_Virtual:80`, obtenemos lo siguiente:

![Foto](../assets/images/practica%206.2/21.png)

No salen las tablas porque un uuario normal no puede verlas, así que, modificaremos el archivo `index.php` para cambiar `$user` por `root`, y `$password` por `secret`, quedando de la siguiente manera:

![Foto](../assets/images/practica%206.2/22.png)

Así, cuando probemos de nuevo, aparecerán todas las tablas:

![Foto](../assets/images/practica%206.2/23.png)

### 7. Esquema de la infraestructura completa de contenedores

![Foto](../assets/images/practica%206.2/24.jpg)