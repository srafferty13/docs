## Raid
### Types de RAID
1. **RAID 0 (Stripping)** :
   - Données réparties entre plusieurs disques.
   - Améliore les performances mais pas de redondance.

2. **RAID 1 (Mirroring)** :
   - Données dupliquées sur deux disques.
   - Redondance mais performances normales en écriture.

3. **RAID 5 (Stripping avec parité)** :
   - Combine RAID 0 et parité répartie.
   - Reconstruction possible en cas de panne d'un disque.

4. **RAID 10 (RAID 1+0)** :
   - Combine RAID 1 et RAID 0.
   - Redondance et performances élevées.

5. **RAID 50 (RAID 5+0)** :
   - Combinaison de sous-ensembles RAID 5 avec stripping.
   - Capacité et performances avec tolérance aux pannes.

### Cas d'utilisation
- **RAID 0** : Applications traitant de gros fichiers sans haute disponibilité.
- **RAID 1** : Applications nécessitant des données très disponibles.
- **RAID 5** : Bases de données ou applications avec besoins modérés.
- **RAID 10** : Besoins élevés en performance et disponibilité.
- **RAID 50** : Bases de données volumineuses.

---
