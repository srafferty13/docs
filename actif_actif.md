# Réplication MariaDB actif / actif (synchronisation bidirectionnelle)

## srv 1
- Ajoutez dans la configuration
```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
```bash
log-slave-updates
master-retry-count = 20
replicate-do-db = gsb_valid
```

## srv 2
- Ajoutez dans la configuration
```bash
binlog_do_db = gsb_valide
log-slave-updates
```
- commenter
```
#bind address
```
- uncomment
```
log_bin
```

- Création du dossier log et permissions
```bash
mkdir -m 2750 /var/log/mysql
chown mysql /var/log/mysql
systemctl restart mariadb.service
```

- Connexion à MariaDB
```sql
create user 'replicateur'@'%' identified by 'Btssio2017';
grant replication slave on *.* to 'replicateur'@'%';
stop slave;
```
crée un utilisateur présent sur toutes les machines (`'@'%'`) avec le mdp Btssio2017  
on donne les droits de réplications sur toutes les tables à l'user replicateur  

- Vérification du statut maître
```sql
show master status;
```
take the master log pos from srv1 and file name to insert in command

- Configuration du master
```sql
change master to master_host='172.16.0.10', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000002', master_log_pos=328;
```

## srv 1
- Vérification du statut maître
```sql
show master status;
```
take the master log pos from srv2 to insert in command

- Configuration du master
```sql
change master to master_host='172.16.0.11', master_user='replicateur', master_password='Btssio2017', master_log_file='mysql-bin.000002', master_log_pos=342;
```

## Sur les deux serveurs
- Démarrage des esclaves et vérification
```sql
start slave;
show slave status \G
```

## Tests de réplication bidirectionnelle
- Sur n'importe quel serveur
```sql
use gsb_valide;
select * from Visiteur;
update gsb_valide.Visiteur set mdp='toto' where login='agest';
```
should work both ways

## Haute disponibilité avec CRM
```bash
crm configure primitive serviceMySQL ocf:heartbeat:mysql params socket=/var/run/mysqld/mysqld.sock
crm configure clone cServiceMySQL serviceMySQL
crm configure show
```
should propagate on both machines; if not, restart
