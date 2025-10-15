---
title: "Processus et services"
---

# Processus et services

## Obtenir la liste des processus

```powershell
# Utilisation simple
C:\> Get-Process -Name thunderbird

 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     85   220,00     290,34       4,83    1144   7 thunderbird
     21    37,55      53,00       0,36    2452   7 thunderbird
     38   195,03     206,14       1,09    7668   7 thunderbird

# Utilisation avancée
C:\> Get-Process -Id 1144, 2452 | Sort-Object -Property WorkingSet -Descending | Format-List -Property Id, Handle, WorkingSet, Name, Path

Id         : 1144
Handle     : 2072
WorkingSet : 327077888
Name       : thunderbird
Path       : C:\Program Files\Mozilla Thunderbird\thunderbird.exe

Id         : 2452
Handle     : 3924
WorkingSet : 52580352
Name       : thunderbird
Path       : C:\Program Files\Mozilla Thunderbird\thunderbird.exe

```

- **Handles** : nombre de références vers un objet système, tel que une connexion réseau ou un fichier ouvert.
- **NPM** : mémoire non paginée (en kilo-octet)
- **PM** : mémoire paginée (en mébioctet)
- **WS** : mémoire en cours d'utilisation (en mébioctet)
- **CPU** : temps processeur total utilisé par le processus depuis son démarrage en seconde
- **Id** : Identifiant du processus
- **SI** : Identifiant de la session de l'utilisateur
- **ProcessName** : Nom du processus

> [!NOTE]
> - La pagination divise la *mémoire virtuelle* en *pages* qui peuvent être déplacées entre la *mémoire vive* et le *disque dur*. La *mémoire virtuelle* permet d'étendre les capacité de la *mémoire vive*, et est généralement paginée.
> - Le mébioctet est égal à **1 048 576 octets**, contrairement au mégaoctet qui est égal à **1 000 000 octets**

## Arrêter un ou plusieurs processus 

```powershell
# Arreter un processus
C:\> Stop-Process -Id 10580

C:\> Get-Process -Id 10580
#Get-Process : Impossible de trouver un processus assorti de l'identificateur 13400.

# Arreter un ensemble de processus
C:\> Stop-Process -Name thunderbird

C:\> Get-Process -Name thunderbird
#Get-Process : Impossible de trouver un processus nommé «thunderbird».
```

## Obtenir la liste des services

```powershell
# Utilisation simple
C:\> Get-Service | Sort-Object -Property Name

Status   Name               DisplayName
------   ----               -----------
Stopped  AarSvc_14a69de0    Agent Activation Runtime_14a69de0
Running  AESMService        Intel® SGX AESM
Stopped  ALG                Service de la passerelle de la couche…

# Utilisation avancée
C:\> Get-Service | Where-Object { $_.UserName -like "*NetworkService*" } | Select-Object Name, StartType, Status, UserName, BinaryPathName | Sort-Object -Property Name | Format-List

Name           : cloudidsvc
StartType      : Manual
Status         : Stopped
UserName       : NT AUTHORITY\NetworkService
BinaryPathName : C:\WINDOWS\system32\svchost.exe -k CloudIdServiceGroup -p
```

## Démarrer et arrêter un ou des services

```powershell
C:\> Stop-Service -Name "MozillaMaintenance"      # Stopped
C:\> Start-Service -Name "MozillaMaintenance"     # Running
C:\> Restart-Service -Name "MozillaMaintenance"   # Running
```

## Supprimer un service

```powershell
C:\> Remove-Service -Name "MozillaMaintenance"
```

## Créer un service

```powershell
$params = @{
  Name = "TestService"
  BinaryPathName = "D:\SOFTS\TestService\TestService.exe"
  DisplayName = "Test service"
  StartupType = "Manual"
  Description = "This is a test service"
}

C:\> New-Service @params
```

> [!NOTE]
> Par défaut, l'utilisateur utilisé pour lancer le service est ```LocalSystem```
