# FICHE RÉVISION ADRESSAGE IPv6

## 1. Pourquoi IPv6 ?

```
IPv4 épuisé → 128 bits = 3,4×10³⁸ adresses (6,6×10²³/m² Terre !)
Dates épuisement : APNIC'11 RIPE'12 ARIN'15 LACNIC'14
2025 : France 2e mondial (68,6% IPv6), 87% fixes/70% mobiles OK
Freins : sites web (35%), SI entreprises
```


## 2. Notation (RFC 5952)

```
128 bits = 8 groupes × 16 bits (hexadécimal)
Ex complet : 2001:0dc8:e004:0001:0000:0000:0000:f00a

RÈGLES SIMPLIFICATION :
- Zéros tête supprimés
- "::" UNE FOIS (plus grand bloc zéros, 1er si égalité) 
- Minuscules

EXEMPLES :
2001:0db8::101:abcd:def1:1234
::a:b.c.d (IPv4 en IPv6)
:: (indéfinie/route défaut)
::1 (loopback)
[2001:db8:1::1]:80 (avec port)
```


## 3. Types adresses (PAS de broadcast !)

| Type | Préfixe | Routable ? | Usage | Obligatoire ? |
| :-- | :-- | :-- | :-- | :-- |
| GUA | 2000::/3 | Internet | Publique | Option |
| LL | fe80::/10 | Lien local | NDP/RA | OUI |
| ULA | fc00::/7 | Privé | Réseau interne | Option |
| Multicast | ff00::/8 | 1→N | Remplace broadcast | - |
| Anycast | Comme unicast | Plus proche | Load balancing | - |

## 4. Structures détaillées

**GUA (Global Unicast)**

```
2000::/3 = 001xxxx...
Global Prefix (FAI ex /48) | Subnet ID | IID (64b)
Ex : 2001:db8:cafe:10::/64
```

**LL (Link-Local - OBLIGATOIRE)**

```
fe80:: + IID/64 (EUI64/random/manuel)
Ex : fe80::a4b2:c3ff:fe45:1234
Passerelle = fe80::1 (souvent)
```

**ULA (Privé)**

```
fc00::/7 → fdXX:XXXX:XXXX::/48 (pratique)
L=1 + Global ID 40b aléatoire + Subnet + IID
Ex : fd a7c4:e291:b3:10::/64 (VLAN 10)
Outil : unique-local-ipv6.com
```


## 5. Multicast clés (1→N)

```
ff02::1     = Tous les hôtes (lien)
ff02::2     = Tous les routeurs
ff02::1:FFXX:XXXX = Solicited-Node (NDP)
ff02::12    = Agents DHCP
ff05::1:3   = Serveurs DHCP (site)
ff0X::FB    = mDNS
ff0X::101   = NTP
```


## 6. IID 64 bits (Interface ID)

```
3 méthodes :
1. EUI-64 ← MAC (stable/traçable)
2. Random/privacy (RFC 8981, clients)
3. Manuel (serveurs)

Serveurs : EUI-64/manuel (stable)
Clients : Privacy extensions (rotation)
```


## 7. Auto-config

```
SLAAC :
• RA (routeur) → préfixe
• Poste = préfixe + IID auto
• DAD (Duplicate Address Detection)
• Pas DNS/NTP

DHCPv6 :
• Stateful = IP + options + logs
• Stateless = SLAAC + options seulement

Recommandations :
• Domestique/invités → SLAAC pur
• PME → SLAAC + DHCPv6 stateless
• Datacenter → DHCPv6 stateful
```


## 8. GUA vs ULA

```
GUA seul      GUA + ULA
Architecture simple    Gestion 2 plages
1 AAAA DNS    Split-DNS interne/externe
FAI change → KO    Réseau interne stable
```


## 9. Configs pratiques

**Windows PowerShell**

```powershell
Get-NetIPAddress -AddressFamily IPv6
New-NetIPAddress -if "Ethernet" -IP 2001:db8:110 -PL 64 -DG fe80::1
Set-DnsClientServerAddress -if "Ethernet" -SA ("2001:4860:4860::8888","2001:4860:4860::8844")
Get-NetRoute -AF IPv6
ping -6 2001:db8::1
```

**Linux /etc/network/interfaces**

```bash
iface eth0 inet6 static
    address 2001:db8:110/64
    gateway fe80::1
```

```bash
ip -6 addr ; ip -6 route
ping6 -c4 2001:4860:4860::8888
traceroute6 google.com
host -t AAAA google.com
```


## 10. Récap 1 page

```
LL  fe80::     → Local uniquement (OBLIGATOIRE)
GUA 2000::/3   → Internet (FAI)
ULA fd00::/8   → Privé (stable)

Passerelle = fe80::1
::1 = loopback
NO Broadcast → Multicast

SLAAC + Stateless DHCPv6 = idéal PME !
```

**À savoir pour le contrôle :**
Dates épuisement IPv4
Règles notation "::"
fe80:: OBLIGATOIRE
Structures GUA/LL/ULA
SLAAC vs DHCPv6
Cmds Windows/Linux
