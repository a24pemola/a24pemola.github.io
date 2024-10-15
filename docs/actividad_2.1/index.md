# Practica 2.1
## Instalación y configuración de servidor web Nginx
Para instalar el servidor nginx en nuestra Debian, lo primero que hacemos es actualizar los repositorios y luego instalamos el paquete necesario:

```
sudo apt update

sudo apt install nginx
```

Después, verificamos que nginx se haya instalado y que esté funcionando bien:

```
systemctl status nginx
```

![Foto 1](../assets/images/practica%202.1/s1.png)

![Foto 2](../assets/images/practica%202.1/s2.png)

![Foto 3](../assets/images/practica%202.1/s3.png)

## Creación de las carpeta del sitio web
Al igual que en Apache, todos los archivos que serán parte de un sitio web servido por nginx se organizan en carpetas. Normalmente, estas carpetas están dentro de `/var/www`.

Vamos a crear la carpeta donde estará nuestro sitio web o dominio:

```
sudo mkdir -p /var/www/nombre_web/html
```

![Foto 4](../assets/images/practica%202.1/s4.png)

El "nombre_web" puede ser lo que quieras, sin espacios.

Luego, dentro de esa carpeta `html`, clona este repositorio:

[https://github.com/cloudacademy/static-website-example](https://github.com/cloudacademy/static-website-example)

![Foto 5](../assets/images/practica%202.1/s5.png)

También vamos a hacer que el propietario de esa carpeta y todo lo que esté dentro sea el usuario `www-data`, que suele ser el usuario del servicio web:

`sudo chown -R www-data:www-data /var/www/nombre_web/html` 

![Foto 6](../assets/images/practica%202.1/s6.png)

Le daremos los permisos adecuados para que no tengamos problemas de acceso no autorizado al entrar en el sitio web:

`sudo chmod -R 755 /var/www/nombre_web` 

![Foto 7](../assets/images/practica%202.1/s7.png)

Para comprobar que el servidor está funcionando bien y sirviendo páginas correctamente, puedes acceder desde tu cliente a:

`http://IP-maq-virtual`

Debería aparecer algo así:

![Foto 8](../assets/images/practica%202.1/s8.png)

Lo que significará que está correcto hasta ahora.

## Configuración de servidor web NGINX

Para que Nginx pueda mostrar el contenido de nuestra web, necesitamos crear un bloque de servidor con las directivas adecuadas. En lugar de modificar el archivo de configuración predeterminado, vamos a crear uno nuevo en `/etc/nginx/sites-available/nombre_web`:

```
sudo nano /etc/nginx/sites-available/vuestro_dominio 
```

El contenido de ese archivo de configuración sería algo como esto:

```
server {
        listen 80;
        listen [::]:80;
        root /ruta/absoluta/archivo/index;
        index index.html index.htm index.nginx-debian.html;
        server_name nombre_web;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

![Foto 9](../assets/images/practica%202.1/s9.png)

En la directiva `root`, tienes que poner la ruta completa donde esté el archivo `index.html` de tu página web, que se encuentra entre los archivos que ya habéis descomprimido.

Ahora vamos a crear un enlace simbólico entre este archivo y el de los sitios habilitados, para que se active automáticamente:

```
sudo ln -s /etc/nginx/sites-available/nombre_web /etc/nginx/sites-enabled/
```

![Foto 10](../assets/images/practica%202.1/s10.png)

Y reiniciamos el servidor para aplicar la configuración:

```
sudo systemctl restart nginx
```

![Foto 11](../assets/images/practica%202.1/s11.png)

## Comprobaciones

### Comprobación del correcto funcionamiento

Como todavía no tenemos un servidor DNS que traduzca los nombres a IPs, lo vamos a hacer manualmente. Editaremos el archivo `/etc/hosts` en nuestra máquina anfitriona para que asocie la IP de la máquina virtual con nuestro `server_name`.

En Linux, este archivo está en `/etc/hosts`.

En Windows lo encontrarás en `C\Windows\System32\drivers\etc\hosts`.

Ahí, debemos añadir la siguiente línea:

`192.168.X.X nombre_web`

Reemplaza "192.168.X.X" por la IP de tu máquina virtual.

![Foto 12](../assets/images/practica%202.1/s12.png)

![Foto 13](../assets/images/practica%202.1/s13.png)

### Comprobar registros del servidor

Asegúrate de que las peticiones se están registrando bien en los archivos de logs, tanto las que salen bien como las que fallan:

- `/var/log/nginx/access.log`: aquí se registra cada solicitud que llega a tu servidor web, a menos que tengas Nginx configurado para otra cosa.

![Foto 14](../assets/images/practica%202.1/s14.png)

- `/var/log/nginx/error.log`: cualquier error que ocurra con Nginx se va a anotar en este archivo.

![Foto 15](../assets/images/practica%202.1/s15.png)

## Configurar servidor SFTP en Debian

Primero, vamos a instalarlo desde los repositorios:

```
sudo apt-get update
sudo apt-get install vsftpd
```

![Foto 16](../assets/images/practica%202.1/s16.png)

Ahora, vamos a crear una carpeta en tu home de Debian:

```
mkdir /home/nombre_usuario/ftp
```

![Foto 17](../assets/images/practica%202.1/s17.png)

En la configuración de vsftpd, indicaremos que esta carpeta será el directorio al que se cambia vsftpd una vez que el usuario se conecta.

Luego, vamos a crear los certificados de seguridad necesarios para añadir una capa de cifrado a la conexión (algo parecido a HTTPS):

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem
```

![Foto 18](../assets/images/practica%202.1/s18.png)

Una vez hecho esto, pasamos a configurar vsftpd como tal. Usa el editor de texto que prefieras para editar el archivo de configuración del servicio, por ejemplo con nano:

```
sudo nano /etc/vsftpd.conf
```

Primero, busca estas líneas en el archivo y elimínalas por completo:

```
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
```

![Foto 19](../assets/images/practica%202.1/s19.png)

Después, añade estas líneas en su lugar:

```
rsa_cert_file=/etc/ssl/private/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.pem
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH

local_root=/home/nombre_usuario/ftp
```

![Foto 20](../assets/images/practica%202.1/s20.png)

Finalmente, guarda los cambios y reinicia el servicio para que aplique la nueva configuración:

```
sudo systemctl restart --now vsftpd
```

![Foto 21](../assets/images/practica%202.1/s21.png)

Una vez terminada esta configuración, ya podrás acceder a tu servidor usando un cliente FTP como Filezilla de dos maneras:

-   **Por el puerto 21** (el puerto por defecto de FTP, que es inseguro), pero utilizando certificados para cifrar los datos, haciéndolo seguro.
    
-   **Usando el protocolo SFTP**, que se dedica al intercambio de datos con una conexión parecida a SSH, usando el puerto 22.

> En mi caso utilicé el puerto 22 porque el 21 parecía dar más error.

Después de descargar un cliente FTP en tu ordenador, introduce los datos necesarios para conectarte a tu servidor FTP en Debian:

![Foto 22](../assets/images/practica%202.1/s22.png)

-   **IP de Debian** (recuadro rojo)

-   **Nombre de usuario** de Debian (recuadro verde)

-   **Contraseña** del usuario (recuadro fucsia)

-   **Puerto de conexión**, que será el **22** conectando por SFTP (recuadro marrón)

Al usar las claves de SSH (como ya hicimos en la Práctica 1), no necesitas poner la contraseña, solo el nombre de usuario.

Al conectarte con claves FTP con _Conexión rápida_, te saldrá el mismo aviso que cuando te conectaste por primera vez por SSH a Debian. Lo aceptas, ya que no hay ningún peligro.

Te conectarás directamente a la carpeta que especificamos en el archivo de configuración: `/home/lara/ftp`.

![Foto 23](../assets/images/practica%202.1/s23.png)

Una vez conectado, en el lado izquierdo de la pantalla (tu ordenador) buscas una imagen cualquiera (por ejemplo). En el lado derecho (el servidor), busca la carpeta donde quieres subir el archivo. Con un doble clic o haciendo clic derecho > _subir_, lo transfieres al servidor.

Recuerda que tu sitio web debe estar en la carpeta `/var/www` y es necesario darle los permisos adecuados, como hicimos con el otro sitio web.

# HTTPS

En esta parte vamos a añadir una capa extra de seguridad a nuestro servidor. Vamos a hacer que todos los sitios web que tengamos usen certificados SSL y que se acceda a ellos a través de HTTPS.

Para hacerlo, y como prueba, vamos a generar unos certificados autofirmados. Luego, en el archivo de configuración de nuestros hosts virtuales (los sitios web que ya configuramos), cambiaremos los parámetros que hagan falta.

Puedes buscar en Internet para guiarte y conseguir el resultado que necesitas.

> En mi caso, me ayudé de internet y la ayuda de unos amigos programadores.

### Redirección HTTP a HTTPS

Cuando hayas terminado de habilitar HTTPS en tus sitios web, puedes pasar a esta tarea.

Fíjate que con la configuración actual, tu sitio web es accesible de dos formas al mismo tiempo: por el puerto 80 (HTTP, que es inseguro) y por el puerto 443 (HTTPS, que es seguro). Como queremos dejar todo bien configurado y sin posibles fallos de seguridad, el objetivo es que, si un usuario accede a tu sitio por el puerto 80 (HTTP), automáticamente se le redirija a HTTPS, en el puerto 443, por seguridad.

Busca la información necesaria para hacer esta redirección automática ajustando tus archivos de configuración de hosts virtuales.

> De nuevo, personalmente me ayudé de internet y unos amigos programadores.

Para finalmente, terminarlo:

![Foto 24](../assets/images/practica%202.1/s24.png)

![Foto 25](../assets/images/practica%202.1/s25.png)

## Cuestiones finales

> ¿Qué pasa si no hago el link simbólico entre `sites-available` y `sites-enabled` de mi sitio web?

Si no haces el enlace simbólico entre `sites-available` y `sites-enabled`, tu sitio web no estará activo y Nginx no lo reconocerá. Esto significa que no podrás acceder a tu sitio a través de la web, ya que Nginx no lo servirá. Así que es un paso importante para que tu sitio esté disponible.

> ¿Qué pasa si no le doy los permisos adecuados a `/var/www/nombre_web`?

Si no le das los permisos adecuados a `/var/www/nombre_web`, tu servidor web no podrá acceder a los archivos de tu sitio. Esto puede causar errores al intentar cargar la página, y los usuarios verán mensajes de acceso denegado. En resumen, tu sitio no funcionará correctamente.