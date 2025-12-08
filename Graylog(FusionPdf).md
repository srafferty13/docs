# Centralisation des logs avec Graylog

## 1. Installation de Graylog

- VM Debian 12 : 8 Go RAM, 20 Go disque  
- Synchronisation NTP : `ntp.univ-rennes2.fr`  
- Installation via tutoriels IT-Connect + documentation officielle  
- URL d'accès : `http://172.16.0.6:9000`  
- Identifiants : `admin` / mot de passe défini à l’installation  

---

## 2. Configuration NTP sur serveurs

Fichier : `/etc/systemd/timesyncd.conf`
```
[Time]
NTP=ntp.univ-rennes2.fr
FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
```

Commandes :
```
timedatectl set-ntp true
timedatectl set-timezone Europe/Paris
systemctl start systemd-timesyncd
```

Port à ouvrir : **UDP 123**

---

## 3. Dashboard Graylog – Logs web

### Étapes
1. **Créer un Input**  
   `System > Inputs → Raw/Plaintext TCP`  
   Port : 5555, bind : `0.0.0.0`

2. **Créer un Index**  
   `System > Indices → Create index set`  
   Template : *7-days hot*  
   Préfixe : `cybersec-test`

3. **Créer un Stream**  
   Nom : `web-test`  
   Lier à l’index

4. **Associer Stream et Input**  
   `Manage rules → match input → Graylog_TCP_test_Linux`

5. **Importer logs**  
   Envoi via `ncat` vers `172.16.0.6:5555`

6. **Parser les logs**
   - Extractor JSON  
   - GROK pattern :
   ```
   %{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:path} HTTP/%{NUMBER:httpversion}" %{NUMBER:status} (?:%{NUMBER:bytes}|-) "(?:%{URI:referrer}|-)" "%{DATA:useragent}"
   ```

7. **Dashboard (4 widgets)**  
   - Nombre de requêtes (courbe)  
   - Liste des requêtes  
   - Top IP sources (barres)  
   - Requêtes suspectes :
     ```
     path:("/hp-admin" OR "/env") OR status:483
     ```

---

## 4. Logs Cisco (Switch)

### Graylog
- Input : Syslog UDP (ex : port 5148)  
- Index dédié  
- Stream + règle match input

### Cisco (CLI)
```
conf t
service timestamps
logging origin-id string SWITCH-ETAGE2-BAIE1
logging trap notifications
logging host 192.168.10.3 transport udp port 5148
exit
copy running-config startup-config
```

Synchronisation NTP :
```
ntp server 0.fr.pool.ntp.org
```

---

## 5. Logs Linux (Rsyslog)

### Graylog
- Input : Syslog UDP 12514  
- Index + Stream dédiés  

### Serveur Linux
```
apt update && apt install rsyslog
```

Fichier `/etc/rsyslog.d/1-Linux-graylog.conf` :
```
*.* @172.16.0.6:12514;RSYSLOG_SyslogProtocol23Format
```

Redémarrer :
```
systemctl restart rsyslog
```

Recherche SSH échoué :
```
message:Failed password AND application_name:sshd
```

Pour filtrer par machine :
```
AND source:srv-web1
```

---

## 6. Logs Windows (NXLog)

### Graylog
- Input : GELF UDP 12201  
- Index : `Index_win_log`  
- Stream associé  

### Windows (SRV-WIN1)

Installer NXLog CE et modifier :
`C:\Program Files\nxlog\conf\nxlog.conf`
```
<Input in>
  Module im_msvistalog
  <QueryXML>
    <QueryList><Query Id='1'><Select Path='Security'>*</Select></Query></QueryList>
  </QueryXML>
</Input>

<Output graylog_udp>
  Module om_udp
  Host 172.16.0.6
  Port 12201
  OutputType GELF_UDP
</Output>

<Route 1>
  Path in => graylog_udp
</Route>
```

Redémarrer :
```
Restart-Service nxlog
```

GPO d’audit (succès/échec) :
- Validation des informations d’identification  
- Authentification Kerberos  
- Opérations ticket Kerberos  
- Autres événements d’ouverture de session  

Application :
```
gpupdate /force
gpresult /R
```

Test Kerbrute (Kali) :
```
kerbrute bruteuser -d sodecaf.local --dc 172.16.0.1 /usr/share/sqlmap/data/txt/wordlist.txt administrateur
```

Filtrage logs :
```
EventID:4776 OR EventID:4771
```

---

## 7. Logs Stormshield

### Stormshield
Configuration > Notifications > Syslog  
- Profil activé  
- Format RFC5424  
- UDP port 1514  
- Destination : SRV-GRAYLOG  

### Graylog
- Input : Syslog UDP 1514  
- Index dédié  
- Stream + règle match input  

### Content Pack
- Télécharger depuis GitHub  
- Remplacer `firewall.lab.lan` par `VMSNSX09K0639A9` dans le JSON  
- Importer → Dashboard automatique

---

## 8. Alertes par email

### Configuration SMTP Graylog
- Serveur : `smtp.gmail.com:587`  
- Utiliser un mot de passe d’application Google  

Test SMTP :
```
echo "Test" | s-nail -s "Test SMTP" votre_mail@gmail.com
```

### Exemple d’alerte SSH échoué
- Condition :  
  ```
  message:Failed password AND application_name:sshd
  ```
- Notification par email  
- Test : tentative SSH avec mauvais mot de passe

---

## Points clés à retenir
- NTP indispensable pour la cohérence des timestamps  
- Structure Graylog : **Input → Index → Stream**  
- GROK / JSON pour parser et extraire les champs  
- Syslog UDP pour équipements réseau  
- GELF pour Windows (NXLog)  
- Content Packs pour dashboards prêts  
- SMTP nécessaire pour alertes  

## Ressources utiles
- IT-Connect – tutoriels Graylog  
- Documentation officielle Graylog  
- Content Pack Stormshield (GitHub)  
- Tutoriels NXLog Windows  
- Alertes email Graylog (IT-Connect)
