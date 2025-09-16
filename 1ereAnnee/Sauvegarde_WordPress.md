## Sauvegarde WordPress
### Créer une archive
```bash
tar -cvzf /var/www/save_wp.tar.gz .
```

### Extraire une archive
```bash
tar -xvzf /var/www/save_wp.tar.gz -C /var/www/
```

### Transfert via SCP
```bash
scp root@ip_container:/var/www/save_wp.tar.gz /var/www/
```

---
