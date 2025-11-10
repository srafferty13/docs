# Étapes du TP – La supervision avec Zabbix

## 1. Installation et paramétrage de Zabbix
- Importer une machine virtuelle **Debian 12** et lui attribuer **2 Go de RAM**.  
- Configurer l’adresse IP statique de cette machine.  
- Suivre le tutoriel d’installation officiel sur : [https://www.zabbix.com/fr/download](https://www.zabbix.com/fr/download).  
- Une fois l’installation terminée :
  - Accéder à l’interface web à l’adresse : `http://172.16.0.5/zabbix`.  
  - Vérifier la présence de tous les paquets nécessaires.  
  - Renseigner le mot de passe de l’utilisateur **Zabbix**.  
  - Configurer le **nom du serveur** et le **fuseau horaire**.  
  - Se connecter avec le compte **Admin / zabbix**.

---

## 2. Supervision du serveur SRV-AD1 (services DHCP et DNS)
- Installer **Zabbix Agent 2** sur le serveur Active Directory (SRV-AD1).  
- Ajouter l’hôte dans Zabbix :
  - Menu **Configuration > Hôtes > Créer un hôte**.  
  - Modèle : **Windows by Zabbix Agent**.  
  - Vérifier la remontée d’informations (RAM, CPU…).  
- Installer et activer **SNMP** sur Windows Server 2022 à l’aide du tutoriel :  
  [https://www.it-connect.fr/configurer-snmp-sous-windows-server-2012-r2/](https://www.it-connect.fr/configurer-snmp-sous-windows-server-2012-r2/).
- Ajouter un élément SNMP dans Zabbix :
  - Menu **Configuration > Hôtes > Éléments** (SRV-AD1) > **Créer un élément**.  
  - Nom : `DhcpNoAddFree`  
  - OID : `1.3.6.1.4.1.311.1.3.2.1.1.3.172.16.0.0`  
  - Objectif : suivre le nombre d’adresses disponibles dans l’étendue DHCP.  
- Tester la supervision :
  - Activer/désactiver le service DHCP pour observer la variation sur le tableau de bord.  
- Pour le DNS :
  - Créer un nouvel élément testant la résolution de `srv-ad1.sodecaf.fr`.  
  - Tester en activant/désactivant le service DNS.

---

## 3. Supervision du serveur SRV-WEB1 (service Apache2)
- Sur le serveur web :
  - Modifier le fichier `/etc/apache2/mods-enabled/status.conf` selon les instructions.  
  - Redémarrer Apache2.  
  - Installer le paquet **zabbix-agent**.  
  - Configurer le fichier `/etc/zabbix/zabbix-agentd.conf` avec l’IP du serveur Zabbix.
- Dans Zabbix :
  - Modifier le modèle **Apache by Zabbix Agent** (changer `httpd` en `apache2` dans les macros).  
  - Ajouter l’hôte **SRV-WEB1** avec ce modèle.  
  - Vérifier la supervision (temps de réponse, requêtes/s, etc.).

---

## 4. Supervision du routeur pare-feu OPNsense
- Dans OPNsense :
  - Installer les greffons **zabbix-agent** et **net-snmp** (menu Système > Firmware > Greffons).  
  - Activer et configurer l’agent Zabbix :  
    Menu **Services > Agent Zabbix > Paramètres** → indiquer l’IP du serveur Zabbix.  
  - Activer **SNMP** (menu **Services > SNMP**) et définir la communauté.  
  - Ouvrir le port **UDP 161** sur le pare-feu LAN.
- Dans Zabbix :
  - Ajouter l’hôte **OPNsense** avec les modèles :
    - **FreeBSD by Zabbix Agent**  
    - **OPNsense by SNMP**
  - Vérifier la supervision (débits WAN, CPU, etc.) sur le tableau de bord.

---

## 5. Création de déclencheurs
- Menu **Configuration > Hôtes > Déclencheurs** (SRV-AD1) > **Créer un déclencheur**.  
- Exemple : alerter quand le nombre d’adresses DHCP disponibles `< 10`.  
- Affecter une sévérité élevée.  
- Modifier temporairement l’étendue DHCP pour générer un incident.  
- Vérifier le changement d’état du déclencheur sur le tableau de bord.

---

## 6. Configuration des notifications par mail
- Sur le serveur Zabbix :
  - Installer les paquets :
    ```bash
    apt install ssmtp mailutils
    ```
  - Modifier le fichier `/etc/ssmtp/ssmtp.conf` :
    ```
    root=adressemail@gmail.com
    mailhub=smtp.gmail.com:465
    hostname=zabbix
    AuthUser=adressemail@gmail.com
    AuthPass=motdepasse
    UseTLS=YES
    ```
  - Tester l’envoi :
    ```bash
    echo "Ceci est un message de Zabbix" | mail -s "Test Zabbix" adressemail@gmail.com
    ```
  - Si nécessaire, activer la double authentification et créer un mot de passe d’application :
    [https://myaccount.google.com/security](https://myaccount.google.com/security)
- Créer le script d’envoi :
  ```bash
  nano /usr/lib/zabbix/alertscripts/zabbix-sendmail
