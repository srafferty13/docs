# Installation et Déploiement de ownCloud dans l’Infrastructure BTS SIO

## 1. Objectif

L’objectif de ce tutoriel est de déployer un **service de partage de fichiers sécurisé** avec **ownCloud** dans l’infrastructure réseau du projet BTS SIO.

ownCloud est une plateforme de stockage et de synchronisation de fichiers similaire à Google Drive ou OneDrive mais **hébergée sur une infrastructure interne**.

Les utilisateurs pourront :

- stocker des fichiers
- partager des documents
- synchroniser leurs données
- accéder aux fichiers via navigateur ou client

---

# 2. Positionnement dans l’Architecture Réseau

Dans l’infrastructure décrite, le serveur ownCloud doit être placé **dans la DMZ intermédiaire (VLAN 200)**.

Raisons :

- Service accessible depuis l'extérieur
- Isolation du réseau interne
- Protection via pare-feu

Architecture logique :

```
Internet
   |
Routeur Pare-feu
   |
DMZ Externe
   |
Reverse Proxy / Load Balancer
   |
DMZ Intermédiaire (VLAN 200)
   |
SRV OWNCLOUD
   |
Pare-feu interne OPNsense
   |
VLAN Serveurs (100)
   |
Serveur SQL
```

Ainsi :

- l'utilisateur externe passe par le **reverse proxy**
- ownCloud est **isolé du réseau interne**
- la base de données peut être **dans le VLAN serveur**

---

# 3. Choix Techniques

Pour ce projet nous utiliserons :

| Composant | Choix |
|-----------|------|
| OS | Ubuntu Server 22.04 LTS |
| Web Server | Apache2 |
| Langage | PHP 7.4 |
| Base de données | MariaDB |
| Application | ownCloud Server |
| Hyperviseur | VMware ESXi02 |

Important : ownCloud **ne supporte pas PHP 8.x**, il faut installer **PHP 7.4**.

---

# 4. Création de la Machine Virtuelle

Sur **VMware ESXi02**

Créer une VM :

Nom :

```
SRV-OWNCLOUD
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
VLAN 200 (DMZ Intermédiaire)
```

Adresse IP statique (exemple) :

```
192.168.200.20
```

DNS :

```
SRV DNS1
SRV DNS2
```

Passerelle :

```
Pare-feu interne OPNsense
```

---

# 5. Installation du système

Installer :

```
Ubuntu Server 22.04 LTS
```

Durant l’installation :

- configurer l’IP statique
- installer **OpenSSH Server**

Connexion au serveur :

```
ssh admin@192.168.200.20
```

Passer en root :

```
sudo -i
```

---

# 6. Mise à jour du système

Toujours commencer par mettre à jour le système.

```
apt update
apt upgrade -y
```

---

# 7. Installation du serveur Web

Installer Apache :

```
apt install apache2 -y
```

Vérifier :

```
systemctl status apache2
```

Tester dans le navigateur :

```
http://IP_DU_SERVEUR
```

---

# 8. Installation de MariaDB

Installer la base de données :

```
apt install mariadb-server -y
```

Sécuriser l'installation :

```
mysql_secure_installation
```

Réponses conseillées :

```
Set root password : YES
Remove anonymous users : YES
Disallow root login remotely : YES
Remove test database : YES
Reload privilege tables : YES
```

---

# 9. Installation de PHP

Ubuntu 22.04 propose PHP 8 mais ownCloud nécessite PHP 7.4.

Ajouter le dépôt PHP :

```
add-apt-repository ppa:ondrej/php -y
```

Mettre à jour :

```
apt update
```

Installer PHP et les modules nécessaires :

```
apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php7.4-intl php7.4-curl php7.4-gd php7.4-xml php7.4-mbstring php7.4-zip php7.4-imagick php7.4-ldap php7.4-apcu php7.4-redis -y
```

Redémarrer Apache :

```
systemctl restart apache2
```

---

# 10. Installation des dépendances supplémentaires

```
apt install smbclient redis-server unzip curl -y
```

---

# 11. Installation de ownCloud

Ajouter le dépôt officiel :

```
echo 'deb http://download.opensuse.org/repositories/isv:/ownCloud:/server:/10/Ubuntu_22.04/ /' | tee /etc/apt/sources.list.d/owncloud.list
```

Ajouter la clé du dépôt :

```
curl -fsSL https://download.opensuse.org/repositories/isv:/ownCloud:/server:/10/Ubuntu_22.04/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/owncloud.gpg > /dev/null
```

Mettre à jour les paquets :

```
apt update
```

Installer ownCloud :

```
apt install owncloud-complete-files -y
```

Les fichiers seront installés dans :

```
/var/www/owncloud
```

---

# 12. Configuration Apache

Créer le fichier de configuration :

```
nano /etc/apache2/sites-available/owncloud.conf
```

Ajouter :

```
Alias /owncloud "/var/www/owncloud/"

<Directory /var/www/owncloud/>
Options +FollowSymlinks
AllowOverride All

<IfModule mod_dav.c>
Dav off
</IfModule>

SetEnv HOME /var/www/owncloud
SetEnv HTTP_HOME /var/www/owncloud

</Directory>
```

Activer le site :

```
a2ensite owncloud.conf
```

Activer les modules nécessaires :

```
a2enmod rewrite headers env dir mime
```

Redémarrer Apache :

```
systemctl restart apache2
```

---

# 13. Création de la base de données

Connexion à MariaDB :

```
mysql -u root -p
```

Créer la base :

```
CREATE DATABASE owncloud;
```

Créer l'utilisateur :

```
CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'MotDePasseFort';
```

Attribuer les droits :

```
GRANT ALL PRIVILEGES ON owncloud.* TO 'ownclouduser'@'localhost';
```

Appliquer :

```
FLUSH PRIVILEGES;
EXIT;
```

---

# 14. Installation initiale de ownCloud

Accéder via navigateur :

```
http://IP_SERVEUR/owncloud
```

Configurer :

Admin :

```
admin
```

Mot de passe :

```
motdepassefort
```

Base de données :

```
MySQL/MariaDB
```

Database user :

```
ownclouduser
```

Database password :

```
MotDePasseFort
```

Database name :

```
owncloud
```

Host :

```
localhost
```

Terminer l'installation.

---

# 15. Sécurisation du serveur

Configurer HTTPS via le **reverse proxy** situé dans la DMZ externe.

Le reverse proxy :

- gère le chiffrement SSL
- protège le serveur ownCloud
- filtre les requêtes externes

Flux :

```
Internet → Reverse Proxy → SRV OWNCLOUD
```

Ports autorisés :

Pare-feu externe :

```
443 → Reverse Proxy
```

Pare-feu interne :

```
Reverse Proxy → SRV OWNCLOUD : 80
```

---

# 16. Configuration Firewall (Paris)

Autoriser les flux nécessaires.

Reverse Proxy vers ownCloud :

```
TCP 80
```

ownCloud vers base SQL :

```
TCP 3306
```

Administration via bastion :

```
SSH 22
```

Flux :

```
SRV BASTION → SRV OWNCLOUD
```

---

# 17. Intégration avec Active Directory (Optionnel)

ownCloud peut être connecté à **Active Directory** pour l’authentification.

Avantages :

- gestion centralisée des comptes
- authentification unique
- synchronisation automatique des utilisateurs

Installer le module LDAP :

```
apt install php7.4-ldap
```

Redémarrer Apache :

```
systemctl restart apache2
```

Configurer dans l’interface :

```
Settings → LDAP / AD integration
```

---

# 18. Sauvegarde

Les sauvegardes doivent être envoyées vers :

```
SRV BACKUP
NAS-SIO2
```

Éléments à sauvegarder :

- base de données
- dossier data ownCloud
- configuration Apache

Sauvegarde base SQL :

```
mysqldump owncloud > backup.sql
```

Sauvegarde fichiers :

```
tar -czvf owncloud-data.tar.gz /var/www/owncloud
```

---

# 19. Supervision

Le serveur doit être supervisé par :

```
Zabbix
```

Éléments à surveiller :

- Apache
- MariaDB
- CPU
- RAM
- espace disque
- disponibilité du service HTTP

---

# 20. Tests de fonctionnement

Tests à réaliser :

Connexion utilisateur.

Upload fichier.

Téléchargement fichier.

Partage utilisateur.

Accès depuis réseau interne.

Accès depuis Internet via reverse proxy.

Test client ownCloud desktop.

---

# 21. Conclusion

Le serveur ownCloud permet :

- centralisation des fichiers
- partage sécurisé entre utilisateurs
- accès distant sécurisé

Dans l’architecture BTS SIO :

- le service est isolé dans une DMZ
- protégé par plusieurs pare-feu
- accessible via reverse proxy
- supervisé par Zabbix
- sauvegardé sur NAS

Cette architecture respecte les objectifs :

- sécurité
- segmentation réseau
- haute disponibilité
- gestion centralisée des données
