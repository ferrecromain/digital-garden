---
title: Chiffrement Bitlocker
---

# Chiffrement Bitlocker

BitLocker permet de **chiffrer des lecteurs** (disques durs, stockage amovible ou encore VHD), afin de protéger leurs contenus **à froid** contre le vol et l'exposition de données.

Nous allons utiliser cette fonctionnalité **intégrée à Windows**, à l'aide de :
1. La commande [manage-bde](https://docs.microsoft.com/fr-fr/windows-server/administration/windows-commands/manage-bde) (à executer en tant qu'administrateur).
2. L'assistant fenêtré dans certains cas.

## Afficher l'état des lecteurs

Saisissons la commande ```manage-bde -status``` afin d'afficher les états de BitLocker pour l'ensemble des lecteurs du système.

Ci-dessous nous constatons que notre unique lecteur (celui du système d'exploitation) n'est pas chiffré.

```powershell
manage-bde -status
```

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.22000
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Volumes de disques pouvant être protégés avec le
Chiffrement de lecteur BitLocker :
Volume C: [Personel]
[Volume du système d’exploitation]
  
    Taille :                     156,79 Go
    Version de BitLocker :       2.0
    État de la conversion :      Intégralement déchiffré
    Pourcentage chiffré :        0,0%
    Méthode de chiffrement :     Aucun
    État de la protection :      Protection désactivée
    État du verrouillage :       Déverrouillé
    Champ d’identification :     Inconnu
    Protecteurs de clés :
        TPM
        Mot de passe numérique
```

## Chiffrer un lecteur

### Préparation

Nous avons choisi de chiffrer le lecteur de la façon suivante :
* Protecteurs de clé : TPM + PIN
* Algorithme de chiffrement XTS AES 256 bits
* Méthode de chiffrement : intégrale
* Clé de récupération : sauvegardée sur notre compte Microsoft

Certaines de ces directives doivent être activées à l'aide de l'**éditeur de stratégie de groupe locale de Windows**, faute de quoi BitLocker ne pourra pas les prendre en charge.

Pour se faire nous devons :
1. Appuyer simultanément sur <kbd>Win</kbd> + <kbd>R</kbd>.
2. Saisir ```gpedit.msc``` puis appuyer sur *OK*
3. Dans l'arborescence à gauche, se rendre sur l'élément : *Configuration ordinateur\Modèles d'administration\Composants Windows\Chiffrement de lecteur BitLocker\Lecteurs du système d'exploitation*.
4. Double cliquer sur l'élément *Exiger une authentification supplémentaire au démarrage*.
5. Choisir *Activé* et cliquer sur *OK*.
6. Double cliquer sur l'élément *Autoriser les codes confidentiels améliorés au démarrage*
7. Choisir *Activé* puis cliquer sur *OK*
### Démarrer le chiffrement

```powershell
manage-bde -on C: -tpmandpin -encryptionmethod "xts_aes256" -recoverypassword
```

Nous sommes ensuite invités à saisir un code PIN d'une longueur maximale de 20 caractères, et à prendre en compte les avertissements  :

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.19041
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Volume C: []
[Volume du système d’exploitation]
Tapez le code PIN à utiliser pour protéger le volume :
Confirmez le code PIN en le tapant à nouveau :
Protecteurs de clés ajoutés :
  
    Mot de passe numérique :
      ID : {458E6A56-9HA0-4EE2-A5EC-123455E868D}
      Mot de passe :
        123456-327536-578439-710468-123456-533236-123456-206954
  
    TPM And PIN :
      ID : {00AFF12B-A5EC-9HA0-8040-123455E868D}
      Profil de validation PCR :
        0, 2, 4, 8, 9, 10, 11
  
ACTIONS REQUISES :
  
    1. Conservez ce mot de passe de récupération numérique dans un endroit
    sûr, loin de votre ordinateur :
  
    123456-327536-578439-710468-123456-533236-123456-206954
  
    Pour éviter toute perte de données, enregistrez ce mot de passe
    immédiatement ; grâce à lui vous êtes sûr de pouvoir déverrouiller le
    volume chiffré.
  
    2. Redémarrez l’ordinateur pour exécuter un test matériel.
    (Tapez « shutdown /? » pour obtenir des instructions de ligne de commande.)
  
    3. Tapez « manage-bde -status » pour vérifier si le test matériel a
    réussi.
  
REMARQUE : le chiffrement commencera après la réussite du test matériel.
```

Nous pouvons à présent redémarrer notre ordinateur afin de débuter l'opération de chiffrement du lecteur :

```powershell
shutdown /r
```
## Gérer les protecteurs de clé
### Afficher les protecteurs de clé d'un lecteur
Pour afficher les informations détaillées sur les protecteurs de clé d'un lecteur :

```powershell
manage-bde -protectors -get C:
```

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.22000
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Volume C: [Personel]
Tous les protecteurs de clés
  
    TPM And PIN :
      ID : {3F6C4C56-2564-492E-8599-4DF1244E19AD}
      Profil de validation PCR :
        7, 11
        (Utilise le démarrage sécurisé pour la validation d’intégrité.)
  
    Mot de passe numérique :
      ID : {570132D0-AC6B-459F-9110-39AA26C957F2}
      Mot de passe :
        695970-708774-360316-079849-474177-198979-076835-400936
```

### Modifier le code PIN

```powershell
manage-bde -changepin C:
```

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.22000
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Tapez le nouveau code PIN :
Confirmez le nouveau code PIN en le tapant à nouveau :
Votre code PIN a été mis à jour avec succès.
```

### Supprimer un protecteur de clé
  
Dans notre exemple, nous souhaitons supprimer l'ensemble des protecteurs de type **mot de passe de récupération** :

```powershell
manage-bde -protectors -delete C: -type recoverypassword
```

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.22000
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Volume C: [Personel]
Protecteurs de clés de type Mot de passe numérique
  
    Mot de passe numérique :
      ID : {A1E4D33C-AF7F-4AEF-A5G1-8B99D709381F}
      Mot de passe :
        549762-316457-986231-457986-531457-958624-165471-871414
  
Le protecteur de clé avec l’ID "{A1E4D33C-AF7F-4AEF-A5G1-8B99D709381F}" a été supprimé.
```

### Ajouter un protecteur de clé

Pour ajouter un nouveau protecteur tel un mot de passe de récupération, nous procédons de la façon suivante :

```powershell  
manage-bde -protectors -add C: -RecoveryPassword
```

```console
Chiffrement de lecteur BitLocker : outil de configuration version 10.0.22000
Copyright (C) 2013 Microsoft Corporation. Tous droits réservés.
  
Protecteurs de clés ajoutés :
  
    Mot de passe numérique :
      ID : {3A09A3B3-7ED6-456C-AFD8-1BED949601AF}
      Mot de passe :
        549762-316457-986231-457986-531457-958624-165471-871414
  
ACTIONS REQUISES :
  
    1. Conservez ce mot de passe de récupération numérique dans un endroit
    sûr, loin de votre ordinateur :
  
    549762-316457-986231-457986-531457-958624-165471-871414
  
    Pour éviter toute perte de données, enregistrez ce mot de passe
    immédiatement ; grâce à lui vous êtes sûr de pouvoir déverrouiller le
    volume chiffré.
```

### Sauvegarder un mot de passe de récupération

Il est recommandé de conserver une copie du mot de passe de récupération dans un endroit sûr. Pour des raisons pratiques, nous avons opté pour la sauvegarde sur notre compte Microsoft.

Depuis le bureau Windows, nous devons :
1. Appuyer simultanément sur <kbd>Win</kbd> + <kbd>R</kbd>.
2. Saisir ```control panel``` puis appuyer sur *OK*.control panel
3. Cliquer sur l'élément *Chiffrement de lecteur Bitlocker*
4. Sur le lecteur correspondant, cliquer sur *Sauvegarder votre clé de récupération*.
5. Depuis la nouvelle fenêtre, cliquer sur *Enregistrer sur votre compte Microsoft*.

Nous pouvons ensuite gérer nos mots de passe de récupération sur [notre compte Microsoft](https://account.microsoft.com/devices/recoverykey), que l'on peut distinguer à l'aide de leurs identifiants.
