#Mise en oeuvre de l'HTTPS sur un serveur web
# Utilisation d'une autorité de certification insterne

## 1. Préparation de la machine CA
- Configuration IP : /etc/network/interfaces
```bash
iface ens33 inet static
        address 172.16.0.20
        netmask 255.255.255.0
        gateway 172.16.0.254
        dns-nameservers 8.8.8.8
```

- Installation d'openssl 
```bash
apt update apt update && apt upgrade -y
apt install openssl

```
