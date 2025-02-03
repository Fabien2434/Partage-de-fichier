# Configuration d'un Serveur de Fichiers sur Windows Server 2022 avec ADDS

## Prérequis
- Un serveur Windows Server 2022 avec le rôle ADDS déployé
- Un poste client Windows 10
- Un accès administrateur sur le serveur et le poste client

---

## 1. Installation du rôle Serveur de fichiers

### Avec le Gestionnaire de Serveur
1. Ouvrir le **Gestionnaire de Serveur**.
2. Cliquer sur **Ajouter des rôles et fonctionnalités**.
3. Sélectionner **Installation basée sur un rôle ou une fonctionnalité**.
4. Choisir le serveur cible.
5. Dans la liste des rôles, cocher **Services de fichiers et de stockage > Serveur de fichiers**.
6. Cliquer sur **Suivant**, puis **Installer**.

### Avec PowerShell
```powershell
Install-WindowsFeature FS-FileServer -IncludeManagementTools
```

Vérification de l'installation :
```powershell
Get-WindowsFeature FS-FileServer
```

---

## 2. Création du dossier "Documents_Entreprise"
```powershell
New-Item -Path "C:\Documents_Entreprise" -ItemType Directory
```

Création des sous-dossiers :
```powershell
New-Item -Path "C:\Documents_Entreprise\RH" -ItemType Directory
New-Item -Path "C:\Documents_Entreprise\Comptabilité" -ItemType Directory
New-Item -Path "C:\Documents_Entreprise\Direction" -ItemType Directory
```

---

## 3. Configuration du partage
```powershell
New-SmbShare -Name "Docs" -Path "C:\Documents_Entreprise" -FullAccess "Administrateurs" -ReadAccess "Utilisateurs du domaine"
```

---

## 4. Configuration des permissions NTFS
### Accès en lecture/écriture pour les groupes spécifiques
```powershell
$acl = Get-Acl "C:\Documents_Entreprise\RH"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("RH","Modify","ContainerInherit,ObjectInherit","None","Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\Documents_Entreprise\RH" $acl

$acl = Get-Acl "C:\Documents_Entreprise\Comptabilité"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("Comptabilité","Modify","ContainerInherit,ObjectInherit","None","Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\Documents_Entreprise\Comptabilité" $acl

$acl = Get-Acl "C:\Documents_Entreprise\Direction"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("Direction","Modify","ContainerInherit,ObjectInherit","None","Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\Documents_Entreprise\Direction" $acl
```

### Accès en lecture seule pour tous les utilisateurs du domaine
```powershell
$acl = Get-Acl "C:\Documents_Entreprise"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("Utilisateurs du domaine","Read","ContainerInherit,ObjectInherit","None","Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\Documents_Entreprise" $acl
```

---

## 5. Vérification des partages sur le serveur
```powershell
Get-SmbShare
```

---

## 6. Configuration du lecteur réseau sur un poste client Windows 10
Sur le poste client, exécuter PowerShell en tant qu'administrateur :
```powershell
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\NomDuServeur\Docs" -Persist
```

Vérification de la connexion :
```powershell
Get-PSDrive Z
```

---

## 7. Test d'accès avec différents comptes
1. Se connecter avec un compte du groupe **RH** et vérifier l'accès en lecture/écriture au dossier RH.
2. Se connecter avec un compte du groupe **Comptabilité** et vérifier l'accès en lecture/écriture au dossier Comptabilité.
3. Se connecter avec un compte du groupe **Direction** et vérifier l'accès à tous les dossiers.
4. Se connecter avec un compte standard du domaine et vérifier l'accès en lecture seule.

---

## 8. Critères d'acceptation
✅ Le serveur de fichiers est installé et configuré.
✅ Le partage "Docs" est accessible depuis le réseau.
✅ Les permissions NTFS et de partage sont appliquées correctement.
✅ Les commandes PowerShell fonctionnent correctement.
✅ Les utilisateurs ont bien les permissions attendues.

---

## 9. Conclusion
Cette procédure permet de mettre en place un partage de fichiers sécurisé sur Windows Server 2022. Grâce à PowerShell, la configuration est automatisable et reproductible.

