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
+ --min-rate: Enviará paquetes no más lento que lo que se  le indique
+ -vvv: Más verboso
+ -Pn: Evitamos host discovery mediante el protocolo ARP.
+ -oG: Exportamos la captura a un archivo en formato grepeable. (Tambien se podría exportar a otros formatos -oA: All formats. -oX: Modo XML -oN: Modo nmap)