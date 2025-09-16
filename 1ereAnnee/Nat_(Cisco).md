## Nat (Cisco)
```cisco
ip route 0.0.0.0 0.0.0.0 20.6.6.1
interface fa0/0
 ip nat inside
interface fa0/1
 ip nat outside
access-list X permit/deny <reseau> <masque inversé>
```

---
