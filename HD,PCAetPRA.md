# Haute Disponibilité (HD), PCA et PRA

## Haute disponibilité (HD)

### Définition
Garantir un service accessible presque tout le temps.  
**Objectif :** éviter toute interruption.  
**Mesurée en % de disponibilité :**

| Disponibilité | Panne/an |
|----------------|-----------|
| 99 % | 3,65 jours |
| 99,9 % | 8 h 45 |
| 99,99 % | 52 min |
| 99,999 % | 5 min |

**SLA (Service Level Agreement)** : contrat précisant le niveau de service (temps de réponse, pénalités…).

### Principes clés
1. Éliminer les points de défaillance uniques.  
2. Résilience du système.  
3. Tolérance aux pannes.

### Moyens techniques
- **Clustering** : serveurs en grappe  
  - *Actif/passif* → un serveur de secours  
  - *Actif/actif* → tous actifs, répartition de charge  
  - *Failover* = basculement automatique  
  - *Failback* = retour au serveur principal  
- **Load balancing** : répartition de charge (Round Robin, Round Robin pondéré, Least Connection)  
- **Redondance** : RAID, double lien réseau (STP), routeurs de secours, sites géographiques doublés  
- **Sauvegarde** :
  - **RPO** = perte de données max. admise  
  - **RTO** = durée d’interruption max. admise

---

## PCA (Plan de Continuité d’Activité)

### Objectif
Maintenir l’activité pendant un sinistre majeur.  
**Responsable :** RPCA (travaille avec RSSI et DSI).  
Déclenché quand tout le reste a échoué.

### Étapes
1. Analyse de risques (menaces humaines/naturelles).  
2. Analyse d’impact (durée d’arrêt max tolérée).  
3. Mise en place des mesures et tests réguliers.  
4. Mise à jour au moins 1×/an.

### 4 Piliers
- Organisation de crise  
- Documentation claire et à jour  
- Sauvegarde efficace et testée  
- Solution technique de secours (redondance, site secondaire)

---

## PRA (Plan de Reprise d’Activité)

### Objectif
Redémarrer le service après sinistre.  
Fait partie du PCA.  
Peut fonctionner en **mode dégradé** (service partiel).  
**Exemple :** redémarrer les serveurs sur un site de secours.

---

## Différences clés

| Terme | Objectif | Moment d’action |
|--------|-----------|----------------|
| HD | Éviter l’arrêt | Avant sinistre |
| PCA | Continuer pendant | Pendant sinistre |
| PRA | Reprendre après | Après sinistre |

---

## Mesures types

- **Préventives :** sauvegarde (règle 3-2-1), redondance, antivirus, formation du personnel  
- **Curatives :** restauration des données, redémarrage des applications/machines

---

## Mode dégradé
Fonctionner avec des moyens réduits (énergie, matériel, personnel) pour assurer le minimum vital.

