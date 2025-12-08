# Fiche de révision – Wazuh SIEM

## 1. Qu'est-ce qu'un SIEM ?
**SIEM** = *Security Information and Event Management*

Fonctions :
- Collecte, analyse et corrélation des logs
- Détection d’incidents via des règles
- Génération d’alertes et de rapports
- Utilisé dans un SOC

**Avantages :**
- Centralisation des données
- Visibilité globale
- Conformité (rapports)
- Réduction du temps de réponse

**Limites :**
- Nécessite une gestion humaine
- Faux positifs possibles
- Ne répond pas automatiquement aux incidents

---

## 2. Architecture Wazuh
```
[Endpoints] → [Agents Wazuh] → [Wazuh Manager] → [Wazuh Indexer]
      ↓               ↓                     ↓
[Windows/Linux]   [Analyse/Alertes]     [Stockage]
                                    ↓
                              [Wazuh Dashboard]
```

**Composants :**
- Agents : collectent les logs
- Wazuh Manager : analyse, règles, alertes
- Wazuh API : interface de programmation
- Wazuh Indexer : stockage Elasticsearch
- Wazuh Dashboard : interface web
- Filebeat : transfert des données vers l’indexer

---

## 3. Installation Wazuh

### A. Préparation SRV-WEB (Debian)
Configuration NTP :
```bash
timedatectl set-timezone Europe/Paris
# /etc/systemd/timesyncd.conf
[Time]
NTP=ntp.univ-rennes2.fr
timedatectl set-ntp true
systemctl restart systemd-timesyncd
```

Installation DVWA :
```bash
apt install curl
bash -c "$(curl -fsSL https://raw.githubusercontent.com/lamCarron/DVWA-Script/main/Install-DVWA.sh)"
```

### B. Préparation SRV-WAZUH (Ubuntu)

Configurer Netplan :
```yaml
# /etc/netplan/99_config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.10.10.2/24
      routes:
        - to: default
          via: 10.10.10.1
      nameservers:
        search: [mydomain, otherdomain]
        addresses: [10.10.10.1, 1.1.1.1]
```

Appliquer :
```bash
sudo netplan apply
```

Installer Wazuh :
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Accès dashboard :
- URL : https://IP_WAZUH/
- User : admin  
- Password : fourni à la fin de l’installation

Récupération mot de passe :
```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

---

## 4. Installation des Agents Wazuh

### Agent Windows  
Documentation officielle :  
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html

### Agent Linux  
Configurer les ports :
```bash
# Ports à ouvrir : 1514 et 1515
```

### Agent OPNsense  
- Ajouter plugin : `os-wazuh-agent`
- Configurer IP du manager : Services > Agent Wazuh > Paramètres

---

## 5. Fonctionnalités Wazuh

### A. Scan de vulnérabilités
Les agents détectent les CVE :
- Critique / High : niveau 15+
- Medium : 12–14
- Low : 7–11

Format CVE :
- `CVE-AAAA-NNNNNN`
- Exemple : `CVE-2021-44228` (Log4Shell)

---

### B. Détection Brute Force SSH (Active Response)

Configuration :
```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5763</rules_id>
  <timeout>180</timeout>
</active-response>
```

Attaque depuis Kali :
```bash
hydra -t 4 -l etudiant -P /usr/share/wordlists/rockyou.txt.gz 192.168.0.1 ssh
```

Informations :
- Règle Wazuh : **5763**
- MITRE : **T1110** Brute Force
- Active Response : blocage IP 180 s

---

### C. Désactivation compte Linux après échecs
- Verrouillage après 3 échecs en < 2 min
- Déverrouillage automatique après 5 min

Vérification :
```bash
passwd --status user1
```

Détection :
- Règle : **120100**
- MITRE : **T1110.001**

---

### D. Détection d'attaques Web (Teler + Wazuh)

Installation :
```bash
wget https://github.com/teler-sh/teler/releases/download/v2.0.0/teler_2.0.0_linux_amd64.tar.gz
tar -xvzf teler_2.0.0_linux_amd64.tar.gz
mkdir /var/log/teler
```

Configuration Teler :
```yaml
log_format: |
  $remote_addr - - [$time_local] "$request_method $request_url $request_protocol" $status $body_bytes_sent "$http_referer" "$http_user_agent"
logs:
  file:
    active: true
    json: true
    path: /var/log/teler/output.log
```

Configuration Wazuh :
```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/teler/output.log</location>
</localfile>
```

Règles personnalisées :
```xml
<group name="teler,">
  <rule id="100012" level="10">
    <decoded_as>json</decoded_as>
    <field name="category" type="pcre2">Common Web Attack</field>
    <mitre><id>T1210</id></mitre>
  </rule>

  <rule id="100013" level="10">
    <field name="category" type="pcre2">Bad (IP Address|Referrer|Crawler)</field>
    <mitre><id>T1590</id></mitre>
  </rule>

  <rule id="100014" level="10">
    <field name="category" type="pcre2">Directory Bruteforce</field>
    <mitre><id>T1595</id></mitre>
  </rule>
</group>
```

Active Response :
```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100012,100013,100014</rules_id>
  <timeout>360</timeout>
</active-response>
```

---

### E. Détection Réseau avec Suricata (NIDS)

Installation :
```bash
apt update && apt install suricata
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.10/emerging.rules.tar.gz
mkdir -p /etc/suricata/rules/
tar -xvzf emerging.rules.tar.gz
mv rules/*.rules /etc/suricata/rules/
chmod 640 /etc/suricata/rules/*.rules
```

Configuration agent :
```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Test :
```bash
ping -c 30 192.168.0.1
```

---

## 6. Syntaxe règles Suricata
Format :
```bash
action proto src_ip src_port -> dest_ip dest_port (msg:"message"; content:"string"; sid:12345;)
```

Exemple :
```bash
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"SSH connection"; flow:to_server,established; content:"SSH-2.0-OpenSSH"; sid:100001)
```

---

## 7. Commandes utiles

Redémarrer services :
```bash
systemctl restart wazuh-manager
systemctl restart wazuh-agent
```

Gestion agents :
```bash
cd /var/ossec/bin
./manage_agents
```

Problème disque VMware :
```bash
lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

---

## 8. Points clés à retenir
1. Architecture : Agents → Manager → Indexer → Dashboard  
2. Modules sécurité :
   - Scan CVE
   - Détection brute force
   - Active Response
   - Intégration Teler et Suricata
3. MITRE ATT&CK intégré  
4. NTP synchronisé partout  
5. Ports 1514/1515 ouverts  
6. Tester chaque règle  

---

## 9. Objectif final
Mettre en place un SIEM opérationnel capable de :
- Centraliser les logs
- Détecter les attaques
- Réagir automatiquement (blocage IP)
- Produire une visibilité complète
- Intégrer IDS externes

**Métrique :** détection et blocage d’une brute force SSH en moins d’une minute.
