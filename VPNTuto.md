### Tutoriel : Mise en place d'un VPN avec Stormshield SNS

Ce tutoriel vous guide à travers la configuration d'un VPN SSL pour les employés nomades, puis d'un tunnel VPN IPSec pour interconnecter deux sites.

---

### Partie A : Création d'un accès VPN SSL/TLS pour les clients nomades

**Objectif :** Permettre à un développeur ou un administrateur d'accéder de manière sécurisée au réseau interne depuis l'extérieur.

#### Étape 1 : Vérification des prérequis

Avant de commencer, assurez-vous que :
- Un annuaire **Active Directory** est configuré et accessible par le pare-feu.
- Les groupes **Admins** et **Developpeurs** existent dans cet annuaire.
- L'interface du pare-feu sur laquelle les clients se connecteront a un profil d'authentification rattaché. (Source: *Fiche 12, p. 3, "Préalables"*).

#### Étape 2 : Configurer le service VPN SSL sur le pare-feu

1.  Dans l'interface d'administration du pare-feu Stormshield, allez dans **Configuration > VPN > VPN SSL**.
2.  Activez le service en cochant **Activer le VPN SSL**.
3.  **Paramètres réseaux :**
    *   Dans le champ **Adresse IP (ou FQDN) de l'UTM utilisé**, entrez l'adresse IP publique du pare-feu (celle qui est accessible depuis Internet).
    *   Dans le champ **Réseaux ou machines accessibles**, sélectionnez ou créez un objet contenant les réseaux internes que les utilisateurs VPN pourront atteindre (par exemple, le LAN et la DMZ). (Source: *Fiche 12, p. 4*).
4.  **Paramètres DNS :**
    *   Indiquez le **Suffixe DNS** de votre domaine.
    *   Renseignez les adresses IP des serveurs **DNS primaire et secondaire**.
5.  **Réseaux clients :**
    *   Définissez les plages d'adresses IP qui seront attribuées aux clients VPN. Respectez les contraintes du TP :
        *   **UDP :** `10.60.0.0/24`
        *   **TCP :** `10.61.0.0/24`
        *(Source: Sujet du TP, p. 2)*.
6.  **Configuration avancée :**
    *   Laissez les ports par défaut (UDP 1194, TCP 443) si votre infrastructure le permet.
7.  **Certificats utilisés :**
    *   Utilisez les certificats par défaut créés par le pare-feu. (Source: *Fiche 12, p. 5*).

#### Étape 3 : Configurer les droits d'accès des utilisateurs

1.  Allez dans **Configuration > Utilisateurs > Droits d'accès**.
2.  Dans la section **VPN SSL**, laissez le paramètre général sur **Interdire**.
3.  Allez dans l'onglet **Accès détaillé** et cliquez sur **Ajouter** pour créer une règle personnalisée.
4.  Créez deux règles distinctes (en les ordonnant correctement) :
    *   **Règle pour les Admins :**
        *   **État :** Actif
        *   **Utilisateur – groupe d'utilisateurs :** Sélectionnez le groupe `Admins`.
        *   **VPN SSL :** `Autoriser`
    *   **Règle pour les Développeurs :**
        *   **État :** Actif
        *   **Utilisateur – groupe d'utilisateurs :** Sélectionnez le groupe `Developpeurs`.
        *   **VPN SSL :** `Autoriser`
*(Source: Fiche 12, p. 5)*.

#### Étape 4 : Ajouter les règles de filtrage

1.  Allez dans **Configuration > Politique de filtrage**.
2.  Ajoutez une règle pour autoriser l'initiation de la connexion VPN depuis Internet :
    *   **Source :** `any` (ou `Internet`)
    *   **Destination :** `IP_publique_du_pare-feu`
    *   **Service :** `UDP 1194` et `TCP 443` (les ports du VPN SSL)
    *   **Action :** `Autoriser`
3.  Ajoutez une règle pour autoriser les utilisateurs VPN à accéder aux ressources internes :
    *   **Source :** Les objets réseaux clients `10.60.0.0/24` et `10.61.0.0/24`
    *   **Destination :** Les objets des ressources internes (LAN, DMZ)
    *   **Action :** `Autoriser`
4.  Ajoutez une règle pour donner accès à Internet aux développeurs (si nécessaire) :
    *   **Source :** Les objets réseaux clients `10.60.0.0/24` et `10.61.0.0/24` (groupe `Developpeurs`)
    *   **Destination :** `any` (ou l'objet `Internet`)
    *   **Service :** `any` (ou `HTTP`, `HTTPS`, etc.)
    *   **Action :** `Autoriser`
*(Source: Fiche 12, p. 6)*.

#### Étape 5 : Configurer un client OpenVPN et tester

1.  **Récupérer la configuration :** Connectez-vous au portail captif du pare-feu (https://adresse_du_pare-feu/auth) avec vos identifiants. Téléchargez le profil VPN (qui est une archive .zip contenant le fichier .ovpn et les certificats). (Source: *Fiche 12, p. 2 & 7*).
2.  **Installer le client :** Téléchargez et installez **OpenVPN Connect** (ou le client Stormshield). (Source: *Fiche 12, p. 7-8*).
3.  **Configurer le client :**
    *   Lancez OpenVPN Connect.
    *   Cliquez sur **FILE** et importez le fichier `openvpn_client.ovpn` que vous avez téléchargé.
    *   Éditez le profil pour enregistrer votre nom d'utilisateur et mot de passe. (Source: *Fiche 12, p. 9*).
4.  **Tester la connexion :** Cliquez sur **CONNECT**. La connexion devrait s'établir. Vérifiez l'adresse IP attribuée (10.60.x.x ou 10.61.x.x) et que vous pouvez accéder aux ressources autorisées (ex: serveur web en DMZ). (Source: *Fiche 12, p. 9*).

#### Étape 6 : Ajouter la double authentification (TOTP)

1.  **Configurer la méthode TOTP :**
    *   Allez dans **Configuration > Utilisateurs > Authentification**, onglet **Méthodes disponibles**.
    *   Cliquez sur **Ajouter une méthode** et sélectionnez **Mot de passe à usage unique (TOTP)**.
    *   Dans le cadre **Mot de passe à usage unique basé sur le temps (TOTP)** , sélectionnez les authentifications compatibles avec le VPN SSL.
    *   Configurez les paramètres des codes (durée de vie, taille, etc.). Laissez les valeurs par défaut si vous n'êtes pas sûr.
    *   Cliquez sur **Appliquer**. (Source: *Note technique TOTP, p. 8*).
2.  **Activer TOTP dans la politique d'authentification :**
    *   Allez dans l'onglet **Politique d'authentification**.
    *   Pour les règles qui concernent vos groupes d'utilisateurs VPN (Admins, Developpeurs), cochez la case dans la colonne **Mot de passe à usage unique**.
    *   Cliquez sur **Appliquer**. (Source: *Note technique TOTP, p. 9*).
3.  **Enrôlement de l'utilisateur :**
    *   L'utilisateur doit se connecter au portail captif (https://adresse_du_pare-feu/auth).
    *   Après s'être authentifié avec son login/mot de passe, la page **Enrôlement TOTP** s'affiche avec un QR code.
    *   L'utilisateur scanne ce QR code avec une application Authenticator (Google Authenticator, etc.) sur son smartphone.
    *   Il saisit ensuite le code à 6 chiffres affiché dans l'application pour valider l'enrôlement. (Source: *Note technique TOTP, p. 10-11*).
4.  **Utilisation du VPN avec TOTP :**
    *   Lors de la prochaine connexion VPN, le client OpenVPN demandera le nom d'utilisateur et le mot de passe.
    *   **Pour le champ mot de passe**, l'utilisateur doit **concaténer son mot de passe AD et le code TOTP**. Exemple : `MotDePasseAD123456`. (Source: *Note technique TOTP, p. 12*).

---

### Partie B : Création d'un tunnel VPN IPSec site-à-site

**Objectif :** Interconnecter deux agences (ex: Montauban et Paris) via un tunnel sécurisé en utilisant une authentification par certificat X.509.

#### Étape 1 : Préparer l'infrastructure (Clonage et paramétrage)

1.  Clonez la VM Stormshield du site de Montauban pour créer celle du site de Paris.
2.  Modifiez les paramètres IP de la nouvelle VM pour qu'ils correspondent au site de Paris (adresses IP des interfaces LAN, WAN, etc.). (Source: *Sujet du TP, p. 4*).

#### Étape 2 : Créer la PKI et les certificats (sur un des pare-feux)

1.  **Créer l'Autorité Racine :**
    *   Sur le pare-feu de Montauban, allez dans **Configuration > Objets > Certificats et PKI**.
    *   Cliquez sur **Ajouter > Autorité racine**.
    *   Remplissez les champs (Nom, etc.) et créez la PKI (ex: `pki.entreprise.fr`). (Source: *Fiche 12, p. 15-16*).
2.  **Générer les certificats serveurs :**
    *   Toujours sur le pare-feu de Montauban, sélectionnez votre nouvelle PKI.
    *   Cliquez sur **Ajouter > Identité serveur**.
    *   Créez un certificat pour le **site de Montauban** (ex: `CN=montauban.entreprise.fr`). (Source: *Fiche 12, p. 16*).
    *   Répétez l'opération pour générer un deuxième certificat pour le **site de Paris** (ex: `CN=paris.entreprise.fr`). (Source: *Fiche 12, p. 18*).

#### Étape 3 : Exporter et importer les certificats

1.  **Exporter le certificat de Paris :**
    *   Sur le pare-feu de Montauban, faites un clic droit sur le certificat de Paris et choisissez **Télécharger > Identité > au format P12**.
    *   Définissez un mot de passe pour protéger la clé privée lors de l'export. (Source: *Fiche 12, p. 18*).
2.  **Importer le certificat sur le pare-feu de Paris :**
    *   Connectez-vous à l'interface d'administration du pare-feu de Paris.
    *   Allez dans **Configuration > Objets > Certificats et PKI**.
    *   Cliquez sur **Ajouter > Importer un fichier**.
    *   Sélectionnez le fichier `.p12` téléchargé et entrez le mot de passe défini lors de l'export. (Source: *Fiche 12, p. 19*).

#### Étape 4 : Configurer le tunnel IPSec sur les deux pare-feux

**Configuration sur le pare-feu de Montauban :**

1.  Allez dans **Configuration > VPN > VPN IPsec**.
2.  Choisissez une politique IPsec libre (ex: "04"), renommez-la (ex: `IPsec-Montauban-Paris`).
3.  Cliquez sur **Ajouter > Tunnel site à site simple** pour lancer l'assistant.
4.  **Ressources locales :** Indiquez le sous-réseau LAN de Montauban.
5.  **Réseaux distants :** Indiquez le sous-réseau LAN du site de Paris.
6.  **Correspondant :** Créez un nouvel objet correspondant avec les paramètres suivants :
    *   **Type d'adresse :** Adresse IP
    *   **Adresse IP :** L'adresse IP publique du pare-feu de Paris.
    *   **Politique IKE :** Par défaut (ou une politique sécurisée).
    *   **Méthode d'authentification :** Sélectionnez le certificat de Montauban que vous avez créé.
    *   **Certificat :** `montauban.entreprise.fr`
    *   **Vérification de l'identifiant de l'homologue :** **Nom du sujet** et renseignez le CN du certificat de Paris (ex: `paris.entreprise.fr`).
7.  **Terminer et activer :**
    *   Une fois l'assistant terminé, activez le tunnel (passer le statut sur **On**).
    *   Définissez une valeur **Keepalive** (ex: 10 secondes) pour maintenir le tunnel actif. (Source: *Fiche 12, p. 20-22*).

**Configuration sur le pare-feu de Paris :**
Répétez la même procédure en inversant les rôles :
1.  **Ressources locales :** Sous-réseau LAN de Paris.
2.  **Réseaux distants :** Sous-réseau LAN de Montauban.
3.  **Correspondant :**
    *   **Adresse IP :** L'adresse IP publique du pare-feu de Montauban.
    *   **Méthode d'authentification :** Sélectionnez le certificat de Paris.
    *   **Certificat :** `paris.entreprise.fr`
    *   **Vérification de l'identifiant de l'homologue :** `montauban.entreprise.fr`.

#### Étape 5 : Appliquer la politique de sécurité

1.  **Phase de test (Pass all) :** Pour tester rapidement la connexion, vous pouvez, sur chaque pare-feu, aller dans **Configuration > Politique de filtrage** et placer une règle `Pass all` en tête de liste. Cela permettra à tous les flux de passer (peu sécurisé).
2.  **Phase finale (Politique restrictive) :**
    *   Supprimez la règle `Pass all`.
    *   Créez des règles de filtrage explicites pour le trafic transitant par le tunnel IPSec.
    *   Par exemple, pour autoriser le trafic du LAN de Paris vers la DMZ de Montauban :
        *   **Source :** `LAN_Paris`
        *   **Destination :** `DMZ_Montauban`
        *   **Action :** `Autoriser`
    *   Pour interdire le trafic du LAN de Paris vers le LAN de Montauban :
        *   **Source :** `LAN_Paris`
        *   **Destination :** `LAN_Montauban`
        *   **Action :** `Interdire`
*(Source: Sujet du TP, p. 4)*.

#### Étape 6 : Tester la solution

*   Utilisez le PC-client du site de Paris pour tenter d'accéder à un serveur dans la DMZ de Montauban (ex: un serveur web). Cela devrait fonctionner.
*   Essayez d'accéder à un serveur dans le LAN de Montauban. Cela devrait échouer.
