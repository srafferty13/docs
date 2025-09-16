## Routage rip (Cisco)
```cisco
router rip
 version 2
 network <réseau>
 passive interface <interface>
 default-information originate
ip route 0.0.0.0 0.0.0.0 <Gateway>
```

---
