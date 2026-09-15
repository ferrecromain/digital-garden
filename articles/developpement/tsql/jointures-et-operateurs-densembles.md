---
title: Jointures et opérateurs d'ensembles
---

# Jointures et opérateurs d'ensemble

## Jointures

Les opérations de jointures sont réalisées en créant une *table virtuelle* issue du [produit cartésien](https://fr.wikipedia.org/wiki/Produit_cart%C3%A9sien) de **deux tables**, qui sera par la suite **filtrée** selon le **type de jointure souhaitée**.

Considerons les jeux de données ci dessous :

```sql
DECLARE @users TABLE (
	Name NVARCHAR(50), 
	GroupId INT)
DECLARE @groups TABLE (
    Id INT,
    Name NVARCHAR(50),
    ParentGroupId INT NULL
)

INSERT INTO @users (Name, GroupId) VALUES
	('John', 1),
	('Alice', 2),
	('Clark', NULL)
INSERT INTO @groups (Id, Name, ParentGroupId) VALUES
(1, 'Administrator', NULL),
(2, 'Moderator', 1),
(3, 'Contributor', 2)
```

### Interne

Conserve les **utilisateurs** appartenant à un **groupe**

> [!note]
> Le mot clé `INNER` qui précède `JOIN` peut être omis bien qu'il soit **recommandé**

```sql
-- Syntaxe ANSI SQL-89 
SELECT u.Name AS 'User', g.Name AS 'Group'
FROM @users AS u, @groups AS g
WHERE g.id = u.GroupId

-- Syntaxe ANSI SQL-92
SELECT u.Name AS 'User', g.Name AS 'Group'
FROM @users AS u
INNER JOIN @groups AS g ON g.id = u.GroupId
```

| User  | Group         |
| ----  | ------------- |
| John  | Administrator |
| Alice | Moderator     |

### Externe

- **Gauche** : Conserve tous les **utilisateurs** avec les **groupes** associés ou `NULL` pour ceux qui n'en ont pas
- **Droite** : Conserve toutes les **groupes** avec leur **utilisateurs** ou `NULL` pour ceux qui n'en ont pas
- **Complète** : Conserve à la fois :
	- tous les **utilisateurs** avec leur **groupe** ou `NULL` pour ceux qui n'en ont pas
	- tous les **groupes** avec les **utilisateurs** associés ou `NULL` pour ceux qui n'en ont pas

> [!note]
> Le mot clé `OUTER` entre `LEFT|RIGHT|FULL` et `JOIN` peut être omis bien qu'il soit **recommandé**

```sql
SELECT u.Name AS 'User', g.Name AS 'Group'
FROM @users AS u
-- Jointure externe gauche
LEFT OUTER JOIN @groups AS g ON g.Id = u.GroupId
-- Jointure externe droite
RIGHT OUTER JOIN @groups AS g ON g.Id = u.GroupId
-- Jointure externe complète
FULL OUTER JOIN @groups AS g ON g.Id = u.GroupId
```


<table>
	<tr>
		<th colspan="2">Gauche</th>
		<th colspan="2">Droite</th>
		<th colspan="2">Complète</th>
	</tr>
	<tr>
		<td>User</td>
		<td>Group</td>
		<td>User</td>
		<td>Group</td>
		<td>User</td>
		<td>Group</td>
	</tr>
	<tr>
		<td>John</td>
		<td>Administrator</td>
		<td>John</td>
		<td>Administrator</td>
		<td>John</td>
		<td>Administrator</td>
	</tr>
	<tr>
		<td>Alice</td>
		<td>Moderator</td>
		<td>Alice</td>
		<td>Moderator</td>
		<td>Alice</td>
		<td>Moderator</td>
	</tr>
	<tr>
		<td>Clark</td>
		<td><i>NULL</i></td>
		<td><i>NULL</i></td>
		<td>Contributor</td>
		<td>Clark</td>
		<td><i>NULL</i></td>
	</tr>
	<tr>
		<td colspan="4"></td>
		<td><i>NULL</i></td>
		<td>Contributor</td>
	</tr>
<table>

### Croisée

Conserve tous les résultats en n'appliquant aucun filtre

```sql
SELECT g.Name AS 'GroupName', lc.MerchantName
FROM [dbo].[Group] AS g
CROSS JOIN [dbo].[LoyaltyCard] AS lc
```

### Réflexive

Conserve tous les **groupes** avec leur **groupe enfant** ou `NULL` pour ceux qui n'en ont pas

```sql
SELECT parentGroup.Name AS 'ParentGroupName', childGroup.Name AS 'ChildGroupName' 
FROM @groups AS parentGroup
LEFT OUTER JOIN @groups AS childGroup ON parentGroup.Id = childGroup.ParentGroupId
```

> [!Note] 
> Le produit cartésien est appliqué à partir de deux instances d'une seule et même table.

| ParentGroupName | ChildGroupName |
| --------------- | -------------- |
| Administrator   | Moderator      |
| Moderator       | Contributor    |
| Contributor     | *NULL*         |


## Opérateurs d'ensembles

Considerons les jeux de données ci dessous :

```sql
DECLARE @player TABLE (
    Id INT PRIMARY KEY,
    NickName NVARCHAR(100)
);
DECLARE @character TABLE (
    Id INT PRIMARY KEY,
    PlayerId INT,
    NickName NVARCHAR(100)
);

INSERT INTO @player VALUES (1, 'johndoe123'), (2, 'janedoe456');
INSERT INTO @character VALUES (1, 1, 'ultrawarrior'), (2, 2, 'janedoe456');
```

> [!NOTE]
> Les opérateurs d'ensemble ne doivent pas être confondu avec ```JOIN``` qui créé un produit cartésien.

```sql
SELECT NickName FROM @player 
-- Unit les lignes deux jeux de résultats en incluant les doublons
UNION ALL
-- Unit les lignes des deux jeux de résultats en excluant les doublons 
UNION
-- Croise les lignes deux jeux de résultats
INTERSECT
-- Soustrait les lignes du deuxième jeu de résultats du premier
EXCEPT
SELECT NickName FROM @character;
```

| ```UNION```  | ```UNION ALL``` | ```ÌNTERSECT``` | ```P EXCEPT C ``` | ```C EXCEPT P``` |
|--------------|-----------------|-----------------|-------------------|------------------|
| janedoe456   | johndoe123      | janedoe456      | johndoe123        | ultrawarrior     |
| johndoe123   | janedoe456      |                 |                   |                  |
| ultrawarrior | ultrawarrior    |                 |                   |                  |
|              | janedoe456      |                 |                   |                  |

