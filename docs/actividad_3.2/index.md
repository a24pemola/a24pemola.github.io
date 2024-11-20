# Practica 3.2 Despliegue de aplicaciones con Node Express

## ¡OJO!

Debemos comprobar que el servidor de Tomcat que hemos instalado anteriormente está desactivado, o dará problemas:

`sudo systemctl status tomcat10`

Y en caso de que esté activo, se para:

`sudo systemctl stop tomcat10`

## Instalación de Node.js, Express y test de la primera aplicación

Primero, instalaremos Node.js y Express en Debian 11. Luego, crearemos un archivo `.js` de prueba para asegurarnos de que el despliegue funciona.

Personalmente he seguido este [tutorial] (https://web.archive.org/web/20240420163631/https://unixcop.com/how-to-install-expressjs-on-debian-11/) para instalar Node.js.

Para Express, recuerda acceder a `http://IP-maq-virtual:3000` desde tu máquina local, usando la IP de tu máquina virtual, en lugar de `http://localhost:3000`.

Recordad parar el servidor (CTRL+C) al acabar la práctica.

![Foto 1](../assets/images/practica%203.2/s1.png)
![Foto 2](../assets/images/practica%203.2/s2.png)

## Despliegue de una nueva aplicación

Vamos a desplegar una app de terceros como ejemplo. Es un prototipo de predicción meteorológica que está en este repositorio de GitHub.

Lo primero es clonar el repositorio siguiendo las instrucciones que ahí se indican:

```
git clone https://github.com/MehedilslamRipon/Shopping-Cart-Application
```

Nos movemos al nuevo directorio:

```
cd Shopping-Cart-Application/
```

![Foto 3](../assets/images/practica%203.2/s3.png)

Instalamos librerías necesarias:

```
npm install
```

Y por último la iniciamos:

```
npm run start
```

Dará un error de este tipo al intentar arrancar:

```
sh: 1: nodemon: not found
```

![Foto 4](../assets/images/practica%203.2/s4.png)

Para solucionar el problema, instalamos nodemon desde consola y listo.

![Foto 5](../assets/images/practica%203.2/s5.png)
![Foto 6](../assets/images/practica%203.2/s6.png)
![Foto 7](../assets/images/practica%203.2/s7.png)

## Cuestiones

Cuando ejecutáis el comando npm run start, lo que estáis haciendo es ejecutar un script:

```
- ¿Donde podemos ver que script se está ejecutando?
```
En el archivo `package.json`, dentro de la sección `"scripts"`. Ahí se define qué hace el comando `npm run start`.

```
- ¿Qué comando está ejecutando?
```
Depende de lo que esté configurado en `"start"` en el `package.json`. Por ejemplo, podría ser algo como node `server.js`, `nodemon app.js`, o cualquier otro comando que se especifique en esa línea.

