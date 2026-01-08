# Workflow Git : branches, Pull Requests et merges

Les branches et les merges forment le cœur d’un workflow Git/GitHub propre pour travailler à plusieurs sur le même projet.

***

## 1. Principe des branches et du merge

- Une **branche** est une ligne de développement indépendante, dérivée en général de `main`. On y fait des modifications sans impacter immédiatement le code principal.
- Un **merge** consiste à intégrer les commits d’une branche (par exemple `feature/x`) dans une autre (souvent `main`).
- En cas de modifications concurrentes sur les mêmes lignes de code, Git signale un **conflit** qu’il faut résoudre manuellement avant de terminer le merge.

Workflow classique :

1. Mettre `main` à jour.
2. Créer une branche à partir de `main`.
3. Travailler et committer sur cette branche.
4. Fusionner la branche dans `main` (souvent via une Pull Request).

***

## 2. Branches et merges côté GitHub

Sur GitHub, on sépare généralement :

- La branche principale (`main` ou `master`) qui sert de référence stable.
- Des branches de travail pour chaque fonctionnalité ou correction.

Processus typique :

1. Pousser la branche de travail vers GitHub.
2. Créer une **Pull Request (PR)** qui propose de fusionner cette branche dans `main`.
3. Relire les changements, éventuellement commenter ou demander des modifications.
4. Lorsque tout est validé, **fusionner la PR** (bouton *Merge* sur GitHub).

Après merge, la branche peut être supprimée pour garder un historique propre.

***

## 3. Gestion des branches dans VS Code

VS Code propose une intégration Git directement dans l’éditeur.

- La **branche courante** apparaît dans la barre de statut, en bas de la fenêtre.
- Cliquer sur le nom de la branche permet :
    - de **changer de branche**,
    - ou de **créer une nouvelle branche** (par exemple `feature/nom-fonctionnalite`).

Workflow recommandé :

1. Se placer sur `main` et synchroniser (pull).
2. Créer une nouvelle branche à partir de `main`.
3. Effectuer les changements sur cette nouvelle branche.

Les fichiers modifiés sont visibles dans la vue **Source Control**. On peut y préparer les fichiers (stage), rédiger un message de commit, puis valider le commit.

***

## 4. Publier une branche et créer une Pull Request avec VS Code

Avec l’intégration GitHub de VS Code (via l’extension de Pull Requests et Issues) :

- Lorsqu’une branche locale n’est pas encore sur GitHub, un bouton **Publish Branch** apparaît dans la vue Source Control.
- Cliquer sur ce bouton publie la branche vers le dépôt distant.

Pour créer une Pull Request :

1. Ouvrir la vue **Pull Requests**.
2. Utiliser la commande **Create Pull Request**.
3. Choisir :
    - la branche de base (souvent `main`),
    - la branche de comparaison (branche de travail).
4. Renseigner le **titre** et la **description** de la PR.
5. Valider la création de la PR.

Cette opération crée la PR sur GitHub, prête à être relue et fusionnée.

***

## 5. Revue, merge et nettoyage de branches dans VS Code

Une fois la PR créée, VS Code peut passer en **Review Mode** :

- Il est possible d’ouvrir la PR, d’inspecter les diffs, d’ajouter des commentaires et de suivre l’état de la revue.
- Quand la PR est prête, une action permet de **fusionner** la PR dans la branche de base (par exemple `main`).

Après le merge :

- VS Code peut proposer de supprimer la branche locale et distante pour éviter l’accumulation de branches obsolètes.
- Le code fusionné se trouve désormais sur la branche principale, qui devient à nouveau la base pour les futurs travaux.

