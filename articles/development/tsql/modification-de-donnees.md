---
title: Modification de données
---

# Modification de données

Les tables ci dessous sont utilisées dans les différents exemples :

```sql
DECLARE @books TABLE (
    Id INT IDENTITY(1,1),
	Title NVARCHAR(50),
    Quantity INT DEFAULT 0
)
DECLARE @orders TABLE (
    Id INT IDENTITY(1,1),
    BookId INT,
    Quantity INT,
    CreatedOn DATETIME DEFAULT GETDATE(),
    Status NVARCHAR(20) DEFAULT 'Pending'
)
```

> [!NOTE]
> Nous pouvons utiliser le mot clé ```DEFAULT``` qui, en fonction de la configuration de la colonne, peut retourner :
> - ```NULL``` 
> - Une valeur par défaut 
> - Une valeur générée automatiquement

## Insertion

### Depuis un jeu de données explicite

Insertion de 3 livres avec leur nombre d'exemplaires

```sql
INSERT INTO @books (Title, Quantity) VALUES
    ('The Great Gatsby', 8),
    ('1984', 3),
    ('To Kill a Mockingbird',  DEFAULT)
```

| Id | Title                 | Quantity |
| -- | -----                 | -------- |
| 1  | The Great Gatsby      | 8        |
| 2  | 1984                  | 3        |
| 3  | To Kill a Mockingbird | 0        |

> [!NOTE]
> La liste des noms de colonnes est facultative, dans ce cas on considère l'ensemble des colonnes de la table.

### Depuis le resultat d'une requête

Création de commandes pour réapprovisionner les livres de la bibilothèque dont le stock est inférieur à 6, afin d'en avoir 15 au total.

```sql
INSERT INTO @orders (BookId, Quantity)
SELECT Id, 15 - b.Quantity
FROM @books AS b
WHERE b.Quantity < 6
```

| Id | BookId | Quantity | CreatedOn               | Status  |
| -- | ------ | -------- | ---------               | ------  |
| 1  | 2      | 12       | 2025-10-17 16:08:01.580 | Pending |
| 2  | 3      | 15       | 2025-10-17 16:08:01.580 | Pending |

> [!NOTE]
> Il existe le jeu d'instruction ```INSERT EXEC``` pour les données provenant de procèdures stockées. Il convient toutefois de vérifier qu'elle ne retournent qu'un seul jeu de résultat.

## Mise à jour

### A l'aide d'une simple expression conditionelle

Fermeture de la première commande.

```sql
UPDATE @orders
SET Status = 'Closed'
WHERE Id = 1
```

| Id | BookId | Quantity | CreatedOn               | Status  |
| -- | ------ | -------- | ---------               | ------  |
| 1  | 2      | 12       | 2025-10-17 16:08:01.580 | Closed  |
| 2  | 3      | 15       | 2025-10-17 16:08:01.580 | Pending |

> [!WARNING]
> En l'absence de clause ```WHERE``` l'ensemble des lignes de la table seront modifiées

### A l'aide d'une jointure

Mise à jour des stocks des livres de la bibliothèque à partir des commandes achevées.

```sql
UPDATE b
SET b.Quantity = b.Quantity + o.Quantity
FROM @books AS b
JOIN @orders AS o ON b.Id = o.BookId
WHERE o.Status = 'Closed'
```

| Id | Title                 | Quantity |
| -- | -----                 | -------- |
| 1  | The Great Gatsby      | 8        |
| 2  | 1984                  | 15       |
| 3  | To Kill a Mockingbird | 0        |

Nous constatons que le nombre d'exemplaire du livre "1984" dans la bibliothèque a bien été mis à 15, ce qui n'est pas le cas de "To Kill a Mockingbird' car la commande est toujours en cours.
