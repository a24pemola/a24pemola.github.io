# Práctica 6.1 - Dockerización del despliegue de una aplicación Node.js

## Despliegue con Docker

Empezaremos con lo básico, debiendo clonar el siguiente repositorio para la práctica:

```
git clone https://github.com/raul-profesor/DAW_practica_6.1_2024.git
```

![Foto](../assets/images/practica%206.1/1.png)

El repositorio en si contiene el `Dockerfile` necesario para construir la imagen y correr el contenedor, pero se hablará sobre él más tarde.

### Instalación de Docker

Antes de seguir, instalaremos Docker en nuestro sistema. Para ello, utilizaremos el siguiente comando:

```
sudo apt install -y docker.io
```

![Foto](../assets/images/practica%206.1/2.png)

### Archivo Dockfile

Ahora que sí tenemos instalado Docker, buscamos el archivo `Dockerfile` del repositorio que hemos clonado, y rellenamos lo que falta, quedando asi:

![Foto](../assets/images/practica%206.1/3.png)

Donde con lo que completamos hará que:

- `RUN mkdir -p /opt/app` : Crea el directorio /opt/app en el contenedor.
- `WORKDIR /opt/app`: Establece /opt/app como directorio de trabajo.
- `COPY src/package.json src/package-lock.json ./`: Copia ficheros package.json y package-lock.json desde el src/ al directorio de trabajo.
- `RUN npm intsall` : Instala las dependencias de la aplicación.
- `COPY src/ .` : Copia el resto del código de la carpeta src/ al contenedor.
- `EXPOSE 3000` : Expone el puerto 3000 para que la app se pueda acceder desde fuera del contenedor en sí.
- `CMD ["npm", "run", "start:dev"]` : Arranca la aplicación en modo desarrollo.

### Construcción de la imagen

En el contexto del directorio actual de trabajo, hacemos el build de la imagen, bajo el nombre de `librodirecciones`.

```
sudo docker build -t librodirecciones .
```

![Foto](../assets/images/practica%206.1/4.png)
![Foto](../assets/images/practica%206.1/5.png)

Lo siguiente será iniciar el contenedor en modo demonio, para ponerlo y que escuche las peticiones del puerto 3000, y que coincida con el puerto 3000 del contenedor en sí.

```
docker run -p 3000:3000 -d librodirecciones
```

![Foto](../assets/images/practica%206.1/6.png)

Si ahora intentamos acceder a nuestro `http://IP_Maq_Virtual:3000`, se produce un error de conexión.

![Foto](../assets/images/practica%206.1/7.png)

## Docker Compose

Para continuar, instalaremos Docker Compose en nuestro sistema, usando el siguiente comando:

```
sudo apt install docker-compose
```

![Foto](../assets/images/practica%206.1/8.png)

Una vez instalado comprobamos la versión:

```
docker-compose --version
```

![Foto](../assets/images/practica%206.1/9.png)

Si no viniese el archivo, se deberá crear un `docker-compose.yml` en el directorio raíz del repositorio que hemos clonado.

Debería tener el próximo contenido:

```
version: "3.9"
services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports: 
      - '5432:5432'
    volumes:
      - addressbook-db:/var/lib/postgresql/data

  addressbook:
    build:
      context: .
    environment:
      DB_SCHEMA: postgres
      DB_USER: postgres
      DB_PASSWORD: postgres
      DB_HOST: postgres
    depends_on:
      - postgres
    ports:
      - '3000:3000'

volumes:
  addressbook-db:
```
![Foto](../assets/images/practica%206.1/10.png)

Antes de iniciar los contenedores, vamos a crear una estructura para la base de datos. Para ello, usamos el siguiente comando:

```
sudo docker-compose run addressbook npm run migrate
```

![Foto](../assets/images/practica%206.1/11.png)
![Foto](../assets/images/practica%206.1/12.png)

Y ahora SÍ podemos iniciar los contenedores:

```
sudo docker-compose up --build -d
```

![Foto](../assets/images/practica%206.1/13.png)

Podemos testear con el siguiente comando para ver que nos da una salida parecida a esto:

```
sudo docker-compose run addressbook npm test
```

![Foto](../assets/images/practica%206.1/14.png)
![Foto](../assets/images/practica%206.1/15.png)
![Foto](../assets/images/practica%206.1/16.png)


## Tarea final

Vamos a probar que la aplicación junto con la BBDD funciona. Para ello probaremos los siguientes comandos:

- `curl -X PUT http://localhost:3000/persons -H 'Content-Type: application/json' -d '{"id": 1, "firstName": "Raúl", "lastName": "Profesor"}'` : Para añadir una persona.
- `curl -X GET http://localhost:3000/persons/all -H 'Content-Type: application/json'` : Para listar todas las personas.
- `curl -X GET http://localhost:3000/persons/1 -H 'Content-Type: application/json'` : Para buscar una persona por ID.
- `curl -X DELETE http://localhost:3000/persons/1 -H 'Content-Type: application/json'` : Eliminar una persona.

Si todos salen bien, deberían salir de esta manera:

![Foto](../assets/images/practica%206.1/17.png)