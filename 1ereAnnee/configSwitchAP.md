## Config Switch AP (Cisco)
```cisco
version 15.2
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
hostname Switch
!
username etudiant privilege 15 secret 5 $1$dTBB$62RXUnxRiGkGZ9Ik/ZWic1
no aaa new-model
system mtu routing 1500
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 description Administration
 switchport access vlan 100
 switchport mode access
!
interface FastEthernet0/2
 description Commerciaux
 switchport access vlan 30
 switchport mode access
!
interface FastEthernet0/3
 description Production
 switchport access vlan 20
 switchport trunk allowed vlan 10,20
 switchport mode access
 switchport port-security
!
interface FastEthernet0/4
 description Serveurs
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/5
 description Sauvegarde
 switchport access vlan 99
 switchport mode access
!
interface Vlan1
 ip address 172.17.1.120 255.255.0.0
!
ip http server
ip http secure-server
!
line con 0
line vty 0 4
 login local
line vty 5 15
 login local
!
end
```

---
