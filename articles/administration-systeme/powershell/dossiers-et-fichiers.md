---
title: "Dossiers et fichiers"
---

# Dossiers et fichiers

## Recherche

### Selon certains attributs

```powershell
# Utilisation simple
C:\> Get-ChildItem

    Directory: C:\Users\romai\source\repos\loyalty-card-manager

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          27/12/2024    11:07                LoyaltyCardManager.Core
d----          27/12/2024    11:07                LoyaltyCardManager.Infrastructure
d----          27/12/2024    11:07                LoyaltyCardManager.Test
d----          27/12/2024    11:07                LoyaltyCardManager.WebApi
-a---          27/12/2024    11:07           6352 .gitignore
-a---          27/12/2024    11:07           1091 LICENSE
-a---          27/12/2024    11:07           2780 LoyaltyCardManager.sln
-a---          27/12/2024    11:07           1180 README.md

# Utilisation avancée
C:\> Get-ChildItem -Recurse -Include "*.sln" -File | Select-Object Mode, LastWriteTime, Length, FullName | Format-List

Mode          : -a---
LastWriteTime : 27/12/2024 11:07:10
Length        : 2780
FullName      : C:\Users\romai\source\repos\loyalty-card-manager\LoyaltyCardManager.sln
```

Les lettres dans l'attribut *mode* signifient :
- ```l``` : lien (link)
- ```d``` : répertoire (directory)
- ```a``` : archive
- ```r``` : en lecture seule (read only)
- ```h``` : caché (hidden)
- ```s``` : système (system)

> [!NOTE]
> Le mode *archive* indique que le fichier ou le dossier a été créé ou modifié après la dernière sauvegarde par un logiciel d'archivage.

### Selon le contenu

```powershell
C:\> Get-ChildItem -Recurse -Include "*.csproj" -File | Select-String -Pattern "Swashbuckle" | Format-List LineNumber, Line, Path

LineNumber : 10
Line       :     <PackageReference Include="Swashbuckle.AspNetCore" Version="6.2.3" />
Path       : C:\Users\romai\source\repos\loyalty-card-manager\LoyaltyCardManager.WebApi\LoyaltyCardManager.WebApi.csproj
```

## Affichage et listage

### Contenu en temps réel

```powershell
C:\> Get-Content -Path \\machine001\LOGS\app\app1_20250407.log, \\machine003\LOGS\app\app1_20250407.log -Tail 1 -Wait
```

## Création de nouveaux éléments

```powershell
C:\> New-Item -Path User.cs -ItemType File # Type par défaut
C:\> New-Item -Path User.cs -ItemType Directory
```

## Copie et déplacement

```powershell
C:\> Copy-Item -Path .\* -Include "*Entity.cs" -Destination .\Entities\
```
