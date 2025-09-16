## Revision ACL (Cisco)

### Fondements des ACL
- **ACL (Access Control List)**: Liste ordonnée de règles filtrant le trafic réseau.
  - Actions: `permit` (autoriser) ou `deny` (bloquer).
  - Règles lues de haut en bas → première correspondance appliquée.
  - `deny any` implicite si aucune règle `permit` ne correspond.

#### Types d'ACL
| Type       | Plage numéro     | Critères                     | Placement recommandé      |
|------------|------------------|------------------------------|---------------------------|
| Standard   | 1-99, 1300-1999  | IP source uniquement         | Près de la destination    |
| Étendue    | 100-199, 2000-2699 | IP source/dest, port, protocole | Près de la source       |

### Configuration des ACL
#### ACL Standard (numérique)
```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any
```

#### ACL Standard (nommée)
```cisco
ip access-list standard MON_ACL
 permit host 192.168.1.1
 deny 192.168.2.0 0.0.0.255
```

#### ACL Étendue (numérique)
```cisco
access-list 100 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.1 eq 80
access-list 100 deny icmp any any
```

#### ACL Étendue (nommée)
```cisco
ip access-list extended MON_ACL_EXT
 permit udp any host 10.0.0.2 eq 53
 deny ip any any log
```

#### Masque Wildcard
- Inverse du masque de sous-réseau :
  - `0.0.0.255` ≡ /24
  - `0.0.0.0` ≡ une seule IP (`host`)

### Application des ACL
#### Sur une interface
```cisco
interface FastEthernet0/0
 ip access-group 10 in
 ip access-group 20 out
```

#### Sur lignes VTY (SSH/Telnet)
```cisco
line vty 0 4
 access-class 30 in
```

### Gestion et modification
#### Vérification
```cisco
show access-lists
show ip interface f0/0
```

#### Modifier une ACL nommée
```cisco
ip access-list standard MON_ACL
 no 10
 15 permit 192.168.3.0 0.0.0.255
```

#### Supprimer une ACL
```cisco
no access-list 100
no ip access-list extended MON_ACL_EXT
```

### Bonnes pratiques & erreurs
- ✅ Règles spécifiques en premier (ex: `host` avant `any`).
- ✅ Désactiver ACL avant modification : `no ip access-group 10 in`.
- ✅ Journaliser les refus : `deny ip any any log`.
- ⚠ Masque wildcard inversé : `0.0.0.255` ≠ `255.255.255.0`.

### Exemples pratiques
#### Autoriser seulement Web et DNS
```cisco
access-list 110 permit tcp any any eq 80
access-list 110 permit udp any any eq 53
access-list 110 deny ip any any
```

#### Bloquer un réseau sauf ping
```cisco
access-list 120 deny ip 192.168.2.0 0.0.0.255 any
access-list 120 permit icmp any any
```

### Tableau des commandes
| Action                    | Commande                                  |
|---------------------------|-------------------------------------------|
| Créer ACL standard        | `access-list 10 permit 192.168.1.0 0.0.0.255` |
| Créer ACL étendue         | `access-list 100 permit tcp any any eq 80` |
| Appliquer sur interface   | `ip access-group 10 in`                   |
| Voir les ACL              | `show access-lists`                       |
| Supprimer ACL             | `no access-list 10`                       |

---
