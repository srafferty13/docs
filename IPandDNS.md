# Configuration d'une IP statique et des serveurs DNS

## 1. Configurer l’IP statique dans `/etc/network/interfaces`
```bash
auto ens33                     # démarre l’interface au boot
iface ens33 inet static        # mode statique
    address 172.16.0.3         # l’IP de la VM
    netmask 255.255.255.0      # masque de sous-réseau
    gateway 172.16.0.1         # passerelle par défaut
```

## 2. Configurer les serveurs DNS dans `/etc/resolv.conf`
```bash
nameserver 172.16.0.1          # DNS primaire (OPNsense / passerelle)
nameserver 8.8.8.8             # DNS secondaire (Google public)
```

## 3. Appliquer la configuration
```bash
sudo systemctl restart networking
```
