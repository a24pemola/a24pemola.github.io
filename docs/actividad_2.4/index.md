# Practica 2.4
## Balanceo de carga con proxy inverso en Nginx

### Configuraciones

```
¡OJO! Como detalle a tener en cuenta, deberemos desactivar los sitios web de las tareas anteriores para que no den problemas.

Dentro de la carpeta `/etc/nginx/sites-enabled` debemos ejecutar `unlink nombre_archivo` para cada uno de los archivos de los sitios web que tenemos.
```

![Foto 1](../assets/images/practica%202.4/s1.png)

### Nginx Servidor Web 1

Debemos configurar este servidor web para que sirva el siguiente `index.html` que debéis crear dentro de la carpeta `/var/www/webserver1/html`:

![Foto 2](../assets/images/practica%202.4/s2.png)

- El nombre del sitio web que se debe utilizar en los archivos (sites-available, etc) para Nginx es `webserver1`, igual que en el resto de configuraciones.

- El sitio web debe escuchar al puerto 8080.

- Se debe añadir una cabecera que se llame `Serv_Web1_vuestronombre`.

![Foto 3](../assets/images/practica%202.4/s13.png)

### Nginx Servidor Web 2

Se debe clonar en la máquina Debian, del servidor web 1.

![Foto 4](../assets/images/practica%202.4/s5.png)

Se debe hacer una configuración idéntica, pero cambiando los nombres de `webserver1` por `webserver2` (incluído el index.html), y el nombre de la cabecera será `Serv_Web2_vuestronombre`.


```
¡OJO! Es importante que no quede ninguna referencia al webserver1, o podría haber problemas.
```

### Nginx Proxy Inverso

Ya que tenemos los servidores web, lo que nos quedará es configurar el proxy inverso.

- En sites-available vamos a crear el archivo de configuración con el nombre `balanceo`.

Y el archivo tendrá el siguiente formato:

```
    upstream backend_hosts {
                random;
                server ________:____;
                server ________:____;
    }
            server {
                listen 80;
                server_name ________;      
                location / {
                    proxy_pass http://backend_hosts;
                }
            }
```

- El bloque upstream → son los servidores entre los que se va a repartir la carga, que son los dos que hemos configurado anteriormente.

- Si miráis el diagrama y tenéis en cuenta la configuración que habéis hecho hasta ahora, aquí deberéis colocar la IP de cada servidor, así como el puerto donde está escuchando las peticiones web.

- A este grupo de servidores le ponemos un nombre, que es `backend_hosts`.

- Pondremos random porque es lo más fácil para comprobar que todo funciona bien en la práctica, pero hay diferentes formas de repartir la carga (las peticiones HTTP). 

![Foto 5](../assets/images/practica%202.4/s11.png)

### Comprobaciones

Se debería poder acceder al sitio web sin problemas, recordando que se debe desactivar el caché.

![Foto 6](../assets/images/practica%202.4/s15.png)

![Foto 7](../assets/images/practica%202.4/s16.png)

Si refrescamos, el servidor es aleatorio, y si se desactiva el servidor 1, se irá al 2, igual que si desactivamos el servidor 2, se irá al 1.

## Cuestiones finales

### Cuestión 1

Busca información de qué otros métodos de balanceo se pueden aplicar con Nginx y describe al menos 3 de ellos.

- `Round Robin`: Asigna las solicitudes en un ciclo uniforme entre todos los servidores, ideal para ambientes de carga similar​.

- `Least Connections`: Redirige las solicitudes al servidor con menos conexiones activas, equilibrando cargas en sistemas con tráfico variable.

- `IP Hash`: Envía solicitudes del mismo cliente al mismo servidor, manteniendo la sesión y datos consistentes, útil para aplicaciones como e-commerce.

### Cuestión 2

Si quiero añadir 2 servidores web más al balanceo de carga, describe detalladamente qué configuración habría que añadir y dónde.

- Si tenemos ya configurado, lo único que debemos hacer es añadir los servidores extra en el bloque de `upstream`.

```
upstream webserverejemplo {
    ...;
    server <host_servidor_1>:<puerto_servidor_1>;
    server <host_servidor_2>:<puerto_servidor_2>;
}
```

### Cuestión 3

Describe todos los pasos que deberíamos seguir y configurar para realizar el balanceo de carga con una de las webs de prácticas anteriores.

Indicad la configuración de todas las máquinas (webservers, proxy...) y de sus servicios.

- Tal y como hemos hecho con este ejercicio, se deberán tener al menos 3 máquinas virtuales. El servidor web original, el servidor 2 clonado del original, y el proxy inverso.

- Para la configuración del servidor web, debemos crear un archivo en el directorio `/etc/nginx/sites-available`, y debemos elegir un nombre. Para este ejemplo, se llamará `webtest` y debe quedar así:

```
/etc/nginx/sites-available/webtest

server {
    listen 8080;
    server_name webtest.local;

    location / {
        root /var/www/webtest/html;
        index index.html;

        add_header webtest_lara webtest;
    }
}
```

- Tras tener configurado el archivo, creamos el enlace:

```
sudo ln -s /etc/nginx/sites-available/webtest /etc/nginx/sites-enabled/
```

- Y lo reiniciamos:

```
sudo systemctl restart nginx
```

- Para la configuración del servidor clonado, solo se tiene que clonar el servidor anterior, pero cambiando el antiguo `server_name` para que no haya errores.

```
/etc/nginx/sites-available/webtest

server {
    listen 8080;
    server_name webtest2.local;

    location / {
        root /var/www/webtest/html;
        index index.html;

        add_header webtest2_lara webtest;
    }
}
```

- Para la configuración del proxy, lo haremos igual que en la práctica en sí. Creamos un archivo que debería llevar esta configuración:

```
/etc/nginx/sites-available/webtest-proxy

    upstream webtest {
                random;
                server webtest.local:8080;
                server webtest2.local:8080;
    }
            server {
                listen 80;
                server_name webtest.local;      
                location / {
                    proxy_pass http://webtest;
                }
            }
```

- No se debe olvidar el crear los enlaces pertinentes con el comando de siempre:

```
sudo ln -s /etc/nginx/sites-available/webtest-proxy /etc/nginx/sites-enabled/
```

- Por último, se reinicia de nuevo:

```
sudo systemctl restart nginx
```

- Como último paso, añadimos al archivo hosts de nuestro pc la IP del proxy para poder probarlo, en Windows, la localización sería:

```
C:\Windows\System32\drivers\etc\hosts

...
ip-proxy webtest.local
```

- En Linux, la localización estaría en:

```
/etc/hosts

...
ip-proxy webtest.local
```