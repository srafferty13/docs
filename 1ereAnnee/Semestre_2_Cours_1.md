## Semestre 2 Cours 1 (Sécurité réseau)

### DHCP Snooping
```cisco
ip dhcp snooping
interface fa0/1
 ip dhcp snooping trust
```

### MAC Flooding
- **Outil** : `macof` (génère des adresses MAC pour overflow de la table CAM).
- **Configuration switch** :
  ```cisco
  interface range Fa0/1-24
   switchport mode access
   switchport port-security
   switchport port-security mac-address sticky
   switchport port-security aging time 2
   switchport port-security maximum 4
  ```

---
