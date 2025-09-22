# DNSSEC

## 1. Purpose of DNSSEC
- DNSSEC secures the Domain Name System by **adding digital signatures** to DNS records.  
- It validates that DNS responses come from the **correct source** and have not been altered in transit.  
- It prevents attacks such as **cache poisoning** and ensures **data integrity** and **authenticity** in DNS lookups.  

---

## 2. Configuration Steps

### Enable DNSSEC validation
Edit `/etc/bind/named.conf.options` and replace:
```bash
dnssec-validation auto;
```
with:
```bash
dnssec-validation yes;
```

---

### Generate keys
Create the keys directory:
```bash
mkdir /etc/bind/keys
cd /etc/bind/keys
```

Generate the **Zone Signing Key (ZSK)**:
```bash
dnssec-keygen -a rsasha1 -b 1024 -n zone sodecaf.fr
```
Example output:
```
Ksodecaf.fr.+005+41725.key
Ksodecaf.fr.+005+41725.private
```

Generate the **Key Signing Key (KSK)**:
```bash
dnssec-keygen -a rsasha1 -b 1024 -f KSK -n zone sodecaf.fr
```
Example output:
```
Ksodecaf.fr.+005+09446.key
Ksodecaf.fr.+005+09446.private
```

---

### Include the public keys in the zone
Edit `/var/cache/bind/db.sodecaf.fr` and add:
```
; KSK
$include "/etc/bind/keys/Ksodecaf.fr.+005+09446.key"

; ZSK
$include "/etc/bind/keys/Ksodecaf.fr.+005+41725.key"
```

---

### Sign the zone
Go to `/var/cache/bind` and run:
```bash
dnssec-signzone -o sodecaf.fr -t -k /etc/bind/keys/Ksodecaf.fr.+005+09446 db.sodecaf.fr /etc/bind/keys/Ksodecaf.fr.+005+41725
```

#### Explanation
- Signs the zone `sodecaf.fr` with DNSSEC.  
- Takes the unsigned zone file `db.sodecaf.fr`.  
- Uses the **ZSK (41725)** to sign all DNS records.  
- Uses the **KSK (09446)** to sign the DNSKEY set.  
- Produces a new signed zone file:  
  ```
  db.sodecaf.fr.signed
  ```

---

## 3. Final Configuration
Edit `/etc/bind/named.conf.local` to reference the signed zone file by adding `.signed`.  
```bash
zone "sodecaf.fr" {
        type master;
        file "db.sodecaf.fr.signed";
        allow-transfer {172.16.0.4;};
};
```

Restart Bind9:
```bash
systemctl restart bind9
```
