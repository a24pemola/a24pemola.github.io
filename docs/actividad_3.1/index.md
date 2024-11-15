# Practica 3.1 Instalación de Tomcat

## Instalación de Tomcat

Realizaremos la instalación del servidor de aplicaciones Tomcat 10, en una máquina virtual corriendo Debian 11 Bullseye.

En mi caso, he seguido el siguiente [tutorial](https://youtu.be/kgSDF6neBK0?si=bdG6H0BA6KMLS6si) para instalarlo con el administrador de paquetes.

## Despliegue manual mediante la GUI de administración

Para esta práctica podemos descargar el WAR [aquí](https://tomcat.apache.org/tomcat-6.0-doc/appdev/sample/).

Realizaremos el despliegue manual de una aplicación ya previamente empaquetada en formato WAR. Para ello:

- Nos logueamos con el usuario previamente creado.

- Buscamos la sección que nos permite desplegar un WAR manualmente, seleccionamos nuestro archivo y lo desplegamos.

![Foto 0](../assets/images/practica%203.1/s0.png)

![Foto 1](../assets/images/practica%203.1/s1.png)


## Despliegue con Maven

### Instalación de Maven

Hay dos maneras de instalarlo, con el administrador de paquetes o manualmente. Yo personalmente voy a explicar cómo instalarlo con el administrador de paquetes siguiendo estos sencillos pasos:

Actualizamos los repositorios:

```
sudo apt update
```

Instalamos Maven:

```
sudo apt install maven
```

Comprobamos que funciona bien:

```
mvn --v
```
![Foto 2](../assets/images/practica%203.1/s2.png)

### Configuración de Maven

Necesitamos realizar la configuración adecuada para Maven, modificando los siguientes archivos:

Modificamos el archivo `/etc/tomcat9/tomcat-users.xml` acorde a nuestras necesidades (los nombres de usuario y contraseña deberán ser los que elijáis para vosotros), debe quedar así:

![Foto 3](../assets/images/practica%203.1/s3.png)

Luego, editamos el archivo `/etc/maven/settings.xml` para indicarle a Maven, un identificador para el servidor sobre el que vamos a desplegar (no es más que un nombre, ponedle el nombre que consideréis), así como las credenciales. Todo esto se hará dentro del bloque servers del XML:

![Foto 4](../assets/images/practica%203.1/s4.png)

### Configurar un proyecto de Java Maven

Como para este paso necesitamos un proyecto de Java que utilice Maven, usaremos un repositorio para la práctica y pondremos el patch-1 para cubrir de paso una tarea que se preguntaba más delante:

```
git clone https://github.com/cameronmcnz/rock-paper-scissors.git && cd rock-paper-scissors && git checkout patch-1
```

Despues de hacerlo, debemos modificar el `POM` del proyecto para que haga referencia a que el despliegue se realice con el plugin de Maven para Tomcat.

![Foto 5](../assets/images/practica%203.1/s5.png)

Este es el bloque a añadir:

```
<build>
        <finalName>war-deploy</finalName> 


        <plugins> 
        <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
            <url>http://localhost:8080/manager/text</url> 


            <server>Tomcat.P.3.1</server> 


            <path>/myapp</path> 


        </configuration>
        </plugin>
        </plugins>
</build>
```

## Despliegue

Teniendo ya todo listo para realizar despliegues, ahora crearemos una aplicación Java de prueba para ver si podemos desplegarla sobre la arquitectura que hemos montado. Para ello utilizamos el comando:

```
 mvn archetype:generate -DgroupId=lara -DartifactId=war-deploy -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
 ```

 Tras generar esta aplicación, usaremos un comando final para desplegar y comprobar que se ha buildeado correctamente:

 ```
 sudo mvn tomcat7:deploy
 ```

![Foto 6](../assets/images/practica%203.1/s6.png)

Y, accediendo a través de la GUI, debemos ver que la aplicación está desplegado y que podemos acceder a ella perfectamente:

![Foto 7](../assets/images/practica%203.1/s7.png)

![Foto 8](../assets/images/practica%203.1/s8.png)

# Cuestiones

```
Habéis visto que los archivos de configuración que hemos tocado contienen contraseñas en texto plano, por lo que cualquiera con acceso a ellos obtendría las credenciales de nuestras herramientas.

En principio esto representa un gran riesgo de seguridad, ¿sabrías razonar o averigüar por qué esto está diseñado de esta forma?
```

Se espera que, al instalar Tomcat, solo root y el usuario que lo ejecuta puedan acceder a esos archivos, para proteger las contraseñas, igual que pasa con el archivo /etc/shadow, que solo los administradores pueden ver.