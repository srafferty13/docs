# Changer le hostname et mettre à jour la résolution locale

## 1. Changer le hostname actif
```bash
sudo hostnamectl set-hostname NOM_MACHINE
```

## 2. Mettre à jour /etc/hosts pour résolution locale
```bash
sudo nano /etc/hosts
```
Ajouter ou modifier la ligne suivante :
```
172.16.0.3 NOM_MACHINE
```

## 3. Vérification
```bash
hostname           # montre le nom actuel
hostnamectl        # détail complet
ping NOM_MACHINE   # teste la résolution locale
