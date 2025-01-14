# Practica 4.1 Configuración de un servidor DNS

⚠️ Atención: antes de empezar esta práctica, elimina las entradas que se han introducido anteriormente en el archivo de `/etc/hosts` para que no interfiera con la práctica.

## Instalación de servidor DNS

Primero, instalaremos Bind, una herramienta para servidores DNS. Utilizaremos la versión Bind9, que es la recomendada para usarse.

```
sudo apt-get install bind9 bind9utils bind9-doc
```

![Foto 1](../assets/images/practica%204.1/1.png)

## Configuración del servidor

Al ser una práctica de clase, utilizaremos IPv4. Lo configuraremos a Bind a través de un archivo llamado `named`, que se encuentra en el directorio `/etc/default`.

Para indicar que se utilizará IPv4, tenemos que modificar esta línea de la siguiente manera:

```
OPTIONS = "-u bind -4"
```

![Foto 2](../assets/images/practica%204.1/2.png)

Luego, el archivo en cuestión de configuración principal se llama `named.conf` y está en el directorio `/etc/bind`.

Al abrirlo lo que nos encontraremos será lo siguiente:

![Foto 3](../assets/images/practica%204.1/3.png)

### Configuración de named.conf.options

Como es buena práctica que se haga copia de seguridad de ciertos archivos, primero utilizaremos el siguiente comando para copiar:

```
sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
```

![Foto 4](../assets/images/practica%204.1/4.png)

Y a continuación, ya editaremos el archivo `named.conf.options` para añadir las líneas necesarias.

- Por seguridad, solo añadiremos una lista de acceso para los hosts que decidamos. En mi caso, el ordenador que hace de host para la práctica, con IP 192.168.X.0/24 (debemos cambiar la X por el de nuestra red). Debemos añadir la línea antes del bloque de `options {`.

![Foto 5](../assets/images/practica%204.1/5.png)

Debemos tener en mente que el directorio donde se guardarán las zonas por defecto es `/var/cache/bind`.

Añadiremos luego unas líneas justo debajo del pequeño bloque de `forwarders {`.

```
allow-recursion {confiables;};
allow-transfer{none;};
listen-on port 53{192.168.137.31;}; // Aquí debéis usar vuestra IP de la máquina virtual
recursion yes;
```
![Foto 6](../assets/images/practica%204.1/6.png)

Una vez guardada la configuración, podemos comprobar si todo salió bien, con el comando:

```
sudo named-checkconf
```

Si no sale nada, está todo bien.

![Foto 7](../assets/images/practica%204.1/7.png)

Luego, reiniciamos el servidor y comprobamos el estado:

```
sudo systemctl restart named

sudo systemctl status named
```

![Foto 8](../assets/images/practica%204.1/8.png)

### Configuración de named.conf.local

En este archivo lo que configuramos son las zonas. Para esta práctica declaramos la zona `deaw.es`.

Añadimos el siguiente bloque:

```
zone "deaw.es" {
        type master;
        file "/etc/bind/db.deaw.es";
};
```

![Foto 9](../assets/images/practica%204.1/9.png)

### Creación del archivo de zona

Este archivo deberemos crearlo, con el nombre que declaramos en el anterior archivo. En mi caso es `db.deaw.es`.

Se debe hacer de la siguiente manera, respetando el formato:

![Foto 10](../assets/images/practica%204.1/10.png)

En mi caso he puesto las dos IPs, la del host y la de la máquina virtual.

### Creación del archivo de zona para la resolución inversa

Como deben existir dos archivos de zona, uno para resolución directa y otro para la inversa, crearemos otro para la zona inversa.

Primero, debemos tener en cuenta que se tienen que añadir unas líneas más en el archivo anterior de `named.conf.local`, de la misma manera en la que hemos añadido la primera zona para `deaw.es`, donde las X de las IP deben ser sustituídas por las nuestras.

```
zone "X.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.X.168.192";
};
```

![Foto 11](../assets/images/practica%204.1/11.png)

Y tal como hemos hecho antes, también haremos un archivo para la resolución inversa de esta nueva zona, donde la X será sustituida por el final de nuestras IP:

![Foto 12](../assets/images/practica%204.1/12.png)

### Comprobación de las configuraciones

- Para comprobar la configuración de la zona de resolución directa, donde X se cambia por vuestra IP:

```
sudo named-checkzone db.deaw.es db.X.168.192
```

![Foto 13](../assets/images/practica%204.1/13.png)

- Y la comprobación de la resolución inversa:

```
sudo named-checkzone db.X.168.192 db.deaw.es
```

![Foto 14](../assets/images/practica%204.1/14.png)

En caso de que todo esté bien, reiniciamos el servicio de nuevo para comprobar que todo está OK.

![Foto 15](../assets/images/practica%204.1/15.png)

## Comprobación de las resoluciones y de las consultas

Desde los clientes, podemos comprobar las resoluciones directas e inversas con los comandos de dig o nslookup:

- Comprobación usando `nslookup`:

![Foto 16](../assets/images/practica%204.1/16.png)
![Foto 17](../assets/images/practica%204.1/17.png)

- Comprobación usando `dig`:

![Foto 18](../assets/images/practica%204.1/18.png)
![Foto 19](../assets/images/practica%204.1/19.png)

# Cuestiones finales

## Cuestión 1

**¿Qué pasará si un cliente de una red diferente a la tuya intenta hacer uso de tu DNS de alguna manera, le funcionará? ¿Por qué, en qué parte de la configuración puede verse?**

No, si no le has dado permiso, no podrá acceder. La configuración `allow-query` en tu DNS es donde decides quién puede hacer consultas.

---

## Cuestión 2

**¿Por qué tenemos que permitir las consultas recursivas en la configuración?**

Porque sin ellas, tu servidor no podrá buscar dominios fuera de su zona. Básicamente, no podría hacer consultas completas, solo resolver los nombres que ya conoce.

---

## Cuestión 3

**El servidor DNS que acabáis de montar, ¿es autoritativo? ¿Por qué?**

Sí, porque es el encargado de gestionar y dar respuestas sobre las zonas que hemos configurado. Es el jefe en ese dominio.

---

## Cuestión 4

**¿Dónde podemos encontrar la directiva $ORIGIN y para qué sirve?**

Está en los archivos de zona. Sirve para definir el dominio base y evitar que tengas que escribirlo todo el tiempo.

---

## Cuestión 5

**¿Una zona es idéntico a un dominio?**

No son lo mismo. La zona es la parte del dominio que el servidor DNS gestiona, pero no siempre coincide uno a uno con el dominio.

---

## Cuestión 6

**¿Pueden editarse los archivos de zona de un servidor esclavo/secundario?**

No, solo el maestro puede hacer cambios. Los esclavos solo sirven la info que les pasan.

---

## Cuestión 7

**¿Por qué podría querer tener más de un servidor esclavo para una misma zona?**

Para que si uno se cae, otro pueda seguir funcionando. Y también para repartir la carga de consultas.

---

## Cuestión 8

**¿Cuántos servidores raíz existen?**

Hay 13, aunque no son solo 13 físicos, están distribuidos por todo el mundo.

---

## Cuestión 9

**¿Qué es una consulta iterativa de referencia?**

Es cuando tu servidor no tiene la respuesta y le dice al cliente “ve a preguntar a otro servidor”.

---

## Cuestión 10

**En una resolución inversa, ¿a qué nombre se mapearía la dirección IP 172.16.34.56?**

Se mapearía a `56.34.16.172.in-addr.arpa`, que es la forma de hacer una búsqueda inversa en DNS.


