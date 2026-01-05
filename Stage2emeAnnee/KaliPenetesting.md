# Guide Complet du Penetesting avec Kali Linux

## Introduction

Ce guide présente une méthodologie structurée de penetesting basée sur le standard **PTES (Penetration Testing Execution Standard)**, la référence du secteur. La même approche s'applique à tous les contextes de test : infrastructure interne ou externe, petit réseau ou grande entreprise.

---

## Méthodologie PTES : Les 5 Phases

### Phase 1 : Reconnaissance & Collecte d'Informations (Information Gathering)

**Objectif :** Comprendre la cible sans l'affecter. C'est l'étape la plus importante (80% du travail de penetesting).

**Approche :**
- Collecte passive d'informations (OSINT) sur les domaines, IP publiques et services.
- Écoute légère du réseau pour comprendre l'architecture.

**Outils Kali Linux :**

| Outil | Utilisation | Commande exemple |
|-------|-----------|------------------|
| `whois` | Récupérer les informations d'enregistrement de domaine | `whois exemple.com` |
| `nslookup` / `dig` | Énumérer les enregistrements DNS | `dig exemple.com` |
| `theHarvester` | Trouver emails, sous-domaines et IP liés à une cible | `theHarvester -d exemple.com -b google` |
| `Wireshark` | Écouter le trafic réseau (ARP, DHCP, DNS) | Interface graphique |

---

### Phase 2 : Scan & Énumération (Scanning)

**Objectif :** Cartographier précisément le réseau. Identifier les machines actives, les ports ouverts et les versions des services.

**Approche :**
- Scan des adresses IP pour déterminer quels hôtes sont actifs.
- Scan des ports pour connaître les services exposés.
- Récupération des versions des logiciels (banner grabbing).

**Outils Kali Linux :**

| Outil | Utilisation | Commande exemple |
|-------|-----------|------------------|
| `Nmap` (L'outil principal) | Scanner complet : hôtes, ports, versions | `nmap -sV -sC -O 192.168.1.0/24` |
| `Nmap` (scan version) | Déterminer les versions des services | `nmap -sV <IP>` |
| `Nmap` (scan aggressive) | Scan plus approfondi avec reconnaissance d'OS | `nmap -A <IP>` |
| `netdiscover` | Découvrir les machines sur un réseau local | `netdiscover -r 192.168.1.0/24` |
| `Enum4linux` | Énumérer les infos Windows/Active Directory | `enum4linux <IP>` |

**Flags Nmap courants :**
- `-p` : Spécifier les ports à scanner
- `-sT` : Scan TCP complet
- `-sU` : Scan UDP
- `--script` : Utiliser les scripts NSE (Nmap Scripting Engine)

---

### Phase 3 : Analyse de Vulnérabilité (Vulnerability Assessment)

**Objectif :** Identifier les failles de sécurité connues (CVE) dans les services détectés.

**Approche :**
- Scanner automatisé pour détecter les vulnérabilités.
- Recherche manuelle des exploits connus.
- Validation des failles trouvées.

**Outils Kali Linux :**

| Outil | Utilisation | Commande exemple |
|-------|-----------|------------------|
| `OpenVAS` (Greenbone) | Scanner de vulnérabilité complet et paramétrable | Interface web (http://localhost:9392) |
| `Nmap` (scripts de vulnérabilité) | Scan léger des vulnérabilités connues | `nmap --script vuln <IP>` |
| `Searchsploit` | Chercher des exploits publics pour une faille donnée | `searchsploit apache 2.4` |
| `Nikto` | Scanner spécialisé pour serveurs web | `nikto -h <URL>` |

**Processus :**
1. Lancer un scan OpenVAS complet ou un scan Nmap vuln rapide.
2. Identifier les CVE avec score CVSS élevé.
3. Vérifier manuellement avec Searchsploit si un exploit existe.

---

### Phase 4 : Exploitation (Gaining Access)

**Objectif :** Confirmer pratiquement que les failles sont réelles et exploitables.

**Approche :**
- Exploitation manuelle ou semi-automatisée des vulnérabilités.
- Test des accès avec identifiants faibles.
- Escalade de privilèges si possible.

**Outils Kali Linux :**

| Outil | Utilisation | Commande exemple |
|-------|-----------|------------------|
| `Metasploit Framework` (`msfconsole`) | Plateforme d'exploitation tout-en-un | `msfconsole` |
| `Hydra` | Attaque par force brute (SSH, FTP, Web, etc.) | `hydra -l admin -P wordlist.txt ssh://192.168.1.1` |
| `Responder` | Capturer les identifiants Windows sur un réseau local | `responder -I eth0` |
| `Burp Suite Community` | Tester les applications web (injection, XSS, etc.) | Interface graphique |
| `SQLmap` | Détecter et exploiter les injections SQL | `sqlmap -u "http://exemple.com?id=1" --dbs` |

**Workflows courants :**
- SSH/Services : Utiliser Hydra pour brute-force, puis Metasploit si une faille existe.
- Réseaux internes : Responder capture les authentifications Windows circulant sur le réseau.
- Web : Burp Suite pour mapper l'application et tester les injections.

---

### Phase 5 : Rapport (Reporting)

**Objectif :** Documenter les résultats de façon claire et exploitable.

**Contenu minimum d'un rapport :**
1. **Faille identifiée** : Description technique (CVE, service affecté, version).
2. **Impact métier** : Quelles données ou services sont menacés ?
3. **Preuve d'exploitation** : Captures d'écran, logs, evidence.
4. **Remédiation** : Patch, mise à jour, configuration de sécurité recommandée.
5. **Priorité** : Critique, Haute, Moyenne, Basse (selon CVSS et impact).

---

## Contextes d'Application

### Pentest Interne ("Grey Box")

**Caractéristiques :**
- Accès au réseau local depuis une machine.
- Objectif : Identifier les failles accessibles depuis l'intérieur.

**Approche spécifique :**
- Mapping du réseau interne avec `netdiscover` et `Nmap`.
- Écoute réseau avec `Wireshark` pour comprendre les flux.
- Audit des serveurs internes avec `OpenVAS`.
- Test de l'Active Directory avec `Enum4linux`.
- Capture des identifiants avec `Responder`.

**Outils clés :**
- OpenVAS, Nmap, Responder, Wireshark, Enum4linux

---

### Pentest Externe ("Black Box" ou "Grey Box")

**Caractéristiques :**
- Accès depuis l'internet (simulation d'une attaque externe).
- Objectif : Identifier les failles accessibles publiquement.
- Contrainte : Aucun déni de service autorisé (pas de flooding, pas de crash).

**Approche spécifique :**
- Scan des IP publiques avec `Nmap` (prudent).
- Énumération OSINT avec `whois`, `dig`, `theHarvester`.
- Scan de vulnérabilités web avec `Nikto` ou `Burp Suite`.
- Injection SQL avec `SQLmap` sur les formulaires web.

**Outils clés :**
- Nmap, OpenVAS, Nikto, Burp Suite, SQLmap, theHarvester

---

## Checklist Avant de Démarrer

Avant tout penetest, valider :

- [ ] **Scope** : Quelles adresses IP ou domaines peut-on tester ?
- [ ] **Horaires** : À quel moment faire le test (heures creuses) ?
- [ ] **Autorisation écrite** : Document signé autorisant les tests.
- [ ] **Point de contact** : Qui appeler en cas de problème ?
- [ ] **Limites** : Interdiction de déni de service ? De données sensibles ? Etc.
- [ ] **Environnement de test** : Machine virtuelle ? Réseau isolé ?

---

## Commandes Essentielles Rapides

```bash
# Reconnaissance
whois exemple.com
dig exemple.com
theHarvester -d exemple.com -b google

# Scan réseau
nmap -sV -sC -O 192.168.1.0/24
netdiscover -r 192.168.1.0/24
Enum4linux 192.168.1.100

# Scan de vulnérabilités
nmap --script vuln 192.168.1.100
nikto -h http://exemple.com

# Recherche d'exploits
searchsploit apache 2.4.49

# Brute force SSH
hydra -l admin -P wordlist.txt ssh://192.168.1.1

# Écoute réseau (AD)
responder -I eth0

# Metasploit
msfconsole
search apache
use exploit/...
set RHOST 192.168.1.100
run
```

---

## Ressources et Références

- **PTES (Penetration Testing Execution Standard)** : Standard méthodologique international.
- **OWASP** : Pour les tests de sécurité web.
- **CVE Database** : https://cve.mitre.org/ (identifiants des vulnérabilités connues).
- **Exploit-DB** : Base de données des exploits publics (accédée via Searchsploit).
- **HackTheBox** / **TryHackMe** : Environnements d'entraînement légaux.

---

## Notes Importantes

1. **Légalité** : Tester uniquement sur les systèmes où vous avez une autorisation écrite.
2. **Éthique** : Le penetesting est un outil défensif ; ne pas l'utiliser à mauvais escient.
3. **Confidentialité** : Les résultats et les données capturées doivent rester confidentiels.
4. **Documentation** : Tout doit être documenté pour justifier les actions entreprises.
5. **Non-destruction** : L'objectif est de trouver les failles, pas de casser l'infrastructure.
