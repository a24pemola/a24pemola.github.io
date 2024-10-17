# Practica 2.2
## Autenticación en Nginx

### Paquetes necesarios

Podemos usar la herramienta openssl para crear contraseñas.

Se puede comprobar si la tenemos instalada con el siguiente comando:

```
dpkg -l | grep openssl
```
En mi caso, estaba instalado:

![Foto 1](../assets/images/practica%202.2/s1.png)

### Creación de usuarios y contraseñas para el acceso web

Crearemos un archivo oculto llamado “.htpasswd” en el directorio de configuración `/etc/nginx` donde guardar nuestros usuarios y contraseñas (la -c es para crear el archivo): 

```
sudo sh -c "echo -n 'vuestro_nombre:' >> /etc/nginx/.htpasswd"
```

![Foto 2](../assets/images/practica%202.2/s2.png)

Y a continuación creamos el password cifrado para el usuario:

```
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

![Foto 3](../assets/images/practica%202.2/s3.png)

Se pueden crear tantos usuarios como haga falta. Para este ejercicio, yo he creado dos, uno para mi nombre, y otro para mi apellido.

Podemos comprobar que los usuarios y contraseñas están cifrados:

```
cat /etc/nginx/.htpasswd
```

![Foto 5](../assets/images/practica%202.2/s5.png)

### Configurando el servidor Nginx para usar autenticación básica

Para poder proteger nuestra web teniendo que introducir el usuario y contraseña, debemos modificar nuestro archivo en:

```
sudo nano /etc/nginx/sites-available/nombre_web
```

![Foto 6](../assets/images/practica%202.2/s6.png)

Y añadir `auth_basic` dentro de location, junto a `auth_basic_user_file` y el fichero que hemos creado anteriormente para las contraseñas.

![Foto 7](../assets/images/practica%202.2/s7.png)

Tras ello, debemos reiniciar el servicio para ue se apliquen los cambios y podamos comprobarlo:

```
sudo systemctl restart nginx
```

![Foto 8](../assets/images/practica%202.2/s8.png)

### Probando la nueva configuración

Al intentar ahora entrar a la web desde mi máquina, pide un usuario y contraseña:

![Foto 9](../assets/images/practica%202.2/s9.png)

Si se cancela, se denega el acceso a la misma:

![Foto 10](../assets/images/practica%202.2/s10.png)

Para la **Tarea 1** se ha probado con un usuario bueno y otro erróneo, y comprobando los logs de `access.log` se pueden ver los logins de los que han podido entrar con éxito:

![Foto 11](../assets/images/practica%202.2/s11.png)

Y si comprobamos los logs de `error.log` podemos ver los que han fallado:

![Foto 12](../assets/images/practica%202.2/s12.png)

A continuación se pide el denegar acceso a solo cierta parte de la página, pero como nuestras webs esán faltas del mencionado `contact.html`, he escrito cómo se vería si quisiésemos denegar acceso al mismo incluso si en mi página no hace nada:

![Foto 13](../assets/images/practica%202.2/s13.png)

### Combinación de la autenticación básica con la restricción de acceso por IP

También podemos usar las restricciones para denegar el acceso a partir de las IP. con la ayuda de `allow` y `deny`. Y se pueden perfectamente combinar las IP con la necesidad de identificación de usuario.

En la **Tarea 1** de esta sección se pedía denegar el acceso a la IP de nuestra máquina anfitriona:

![Foto 18](../assets/images/practica%202.2/s18.png)

Y si comprobamos tras guardar y reiniciar, debería darnos otro error:

![Foto 14](../assets/images/practica%202.2/s14.png)

El cuál también podemos comprobar a través del `error.log`:

![Foto 15](../assets/images/practica%202.2/s15.png)

Como se mencionó anteriormente, también se puede configurar para que se necesite acceso desde cierta IP o con la autentificación de usuario, y lo podemos hacer así:

![Foto 16](../assets/images/practica%202.2/s16.png)

Al probar entrar a la página desde mi máquina, se comprueba que esta vez me deja entrar, por darme derecho a la IP de que puedo acceder:

![Foto 17](../assets/images/practica%202.2/s17.png)

## Cuestiones finales

### Cuestión 1

Supongamos que yo soy el cliente con la IP 172.1.10.15 e intento acceder al directorio `web_muy_guay` de mi sitio web, equivocándome al poner el usuario y contraseña. ¿Podré acceder?¿Por qué?

```
    location /web_muy_guay {
    #...
    satisfy all;    
    deny  172.1.10.6;
    allow 172.1.10.15;
    allow 172.1.3.14;
    deny  all;
    auth_basic "Cuestión final 1";
    auth_basic_user_file conf/htpasswd;
}
```

**Respuesta**: Supuestamente no debería dejar entrar, ya que pide tanto la IP como el usuario correcto.

### Cuestión 2

Supongamos que yo soy el cliente con la IP 172.1.10.15 e intento acceder al directorio `web_muy_guay` de mi sitio web, introduciendo correctamente usuario y contraseña. ¿Podré acceder?¿Por qué?

```
    location /web_muy_guay {
    #...
    satisfy all;    
    deny  all;
    deny  172.1.10.6;
    allow 172.1.10.15;
    allow 172.1.3.14;

    auth_basic "Cuestión final 2: The revenge";
    auth_basic_user_file conf/htpasswd;
}
```

**Respuesta**: Esta vez se puede acceder, porque esa IP en concreto tiene el permiso para entrar Y el usuario se ha introducido bien, incluso con el `deny all` de por medio.

### Cuestión 3

Supongamos que yo soy el cliente con la IP 172.1.10.15 e intento acceder al directorio `web_muy_guay` de mi sitio web, introduciendo correctamente usuario y contraseña. ¿Podré acceder?¿Por qué?

```
    location /web_muy_guay {
    #...
    satisfy any;    
    deny  172.1.10.6;
    deny 172.1.10.15;
    allow 172.1.3.14;

    auth_basic "Cuestión final 3: The final combat";
    auth_basic_user_file conf/htpasswd;
}
```

**Respuesta**: No podríamos acceder desde esa IP porque está denegada. Esa prohibición tiene prioridad sobre el `satisfy any`.

### Cuestión 4

A lo mejor no sabéis que tengo una web para documentar todas mis excursiones espaciales con Jeff, es esta: Jeff Bezos y yo.

Supongamos que quiero restringir el acceso al directorio de proyectos porque es muy secreto, eso quiere decir añadir autenticación básica a la URL:`Proyectos`

Completa la configuración para conseguirlo:

**Ejemplo sin el código**

```
    server {
        listen 80;
        listen [::]:80;
        root /var/www/freewebsitetemplates.com/preview/space-science;
        index index.html index.htm index.nginx-debian.html;
        server_name freewebsitetemplates.com www.freewebsitetemplates.com;
        location              {

            try_files $uri $uri/ =404;
        }
    }
```

**Código modificado**

```
    server {
        listen 80;
        listen [::]:80;
        root /var/www/freewebsitetemplates.com/preview/space-science;
        index index.html index.htm index.nginx-debian.html;
        server_name freewebsitetemplates.com www.freewebsitetemplates.com;
        location              {

            try_files $uri $uri/ =404;
        }
        location /Proyectos {
                auth_basic "Área restringida";
                auth_basic_user_file /etc/nginx/.htpasswd;
                try_files $uri $uri/ =404;
        }
    }
```

