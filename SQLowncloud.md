# Mise en place d’un serveur SQL pour ownCloud
Infrastructure BTS SIO 2025-2026 – SISR

Ce tutoriel explique **comment installer et configurer un serveur SQL (MariaDB)** nécessaire au fonctionnement de **ownCloud** dans l’infrastructure réseau décrite.

Le tutoriel est conçu pour être **compréhensible par un débutant** et **adapté à l’architecture réseau BTS SIO**.

---

# 1. Objectif

ownCloud nécessite une **base de données** pour stocker :

- les comptes utilisateurs
- les métadonnées des fichiers
- les partages
- les paramètres de configuration
- les journaux d'activité

La base de données sera installée sur un **serveur SQL dédié**.

Dans l’architecture fournie, ces serveurs existent déjà conceptuellement :

```
SRV SQL1
SRV SQL2
```

Ces serveurs se trouvent dans :

```
VLAN 150 – DMZ INTERNE
```

Cette zone est destinée aux **bases de données internes**, isolées pour des raisons de sécurité.

---

# 2. Positionnement dans l’architecture réseau

Le serveur SQL ne doit **jamais être exposé à Internet**.

Architecture logique :

```
Internet
   |
Routeur Pare-feu
   |
DMZ Externe
   |
Reverse Proxy
   |
DMZ Intermédiaire (VLAN 200)
   |
SRV OWNCLOUD
   |
Pare-feu interne OPNsense
   |
DMZ Interne (VLAN 150)
   |
SRV SQL1
```

Flux autorisé :

```
SRV OWNCLOUD → SRV SQL1 : TCP 3306
```

Flux interdit :

```
Internet → SQL
```

---

# 3. Création de la DMZ interne (VLAN 150) dans OPNsense

Avant de créer le serveur SQL, il faut **créer le réseau DMZ interne sur le pare-feu OPNsense**.

La DMZ interne permettra d’isoler les serveurs de bases de données du reste du réseau.

---

## 3.1 Accès à l’interface d’administration

Depuis un navigateur :

```
https://IP_OPNSENSE
```

Se connecter avec un compte administrateur.

---

## 3.2 Création du VLAN

Menu :

```
Interfaces → Other Types → VLAN
```

Cliquer sur :

```
Add
```

Configurer :

Parent Interface :

```
Interface connectée au switch central
```

VLAN Tag :

```
150
```

Description :

```
DMZ_INTERNE_SQL
```

Cliquer :

```
Save
```

Puis :

```
Apply Changes
```

---

## 3.3 Attribution du VLAN à une interface

Menu :

```
Interfaces → Assignments
```

Dans :

```
Available network ports
```

Sélectionner :

```
VLAN 150
```

Cliquer :

```
Add
```

Une nouvelle interface apparaît :

```
OPT1
```

---

## 3.4 Configuration de l’interface DMZ

Cliquer sur :

```
OPT1
```

Configurer :

Enable interface :

```
✔ activé
```

Description :

```
DMZ_INTERNE_SQL
```

IPv4 Configuration :

```
Static IPv4
```

Adresse :

```
192.168.150.1 /24
```

Cette adresse sera la **passerelle du réseau SQL**.

Sauvegarder :

```
Save
```

Puis :

```
Apply Changes
```

---

## 3.5 Configuration DHCP (optionnel)

Si vous souhaitez distribuer automatiquement les adresses IP :

Menu :

```
Services → DHCPv4 → DMZ_INTERNE_SQL
```

Activer :

```
Enable DHCP server
```

Plage IP :

```
192.168.150.100 – 192.168.150.200
```

Sauvegarder.

---

## 3.6 Création des règles firewall

Menu :

```
Firewall → Rules → DMZ_INTERNE_SQL
```

Ajouter une règle.

Action :

```
Pass
```

Interface :

```
DMZ_INTERNE_SQL
```

Source :

```
SRV OWNCLOUD
```

Destination :

```
SRV SQL1
```

Port :

```
3306
```

Description :

```
Autoriser ownCloud vers SQL
```

Sauvegarder puis :

```
Apply Changes
```

Créer également une règle pour autoriser l’administration via le bastion :

Source :

```
SRV BASTION
```

Destination :

```
DMZ_INTERNE_SQL
```

Ports :

```
22 (SSH)
```

---

# 4. Configuration du switch pour le VLAN 150

Le switch doit également connaître le VLAN.

Exemple sur un switch Cisco :

Créer le VLAN :

```
vlan 150
name DMZ_INTERNE_SQL
```

Configurer le port connecté au serveur SQL :

```
interface FastEthernet0/10
switchport mode access
switchport access vlan 150
```

Configurer le trunk vers le pare-feu :

```
interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 150
```

---

# 5. Création de la machine virtuelle SQL

Sur l’hyperviseur :

```
VMware ESXi02
```

Créer une nouvelle machine virtuelle.

Nom :

```
SRV-SQL1
```

Configuration recommandée :

CPU :

```
2 vCPU
```

RAM :

```
4 Go
```

Disque :

```
100 Go
```

Carte réseau :

```
VLAN 150 – DMZ INTERNE
```

Adresse IP statique :

```
192.168.150.10
```

Passerelle :

```
192.168.150.1
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
Ubuntu Server 20.04 LTS
```

Durant l’installation :

- configurer l’adresse IP statique
- installer **OpenSSH Server**

Connexion au serveur :

```
ssh admin@192.168.150.10
```

Passer en root :

```
sudo -i
```

---

# 7. Mise à jour du système

Toujours mettre à jour le système avant d’installer des services.

```
apt update
apt upgrade -y
```

---

# 8. Installation du serveur MariaDB

MariaDB est un **fork de MySQL** très utilisé dans les environnements Linux.

Installer le serveur :

```
apt install mariadb-server mariadb-client -y
```

Vérifier que le service fonctionne :

```
systemctl status mariadb
```

Le service doit être :

```
active (running)
```

Activer le démarrage automatique :

```
systemctl enable mariadb
```

---

# 9. Sécurisation du serveur SQL

MariaDB fournit un script de sécurisation.

Lancer :

```
mysql_secure_installation
```

Répondre :

```
Enter current password : ENTER
Set root password : Y
New password : motdepassefort
Remove anonymous users : Y
Disallow root login remotely : Y
Remove test database : Y
Reload privilege tables : Y
```

Cela permet de :

- supprimer les utilisateurs anonymes
- empêcher l’accès root à distance
- supprimer la base de test

---

# 10. Vérification du serveur SQL

Connexion à MariaDB :

```
mysql -u root -p
```

Tester :

```
SHOW DATABASES;
```

Vous devriez voir :

```
information_schema
mysql
performance_schema
```

Quitter :

```
EXIT;
```

---

# 11. Configuration réseau du serveur SQL

Par défaut MariaDB écoute seulement sur :

```
127.0.0.1
```

ownCloud étant sur un **serveur différent**, il faut autoriser les connexions réseau.

Modifier la configuration :

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Chercher :

```
bind-address = 127.0.0.1
```

Remplacer par :

```
bind-address = 0.0.0.0
```

Sauvegarder.

Redémarrer MariaDB :

```
systemctl restart mariadb
```

---

# 12. Création de la base de données ownCloud

Connexion :

```
mysql -u root -p
```

Créer la base :

```
CREATE DATABASE owncloud;
```

Créer un utilisateur dédié :

```
CREATE USER 'ownclouduser'@'192.168.200.%' IDENTIFIED BY 'MotDePasseFort';
```

Attribuer les droits :

```
GRANT ALL PRIVILEGES ON owncloud.* TO 'ownclouduser'@'192.168.200.%';
```

Appliquer :

```
FLUSH PRIVILEGES;
```

Quitter :

```
EXIT;
```

---

# 13. Test de connexion depuis le serveur ownCloud

Depuis le serveur :

```
SRV OWNCLOUD
```

Installer le client MySQL :

```
apt install mariadb-client -y
```

Tester la connexion :

```
mysql -u ownclouduser -p -h 192.168.150.10
```

Si la connexion fonctionne, la base SQL est correctement configurée.

---

# 14. Supervision du serveur SQL

Le serveur doit être supervisé par :

```
SRV ZABBIX
```

Éléments à surveiller :

- disponibilité du service MariaDB
- charge CPU
- mémoire
- utilisation disque
- nombre de connexions SQL

---

# 15. Sauvegarde de la base de données

La base doit être sauvegardée régulièrement.

Commande :

```
mysqldump -u root -p owncloud > owncloud_backup.sql
```

Automatiser avec une tâche cron.

Exemple :

```
crontab -e
```

Ajouter :

```
0 2 * * * mysqldump -u root -pMotDePasse owncloud > /backup/owncloud.sql
```

Les sauvegardes doivent être envoyées vers :

```
SRV BACKUP
NAS-SIO2
```

---

# 16. Haute disponibilité (optionnel)

L’infrastructure prévoit :

```
SRV SQL1
SRV SQL2
```

Il est possible de mettre en place :

- réplication MariaDB
- cluster Galera
- bascule automatique

Architecture :

```
SRV OWNCLOUD
     |
     |----> SRV SQL1
     |
     |----> SRV SQL2
```

Cela permet :

- tolérance aux pannes
- meilleure disponibilité

---

# 17. Vérification finale

Vérifier :

- MariaDB fonctionne
- la base owncloud existe
- l’utilisateur ownclouduser fonctionne
- le pare-feu autorise le port 3306
- le serveur ownCloud peut se connecter

Commande utile :

```
systemctl status mariadb
```

---

# 18. Conclusion

Le serveur SQL constitue un **élément critique** pour le fonctionnement de ownCloud.

Dans cette architecture :

- la base est placée dans **la DMZ interne VLAN 150**
- elle est **isolée d’Internet**
- seul le serveur ownCloud peut s’y connecter
- les données sont **sauvegardées et supervisées**

Cette architecture respecte les bonnes pratiques :

- séparation des services
- segmentation réseau
- sécurité des bases de données
- contrôle des flux réseau
