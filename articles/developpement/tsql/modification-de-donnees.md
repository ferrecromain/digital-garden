---
title: Modification de données
---

# Modification de données

Les tables ci-dessous sont utilisées dans les différents exemples :

```sql
DECLARE @books TABLE (
    Id INT IDENTITY(1,1),
	Title NVARCHAR(50),
    ISIN NVARCHAR(13),
    Quantity INT DEFAULT 0,
    QuantityUpdatedOn DATETIME NULL
)
DECLARE @orders TABLE (
    Id INT IDENTITY(1,1),
    Title NVARCHAR(50),
    ISIN NVARCHAR(13),
    Quantity INT,
    UpdatedOn DATETIME DEFAULT GETDATE(),
    Status NVARCHAR(20) DEFAULT 'Pending'
)
```

> [!NOTE]
> Nous pouvons utiliser le mot clé ```DEFAULT``` qui, en fonction de la configuration de la colonne, peut retourner :
> - ```NULL``` 
> - Une valeur par défaut 
> - Une valeur générée automatiquement

## Insertion

### Jeu de données explicite

Insertion de 3 livres avec leur nombre d'exemplaires.

```sql
INSERT INTO @books (Title, ISIN, Quantity) VALUES
    ('The Great Gatsby', '9780743273565', 8),
    ('Fahrenheit 451', '9780451524935', 3),
    ('To Kill a Mockingbird', '9780061120084', DEFAULT)
```

| Id | Title                 | ISIN          | Quantity |
|----|-----------------------|---------------|----------|
| 1  | The Great Gatsby      | 9780743273565 | 8        |
| 2  | Fahrenheit 451        | 9780451524935 | 3        |
| 3  | To Kill a Mockingbird | 9780061120084 | 0        |

> [!NOTE]
> La liste des noms de colonnes est facultative, dans ce cas on considère l'ensemble des colonnes de la table.

### Résultat d'une requête

Création de commandes pour réapprovisionner les livres de la bibilothèque dont le stock est inférieur à 6, afin d'en avoir 15 au total.

```sql
INSERT INTO @orders (Title, ISIN, Quantity)
SELECT Title, ISIN, 15 - b.Quantity
FROM @books AS b
WHERE b.Quantity <= 5
```

| Id | Title                 | ISIN          | Quantity | UpdatedOn               | Status  |
|----|-----------------------|---------------|----------|-------------------------|---------|
| 1  | Fahrenheit 451        | 9780451524935 | 12       | 2025-10-17 16:08:01.580 | Pending |
| 2  | To Kill a Mockingbird | 9780061120084 | 15       | 2025-10-17 16:08:01.580 | Pending |

> [!NOTE]
> Il existe le jeu d'instruction ```INSERT EXEC``` pour les données provenant de procèdures stockées. Il convient toutefois de vérifier qu'elles ne retournent qu'un seul jeu de résultats.

## Mise à jour

### Par simple expression conditionelle

Fermeture de la première commande.

```sql
UPDATE @orders
SET Status = 'Closed', UpdatedOn = GETDATE()
WHERE Id = 1
```

| Id | Title                 | ISIN          | Quantity | UpdatedOn               | Status  |
|----|-----------------------|---------------|----------|-------------------------|---------|
| 1  | Fahrenheit 451        | 9780451524935 | 12       | 2025-10-17 16:08:01.580 | Closed  |
| 2  | To Kill a Mockingbird | 9780061120084 | 15       | 2025-10-17 16:08:01.580 | Pending |

> [!WARNING]
> En l'absence de clause ```WHERE``` l'ensemble des lignes de la table seront modifiées

### Par jointure

Mise à jour des stocks des livres de la bibliothèque à partir des commandes achevées.

```sql
UPDATE b
SET b.Quantity = b.Quantity + o.Quantity, b.QuantityUpdatedOn = GETDATE()
FROM @books AS b
JOIN @orders AS o ON b.ISIN = o.ISIN
WHERE o.Status = 'Closed' AND o.UpdatedOn > b.QuantityUpdatedOn
```

| Id | Title                 | ISIN          | Quantity |
|----|-----------------------|---------------|----------|
| 1  | The Great Gatsby      | 9780743273565 | 8        |
| 2  | Fahrenheit 451        | 9780451524935 | 15       |
| 3  | To Kill a Mockingbird | 9780061120084 | 0        |

Nous constatons que le nombre d'exemplaire du livre *Fahrenheit 451* dans la bibliothèque a bien été mis à 15, ce qui n'est pas le cas de *To Kill a Mockingbird* car la commande est toujours en cours.

## Suppression

### Par simple expression conditionnelle

Suppression du livre *Fahrenheit 451* qui ne sera plus proposé par la bibliothèque.

```sql
DELETE FROM @books
WHERE Title = 'Fahrenheit 451'
```

| Id | Title                 | ISIN          | Quantity |
|----|-----------------------|---------------|----------|
| 1  | The Great Gatsby      | 9780743273565 | 8        |
| 3  | To Kill a Mockingbird | 9780061120084 | 0        |

> [!WARNING]
> En l'absence de clause ```WHERE``` l'ensemble des lignes de la table seront supprimmées.

### Toutes les lignes d'une table

```sql
TRUNCATE TABLE @orders
```

> [!TIP]
> ```TRUNCATE TABLE``` est plus performant qu'un ```DELETE FROM``` sans clause ```WHERE```.

## Fusion

Les livres referencés dans la bibliothèque sont **synchronisés** par rapport aux commandes effectuées. Autrement dit :
1. On passe en revu chaque commandes effectuées
2. On teste la correspondance vers les livres referencés à l'aide de la clause `ON`
3. On effectue les opérations souhaitées selon la structure conditionelle définie

Dans l'exemple suivant :
- La **table cible** représente les livres referencés
- La **table source** représente les commandes

```sql
MERGE INTO @books AS b
USING @orders AS o
ON b.ISIN = o.ISIN
-- Si la commande est achevée pour un livre déjà referencé, on met à jour le nombre d'exemplaires disponible
WHEN MATCHED AND o.Status = 'Closed' AND o.UpdatedOn > b.QuantityUpdatedOn THEN
    UPDATE SET b.Quantity = b.Quantity + o.Quantity, b.QuantityUpdatedOn = GETDATE()
-- Si la commande n'est pas achevée mais que le livre n'est pas referencé, on le créé
WHEN NOT MATCHED BY TARGET AND o.Status != 'Close' THEN
    INSERT (Title, ISIN)
    VALUES (o.Title, o.ISIN)
-- Si la commande est achevée mais que le livre n'est pas referencé, on le créé avec le nombre d'exemplaire reçu
WHEN NOT MATCHED BY TARGET AND o.Status = 'Close' THEN
    INSERT (Title, ISIN, Quantity, QuantityUpdatedOn)
    VALUES (o.Title, o.ISIN, o.Quantity, GETDATE());
```

> [!NOTE]
> Il existe également la condition ```WHEN NOT MATCHED BY SOURCE THEN``` que l'on utilise généralement avec l'action `DELETE`. On peut supprimer ainsi les lignes de la table *cible* qui n'ont pas de correspondances avec la table *source*.
