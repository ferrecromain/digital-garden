---
title: Création d'objets
---

# Création de tables et insertion de données

| Type               | Syntaxe           | Caractéristiques                                                           |
|--------------------|-------------------|----------------------------------------------------------------------------|
| Régulière          | ```TableName```   | Persistées dans la base de données                                         |
| Temporaire locale  | ```#TableName```  | N'existe que pendant la durée de la session où elle a été créée            |
| Temporaire globale | ```##TableName``` | Existe dans toutes les sessions pour la durée de celle où elle a été créée |

> [!INFO]
> Une session a un identifiant qui peut être obtenu à l'aide de la requête ```SELECT @@SPID```

```sql
-- Create tables
CREATE TABLE [Group]
(
    Id INT PRIMARY KEY NOT NULL,
    Name VARCHAR(100) NOT NULL
)

CREATE TABLE [User]
(
    Id INT PRIMARY KEY NOT NULL,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Birthday DATE NOT NULL,
    UpdatedOn DATETIME DEFAULT GETDATE(),
    GroupId INT NULL FOREIGN KEY REFERENCES [Group](Id)
);

-- Insert data
INSERT INTO [Group] (Id, Name) VALUES 
(1, 'Family'),
(2, 'Friends'),
(3, 'Work');

INSERT INTO [User] (Id, FirstName, LastName, Birthday, GroupId) VALUES 
(1, 'John', 'Doe', '1990-01-01', 1),
(2, 'Jane', 'Smith', '1985-05-15', 2),
(3, 'Alice', 'Johnson', '1992-07-20', 3),
(4, 'Bob', 'Brown', '1988-03-10', NULL),
(5, 'Charlie', 'Davis', '1995-12-25', 1);
```

# Créations de vues

Elles permettent d’introduire une couche d’abstraction afin de masquer la structure des tables sous-jacentes, susceptible d’évoluer.

```sql
-- Create view
CREATE VIEW UsersStatsByGroups
AS
SELECT 
    g.Name AS "GroupName", 
    COUNT(u.Id) AS UserCount, 
    MAX(u.UpdatedOn) AS LastUserUpdate
FROM [User] AS u
JOIN [Group] AS g ON g.Id = u.GroupId
GROUP BY g.Id, g.Name

-- Use view
SELECT GroupName, LastUserUpdate
FROM dbo.UsersStatsByGroups
```

# Common Table Expression (CTE)

Elles permettent de définir un jeu de résultats nommé, réutilisable dans le périmétre d'execution d'une requête.

> [!WARNING]
> Après l'execution de la requête ```SELECT ... FROM CTE_UsersByGroups```, la CTE est détruit et ne peut plus être réutilisé

```sql
WITH CTE_UsersByGroups -- CTE name
AS
(
    SELECT -- CTE request
        u.FirstName, u.LastName, u.UpdatedOn,
        g.Name AS GroupName,
        ROW_NUMBER() OVER (PARTITION BY u.GroupId ORDER BY u.UpdatedOn DESC) AS RowNum
    FROM [User] AS u
    JOIN [Group] AS g ON u.GroupId = g.Id
)
SELECT FirstName, LastName, UpdatedOn, GroupName
FROM [CTE_UsersByGroups] -- CTE usage
WHERE RowNum = 1
```

# Tables dérivées

Elles permettent de définir un jeu de résultats nommé, issue d'une **requête interne**.
A la différence des CTE, la **requête externe** ne peut consommer ces données qu'une seule fois.

> [!CAUTION]
> * Bien que la requête externe n'en ai pas systématiquement besoin, la requête interne doit avoir un alias.
> * On ne parle de table dérivée que si la requête interne est définie dans la calsue ```FROM``` de la requête externe.

```sql
SELECT FirstName, LastName, UpdatedOn, GroupName
FROM (SELECT u.FirstName, u.LastName, u.UpdatedOn, g.Name AS GroupName,
             ROW_NUMBER() OVER (PARTITION BY g.Name ORDER BY u.UpdatedOn DESC) AS RowNum
      FROM [User] AS u
      JOIN [Group] AS g ON u.GroupId = g.Id) AS RankedUsers
WHERE RowNum = 1
```