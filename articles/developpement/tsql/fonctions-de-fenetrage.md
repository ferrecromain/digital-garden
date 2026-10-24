---
title: Fonction de fenêtrage
---

# Fonctions de fenêtrage

La fonction ```OVER``` définit les lignes sur lesquelles une fonction de fenêtrage est appliqué. Elle peut-être utilisée avec :
- ```PARTITION BY``` : Groupe les ligne par colonnes
- ```ORDER BY``` : Trie les lignes
- ```ROW``` / ```RANGE``` : Délimite les lignes

```sql
DECLARE @Group TABLE 
(
    Id INT PRIMARY KEY NOT NULL,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Class VARCHAR(50) NOT NULL,
    Rank INT NOT NULL,
    Points INT NOT NULL
)

INSERT INTO @Group (Id, FirstName, LastName, Class, Rank, Points) VALUES
(1, 'John', 'Doe', 'A', 1, 95),
(2, 'Jane', 'Smith', 'B', 2, 88),
(3, 'Emily', 'Johnson', 'A', 3, 92),
(4, 'Michael', 'Brown', 'C', 4, 85),
(5, 'Sarah', 'Davis', 'B', 5, 90);
```

## Agrégation

```sql
SELECT FirstName, LastName, Class, Rank, Points, 
    AVG(Points) OVER (PARTITION BY Class) AS AveragePointsByClass
FROM @Group
```

| FirstName  | LastName   | Class      | Rank       | Points     | AveragePoints |
| ---------- | ---------- | ---------- | ---------- | ---------- | ------------- |
| John       | Doe        | A          | 1          | 95         | 93            |
| Emily      | Johnson    | A          | 3          | 92         | 93            |
| Jane       | Smith      | B          | 2          | 88         | 89            |
| Sarah      | Davis      | B          | 5          | 90         | 89            |
| Michael    | Brown      | C          | 4          | 85         | 85            |


## Classement

```sql

```
