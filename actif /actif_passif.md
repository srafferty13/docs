# Réplication MariaDB actif / passif

## 1. Configuration du serveur web 1
- Éditez le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf`
```bash
server id =1
```
- uncomment 
```
log_error
server-id
log_bin
expire_log_days
max_binlog_size
```
- add
```
binlog_do_db = gsb_valide
```

- Création du dossier log
```bash
mkdir -m 2750 /var/log/mysql
chown ysql /var/log/mysql
systemctl restart mariadb.service
```

- Connexion à MariaDB
```sql
create user 'replicateur'@'%' identified by 'Btssio2017';
#crée un utilisateur présent sur toutes les machines (`'@'%'`) avec le mdp `Btssio2017`  
grant replication slave on *.* to 'replicateur'@'%';
#on donne les droits de réplications sur toutes les tables à l'user `replicateur`
flush tables with read lock;
#met les tables en lecture seule
```
---

## 2. Configuration du serveur web 2
- Éditez le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf`
```bash
server id =2
expire_logs_days        = 10
max_binlog_size        = 100M
master-retry-count = 20
replicate-do-db = gsb_valide
```

- Redémarrage du service
```bash
systemctl restart mariadb.service
```

- Connexion à MySQL
```sql
stop slave;

change master to master_host='172.16.0.10', 
    master_user='replicateur', 
    master_password='Btssio2017', 
    master_log_file='mysql-bin.000001', 
    master_log_pos=328;

start slave;

show slave running;
```

---

## 3. Tests de réplication
- Sur `srv-web1`
```sql
unlock tables;
use gsb_valide;
show tables;
select * from Visiteur;
update gsb_valide.Visiteur set mdp='toto' where login='agest';
```
changes user agest's password to toto in the table visiteur from gsb_valide  

- Sur `srv-web2`
```sql
select * from Visiteur;
```
should sync the change on srv-web2  

- Vérification
```sql
show master status; 
show slave status \G;
```
the position increments with each command  
the change should be recorded here
