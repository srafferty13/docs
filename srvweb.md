# Mise en oeuvre de l'HTTPS sur un srv web
# Utilisation d'une Autorité de Certification interne

## 1. Préparation de la machine CA
- Configuration IP : /etc/network/interfaces
```bash
allow-hotplug ens33
iface ens33 inet static
		address 172.16.0.20/24
		gateway 172.16.0.254
```
- installation d'openssl
```bash
apt update && sudo apt upgrade -y
apt install openssl
```
