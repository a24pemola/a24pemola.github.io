# Practica 3.4 Despliegue de una aplicación Flask (Python)

## Procedimiento completo para el despliegue

Primero instalamos el gestor de paquetes de Python pip:

`sudo apt update`

`sudo apt install python3-pip`

![Foto 1](../assets/images/practica%203.4/s1.png)

Luego instalamos el paquete `pipenv` para gestionar los entornos virtuales:

`sudo apt install pipenv`

![Foto 2](../assets/images/practica%203.4/s2.png)

Y comprobamos que está instalado correctamente mostrando su versión:

`pipenv --version`

![Foto 3](../assets/images/practica%203.4/s3.png)

Creamos el directorio en el que almacenaremos nuestro proyecto:

`sudo mkdir /var/www/nombre_mi_aplicacion`

Cuando lo creamos con `sudo`, los permisos pertenecerán a root, por lo que debemos cambiarlos para darle los permisos a nuestro usuario (`lara` en mi caso), y que pertenezca al grupo `www-data`, con los siguientes comandos:

`sudo chown -R $USER:www-data /var/www/mi_aplicacion`

`chmod -R 775 /var/www/mi_aplicacion`

![Foto 4](../assets/images/practica%203.4/s4.png)

```
Es indispensable que asignemos los permisos para que no haya errores.
```

A continuación, dentro del directorio de nuestra aplicación, creamos un archivo oculto `.env` que contendrá las variables necesarias:

`touch .env`

![Foto 5](../assets/images/practica%203.4/s5.png)

Editamos el archivo y añadimos las variables, indicando cuál es el archivo `.py` de la aplicación y el entorno, que en nuestro caso será producción:

![Foto 6](../assets/images/practica%203.4/s6.png)

Ahora vamos ya a iniciar nuestro entorno virtual con el siguiente comando:

`pipenv shell`

Si todo va bien, el nombre del entorno virtual debe salir antes del nombre y ruta en consola:

![Foto 7](../assets/images/practica%203.4/s7.png)

Desde dentro del entorno virtual, instalamos las dependencias necesarias para el proyecto:

`pipenv install flask gunicorn`

![Foto 8](../assets/images/practica%203.4/s8.png)

El archivo que contendrá la aplicación propiamente dicha será `application.py`, y `wsgi.py` se encargará únicamente de iniciarla y dejarla corriendo:

`touch application.py wsgi.py`

Una vez creamos los archivos, debemos editarlos para dejarlos así:

![Foto 9](../assets/images/practica%203.4/s9.png)

![Foto 10](../assets/images/practica%203.4/s10.png)

Con el servidor web integrado de Flask, vamos a correr la aplicación para comprobar que todo funciona bien. Especificaremos la dirección 0.0.0.0:

![Foto 11](../assets/images/practica%203.4/s11.png)

Si intentamos entrar en la url, debería funcionarnos. Recordad ponerlo de manera que `http://IP-maq-virtual:5000`

![Foto 12](../assets/images/practica%203.4/s12.png)

¡Y voilá! Una vez lo hemos comprobado, paramos el servidor con `CTRL+C`.

## Comprobación de Gunicorn

Si Flask ha funcionado bien, usando el siguiente comando nos dirá si Gunicorn funciona:

`gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app`

![Foto 13](../assets/images/practica%203.4/s13.png)

Todavía en el entorno virtual, vamos a tomar nota del path desde la que se ejecuta `gunicorn`, la vamos a necesitar para más adelante:

![Foto 14](../assets/images/practica%203.4/s14.png)

Como ya tendríamos que tener Nginx instalado en nuestro sistema por practicas anteriores, lo iniciamos y comprobamos su estado:

```
sudo systemctl start nginx

sudo systemctl status nginx
```

![Foto 15](../assets/images/practica%203.4/s15.png)

Ya fuera del entorno virtual, creamos un archivo para systemd, para que corra Gunicorn como un servicio más:

![Foto 16](../assets/images/practica%203.4/s16.png)

Tened en cuenta que:

- `User`: Será nuestro usuario.

- `Group`: Pondremos `www-data`

- `Environment`: Establecemos el directorio `bin`. La ruta que guardamos antes pero sin el 'gunicorn'.

- `WorkingDirectory`: El directorio donde tenemos el proyecto.

- `ExecStart`: La ruta que guardamos antes para Gunicorn, completa.

Una vez configurado eso, habilitamos el servicio:

```
systemctl enable nombre_mi_servicio

systemctl start nombre_mi_servicio
```

![Foto 17](../assets/images/practica%203.4/s17.png)

El nombre del servicio es el nombre del archivo que creamos antes.

Ahora deberemos configurar Nginx, yendo a la ruta de `/etc/nginx/sites-available/nombre_aplicacion`:

```
server {
    listen 80;
    server_name mi_aplicacion www.mi_aplicacion; 



    access_log /var/log/nginx/mi_aplicacion.access.log; 


    error_log /var/log/nginx/mi_aplicacion.error.log;

    location / { 
            include proxy_params;
            proxy_pass http://unix:/var/www/nombre_aplicacion/nombre_aplicacion.sock; 


    }
}   
```

![Foto 18](../assets/images/practica%203.4/s18.png)

Recordemos que ahora debemos crear un link simbólico del archivo de sitios webs disponibles al de sitios web activos:

`sudo ln -s /etc/nginx/sites-available/nombre_aplicacion /etc/nginx/sites-enabled/`

Y nos aseguramos de que se ha creado dicho link simbólico:

`ls -l /etc/nginx/sites-enabled/ | grep nombre_aplicacion`

![Foto 19](../assets/images/practica%203.4/s19.png)

Como siempre, comprobamos que no haya errores y reiniciamos Nginx:

```
nginx -t

sudo systemctl restart nginx

sudo systemctl status nginx
```

Como ya no podemos acceder por IP ya que la app está siendo servida por Gunicorn y Nginx, accedemos por el `server_name`.

Debemos editar el archivo `hosts` de la máquina anfitriona para añadir la IP de la máquina virtual y el nombre del proyecto:

![Foto 20](../assets/images/practica%203.4/s20.png)

Este archivo, en Linux, está en: `/etc/hosts`

Y en Windows: `C:\Windows\System32\drivers\etc\hosts`

Ya guardado el archivo `hosts`, intentamos acceder a la web a través de `http://nombre_aplicacion` o `http://www.nombre_aplicacion`:

![Foto 21](../assets/images/practica%203.4/s21.png)

![Foto 22](../assets/images/practica%203.4/s22.png)

## Mismo proceso con un repositorio de Git

Primero, comenzamos con clonar el repositorio en cuestión:

`git clone https://github.com/raul-profesor/Practica-3.5`

![Foto 23](../assets/images/practica%203.4/s23.png)

Lo siguiente que debemos hacer es dar los permisos necesarios a nuestra aplicación, tal y como hicimos anteriormente, con los comandos, y siempre teniendo en cuenta que esta vez el nombre de la aplicación es diferente:

```
sudo chown -R $USER:www-data /var/www/mi_aplicacion

chmod -R 775 /var/www/mi_aplicacion   
```

![Foto 24](../assets/images/practica%203.4/s24.png)

Dentro del directorio de la app, creamos el archivo oculto `.env` para la creación de otro entorno virtual:

`touch .env`

Y editamos el archivo para que quede así:

![Foto 25](../assets/images/practica%203.4/s25.png)

Lanzamos el entorno virtual:

`pipenv shell`

![Foto 26](../assets/images/practica%203.4/s26.png)

E instalamos las dependencias correspondientes:

`pipenv install -r requirements.txt`

`pipenv install flask gunicorn`

![Foto 27](../assets/images/practica%203.4/s27.png)

![Foto 28](../assets/images/practica%203.4/s28.png)

Creamos el archivo `wsgi.py` solo ya que el otro no hace falta al existir ya uno:

`touch wsgi.py`

Y lo editamos para dejarlo así:

![Foto 29](../assets/images/practica%203.4/s29.png)

Ya creado y guardado, probamos que lo corre, usando la dirección `0.0.0.0` como la otra vez:

`flask run --host '0.0.0.0'`

Y al entrar en la url `http://IP-maq-virtual:5000` debería verse lo siguiente:

![Foto 30](../assets/images/practica%203.4/s30.png)

## Comprobaación de Gunicorn 2

Si Flask ha funcionado bien, usando el siguiente comando nos dirá si Gunicorn funciona:

`gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app`

![Foto 31](../assets/images/practica%203.4/s31.png)

Todavía en el entorno virtual, vamos a tomar nota del path desde la que se ejecuta `gunicorn`, la vamos a necesitar para más adelante:

![Foto 32](../assets/images/practica%203.4/s32.png)

Como ya tendríamos que tener Nginx instalado en nuestro sistema por practicas anteriores, lo iniciamos y comprobamos su estado:

```
sudo systemctl start nginx

sudo systemctl status nginx
```

Ya fuera del entorno virtual, creamos un archivo para systemd, para que corra Gunicorn como un servicio más:

![Foto 33](../assets/images/practica%203.4/s33.png)

Tened en cuenta que:

- `User`: Será nuestro usuario.

- `Group`: Pondremos `www-data`

- `Environment`: Establecemos el directorio `bin`. La ruta que guardamos antes pero sin el 'gunicorn'.

- `WorkingDirectory`: El directorio donde tenemos el proyecto.

- `ExecStart`: La ruta que guardamos antes para Gunicorn, completa.

Una vez configurado eso, habilitamos el servicio:

```
systemctl enable nombre_mi_servicio

systemctl start nombre_mi_servicio
```

![Foto 34](../assets/images/practica%203.4/s34.png)

El nombre del servicio es el nombre del archivo que creamos antes.

Ahora deberemos configurar Nginx, yendo a la ruta de `/etc/nginx/sites-available/nombre_aplicacion`:

```
server {
    listen 80;
    server_name mi_aplicacion www.mi_aplicacion; 



    access_log /var/log/nginx/mi_aplicacion.access.log; 


    error_log /var/log/nginx/mi_aplicacion.error.log;

    location / { 
            include proxy_params;
            proxy_pass http://unix:/var/www/nombre_aplicacion/nombre_aplicacion.sock; 


    }
}   
```

![Foto 35](../assets/images/practica%203.4/s35.png)

Recordemos que ahora debemos crear un link simbólico del archivo de sitios webs disponibles al de sitios web activos:

`sudo ln -s /etc/nginx/sites-available/nombre_aplicacion /etc/nginx/sites-enabled/`

Y nos aseguramos de que se ha creado dicho link simbólico:

`ls -l /etc/nginx/sites-enabled/ | grep nombre_aplicacion`

Como siempre, comprobamos que no haya errores y reiniciamos Nginx:

```
nginx -t

sudo systemctl restart nginx

sudo systemctl status nginx
```

Como ya no podemos acceder por IP ya que la app está siendo servida por Gunicorn y Nginx, accedemos por el `server_name`.

Debemos editar el archivo `hosts` de la máquina anfitriona para añadir la IP de la máquina virtual y el nombre del proyecto:

![Foto 36](../assets/images/practica%203.4/s36.png)

Este archivo, en Linux, está en: `/etc/hosts`

Y en Windows: `C:\Windows\System32\drivers\etc\hosts`

Ya guardado el archivo `hosts`, intentamos acceder a la web a través de `http://nombre_aplicacion` o `http://www.nombre_aplicacion`:

![Foto 37](../assets/images/practica%203.4/s37.png)

## Cuestiones

```
Busca, lee, entiende y explica qué es y para que sirve un servidor WSGI.
```

Un servidor WSGI es como el intermediario que traduce entre un servidor web (como Nginx o Apache) y una aplicación web hecha en Python (por ejemplo, Flask o Django). Recibe las solicitudes del navegador, las convierte en algo que la aplicación entiende, y luego toma la respuesta de la aplicación y se la devuelve al cliente en un formato listo para la web. Este estándar hace que cualquier servidor WSGI pueda trabajar con cualquier aplicación Python compatible, facilitando el despliegue en producción, especialmente con herramientas como Gunicorn o uWSGI, que manejan múltiples solicitudes al mismo tiempo para maximizar el rendimiento.