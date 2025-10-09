# Fiche de Révision — TP5 PowerShell & Active Directory

## Objectif

Gérer un **annuaire Active Directory (AD)** avec PowerShell :

- Créer OUs, groupes, utilisateurs  
- Automatiser ces actions avec un **script PowerShell**

---

## Mise en place

```powershell
# Installation du module AD
Add-WindowsFeature RSAT-AD-PowerShell
Import-Module ActiveDirectory
```

---

## Commandes essentielles

### Unité d’organisation (OU)

```powershell
New-ADOrganizationalUnit -Name "Employés" -Path "dc=sodecaf,dc=local"
```

### Utilisateurs

```powershell
# Créer un utilisateur
New-ADUser -Name "Paul Bismuth" -GivenName Paul -Surname Bismuth `
-SamAccountName pbismuth -UserPrincipalName pbismuth@supinfo.com `
-AccountPassword (Read-Host -AsSecureString "MotDePasse") -PassThru | Enable-ADAccount

# Lister tous les utilisateurs
Get-ADUser -Filter * | Format-List

# Supprimer un utilisateur
Remove-ADUser pbismuth

# Activer / Désactiver / Déverrouiller
Enable-ADAccount pbismuth
Disable-ADAccount pbismuth
Unlock-ADAccount pbismuth
```

### Groupes

```powershell
# Créer un groupe
New-ADGroup -Name "Politique" -GroupScope Global -Path "ou=Employés,dc=sodecaf,dc=local"

# Ajouter un utilisateur dans un groupe
Add-ADGroupMember -Identity "Politique" -Members "pbismuth"
```

---

## Script d’automatisation (structure type)

```powershell
Import-Module ActiveDirectory

# Génère un mot de passe aléatoire
function Get-RandomPassword {
    [System.Web.Security.Membership]::GeneratePassword(12,2)
}

# Importer le fichier CSV
$users = Import-Csv "utilisateurs_sodecaf.csv"

foreach ($user in $users) {
    $password = (Get-RandomPassword | ConvertTo-SecureString -AsPlainText -Force)
    New-ADUser -Name $user.Nom -GivenName $user.Prenom -SamAccountName $user.Login `
    -UserPrincipalName "$($user.Login)@sodecaf.local" -AccountPassword $password -PassThru | Enable-ADAccount
}
```

---

## Lecteur réseau personnel

(Mettre PowerShell v4 + module **NTFSSecurity**)

```powershell
Import-Module "C:\Users\Administrateur\Documents\WindowsPowerShell\NTFSSecurity"
# Gestion des permissions NTFS via le module NTFSSecurity
```

---

## Commande utile

Lister toutes les commandes du module AD :

```powershell
Get-Command *-AD*
```

