# Déploiement d’une authentification 802.1X avec PacketFence et serveur RADIUS
Infrastructure BTS SIO 2025-2026 – SISR

Ce tutoriel explique comment mettre en place une **authentification réseau sécurisée 802.1X** à l’aide de **PacketFence** afin de contrôler l’accès au réseau filaire et WiFi de l’entreprise.

Le système s’appuie sur :

- protocole 802.1X
- serveur RADIUS
- solution NAC PacketFence
- Active Directory
- switch d’accès
- pare-feu OPNsense

---

# 1. Objectif du TP

L’objectif est de déployer une solution permettant :

- d’authentifier les utilisateurs réseau
- de contrôler l’accès aux ports réseau
- de centraliser l’authentification
- d’utiliser Active Directory comme base d’utilisateurs
- de tracer les connexions réseau

Cette solution est basée sur :

```
PacketFence (NAC + RADIUS)
802.1X
Active Directory
```

---

# 2. Principe de fonctionnement de 802.1X

Le protocole **802.1X** permet de contrôler l’accès à un port réseau.

Un port reste **bloqué tant que l’utilisateur n’est pas authentifié**.

Trois composants interviennent :

### Supplicant

Le client qui souhaite accéder au réseau.

Exemples :

```
PC utilisateur
ordinateur portable
smartphone
```

---

### Authenticator

L’équipement réseau qui contrôle l’accès.

Exemples :

```
switch
point d’accès WiFi
```

---

### Serveur d’authentification

Serveur qui vérifie l’identité de l’utilisateur.

Dans ce TP :

```
PacketFence (serveur RADIUS)
```

---

# 3. Architecture de la solution

Architecture logique :

```
Utilisateur
     |
     | 802.1X
     |
Switch (Authenticator)
     |
     | RADIUS
     |
PacketFence
     |
     | LDAP / Kerberos
     |
Active Directory
```

---

# 4. Positionnement dans l’architecture BTS SIO

Le serveur PacketFence doit être placé dans :

```
VLAN 100 – VLAN SERVEURS
```

Nom de la machine :

```
SRV PACKETFENCE
```

Flux réseau nécessaires :

```
Switch → PacketFence : UDP 1812 (RADIUS)
Switch → PacketFence : UDP 1813 (Accounting)

PacketFence → Active Directory : LDAP 389
PacketFence → AD : Kerberos 88

Administration :
Bastion → PacketFence : SSH 22
```

---

# 5. Création de la machine virtuelle PacketFence

Sur :

```
VMware ESXi02
```

Créer une VM :

Nom :

```
SRV-PACKETFENCE
```

Configuration :

CPU

```
2 vCPU
```

RAM

```
8 Go
```

Disque

```
100 Go
```

Carte réseau

```
VLAN 100
```

Adresse IP exemple :

```
192.168.100.40
```

DNS :

```
SRV DNS1
SRV DNS2
```

---

# 6. Installation du système

Installer :

```
Debian 12
```

Configurer :

- IP statique
- OpenSSH

Connexion :

```
ssh admin@192.168.100.40
```

---

# 7. Synchronisation NTP

La synchronisation horaire est obligatoire pour Kerberos.

Installer NTP :

```
apt install chrony
```

Configurer le même serveur NTP que :

```
Active Directory
```

---

# 8. Installation de PacketFence

Mettre à jour le système :

```
apt update
apt upgrade -y
```

Installer PacketFence selon la documentation officielle.

Télécharger le dépôt PacketFence :

```
https://packetfence.org/doc/PacketFence_Installation_Guide.html
```

Lancer l’installation.

Pendant l’installation définir :

```
domaine : SODECAF.LOCAL
```

---

# 9. Accès à l’interface PacketFence

Accéder via navigateur :

```
https://IP_PACKETFENCE:1443
```

Se connecter avec :

```
admin
```

Mot de passe défini lors de l’installation.

---

# 10. Création du compte AD pour PacketFence

Sur le serveur Active Directory.

Créer un utilisateur :

```
packetfence
```

Dans l’unité d’organisation :

```
OU = Services
```

Ce compte servira à :

```
connecter PacketFence à Active Directory
```

---

# 11. Configuration DHCP

Configurer les étendues DHCP sur :

```
SRV DHCP1
SRV DHCP2
```

Créer une plage pour les postes utilisateurs.

Exemple :

```
192.168.20.100
192.168.20.200
```

Passerelle :

```
OPNsense
```

DNS :

```
SRV DNS1
SRV DNS2
```

---

# 12. Configuration du pare-feu OPNsense

Dans l’interface :

```
Firewall → Rules
```

Autoriser les flux RADIUS :

Source :

```
Switch
```

Destination :

```
SRV PACKETFENCE
```

Ports :

```
UDP 1812
UDP 1813
```

Autoriser également :

```
PacketFence → Active Directory
LDAP
Kerberos
```

---

# 13. Configuration du commutateur

Accéder au switch Cisco.

Activer AAA :

```
aaa new-model
```

Configurer le serveur RADIUS :

```
radius-server host 192.168.100.40 key radiussecret
```

Configurer l’authentification 802.1X :

```
aaa authentication dot1x default group radius
```

Activer le service 802.1X :

```
dot1x system-auth-control
```

Configurer un port :

```
interface FastEthernet0/10
switchport mode access
authentication port-control auto
dot1x pae authenticator
```

---

# 14. Liaison PacketFence avec Active Directory

Dans l’interface PacketFence :

Menu :

```
Configuration
Politiques et contrôle d'accès
Sources d'authentification
```

Ajouter :

```
Active Directory
```

Configurer :

Serveur LDAP :

```
IP du contrôleur de domaine
```

Utilisateur :

```
packetfence
```

Mot de passe :

```
mot de passe AD
```

Domaine :

```
SODECAF.LOCAL
```

---

# 15. Configuration de la méthode d’authentification

Configurer l’authentification :

```
PEAP
```

PEAP permet :

```
authentification sécurisée via tunnel TLS
```

---

# 16. Configuration du client Windows

Sur un poste utilisateur :

Ouvrir :

```
Paramètres réseau
```

Configurer :

```
Authentification IEEE 802.1X
```

Méthode :

```
PEAP
```

Entrer :

```
identifiant Active Directory
mot de passe
```

---

# 17. Tests de fonctionnement

Tester :

Connexion avec utilisateur AD valide.

Connexion avec mauvais mot de passe.

Connexion depuis un nouveau poste.

Vérifier les logs PacketFence.

---

# 18. Journalisation

Logs disponibles dans :

```
/usr/local/pf/logs/
```

Ces logs permettent de vérifier :

```
authentifications réussies
échecs d'accès
équipements connectés
```

---

# 19. Supervision

Le serveur doit être supervisé par :

```
SRV ZABBIX
```

Surveiller :

```
service packetfence
CPU
RAM
réseau
```

---

# 20. Conclusion

La solution PacketFence permet de déployer un **NAC complet basé sur 802.1X**.

Elle permet :

- authentification centralisée
- contrôle d’accès réseau
- intégration Active Directory
- traçabilité des connexions

Dans l’infrastructure BTS SIO :

```
Switch → contrôle d'accès
PacketFence → serveur RADIUS
Active Directory → base utilisateurs
OPNsense → filtrage réseau
```

Cette architecture renforce considérablement la **sécurité du réseau de l’entreprise**.
