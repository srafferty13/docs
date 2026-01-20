# Installation et configuration de WireGuard sur Debian/Ubuntu

Ce guide décrit les étapes pour installer **WireGuard**, le configurer via le fichier `wg0.conf` et activer son lancement automatique au démarrage du système.

***

## 1. Installation des paquets nécessaires

```bash
sudo apt update
sudo apt install wireguard -y
sudo apt install systemd-resolved -y
```


***

## 2. Création du répertoire de configuration

```bash
sudo mkdir -p /etc/wireguard
```


***

## 3. Création et édition du fichier de configuration

Ouvre le fichier de configuration principal de l’interface `wg0` :

```bash
sudo nano /etc/wireguard/wg0.conf
```


### Exemple de configuration

```ini
[Interface]
PrivateKey = AP1Gnu6KPKp1D0+JMTLhWAoT8Kx0I481A+QQ5bHPFUU=
Address = 172.16.35.21/32
PostUp = resolvectl dns %i 192.168.135.19; resolvectl domain %i ~thorigne.isitix.info ~inside.isitix.info

[Peer]
PublicKey = ppbDsDTTmGqrdfvAjlALaFLjkJl0BMEhiNCCcCPsbSQ=
AllowedIPs = 192.168.35.0/24, 192.168.135.0/24, 192.168.136.0/24
Endpoint = 176.188.254.231:52000
```

> Remplace les clés privées et publiques par celles correspondant à ton propre environnement WireGuard.

***

## 4. Activation du service au démarrage

Active et démarre le service WireGuard correspondant à l’interface `wg0` :

```bash
sudo systemctl enable wg-quick@wg0.service
sudo systemctl daemon-reload
sudo systemctl start wg-quick@wg0.service
```


***

## 5. Redémarrage et vérification

Redémarre le système pour valider le démarrage automatique de l’interface :

```bash
sudo reboot
```

Après redémarrage, vérifie le statut du service :

```bash
sudo systemctl status wg-quick@wg0.service
```

Tu peux aussi consulter les interfaces réseau actives :

```bash
ip a
```
