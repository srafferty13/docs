# PowerShell Scripts Overview

## Script 1 – File Search
```powershell
$cherche = $args[0]
$dossier = $args[1]
echo "Recherche du fichier $cherche dans le dossier $dossier"
Get-ChildItem -Path $dossier -Recurse -ErrorAction SilentlyContinue | 
    Where-Object {$_.Name -eq $cherche} |
    ForEach-Object {
        echo ("le fichier $cherche est dans "+$_.DirectoryName) 
    } 
```

**Explanation:**
1. `$args[0]`, `$args[1]` – Command-line arguments: filename and folder path.
2. `echo` – Prints text to console.
3. `Get-ChildItem -Recurse -ErrorAction SilentlyContinue` – Lists all files/folders recursively, ignores errors.
4. `Where-Object {$_.Name -eq $cherche}` – Filters items by name.
5. `ForEach-Object {$_}` – Iterates over matching files, prints directory.

**Use case:** Search a specific file recursively in a folder.

---

## Script 2 – Folder Size Calculation
```powershell
$dossier = $args[0]
echo "calcul en cours sur $dossier"
Get-ChildItem -Path $dossier -Recurse -Force -ErrorAction SilentlyContinue | `
Where-Object {$_PsisContainer -ne 0} | `
Where-Object {$_.Length -gt 1MB} | `
Measure-Object -property Length -Sum | `
ForEach-Object {
    $total = $_.sum / 1MB
    write-host -foregroundColor yellow ("le dossier "+$dossier+" contient {0:#,##0.0} MB" -f $total)
}
```

**Explanation:**
- `-Force` – Includes hidden/system files.
- `Where-Object {$_PsisContainer -ne 0}` – Selects only files.
- `Where-Object {$_.Length -gt 1MB}` – Filters files larger than 1 MB.
- `Measure-Object -Sum` – Computes total size.
- `write-host -ForegroundColor yellow` – Displays text in color.

**Use case:** Calculate total size of files over 1MB in a folder.

---

## Script 3 – Color Display
```powershell
$listeCouleurs = @("Black","DarkBlue","DarkGreen", ...)
$invite = "saisissez une couleur"
$couleur =""
while ($couleur -ne 'stop') {
    $couleur = Read-Host $invite 
    $z = $listeCouleurs | where-object {$_ -match $couleur}
    if ($z -ne $null)  {
        Write-Host -ForegroundColor $couleur ("vous avez demandé à écrire en "+$couleur)
    }
    else {
        write-host ("la couleur "+$couleur+" n'existe pas.")
    }
}
```

**Explanation:**
- `$listeCouleurs` – Array of valid colors.
- `Read-Host` – Prompts user input.
- `Where-Object {$_ -match $couleur}` – Checks if input matches array.
- `Write-Host -ForegroundColor` – Displays text in chosen color.

**Use case:** Interactive demonstration of console text colors.

---

## Script 4 – CSV Import and Directory Creation
```powershell
ipcsv ".\utilisateurs sodecaf.csv" -Delimiter ";" | foreach { 
    $agence = $_.agency;
    if ((Test-Path ("c:\"+$agence)) -eq $false) {
        New-Item ("c:\"+$agence) -ItemType "Directory" | Out-Null
    }
    ...
}
```

**Explanation:**
- `Import-Csv` – Reads CSV file, converts rows to objects.
- `-Delimiter ";"` – Semicolon separator.
- `foreach { ... }` – Iterates rows.
- `Test-Path` – Checks existence.
- `New-Item -ItemType "Directory"` – Creates folders.
- `Out-Null` – Suppresses output.
- `switch($_.function) { ... }` – Conditional actions.

**Use case:** Automate folder creation and display user info from CSV.

---

## Script 5 – DHCP Scope Configuration
```powershell
Add-DhcpServerv4Scope -Name $NomEtendue -StartRange $DebutEtendueDHCP  -EndRange $FinEtendueDHCP -SubnetMask $MasqueIP
Set-DhcpServerv4OptionValue -ScopeId $IPreseau -OptionId 3 -Value $IPPasserelle
Set-DhcpServerv4Scope -ScopeId $IPreseau -Name $NomEtendue -State Active
```

**Explanation:**
- `Add-DhcpServerv4Scope` – Creates new DHCP scope.
- `Set-DhcpServerv4OptionValue` – Configures gateway, DNS, etc.
- `Set-DhcpServerv4Scope -State Active` – Activates the scope.
- `Remove-DhcpServerv4Scope` – Deletes an existing scope.

**Use case:** Automate DHCP scope creation and configuration.

---

## Active Directory Scripts (TP5)

### TP5_scriptAD1.ps1 – Creating Users and Groups
```powershell
Import-Module ActiveDirectory
New-ADOrganizationalUnit -Name "Employés" -Path "dc=sodecaf,dc=local" -ProtectedFromAccidentalDeletion $false
New-ADUser -Name "Paul Bismuth" -GivenName Paul -Surname Bismuth `
-SamAccountName pbismuth -UserPrincipalName pbismuth@supinfo.com `
-AccountPassword (Read-Host -AsSecureString "Mettez ici votre mot de passe") `
-Path "ou=Employés,dc=sodecaf,dc=local" `
-PassThru | Enable-ADAccount
New-ADGroup -name "Politique" -groupscope Global -Path "ou=Employés,dc=sodecaf,dc=local"
Add-ADGroupMember -Identity "Politique" -Members "pbismuth"
```

**Key Commands:**
- `Import-Module ActiveDirectory` – Loads AD cmdlets.
- `New-ADOrganizationalUnit` – Creates OU.
- `New-ADUser` – Creates user account.
- `Enable-ADAccount` – Activates user.
- `New-ADGroup` – Creates group.
- `Add-ADGroupMember` – Adds user to group.

---

### TP5_scriptADNettoyage.ps1 – Deleting OUs
```powershell
Function effaceUO ($name) {
    if (Get-ADOrganizationalUnit -filter "DistinguishedName -like 'ou=$name,dc=sodecaf,dc=local'") {
        Remove-ADOrganizationalUnit -Identity "ou=$name,dc=sodecaf,dc=local" -Recursive -Confirm:$false
        Write-Host "UO $name a été supprimée"
    }
    else
    {
        Write-Host "UO $name n'existe pas"
    }
}
```

**Explanation:**
- `Get-ADOrganizationalUnit` – Checks if OU exists.
- `Remove-ADOrganizationalUnit -Recursive -Confirm:$false` – Deletes OU and children without confirmation.
- Functions encapsulate reusable logic.

---

### TP5_scriptAD2.ps1 – Creating Users from CSV
**Highlights:**
- `Import-Csv` – Reads user info.
- `CreatePassword()` – Generates random passwords.
- `New-ADOrganizationalUnit` – Creates OUs per department/function.
- `New-ADGroup` – Creates groups.
- `New-ADUser` – Creates users with generated passwords.
- `Add-ADGroupMember` – Assigns users to groups.
- `try { ... } catch { ... }` – Handles errors for existing users.

**Use case:** Automate bulk Active Directory account creation and group assignment.
