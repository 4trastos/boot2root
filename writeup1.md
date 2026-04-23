# Writeup 1 - Raiders of the Lost Ark (En busca del arca perdida)

## Writeups Map

![map](img/map.png)

## Índice:

- [1. Encontrar la IP](#1-encontrar-la-ip)
- [2. Reconocimiento de servicios](#2-reconocimiento-de-servicios)
- [3. Explorar el servidor web](#3-explorar-el-servidor-web)
- [4. Fuzzing paths de acceso](#4-fuzzing-paths-de-acceso)
- [5. Explorando el foro](#5-explorando-el-foro)
- [6. Logging en el foro](#6-logging-en-el-foro)


## Explotación

### 1. Encontrar la IP

> Ten en cuenta que esta sección puede variar según la configuración de tu red.

Listamos la VM que está corriendo en el host:

```bash
VBoxManage list runningvms
"boot2root" {656d8bc0-eeee-4e29-93b6-574bfd23da96}
```

Intentamos que nos muestre la configuración de red pero no devuelve nada:
```bash
VBoxManage showvminfo boot2root | grep -i network
```

Así que intentamos ver los adaptadores de red configurados y encontramos una información clave `Attachment: Bridged Interface 'wlo1'`:

```bash
VBoxManage showvminfo "boot2root" | grep -i "nic\|nat\|bridge\|host"
CPUProfile:                  host
NIC 1:                       MAC: 0800275EB507, Attachment: Bridged Interface 'wlo1', Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:                       disabled
NIC 3:                       disabled
NIC 4:                       disabled
NIC 5:                       disabled
NIC 6:                       disabled
NIC 7:                       disabled
NIC 8:                       disabled
Name: 'vmbox_share', Host path: '/home/davgalle/Escritorio/VM_TRANSFER' (global mapping), writable, auto-mount, mount-point: '/media/sf_vmbox_share'
    Destination:             File
```

`wlo1` es nuestra interfaz `WiFi`. Con `Bridged` la VM está en la misma red que nuetro host. Eso significa que tiene una IP en el mismo rango que nosotros.

Usamos el comando `ip addr show wlo1` y separamos todo lo que necesitamos:
```bash
ip addr show wlo1
3: wlo1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether dc:fb:48:d1:41:79 brd ff:ff:ff:ff:ff:ff
    altname wlp0s20f3
    inet 192.168.0.19/24 brd 192.168.0.255 scope global dynamic noprefixroute wlo1
       valid_lft 80384sec preferred_lft 80384sec
    inet6 fe80::f34f:7e0b:28ca:96a5/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

```text
Nuetra IP:      192.168.0.19
Rango:          192.168.0.0/24 
```

La VM está en el mismo rango. Puesto que el campus no disponemos de `nmap`,  usamos una función en bash para escanear las IP:
```bash
 for i in {1..254}; do 
  (ping -c 1 -W 1 192.168.0.$i | grep "from" | cut -d " " -f 4 | tr -d ":" &) 
done; wait
192.168.0.1             ←  // router
192.168.0.12
192.168.0.19            ←  // Nuestra IP
192.168.0.25
192.168.0.17
192.168.0.26
192.168.0.27
192.168.0.30
192.168.0.10
```
Descartamos las IP `192.168.0.19` y `192.168.0.1`, que son la nuestra y la del router. Creamos una función en `bash` para escanear las IP restantes en busca de los puertos abiertos y esto nos devuelve la IP `192.168.0.30` que tiene varios puertos abiertos:

```bash
for ip in 10 12 17 25 26 27 30; do
    echo "--- IP 192.168.0.$ip ---"
    for port in 21 22 25 80 443 3306; do
        (timeout 0.5 bash -c "echo > /dev/tcp/192.168.0.$ip/$port" 2>/dev/null) && echo "[+] Puerto $port abierto"
    done
done
--- IP 192.168.0.10 ---
--- IP 192.168.0.12 ---
--- IP 192.168.0.17 ---
--- IP 192.168.0.25 ---
--- IP 192.168.0.26 ---
[+] Puerto 443 abierto
--- IP 192.168.0.27 ---
[+] Puerto 443 abierto
--- IP 192.168.0.30 ---
[+] Puerto 21 abierto
[+] Puerto 22 abierto
[+] Puerto 80 abierto
[+] Puerto 443 abierto
```

### 2. Reconocimiento de servicios

Con la IP identificada exploramos los servicios disponibles en los puertos abiertos:

**Puerto 80 — HTTP:**
```bash
curl -I http://192.168.0.30
HTTP/1.1 200 OK
Server: Apache/2.2.22 (Ubuntu)
Last-Modified: Wed, 07 Oct 2015 23:37:54 GMT
Content-Type: text/html
```

El servidor web es **Apache 2.2.22** corriendo sobre **Ubuntu**.

**Puerto 22 — SSH:**
```bash
nc -vn 192.168.0.30 22
Connection to 192.168.0.30 22 port [tcp/*] succeeded!
SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.7
```

La versión de SSH es **OpenSSH 5.9p1** — versión antigua con posibles vulnerabilidades conocidas.

**Puerto 22 — Banner SSH:**
```bash
ssh -p 22 192.168.0.30
The authenticity of host '192.168.0.30 (192.168.0.30)' can't be established.
ECDSA key fingerprint is SHA256:d5T03f+nYmKY3NWZAinFBqIMEK1U0if222A1JeR8lYE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes 
Warning: Permanently added '192.168.0.30' (ECDSA) to the list of known hosts.
        ____                _______    _____           
       |  _ \              |__   __|  / ____|          
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___ 
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__ 
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
davgalle@192.168.0.30's password: 
```

El servidor pide credenciales que aun no tenemos.

**Resumen del servidor:**

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 21 | FTP | ? |
| 22 | SSH | OpenSSH 5.9p1 |
| 80 | HTTP | Apache 2.2.22 |
| 443 | HTTPS | Apache 2.2.22 |

### 3. Explorar el servidor web

Exploramos `https` en el terminal y recibimos `404 Not Found`. Aunque parece que hay algo configurado pero no encuentra el recurso raíz.:
```bash
davgalle@davgalle-Latitude-5400:~/Documents/RNCP7/boot2root$ curl -kI https://192.168.0.30
HTTP/1.1 404 Not Found
Date: Thu, 23 Apr 2026 16:56:19 GMT
Server: Apache/2.2.22 (Ubuntu)
Vary: Accept-Encoding
Content-Type: text/html; charset=iso-8859-1
```

Exploramos `http` en el navegador y nuestra una página de "coming soon" aparentemente vacia:

![HTTP index](img/http_index.png)

Exploramos el código fuente de `http` en busca de **directorios ocultos** y no vemos nada fuera de lo normal ni directorios ocultos. Los **enlaces son reales**.   
```bash
 curl http://192.168.0.30
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	<title>Hack me if you can</title>
	<meta name='description' content='Simple and clean HTML coming soon / under construction page'/>
	<meta name='keywords' content='coming soon, html, html5, css3, css, under construction'/>	
	<link rel="stylesheet" href="style.css" type="text/css" media="screen, projection" />
	<link href='http://fonts.googleapis.com/css?family=Coustard' rel='stylesheet' type='text/css'>

</head>
<body>
	<div id="wrapper">
		<h1>Hack me</h1>
		<h2>We're Coming Soon</h2>
		<p>We're wetting our shirts to launch the website.<br />
		In the mean time, you can connect with us trought</p>
		<p><a href="https://fr-fr.facebook.com/42Born2Code"><img src="fb.png" alt="Facebook" /></a> <a href="https://plus.google.com/+42Frborn2code"><img src="+.png" alt="Google +" /></a> <a href="https://twitter.com/42born2code"><img src="twitter.png" alt="Twitter" /></a></p>
	</div>
</body>
</html>
```

Como sabemos que el servidor es Apache.  En Apache hay algunos archivos y directorios que existen por defecto o son muy comunes en cualquier servidor web:

- `robots.txt` — le dice a los buscadores qué no indexar
- `sitemap.xml` — mapa del sitio
- `.htaccess` — configuración de Apache
- `server-status` — estado del servidor

Probamos con todos en el puerto `80` y el `443` y no encontramos nada de información importante salvo por un detalle:

- `curl http://192.168.0.30/.htaccess` -> Devuelve `403`
- `curl -k https://192.168.0.30/.htaccess` -> Devuelve `404`

```bash
curl http://192.168.0.30/.htaccess
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /.htaccess
on this server.</p>
<hr>
<address>Apache/2.2.22 (Ubuntu) Server at 192.168.0.30 Port 80</address>
</body></html>
```
```bash
curl -k https://192.168.0.30/.htaccess
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /.htaccess was not found on this server.</p>
<hr>
<address>Apache/2.2.22 (Ubuntu) Server at 192.168.0.30 Port 443</address>
</body></html>
```

Esta variación significa que en `HTTPS` la configuración es diferente al `HTTP`.

Los certificados SSL autofirmados a veces revelan nombres de dominio, emails o información del servidor en los campos:

```bash
openssl s_client -connect 192.168.0.30:443 2>/dev/null | openssl x509 -noout -text | grep Subject
Subject: CN = BornToSec
Subject Public Key Info:
```
El certificado SSL es **autofirmado** y revela el nombre `CN = BornToSec` —
el mismo nombre que aparece en el prompt de login de la VM. Esto sugiere que
hay un **virtual host** configurado con ese nombre en Apache.

Sin acceso a `/etc/hosts` en el campus, pasamos el header `Host` directamente
con `curl` para forzar la resolución del virtual host:

```bash
curl -k -H "Host: BornToSec" https://192.168.0.30/
HTTP/1.1 404 Not Found
address>Apache/2.2.22 (Ubuntu) Server at borntosec Port 443
```

El servidor responde como `borntosec` (en minúsculas) — confirma que el virtual
host existe. Probamos los mismos archivos comunes pero sin resultado.

**Resumen final del reconocimiento:**

| Puerto | Servicio | Versión | Notas |
| --- | --- | --- | --- |
| 21 | FTP | vsftpd | Anónimo configurado pero con error de permisos |
| 22 | SSH | OpenSSH 5.9p1 | Versión antigua — pide credenciales |
| 80 | HTTP | Apache 2.2.22 | Página "coming soon" — `.htaccess` existe (403) |
| 443 | HTTPS | Apache 2.2.22 | Virtual host `borntosec` — certificado autofirmado |


### 4. Fuzzing paths de acceso

Una vez que el reconocimiento está completo. Vamos a explorar los directorios sin el comando `dirb` porque no se encuentra instalado en el campus. 

Creamos un script bash con los directorios mas comunes de las webs. Las wordlists más comunes están en `/usr/share/wordlists/` en `Kali` o similares.

Deescubirmos que `/forum/` está bloqueado en `HTTP (403)` pero accesible en `HTTPS`.

```bash
for dir in admin login wp-admin phpmyadmin forum blog uploads files backup webmail tmp test; do
    code=$(curl -s -o /dev/null -w "%{http_code}" http://192.168.0.30/$dir/)
    echo "$code http://192.168.0.30/$dir/"
done
404 http://192.168.0.30/admin/
404 http://192.168.0.30/login/
404 http://192.168.0.30/wp-admin/
404 http://192.168.0.30/phpmyadmin/
403 http://192.168.0.30/forum/       ←
404 http://192.168.0.30/blog/
404 http://192.168.0.30/uploads/
404 http://192.168.0.30/files/
404 http://192.168.0.30/backup/
404 http://192.168.0.30/webmail/
404 http://192.168.0.30/tmp/
404 http://192.168.0.30/test/
```

Lo probamos en el navegador `https://192.168.0.30/forum/`, nos muetra la página del foro y vemos que podemos extraer información muy importante.

![HTTPS forum](img/https_forum.png)

### 5. Explorando el foro

El foro es **"my little forum 2.3.4"**. Extraemos información muy valiosa:

**6 usuarios registrados:**

| Usuario | Rol |
| --- | --- |
| `admin` | Administrador |
| `lmezard` | Usuario |
| `qudevide` | Usuario |
| `zaz` | Usuario |
| `wandre` | Usuario |
| `thor` | Usuario |

**Hilos publicados:**

| ID | Título | Autor |
| --- | --- | --- |
| 1 | Welcome to this new Forum ! | `admin` |
| 6 | Probleme login ? | `lmezard` |
| 4 | Gasolina | `qudevide` |
| 2 | Les mouettes ! | `wandre` |

El hilo más interesante es **"Probleme login ?"** publicado por `lmezard`. Puede contener credenciales o pistas de acceso.

Después de revisar todo el hilo **"Probleme login ?"** encontramos un registro de un error de inicio de sesión, en el que parece el típico error en el que el usuario a puesto el password `!q\]Ej?*5K5cy*AJ` en el campo del usuario.

### 6. Logging en el foro

Y más abajo el usuario `lmezard` se loguea correctamente:

![HTTPS password](img/password.png)

Probamos a logueranos en el blog en el navegador con las credenciales que hemos encontrado:

![HTTPS back](img/back.png)

![HTTPS back](img/loging_01.png)

![HTTPS back](img/loging_02.png)

¡¡¡Y ya estamos dentro!!!

![HTTPS back](img/loging_03.png)


Buscando en el perfil del usuairo descubrimos su email `laurie@borntosec.net`. Y con esa información descubrimos dos cosas:

![HTTPS back](img/mail_01.png)

- El nombre real de `lmezard` es `Laurie`
- El dominio es `borntosec.net` (que coincide con el virtual host borntosec que encontramos antes) 

Ahora que tenemos su cuenta de correo vamos a probar a loguearnos en `webmail`.
```text
https://192.168.0.30/webmail
```

![HTTPS back](img/webmail_01.png)

![HTTPS back](img/webmail_02.png)