# Practica 2.5
## Proxy inverso y balanceo de carga con SSL en NGINX

### Configuraciones

Partiendo de la configuración exacta de la práctica anterior, vamos a añadir la configuración SSL para el cifrado del Proxy Inverso.

### Creación de certificado autofirmado

Nosotros no usaremos un CA de confianza, ya que no estamos publicando en internet Y los certificados son de pago.

Nosotros crearemos el nuestro propio para simular el escenario, por lo que la primera vez que accedamos el sitio web, saldrá una advertencia.

En primer lugar, tenemos que crear el sguiente directorio:

`/etc/nginx/ssl`

![Foto 1](../assets/images/practica%202.5/s1.png)

Luego, podemos crear el certificado con un único comando:

`sudo openssl rew -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/server.key -out /etc/nginx/ssl/server.crt`

![Foto 2](../assets/images/practica%202.5/s2.png)

Nos preguntará el introducir una serie de parámetros. Se deben introducir los parámetros de la siguiente imagen, a excepción del “Organizational Unit Name”. Ahí deberéis poner `2DAW – DEAW - Vuestronombre`.

![Foto 3](../assets/images/practica%202.5/s3.png)

### Configuración SSL en el proxy inverso

De la práctica anterior, ya deberías tener en el directorio `/etc/nginx/sites-available` un archivo de configuración llamado “balanceo”. Aquí es donde vamos a configurar el acceso al sitio web para que funcione con SSL (HTTPS).

Dentro del bloque `server {…}`, cambia el puerto de escucha (`listen 80`) por lo que ves en la imagen de abajo. Además, añade las siguientes líneas de configuración para que quede así:

![Foto 4](../assets/images/practica%202.5/s4.png)

Recordad el reiniciar el servicio de Nginx como de costumbre tras el cambio.

### Comprobaciones

- Si ahora accedes a `https://balanceo.local`, verás un aviso de seguridad porque nuestro certificado es autofirmado, como mencionamos antes.

- Si agregas una excepción, podrás entrar al sitio, y al recargar la página varias veces con F5, verás que el balanceo de carga funciona bien usando HTTPS.

- Para confirmar que los datos del certificado son los tuyos, haz clic en el candado en la barra de búsqueda:

![Foto 5](../assets/images/practica%202.5/s5.png)

![Foto 6](../assets/images/practica%202.5/s6.png)

![Foto 7](../assets/images/practica%202.5/s7.png)

### Redirección forzosa a HTTPS

Para que sí o sí nos fuerce a usar HTTPS, haremos una configuración adicional.

Añadiremos un nuevo bloque 'server' separado del otro, en el archivo de configuración de 'balanceo'.

Debe quedar algo así:

![Foto 8](../assets/images/practica%202.5/s8.png)

Lo que estamos haciendo es que cuando se reciba una petición HTTP (puerto 80) en `http://balanceo.local`, se redirija a `https://balanceo.local` (HTTPS).

La tarea extra ha consistido en:

- Eliminar del otro bloque `server{...}` las líneas que hacían referencia al puerto 80.

- Hemos reiniciado el servicio.

- Se ha comprobado que ahora fuerza siempre la redirección a HTTPS.

- Y se ha comprobado que cuando realizamos una petición en el archivo de log `http_access.log` aparece la redirección 301 y que, de la misma manera, aparece una petición GET en `https_access.log`.

![Foto 9](../assets/images/practica%202.5/s9.png)

## Cuestiones finales

### Cuestión 1

Hemos configurado nuestro proxy inverso con todo lo que nos hace falta pero no nos funciona y da un error del tipo `This site can't provide a secure connection, ERR_SSL_PROTOCOL_ERROR`.

Dentro de nuestro server block tenemos esto:

```
server {
    listen 443;
    ssl_certificate /etc/nginx/ssl/enrico-berlinguer/server.crt;
    ssl_certificate_key /etc/nginx/ssl/enrico-berlinguer/server.key;
    ssl_protocols TLSv1.3;
    ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;
    server_name enrico-berlinguer;
    access_log /var/log/nginx/https_access.log;

    location / {
        proxy_pass http://red-party;
        }
    }
```

En el primer `listen 443` ha faltado poner el `ssl`. Debería quedar como:

`listen 443 ssl;`

### Cuestión 2

Imaginad que intentamos acceder a nuestro sitio web HTTPS y nos encontramos con el siguiente error, luego, investigad qué está pasando y como se ha de solucionar.

![Foto 10](../assets/images/practica%202.5/s10.png)

El error `NET::ERR_CERT_REVOKED` en una conexión HTTPS significa que el certificado del sitio ha sido revocado por la entidad certificadora, lo que indica que ya no es confiable. Esto puede suceder si el certificado fue comprometido, emitido por error, o revocado a petición del propietario. Para solucionarlo, el administrador del sitio debe reemplazar el certificado por uno nuevo y válido; como usuario, no puedes evitar el error hasta que el sitio lo corrija.