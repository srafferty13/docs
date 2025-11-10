# Étapes du TP – Prise en main du firewall Stormshield SN210W

## 1. Préparation
- Importer les machines virtuelles à partir du réseau SIO (pare-feu Stormshield + machines clientes/serveurs).
- Dans votre hyperviseur (VMware ou VirtualBox), configurer les interfaces de la VM Stormshield (ajouter une interface si nécessaire).
- Démarrer la VM du pare-feu.
- Configurer l’adressage IP des interfaces et le mot de passe administrateur.
- Depuis le PC client Windows, accéder à l’interface web d’administration du pare-feu.
- Mettre à jour le système (OS) et la licence à l’aide des fichiers fournis.

---

## 2. Création des objets réseaux
- Accéder à **Configuration > Objets > Objets réseau** pour consulter les objets prédéfinis.
- Relever l’adresse IP du côté WAN.
- Créer les objets suivants :
  - `LAN_SODECAF` : 172.16.0.0/24  
  - `DMZ_SODECAF` : 192.168.0.0/24  
  - `Fw_LAN_SODECAF` : 172.16.0.254  
  - `Fw_DMZ_SODECAF` : 192.168.0.254  
  - `SRV_WEB` : 192.168.0.1  
  - `SRV_AD` : 172.16.0.1  

---

## 3. Configuration réseau
- Activer la politique de sécurité **Pass all** :  
  `Configuration > Politique de sécurité > Filtrage et NAT > Pass all`
- Aller dans **Configuration > Réseau > Interfaces** :
  - Retirer les interfaces `IN` et `DMZ1` du bridge.
  - Configurer :
    - Interface `IN` → mode interne, IP : 172.16.0.254  
    - Interface `DMZ1` → mode interne, IP : 192.168.0.254
- Configurer la passerelle par défaut :  
  `Configuration > Réseau > Routage > Passerelle par défaut` = 172.17.255.253
- Tester la connectivité entre le LAN (172.16.0.x) et le serveur web DMZ (192.168.0.1).

---

## 4. NAT dynamique (sortant)
- Dans **Filtrage et NAT > NAT**, avec la politique “Pass all” :
  - Créer une règle NAT dynamique :
    - Source : `LAN_SODECAF` sur interface `IN`
    - Destination : `Firewall_out` (adresse publique)
    - Port : attribution dynamique
  - Créer une règle équivalente pour `DMZ_SODECAF` (interface DMZ1).
- Vérifier que les machines du LAN et de la DMZ accèdent à Internet.

---

## 5. NAT statique (entrant)
- Toujours dans **Filtrage et NAT > NAT** :
  - Créer une règle NAT statique pour rendre le serveur web (`SRV_WEB`) accessible depuis l’extérieur.
  - Rediriger le port 80 de l’adresse WAN vers `SRV_WEB` (192.168.0.1).
- Tester l’accès depuis une machine du réseau SIO via l’adresse WAN du pare-feu.

---

## 6. Filtrage
- Copier les règles NAT de la politique “(10) Pass all” vers la politique “(5) Filter 05”.
- Renommer “Filter 05” → “Sodecaf”.
- Appliquer la politique “Sodecaf” (au départ, tout est bloqué).
- Ajouter des règles :
  1. Autoriser le trafic HTTP sur toutes les interfaces.
  2. Permettre :
     - Ping depuis `LAN_SODECAF` uniquement.
     - Navigation HTTP/HTTPS depuis `LAN_SODECAF`, sauf vers les sites russes.
     - Blocage de `www.youtube.com` via un objet DNS FQDN.

---

## 7. Filtrage applicatif (HTTP/HTTPS)
- Vérifier les catégories des sites :  
  `Configuration > Objets > URL > Vérification d’URL`
  - `facebook.com`, `credit-agricole.fr`, `ouest-france.fr`, `ldlc.com`
- Modifier la page de blocage :  
  `Configuration > Notifications > Messages de blocage > BLOCKPAGE_01`  
  → Personnaliser pour SODECAF.
- Créer deux catégories de certificats (CN) :
  - **white-list** : `*.ouest-france.fr`, `*.ldlc.com`
  - **black-list** : `*.facebook.com`, `*.credit-agricole.fr`
- Configurer le filtrage d’URL :  
  `Configuration > Politique de sécurité > Filtrage URL > URLFilter_00`
  - Autoriser tout sauf :
    - Sites Facebook, shopping, news
    - Sauf exceptions (Ouest-France et LDLC autorisés)
- Configurer le filtrage SSL sans déchiffrement :  
  `Configuration > Politique de sécurité > Filtrage SSL > SSLFilter_00`
- Adapter les règles dans **Filtrage et NAT** pour appliquer les filtres URL et SSL.
- Tester :
  - Sites autorisés/interdits
  - Import du certificat proxy :  
    `Configuration > Objets > Certificats et PKI > SSL proxy default authority`
  - Télécharger et importer dans le navigateur client.

---

**Fin du TP :** Le pare-feu est fonctionnel avec segmentation réseau, NAT, filtrage, et filtrage de contenu appliqué.
