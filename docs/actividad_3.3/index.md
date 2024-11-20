# Practica 3.3 Despliegue de una aplicación una aplicación React en Netlify (PaaS)

## Creación de nuestra aplicación

Después de iniciar sesión por SSH en nuestro Debian, creamos una carpeta para la aplicación con el nombre que prefiramos. Dentro de esa carpeta, añadimos los 3 archivos necesarios: dos `.html` y un `.js` para nuestra sencilla app de ejemplo.

El directorio se puede crear con el comando ` mkdir` como hemos hecho otras veces, mientras que los archivos en cuestion se pueden crear usando el comando `sudo nano head.html` por ejemplo.

head.html
```
<!DOCTYPE html>
<html>
<head>
        <title>Hola Mundo</title>
</head>
<body>

        <h1>Esta es la pagina principal</h1>

<p><a href="/tailPage">Ir a la siguiente pagina</a></p>


</body>
```

![Foto 1](../assets/images/practica%203.3/s1.png)

tail.html
```
<!DOCTYPE html>
<html>
<head>
        <title>Hola Mundo</title>
</head>
<body>
        <h1>FUNCIONA</h1>

</body>
```

![Foto 2](../assets/images/practica%203.3/s2.png)

aplicacion.js
```
var http = require('http');
var fs = require('fs'); // para obtener los datos del archivo html
var port = process.env.PORT || 8080; 

http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/html' });

    // req.url almacena el path o ruta de la URL
    var url = req.url;
    if (url === "/") {
// fs.readFile busca el archivo HTML
// el primer parámetro es el path al archivo HTML
// y el segundo es el callback de la función
// si el archivo no se encuentra, la función devuelve un error
// si el archivo se encuentra, el contenido del mismo se encuentra en pgres    
        fs.readFile("head.html", function (err, pgres) {
            if (err)
                res.write("HEAD.HTML NOT FOUND");
            else {
                // Las siguientes 3 lineas
                // tienen la función de enviar el archivo html
                // y finalizar el proceso de respuesta
                res.writeHead(200, { 'Content-Type': 'text/html' });
                res.write(pgres);
                res.end();
            }
        });
    }
    else if (url === "/tailPage") {
        fs.readFile("tail.html", function (err, pgres) {
            if (err)
                res.write("TAIL.HTML NOT FOUND");
            else {
                res.writeHead(200, { 'Content-Type': 'text/html' });
                res.write(pgres);
                res.end();
            }
        });
    }

}).listen(port, function () {
    console.log("SERVER STARTED PORT: 8080");
});
```

![Foto 3](../assets/images/practica%203.3/s3.png)

A continuación, creamos el archivo `package.json` con el siguiente comando:

`npm init`

![Foto 4](../assets/images/practica%203.3/s4.png)

Y podemos probar si funciona:

`node aplicacion.js`

Tras hacer eso, deberíamos poder acceder con nuestra máquina anfitriona a `http://IP-maq-virtual:8080`

![Foto 5](../assets/images/practica%203.3/s5.png)
![Foto 6](../assets/images/practica%203.3/s6.png)

## Aplicación para Netlify

Utilizaremos una aplicación de ejemplo para esta práctica.

Simplemente, clonaremos el siguiente repositorio con el comando:

`git clone https://github.com/StackAbuse/color-shades-generator`

![Foto 7](../assets/images/practica%203.3/s7.png)

## Proceso de despliegue en Netlify

Veremos dos métodos de despliegue en Netlify, el primero será el despliegue manual desde el CLI de Netlify, y el otro será con conexión a un código publicado en Github.

Tened en cuenta que debéis haceros una cuenta en [Netlify](https://www.netlify.com/).

## Despliegue mediante CLI

Una vez estemos registrados, instalamos el CLI de Netlify:

`sudo npm install netlify-cli -g`

![Foto 8](../assets/images/practica%203.3/s8.png)

Pedirá autentificación, lo haremos con el comando:

`netlify login`

Como estamos conectados por SSH, podemos generar un token desde Netlify, y lo establecemos como variable de ambiente:

![Foto 9](../assets/images/practica%203.3/s9.png)

![Foto 10](../assets/images/practica%203.3/s10.png)

Y ya podremos logearnos con

`netlify login`

Como buenos desarrolladores, haremos un build de la aplicación para antes de desplegarla.

Primero debemos instalar las dependencias indicadas en el archivo `package.json`:

`npm install`

![Foto 11](../assets/images/practica%203.3/s11.png)

Y cuando lo tengamos procedemos al build:

`npm run build`

![Foto 12](../assets/images/practica%203.3/s12.png)

Esto nos crea la carptea `build` que contendrá la aplicación a desplegar.

Haremos un pre-deploy de la app con el siguiente comando:

`netlify deploy`

![Foto 13](../assets/images/practica%203.3/s13.png)

Nos hará unas cuantas preguntas para el despliegue:

- Indicamos que queremos crear y configurar una nueva site.

- El Team se deja por defecto.

- El nombre para la web será `nombre-practica3-4` y el directorio que utilizaremos será `./build`.

Si nos indica que todo ha ido bien e incluso podemos ver el "borrador" (Website Draft URL) de la web que nos aporta, podemos pasarla a producción finalmente tal y como nos indica la misma salida del comando:

```
If everything looks good on your draft URL, deploy it to your main site URL with the --prod flag.
netlify deploy --prod
```

![Foto 14](../assets/images/practica%203.3/s14.png)

No olvideis desplegar finalmente para poder comprobar que se puede acceder a la URL:

![Foto 15](../assets/images/practica%203.3/s15.png)

## Despliegue mediante conexión con Github

primero eliminamos la site que hemos desplegado en Netlify para evitar conflictos:

![Foto 16](../assets/images/practica%203.3/s16.png)

Luego borramos el directorio donde hemos clonado el repositorio para empezar de 0:

`rm -rf directorio_repositorio`

A continuación, descargamos un `.zip` sin que tenga referencia a Github:

`wget https://github.com/StackAbuse/color-shades-generator/archive/refs/heads/main.zip`

![Foto 17](../assets/images/practica%203.3/s17.png)

Y creamos la carpeta nueva para descomprimir el .zip allí:

```
mkdir practica3.4

unzip main.zip -d practica3.4/
```

![Foto 18](../assets/images/practica%203.3/s18.png)

Ya podemos entrar en la carptea donde está el código:

`cd practica3.4/color-shades-generator-main/`

Ahora deberemos crear un repositorio completamente vacío en Github que se llame `practicaTresCuatro`.

Y tras hacerlo, volvemos a la terminal para iniciar el repositorio que queremos subir:

```
$ git init
$ git add .
$ git commit -m "Subiendo el código..."
$ git branch -M main
```

![Foto 19](../assets/images/practica%203.3/s19.png)

Y ya solo queda hacer referencia al recién creado Github para hacer un `push`:

```
$ git remote add origin https://github.com/username/practicaTresCuatro.git
$ git push -u origin main
```

![Foto 20](../assets/images/practica%203.3/s20.png)

Con el código subido a Github, ahora lo que tenemos que hacer es vincular nuestra cuenta de Github con Netlify. Al hacerlo, pedirá permisos y nosotros lo autorizaremos, pero un detalle a tener en cuenta es que cuando pregunte sobre qué repositorios instalar, solo elegimos el que hemos hecho para la práctica.

![Foto 21](../assets/images/practica%203.3/s21.png)

Y ya lo tendremos todo listo.

![Foto 22](../assets/images/practica%203.3/s22.png)

Luego estaremos listos para desplegar la aplicación.

![Foto 23](../assets/images/practica%203.3/s23.png)

Netlify se encargará de hacer el `build` de forma automática tal y como hemos visto en la imagen de arriba, con el comando `npm run build`, publicando el contenido del directorio `build`.

Con esta conexión, cualquier cambio que hagamos en el proyecto y al que le hagamos un `commit` y `push` desde Github, se actualizará automaticamente en Netlify.

### Comprobación de que funciona así:

- Dentro de la carpeta `public` existe el archivo `robots.txt`, que indica a los rastreadores de los buscadores qué URLs pueden acceder. Se puede acceder a través de la site en sí.

![Foto 24](../assets/images/practica%203.3/s24.png)

- Lo que vamos a hacer es acceder la carpeta `public` donde se encuentra el archivo `robots.txt` para modificarlo, añadiendo `nombre_apellido` en la sección de Disallow.

![Foto 25](../assets/images/practica%203.3/s25.png)

- Haremos un nuevo `commit` y `push` tal y como lo hemos hecho antes.

- Y comprobaremos que tanto en la consola como en Netlify hay un nuevo deploy de la app.

![Foto 26](../assets/images/practica%203.3/s26.png)

![Foto 27](../assets/images/practica%203.3/s27.png)

- Por último, accedemos a la url `https://url_de_la_aplicacion/robots.txt` y comprobamos que el cambio se ha reflejado.

![Foto 28](../assets/images/practica%203.3/s28.png)