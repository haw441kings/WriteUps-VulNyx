# Fase 1- Tanteo

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/6dbe3755-5335-45eb-8ffd-39504fc077ef)

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

1. **22/tcp (SSH):** OpenSSH 9.2p1 Debian 2+deb12u2.
2. **80/tcp (HTTP):** Apache httpd 2.4.57 en Debian.

### Puerto 80

Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Esto llevó a acceder a la página por defecto de Apache en la máquina vulnerable, confirmando así el correcto funcionamiento del servidor web Apache en Debian. La página por defecto de Apache en Debian muestra el mensaje "Apache2 Debian Default Page".

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/5a18a383-bd73-41a6-94a6-29932abfac9d)

# Fase 2- Exploración

### Archivo robots.txt

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/4617c549-5a1c-4ccc-84fd-1fd1cc2fa762)
Al acceder a `http://192.168.52.139/robots.txt`, se encontró la siguiente configuración de restricción para rastreadores web:

### Disallow: /ritedev/

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/45b7d4c4-4579-4150-8013-b666b5574c9c)
Al ingresar a `http://192.168.52.139/ritedev/`, se encontró una página de RiteCMS, confirmando así la presencia y funcionamiento del sistema de gestión de contenidos en la máquina vulnerable.

### Gobuster

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/67130fc3-6d24-4fbf-8183-a606c3f1d6e0)
Al utilizar el comando `gobuster dir -u http://192.168.52.139/ritedev/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,py,sh`, se encontraron los siguientes archivos y directorios:

- `/admin.php` (Status: 200) - Página de administración de RiteCMS.
- `/templates` (Status: 301) - Directorio de plantillas.
- `/media` (Status: 301) - Directorio de medios.
- `/files` (Status: 301) - Directorio de archivos.
- `/data` (Status: 301) - Directorio de datos.
- `/cms` (Status: 301) - Directorio de sistema de gestión de contenidos.

Además, se encontraron archivos `.php` que devolvieron un código de estado 403, indicando acceso prohibido.

### Admin.php

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/07e65a56-6d7a-4860-89c3-6852d8425872)
Al acceder a `http://192.168.52.139/ritedev/admin.php`, se encontró un panel de login, indicando la presencia de un área administrativa protegida por contraseña.

#### Admin-Admin

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/eb1db5ee-105d-484b-955e-cfc36dd89fed)
Se ingresaron las siguientes credenciales:

- Usuario: admin
- Contraseña: admin

Esto permitió acceder con éxito al panel de login, lo que indica que las credenciales por defecto "admin/admin" fueron válidas para ingresar al sistema.

# Fase 3- Explotación

### File Manager

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/5918e574-8135-4691-9db0-2745036ce773)
Durante el proceso de exploración, se identificó la presencia de un file manager en el panel de administración que permite la carga de archivos. Esta funcionalidad representa un posible punto de explotación si se logra colar un archivo malicioso en el sistema.

### Revershell.php

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/693c76d4-4de3-4b22-9747-f50f327eecb5)
Se subio un archivo malicisoso .php que contiene una reverse shell. De forma tal que con el directorio `media` que encontramos anteriormente ejecutamos el archivo .php.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/7c8ce23b-a5d3-4ef9-91b9-67d66613bdda)
Y estamos dentro y somos el usuario www-data ósea que tenemos que realizar escalado de privilegios.

# Fase 4- Privilegios


![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/d5e2e479-cb18-428a-8517-6807ebbb858b)
Se realizo el tratamiento de la TTY. 

### Travis??
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/f153e67d-7f40-41ca-8e14-1038242b22cf)
El usuario `www-data` tiene las siguientes configuraciones predeterminadas en el sistema `load`:

- `env_reset`: Restablecimiento del entorno de ejecución para comandos.
- `mail_badpass`: Envío de correo en caso de intentos fallidos de contraseña.
- `secure_path`: Definición de rutas seguras para la ejecución de comandos.
- `use_pty`: Permite el uso de pseudo terminales para sesiones interactivas.

#### Permisos de Ejecución de Comandos

El usuario `www-data` tiene permisos especiales para ejecutar el siguiente comando en el sistema `load`:

- Comando: `/usr/bin/crash`
- Como Usuario: `travis`
- Sin Requerir Contraseña (`NOPASSWD`)

Esto significa que el usuario `www-data` puede ejecutar el comando `/usr/bin/crash` como el usuario `travis` sin necesidad de proporcionar una contraseña.

#### Ejecución Exitosa de Comando con Privilegios

1. **Usuario:** `travis`
2. **Comando Ejecutado:** `sudo -u travis /usr/bin/crash -h`
3. **Acción Realizada:** Obtención de una shell de bash como el usuario `travis` tras ejecutar `!` seguido de `/bin/bash` en `crash`.
#### Permisos de Sudo para Usuario travis

1. **Configuraciones Predeterminadas:**
    - `env_reset`, `mail_badpass`, `secure_path`, `use_pty`
2. **Permisos de Ejecución:**
    - Comando: `/usr/bin/xauth`
    - Como Usuario: `root`
    - Sin Contraseña (`NOPASSWD`)
    
![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/3c58f594-6471-4dc7-9f77-090018c057a9)
**Comando Ejecutado:** `sudo -u root /usr/bin/xauth source /root/.ssh/id_rsa`

**Resultado de la Ejecución:**
1. Error: `file /root/.Xauthority does not exist`: Indica que el archivo de autoridad `/root/.Xauthority` no se encontró en la ubicación esperada.
2. Errores de Comando Desconocido: `xauth` interpretó los comandos del archivo `.ssh/id_rsa` como comandos desconocidos, lo que sugiere que este archivo no está en el formato esperado por `xauth`. Pero salio lo que queria lol.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/766e41d5-24ed-463f-ab30-a0b86f4e4a85)
**Comando 1:** `cat texto | awk '{print $5}'`

**Resultado 1:**

- "-----BEGIN"
- "MIIEpAIBAAKCAQEAn1xk2mDBXCTen7d97aY7rEVweRUsVE5Zl4sGPG/yXLAAuodz"
- "xjGuAqvTRhG4omhxiJeDr9taOePsIaUGI3Q/qBqUsbnuM/86vu/ANM6+Olzt80fc"
- ...

**Análisis del Resultado 1:**

- El comando `awk '{print $5}'` extrae la quinta columna de cada línea del archivo "texto".
- Se obtienen cadenas de texto que parecen ser partes de una clave RSA privada, como "-----BEGIN" y secuencias de caracteres cifrados.

![image](https://github.com/haw441kings/WriteUps-VulNyx/assets/136659799/73b28ad8-7644-44e6-afa1-dd9e38092d1c)
Finalizada con privilegios de administrador.
