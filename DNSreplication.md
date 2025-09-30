# DNS Replication

## 1. Préparation
- Changer IP, DNS et nom de la machine.
- Installation de Bind9 et de ses outils
```bash
sudo apt update
sudo apt install bind9 bind9-utils -y
```
- Cloner la machine.
- Changer nom et IP sur la machine clonée.

---

## 2. Configuration sur DNS 1 (Master)
- Éditez `/etc/bind/named.conf.local` pour définir les zones :

```bash
zone "sodecaf.fr" {
    type master;
    file "/etc/bind/db.sodecaf.fr";
};
```

- Créer le fichier de zone dans `/var/cache/bind/db.sodecaf.fr` :

```
; fichier de zone db.sodecaf.fr
$TTL 86400
@               IN      SOA     srv-dns1.sodecaf.fr. hostmaster.sodecaf.fr (
                                2025092201      ;serial
                                86400           ;refresh
                                21600           ;retry
                                3600000         ;expire
                                3600 )          ;negative cache TTL
@               IN      NS      srv-dns1.sodecaf.fr.
@               IN      NS      srv-dns2.sodecaf.fr.

srv-dns1        IN      A       172.16.0.3
srv-dns2        IN      A       172.16.0.4
srv-web1        IN      A       172.16.0.10
srv-web2        IN      A       172.16.0.11
www             IN      A       172.16.0.12
web1            IN      CNAME   srv-web1.sodecaf.fr.
web2            IN      CNAME   srv-web2.sodecaf.fr.
```

### Explications
- TTL = Time To Live  
- Serial = numéro de version de la zone (incrémente à chaque modification)  
- Refresh = fréquence de vérification par les DNS secondaires  
- Retry = délai avant nouvelle tentative si Refresh échoue  
- Expire = délai après lequel un DNS secondaire considère la zone périmée  
<br>

- Vérifier le fichier de zone :
```bash
named-checkzone sodecaf.fr db.sodecaf.fr
```

- Redémarrer Bind9 :
```bash
systemctl restart bind9
```

- Test :
```bash
# Sur Windows
nslookup web1.sodecaf.fr 172.16.0.3

# Sur Linux
dig web1.sodecaf.fr @172.16.0.3
```

---

## 3. Mise en place du reverse

- Dans `/etc/bind/named.conf.local` :

```bash
zone "0.16.172.in-addr.arpa" {
    type master;
    file "db.172.16.0.rev";
};
```

- Copier le fichier `db.sodecaf.fr` en `db.172.16.0.rev` dans `/var/cache/bind` :

```
; fichier de zone db.172.16.0.rev
$TTL 86400
@               IN      SOA     srv-dns1.sodecaf.fr. hostmaster.sodecaf.fr (
                                2025092201      ;serial
                                86400           ;refresh
                                21600           ;retry
                                3600000         ;expire
                                3600 )          ;negative cache TTL
@               IN      NS      srv-dns1.sodecaf.fr.
@               IN      NS      srv-dns2.sodecaf.fr.

3               IN      PTR     srv-dns1.sodecaf.fr.
4               IN      PTR     srv-dns2.sodecaf.fr.
10              IN      PTR     srv-web1.sodecaf.fr.
11              IN      PTR     srv-web2.sodecaf.fr.
12              IN      PTR     www.sodecaf.fr.
```

### Explication
- `3` correspond à `172.16.0.3`, `4` à `172.16.0.4`, etc.

---

## 4. Configuration du DNS 2 (Slave)

- Dans `/etc/bind/named.conf.local` :

```bash
zone "sodecaf.fr" {
    type slave;
    file "slave/db.sodecaf.fr";
    masters {172.16.0.3;};
};

zone "0.16.172.in-addr.arpa" {
    type slave;
    file "slave/db.172.16.0.rev";
    masters {172.16.0.3;};
};
```

- Créer le dossier `slave` dans `/var/cache/bind` :
```bash
chgrp bind slave/
chmod g+w slave/
```

- Sur le DNS 1, autoriser le transfert vers le slave :
```bash
# Dans named.conf.local, ajouter à chaque zone
allow-transfer {172.16.0.4;};
```

- Redémarrer Bind9 sur les deux machines :
```bash
systemctl restart bind9
```

- Vérification sur DNS 2 :
```bash
ls /var/cache/bind/slave
# doit afficher db.172.16.0.rev  db.sodecaf.fr
```

- Tests depuis Windows :
```text
U:\>nslookup web1.sodecaf.fr 172.16.0.4
Serveur :   srv-dns2.sodecaf.fr
Address:  172.16.0.4

Nom :    srv-web1.sodecaf.fr
Address:  172.16.0.10
Aliases:  web1.sodecaf.fr

U:\>nslookup 172.16.0.10 172.16.0.4
Serveur :   srv-dns2.sodecaf.fr
Address:  172.16.0.4

Nom :    srv-web1.sodecaf.fr
Address:  172.16.0.10
```

---

## 5. Configuration des forwarders

- Dans `/etc/bind/named.conf.options` :
```bash
forwarders {
    8.8.8.8;
};
```

### Explication
- Le DNS redirigera toute requête qu’il ne peut pas résoudre directement vers le DNS public de Google.
