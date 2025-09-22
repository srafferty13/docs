# Résumé : Le DNS (Domain Name System)

## Sujet principal
Explication du fonctionnement du DNS.

## Problème résolu par le DNS
- Les humains retiennent facilement des **noms de domaine** (ex: `youtube.com`).
- Les navigateurs ont besoin d'**adresses IP numériques** (ex: `142.251.42.206`) pour se connecter aux serveurs.
- Le DNS fait la traduction entre les deux.

## Définition du DNS
- DNS signifie **Domain Name System** (Système de Noms de Domaine).
- C'est un système qui **traduit les noms de domaine en adresses IP**.
- Cette traduction s'appelle la **résolution de nom de domaine**.

## Fonctionnement du DNS

### Hiérarchie
Le DNS est organisé comme une arborescence :
- **Racine** : Le niveau le plus haut, représenté par un point (`.`), souvent invisible.
- **Domaines de premier niveau (TLD)** : Juste en dessous de la racine (ex: `.fr`, `.com`, `.org`).
- **Sous-domaines** : Séparés par des points, les domaines les plus importants étant à droite.

### Processus de résolution (exemple avec `fr.wikipedia.org`)
1. Le navigateur contacte un **serveur DNS récursif** (fourni par le FAI ou un service public comme Google DNS).
2. Le serveur récursif effectue une requête **itérative** :
   - Il interroge un **serveur DNS racine**.
   - Le serveur racine l’oriente vers un serveur du **TLD `.org`**.
   - Le serveur `.org` l’oriente vers le serveur faisant **autorité pour `wikipedia.org`**.
   - Ce dernier connaît l’IP du sous-domaine `fr` et la renvoie au serveur récursif.
3. Le serveur récursif transmet l’adresse IP au navigateur.
4. Le navigateur peut alors se connecter au site web.

## En résumé
Le DNS est **l’annuaire de l’internet**.  
Il permet de trouver l’adresse technique (IP) d’un site à partir de son nom facile à retenir (domaine), en suivant une **hiérarchie bien précise**.

[<u>**source**</u>](https://www.youtube.com/watch?v=qzWdzAvfBoo)
