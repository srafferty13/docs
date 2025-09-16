## Cours Snort Honeypot Cowrie

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
