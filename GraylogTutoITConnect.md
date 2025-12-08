# Fiche de révision – Tutoriel Graylog sur Debian

## Présentation de Graylog
Graylog est une solution open source de SIEM (Security Information & Event Management).  
Il permet de collecter, stocker et analyser des logs en temps réel via une interface web.

Architecture :  
- Elasticsearch : stockage des logs  
- MongoDB : stockage de la configuration  
- Graylog Server : traitement, interface, alertes

## Architecture Graylog
Sources de logs (Syslog, GELF, Beats…)  
→ Graylog Server  
→ Elasticsearch  
→ Interface Web  
→ Utilisateurs et alertes  
Note : MongoDB stocke la configuration, pas les logs.

---

## Prérequis

### Matériel
- 4 Go RAM (8 Go recommandé)  
- 2 à 4 CPU  
- 20 Go disque minimum

### Logiciels
- Debian 11 ou 12  
- Java OpenJDK 11 ou 17  
- MongoDB 5.0+  
- Elasticsearch 7.10 (pas la 8)

---

## Installation pas à pas

### 1. Préparation système
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apt-transport-https openjdk-17-jre-headless -y
```

### 2. Installation de MongoDB
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/5.0 main" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

sudo apt update
sudo apt install mongodb-org -y
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

### 3. Installation d’Elasticsearch
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

sudo apt update
sudo apt install elasticsearch-oss -y
```

Configuration :
```yaml
cluster.name: graylog
network.host: 127.0.0.1
discovery.type: single-node
```

Démarrage :
```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
curl -X GET "localhost:9200"
```

### 4. Installation de Graylog
```bash
wget https://packages.graylog2.org/repo/packages/graylog-5.1-repository_latest.deb
sudo dpkg -i graylog-5.1-repository_latest.deb
sudo apt update
sudo apt install graylog-server -y
```

### 5. Configuration de Graylog
Génération des clés :
```bash
openssl rand -base64 96 | tr -d '\n'
echo -n "VotreMotDePasse" | sha256sum
```

Éditer :
```bash
sudo nano /etc/graylog/server/server.conf
```

Variables importantes :
```
password_secret = [clé générée]
root_password_sha2 = [hash SHA256]
http_bind_address = 0.0.0.0:9000
http_external_uri = http://VOTRE_IP:9000/
```

Démarrage :
```bash
sudo systemctl start graylog-server
sudo systemctl enable graylog-server
sudo systemctl status graylog-server
```

### 6. Accès web
- URL : http://VOTRE_IP:9000  
- Login : admin  
- Mot de passe : celui utilisé pour le hash  

---

## Configuration initiale

### 1. Inputs
System → Inputs  
Exemples :
- Syslog UDP (1514)
- Syslog TCP
- GELF UDP

### 2. Index Set
System → Indices  
- Prefix : graylog_  
- Rétention : 30 jours

### 3. Streams
Streams → Create  
- Associer un index  
- Créer des règles de routage

---

## Tests

### Envoyer un log de test
```bash
echo "Test log from $(hostname)" | nc -u VOTRE_IP 1514
```

Vérifier dans la Search de Graylog.

---

## Envoi de logs Linux

### Avec rsyslog
```bash
sudo nano /etc/rsyslog.d/90-graylog.conf
```

Contenu :
```
*.* @VOTRE_IP:1514;RSYSLOG_SyslogProtocol23Format
```

TCP :
```
*.* @@VOTRE_IP:1514;RSYSLOG_SyslogProtocol23Format
```

Redémarrer :
```bash
sudo systemctl restart rsyslog
```

---

## Envoi de logs Windows

### Avec NXLog
Configuration :
```xml
<Input in>
    Module im_msvistalog
</Input>
<Output out>
    Module om_udp
    Host VOTRE_IP
    Port 12201
    OutputType GELF_UDP
</Output>
<Route 1>
    Path in => out
</Route>
```

---

## Dashboards

Widgets utiles :
- Message Count  
- Quick Values  
- Field Graph  
- Search Result  

Exemple "Top Sources" :
```
Type: Quick Values
Field: source
Limit: 10
```

---

## Alertes

Exemple : erreurs SSH
```
message:"Failed password" AND application_name:"sshd"
count() > 5 dans les 5 dernières minutes
```

Types :
- Message count  
- Field value  
- Contenu de champ  
- Notifications email, Slack, webhook  

---

## Recherches avancées

Exemples :
```
error OR warning
level:3 OR level:4
message:"connection refused"
source:web* AND message:*error*
timestamp:["2024-01-01 00:00:00" TO "2024-01-01 23:59:59"]
```

---

## Sécurisation

server.conf :
```
http_bind_address = 127.0.0.1:9000
http_enable_tls = true
```

Pare-feu :
```bash
sudo ufw allow 9000/tcp
sudo ufw allow 1514/udp
sudo ufw allow 12201/udp
```

---

## Maintenance

Commandes :
```bash
sudo systemctl status graylog-server elasticsearch mongod
sudo tail -f /var/log/graylog-server/server.log
sudo systemctl restart graylog-server elasticsearch mongod
```

Problèmes courants :  
- Interface inaccessible : vérifier firewall et binding  
- Pas de logs : vérifier Inputs  
- Recherche lente : ajouter RAM  
- Disque plein : gérer la rétention  

Sauvegardes :
```bash
mongodump --out /backup/graylog-config-$(date +%Y%m%d)
```

---

## Checklist d’installation
- NTP synchronisé  
- MongoDB opérationnel  
- Elasticsearch répond sur 9200  
- Graylog accessible  
- Input créé  
- Premier log reçu  
- Mot de passe admin changé  
- Backup configuré  
- Alertes de base  

---

## Objectif final
Une plateforme centralisée permettant la centralisation des logs, la recherche rapide, l’alerte automatique et la génération de rapports.

