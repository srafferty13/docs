## GLPI
### Configuration réseau
```bash
nano /etc/network/interfaces  # set ip
nano /etc/resolv.conf
```
Contenu de `resolv.conf` :
```
domain geltram.intra
search geltram.intra
nameserver 192.168.10.1.8.8.8.8
```

### Installation
```bash
apt update -y
cd /var/www
tar -xzvf glpi-10.x.x.tar.gz
cd /var/lib
mkdir glpi
chown www-data:www-data glpi
```

---
