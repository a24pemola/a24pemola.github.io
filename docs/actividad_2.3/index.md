# Practica 2.3
## Proxy inverso con Nginx

### Configuraciones

**Nginx servidor web**

Vamos a clonar nuestra primera máquina de Debian para poder tener dos, siguiendo estas pautas:

- Uno servirá para la web que ya hemos montado en la práctica anterior (Actividad 2.1).
- La nueva máquina clonada se usará para el proxy inverso.
- Realizaremos las peticiones HTTP desde el navegador de nuestra máquina física/anfitrión al proxy clonado, que luego redirigirá al servidor web original.

```
¡OJO! Al clonar las máquinas virtuales porque hay que darle a crear una nueva MAC, de lo contrario no tendréis IP en esa máquina.
```

![Foto 1](../assets/images/practica%202.3/s1.png)

![Foto 2](../assets/images/practica%202.3/s2.png)

Para diferenciarlo mejor, vamos a modificar los servidores para que cada uno haga la petición desde un puerto diferente.

1. Primero, vamos a cambiar el nombre antiguo del servidor web original por el de `webserver`, y eso implica:

- Cambiar el nombre del archivo de configuración de sitios disponibles para Nginx.

- Cambiar el nombre del sitio web dentro de este archivo de configuración donde haga falta.

![Foto 5](../assets/images/practica%202.3/s5.png)

- No os olvidéis de eliminar el link simbólico antiguo con el comando `unlink nombre_del_link` dentro de la carpeta `sites-enabled` y crear el nuevo para el nuevo nombre de archivo.

![Foto 4](../assets/images/practica%202.3/s4.png)

![Foto 6](../assets/images/practica%202.3/s6.png)

![Foto 7](../assets/images/practica%202.3/s7.png)

2. En el archivo de configuración del sitio web, en lugar de hacer que el servidor escuche en el puerto 80, cambiadlo al 8080.

![Foto 3](../assets/images/practica%202.3/s3.png)

3. Reiniciamos Nginx.

![Foto 8](../assets/images/practica%202.3/s8.png)

**Nginx proxy inverso**

Ahora, cuando intentamos acceder a `http://ejemplo-proxy` (o el nombre que tuvieráis de vuestra web de las prácticas anteriores), en realidad estaremos accediendo al proxy, que nos redirigirá a `http://webserver:8080`, el servidor web que acabamos de configurar para que escuche con ese nombre en el puerto 8080. Pero para ello debemos seguir los siguientes pasos:

- Crear un archivo de configuración en sites-available con el nombre ejemplo-proxy (o el que tuvieráis vosotros).

- Este archivo de configuración será más simple, tendrá la siguiente forma.

```
server { 
    listen __; 
    server_name ____________; 
    location / { 
    proxy_pass http://_________:____; 
    } 
} 
```
Donde, mirando el diagrama de red y teniendo en cuenta la configuración hecha hasta ahora, debéis completar:

- El puerto donde está escuchando el proxy inverso

- El nombre de vuestro dominio o sitio web original al que accedemos en el proxy

- La directiva `proxy_pass` indica a dónde se van a redirigir las peticiones, esto es, al servidor web. Por tanto, debéis poner la IP y número de puerto adecuados de vuestro sitio web configurado en el apartado anterior.

- Crear el link simbólico pertinente

![Foto 9](../assets/images/practica%202.3/s9.png)

![Foto 10](../assets/images/practica%202.3/s10.png)

**¡OJO!** En mi caso en concreto he tenido que poner el link en el `proxy_pass` sin el `8080` al final, de otra manera no me llegaba a funcionar.

```
Debéis modificar el archivo host que configurastéis en la práctica 2.1. Si miráis el diagrama de red, ahora el nombre de vuestro sitio web se corresponderá con la IP de la nueva máquina clon que hace de proxy. Será ésta la encargada de redirigirnos automáticamente al verdadero sitio web.
```

![Foto 11](../assets/images/practica%202.3/s11.png)

### Comprobaciones

Deberíais poder acceder a vuestro sitio web sin problemas.

- Comprobad en los access.log de los dos servidores que llega la petición

- Comprobad además la petición y respuesta con las herramientas de desarrollador de Firefox en Xubuntu. Pulsando F12 en el navegador os aparecerán estas herramientas.

![Foto 12](../assets/images/practica%202.3/s12.png)

![Foto 13](../assets/images/practica%202.3/s13.png)

Debéis entrar en Red > luego donde se puede ver la respuesta > y ver la respuesta HTTP como petición GET HTTP (200 OK).

También deberían aparecer las cabeceras, de las cuales podemos añadir algunas nosotros.

**Añadiendo cabeceras**

```
Es importante desactivar el caché para hacer las pruebas siguientes.
```

Como una manera extra de comprobar que todo va bien, podemos añadir las cabeceras de la siguiente manera:

- Para añadir cabeceras, en el archivo de configuración del sitio web debemos añadir dentro del bloque `location / { … }` debemos añadir la directiva: 

```
add_header Host nombre_del_host;
```

1. Añadimos primero esta cabecera solo en el archivo de configuración del proxy inverso. El Nombre_del_host será Proxy_inverso_vuestronombre.

![Foto 3](../assets/images/practica%202.3/s3.png)

2. Reiniciamos Nginx.

3. Comprobamos que podemos acceder al sitio web sin problemas.

4. Con las herramientas de desarrollador comprobamos que el nuevo header aparece en la respuesta.

5. Luego hacemos lo mismo pero con el archivo del servidor web original. Esta vez el `Nombre_del_host` será `servidor_web_vuestronombre`.

Si todo está bien, ahora deberían aparecer los dos.

![Foto 14](../assets/images/practica%202.3/s14.png)

