# Notes Techniques SISR

## Revision ACL (Cisco)

### Fondements des ACL
- **ACL (Access Control List)**: Liste ordonnée de règles filtrant le trafic réseau.
  - Actions: `permit` (autoriser) ou `deny` (bloquer).
  - Règles lues de haut en bas → première correspondance appliquée.
  - `deny any` implicite si aucune règle `permit` ne correspond.

#### Types d'ACL
| Type       | Plage numéro     | Critères                     | Placement recommandé      |
|------------|------------------|------------------------------|---------------------------|
| Standard   | 1-99, 1300-1999  | IP source uniquement         | Près de la destination    |
| Étendue    | 100-199, 2000-2699 | IP source/dest, port, protocole | Près de la source       |

### Configuration des ACL
#### ACL Standard (numérique)
```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any
```

#### ACL Standard (nommée)
```cisco
ip access-list standard MON_ACL
 permit host 192.168.1.1
 deny 192.168.2.0 0.0.0.255
```

#### ACL Étendue (numérique)
```cisco
access-list 100 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.1 eq 80
access-list 100 deny icmp any any
```

#### ACL Étendue (nommée)
```cisco
ip access-list extended MON_ACL_EXT
 permit udp any host 10.0.0.2 eq 53
 deny ip any any log
```

#### Masque Wildcard
- Inverse du masque de sous-réseau :
  - `0.0.0.255` ≡ /24
  - `0.0.0.0` ≡ une seule IP (`host`)

### Application des ACL
#### Sur une interface
```cisco
interface FastEthernet0/0
 ip access-group 10 in
 ip access-group 20 out
```

#### Sur lignes VTY (SSH/Telnet)
```cisco
line vty 0 4
 access-class 30 in
```

### Gestion et modification
#### Vérification
```cisco
show access-lists
show ip interface f0/0
```

#### Modifier une ACL nommée
```cisco
ip access-list standard MON_ACL
 no 10
 15 permit 192.168.3.0 0.0.0.255
```

#### Supprimer une ACL
```cisco
no access-list 100
no ip access-list extended MON_ACL_EXT
```

### Bonnes pratiques & erreurs
- ✅ Règles spécifiques en premier (ex: `host` avant `any`).
- ✅ Désactiver ACL avant modification : `no ip access-group 10 in`.
- ✅ Journaliser les refus : `deny ip any any log`.
- ⚠ Masque wildcard inversé : `0.0.0.255` ≠ `255.255.255.0`.

### Exemples pratiques
#### Autoriser seulement Web et DNS
```cisco
access-list 110 permit tcp any any eq 80
access-list 110 permit udp any any eq 53
access-list 110 deny ip any any
```

#### Bloquer un réseau sauf ping
```cisco
access-list 120 deny ip 192.168.2.0 0.0.0.255 any
access-list 120 permit icmp any any
```

### Tableau des commandes
| Action                    | Commande                                  |
|---------------------------|-------------------------------------------|
| Créer ACL standard        | `access-list 10 permit 192.168.1.0 0.0.0.255` |
| Créer ACL étendue         | `access-list 100 permit tcp any any eq 80` |
| Appliquer sur interface   | `ip access-group 10 in`                   |
| Voir les ACL              | `show access-lists`                       |
| Supprimer ACL             | `no access-list 10`                       |

---

## Revision B3.4 (Sécurité BDD & Web)

### 1. Gestion des droits d’accès (B3.4.1)
#### MySQL
```sql
CREATE USER 'internaute'@'localhost' IDENTIFIED BY 'MotDePasseComplexe123!';
GRANT SELECT ON Dutoit.* TO 'internaute'@'localhost';
FLUSH PRIVILEGES;
REVOKE UPDATE ON Dutoit.utilisateur FROM 'assistant'@'localhost';
DROP USER 'internaute'@'localhost';
```

#### PostgreSQL
```sql
CREATE ROLE assistant WITH LOGIN PASSWORD 'MotDePasseComplexe123!';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO assistant;
```

#### Bonnes pratiques
- Ne jamais utiliser "root" pour une app web.
- Supprimer les comptes par défaut.
- Vérifier les droits : `SHOW GRANTS FOR 'assistant'@'localhost';`

### 2. Chiffrement et protection des données (B3.4.2)
#### Hachage PHP (bcrypt)
```php
$hash = password_hash("MotDePasse123!", PASSWORD_DEFAULT);
$verif = password_verify("MotDePasse123!", $hash);
```

#### Libsodium (chiffrement symétrique)
```php
$cle = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES);
$nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
$chiffre = sodium_crypto_secretbox("Donnée secrète", $nonce, $cle);
```

#### Pseudonymisation vs Anonymisation
- **Pseudonymisation** : réversible (ex: chiffrer nom/prénom).
- **Anonymisation** : irréversible (ex: supprimer email/téléphone).

### 3. Respect du RGPD (B3.4.3)
- Droits des utilisateurs : information, accès, rectification, portabilité, oubli.
- Consentement : cases décochées par défaut, bannière cookies obligatoire.
- Portabilité (API REST) :
  ```php
  header('Content-Type: application/json');
  echo json_encode(["nom" => "Dupont", "email" => "dupont@exemple.com"]);
  ```

### 4. Prévention des injections SQL (B3.4.4)
#### Requêtes préparées (PDO)
```php
$stmt = $db->prepare("SELECT * FROM users WHERE email = :email");
$stmt->bindValue(':email', $email, PDO::PARAM_STR);
$stmt->execute();
```

#### Validation des saisies
- `filter_var()`, `is_numeric()`
- `htmlspecialchars()` pour protéger les entrées utilisateur.

#### Types d’attaques
- Commentaires : `'admin'--'`
- UNION : `SELECT * FROM produits UNION SELECT * FROM users`
- OR 1=1 : `SELECT * FROM users WHERE user='admin' OR 1=1`

### 5. Sécurité des sessions (B3.4.5)
#### Démarrage et sécurisation
```php
session_start();
$_SESSION['user_id'] = 123;
session_set_cookie_params([
    'lifetime' => 3600,
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'
]);
```

#### Prévention fixation de session
```php
session_regenerate_id(true);
```

#### CSRF
- Utiliser un jeton (token) unique dans les formulaires.

#### Jetons OTP
- Mot de passe à usage unique (ex: SMS, Google Authenticator).

### 6. Bonnes pratiques générales
- ✅ HTTPS obligatoire
- ✅ Mots de passe forts : 12+ caractères, majuscules, chiffres, symboles
- ✅ Journalisation des accès
- ✅ Mises à jour régulières (PHP, OpenSSL, etc.)

---

## Routeur Config (Cisco)
```cisco
version 16.6
service timestamps debug datetime msec
service timestamps log datetime msec
platform qfp utilization monitor load 80
no platform punt-keepalive disable-kernel-core
hostname Router
!
vrf definition Mgmt-intf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
no aaa new-model
!
license udi pid ISR4321/K9 sn FDO23441558
diagnostic bootup level minimal
spanning-tree extend system-id
!
redundancy
 mode none
!
interface GigabitEthernet0/0/0
 no ip address
 shutdown
 negotiation auto
!
interface GigabitEthernet0/0/1
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.10.1
!
interface GigabitEthernet0/0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.10.1
!
interface GigabitEthernet0/0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.10.1
!
interface GigabitEthernet0/0/1.100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.0
 ip helper-address 192.168.10.1
!
interface GigabitEthernet0
 vrf forwarding Mgmt-intf
 no ip address
 shutdown
 negotiation auto
!
ip forward-protocol nd
ip http server
ip http authentication local
ip http secure-server
ip tftp source-interface GigabitEthernet0
!
line con 0
 transport input none
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 login
!
end
```

---

## Fonctions machines AP
- **Srv Web** : héberge site web
- **Honey pot** : leurre
- **WSUS** : déploiement de MAJ
- **AD, DHCP** : Active Directory, distribue les IP
- **IPAM** : gestion des adresses IP
- **Srv WIKI** : site wiki
- **GLPI** : gestion du parc informatique
- **FOG** : déploiement d’images système

---

## Fichiers de conf lieu
- Apache : `nano /etc/apache2/apache2.conf`
- Site GLPI : `nano /etc/apache2/sites-available/glpi.conf`
- SSH : `nano /etc/ssh/sshd_config`

---

## GPT Snort Honeypot Cowrie

### Snort
- IDS/IPS installé sur pfSense pour détecter et bloquer des menaces en temps réel.

### Cowrie (Honeypot SSH)
#### Installation des dépendances
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-venv python3-pip python3-dev libssl-dev libffi-dev build-essential
```

#### Création d’un utilisateur dédié
```bash
sudo adduser --disabled-password cowrie
sudo su - cowrie
```

#### Installation de Cowrie
```bash
git clone https://github.com/cowrie/cowrie
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

#### Configuration
```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
nano etc/cowrie.cfg
```
Modifier :
- `hostname`
- `listen_port` (par défaut 2222)
- `output_jsonlog = true` (logs en JSON)

#### Démarrer Cowrie
```bash
source cowrie-env/bin/activate
./bin/crowrie start
./bin/cowrie status  # Vérification du fonctionnement
```

#### Redirection du port SSH (optionnel)
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

#### Surveillance des logs
```bash
tail -f var/log/cowrie/cowrie.log
ss -tunalp  # permet d'écouter les ports
```

#### Lancer Cowrie en service
Créer un fichier systemd : `sudo nano /etc/systemd/system/cowrie.service`
```
[Unit]
Description=Cowrie SSH Honeypot
After=network.target

[Service]
User=cowrie
WorkingDirectory=/home/cowrie/cowrie
ExecStart=/home/cowrie/cowrie/cowrie-env/bin/python /home/cowrie/cowrie/bin/cowrie start
Restart=always

[Install]
WantedBy=multi-user.target
```
Activer et démarrer le service :
```bash
sudo systemctl enable cowrie
sudo systemctl start cowrie
```

---

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

## GPT Raid
### Types de RAID
1. **RAID 0 (Stripping)** :
   - Données réparties entre plusieurs disques.
   - Améliore les performances mais pas de redondance.

2. **RAID 1 (Mirroring)** :
   - Données dupliquées sur deux disques.
   - Redondance mais performances normales en écriture.

3. **RAID 5 (Stripping avec parité)** :
   - Combine RAID 0 et parité répartie.
   - Reconstruction possible en cas de panne d'un disque.

4. **RAID 10 (RAID 1+0)** :
   - Combine RAID 1 et RAID 0.
   - Redondance et performances élevées.

5. **RAID 50 (RAID 5+0)** :
   - Combinaison de sous-ensembles RAID 5 avec stripping.
   - Capacité et performances avec tolérance aux pannes.

### Cas d'utilisation
- **RAID 0** : Applications traitant de gros fichiers sans haute disponibilité.
- **RAID 1** : Applications nécessitant des données très disponibles.
- **RAID 5** : Bases de données ou applications avec besoins modérés.
- **RAID 10** : Besoins élevés en performance et disponibilité.
- **RAID 50** : Bases de données volumineuses.

---

## Raid (Simplifié)
- **RAID 0** : disques conglomerés (vitesse mais pas de redondance).
- **RAID 1** : disque miroir (copie en cas de perte).
- **RAID 5** : disque partagé avec stripping.
- **RAID 10** : RAID 1 + RAID 0 (sous-système RAID 1 en RAID 0).

---

## Config Switch AP (Cisco)
```cisco
version 15.2
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
hostname Switch
!
username etudiant privilege 15 secret 5 $1$dTBB$62RXUnxRiGkGZ9Ik/ZWic1
no aaa new-model
system mtu routing 1500
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 description Administration
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/2
 description Commerciaux
 switchport access vlan 30
 switchport mode access
!
interface FastEthernet0/3
 description Production
 switchport access vlan 20
 switchport trunk allowed vlan 10,20
 switchport mode access
 switchport port-security
!
interface FastEthernet0/4
 description Serveurs
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/5
 description Sauvegarde
 switchport access vlan 99
 switchport mode access
!
interface Vlan1
 ip address 172.17.1.120 255.255.0.0
!
ip http server
ip http secure-server
!
line con 0
line vty 0 4
 login local
line vty 5 15
 login local
!
end
```

---

## Nat (Cisco)
```cisco
ip route 0.0.0.0 0.0.0.0 20.6.6.1
interface fa0/0
 ip nat inside
interface fa0/1
 ip nat outside
access-list X permit/deny <reseau> <masque inversé>
```

---

## Routage rip (Cisco)
```cisco
router rip
 version 2
 network <réseau>
 passive interface <interface>
 default-information originate
ip route 0.0.0.0 0.0.0.0 <Gateway>
```

---

## Sauvegarde WordPress
### Créer une archive
```bash
tar -cvzf /var/www/save_wp.tar.gz .
```

### Extraire une archive
```bash
tar -xvzf /var/www/save_wp.tar.gz -C /var/www/
```

### Transfert via SCP
```bash
scp root@ip_container:/var/www/save_wp.tar.gz /var/www/
```

---

## Cours Pare-feu
- **Intranet** : réseau contrôlé.
- **Extranet** : réseau externe hors de notre contrôle (internet + réseaux fermés).
- **DMZ** : zone démilitarisée pour services accessibles depuis l'extérieur (ex: serveurs Web, FTP, DNS).
- **Whitelist** : tout bloqué sauf exceptions.
- **Blacklist** : tout accepté sauf exceptions.
- **Firewall stateful** : autorise automatiquement les réponses aux paquets.
- **NAT** : translation d'adresses réseau.
- **PAT** : translation de ports.
- **FTP** : `ftp 192.168.0.254` (ou toute IP).

---

## IPAM AD
- **Gestionnaire de serveur** : renommer l'appareil (IPAM).
- **IPAM IP** : 172.90.10.2
- **Active Directory** : 172.90.10.1
- **Renouvellement SID** : `C:\Windows\system32\sysprep\`
- **Utilisateurs et Ordinateurs sous AD** : activer les fonctionnalités avancées.
- **Partage SYSVOL** : `C:\Windows\SYSVOL\sysvol`

---

## WikiPFSR
- **Root** : usual password
- **Adminer** : Adminer84828482
- **Wordpress Admin** : Wpadmin84828482
- **URL** : http://172.17.1.156
- **Fichiers de configuration** :
  - `glpi.conf` = `pfSR.conf`
  - `glpi-ssr.conf` = `pfSR-ssr.conf`
- **Certificats SSL** :
  - `SSLCertificateFile /opt/ca/pfSR.crt`
  - `SSLCertificateKeyFile /opt/ca/pfSR.key`
  - `SSLCACertificateFile /opt/ca/FLDca.crt`
- **Server name** :
  - `wikiSR.pfsio1.bts-sio.local`
  - `pfSR.pfsio1.bts-sio.local`

---

## Semestre 2 Cours 1 (Sécurité réseau)

### DHCP Snooping
```cisco
ip dhcp snooping
interface fa0/1
 ip dhcp snooping trust
```

### MAC Flooding
- **Outil** : `macof` (génère des adresses MAC pour overflow de la table CAM).
- **Configuration switch** :
  ```cisco
  interface range Fa0/1-24
   switchport mode access
   switchport port-security
   switchport port-security mac-address sticky
   switchport port-security aging time 2
   switchport port-security maximum 4
  ```

---

## Linux commandes
- `cd` : change directory
- `cd ~` : aller dans le répertoire de travail
- `cd ..` : reculer d'un répertoire
- `cd /home/rep_c` ou `cd rep_c`
- `ls -l` : lister les fichiers avec leurs droits
- `id eve` : infos sur l'utilisateur eve
- `pwd` : afficher le dossier courant
- `chown user:groupe chemin` : changer propriétaire/groupe
- `chmod XXX chemin` : changer les droits (R=4, W=2, X=1)
- `chmod u+x chemin` : ajouter droit d'exécution
- `mkdir /home/partage` : créer un répertoire
- `touch /home/partage/fichier` : créer un fichier
- `rm /home/partage` : supprimer (ajouter `-r` pour répertoire)
- `mv cible dest` : déplacer
- `cp cible dest` : copier
- `su eve` : changer d'utilisateur
- `su -eve` : changer d'utilisateur et aller dans son home
- `exit` : quitter une session
- `ssh user@ip` : connexion SSH
- `apt-get update` : mettre à jour la liste des paquets
- `apt-get upgrade` : mettre à jour les paquets
- `apt-install apache2` : installer Apache
- `adduser admin` : ajouter un utilisateur

---

## Slam SQL utilisateur
### Création d’utilisateur
```sql
CREATE USER 'internaute'@'localhost' IDENTIFIED BY 'mmdDpP%ss2';
```

### Suppression d’utilisateur
```sql
DROP USER 'internaute'@'localhost';
```

### Modification du mot de passe
```sql
SET PASSWORD FOR 'internaute'@'localhost' = PASSWORD('pP@sSW0rRDd');
```
