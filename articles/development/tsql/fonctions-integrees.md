---
title: Fonction intégrées
---

# Fonctions intégrées

## Scalaires

Elles ne peuvent retourner qu'une seule valeur

### Dates

- [DATENAME][1] : Retourne une chaine de caractère représentant la partie spécifiée
- [YEAR][2] : Retourne un entier représentant l'année de la date
- [DATEDIFF][3] : Retourne un nombre représentant la différence entre deux dates selon une partie spécifiée 
- [DAY][4] : Retourne un entier représentant le jour de la date

```sql
DECLARE @date DATETIME = '2025-01-01'
SELECT DATENAME(mm, @date) UNION ALL
-- janvier
SELECT CAST(YEAR(@date) AS VARCHAR) UNION ALL
-- 2025
SELECT CAST(DATEDIFF(dd, '2024-01-01', @date) AS VARCHAR) UNION ALL
-- 366
SELECT CAST(DAY(@date) AS VARCHAR)
-- 1
```

### Nombres

- [ROUND][5] : Retourne une valeur numérique, arrondie à la longueur ou à la précision indiquée
- [FLOOR][6] : Retourne le nombre entier à l'arrondi infèrieur
- [CEILING][7] : Retourne le nombre entier à l'arrondi superieur
- [SQUARE][8] : Retourne le carré de la valeur spécifiée
- [SQRT][9] : Retourne la racine carrée de la valeur spécifiée
- [LOG][10] : Retourne le logarithme népérien de la valeur spécifiée
- [RAND][11] : Retourne un flottant pseudo-aléatoire, compris entre 0 et 1, valeurs exclues

```sql
DECLARE @number FLOAT = 2620.496545
SELECT ROUND(@number, 3) UNION ALL
-- 2620,497 
SELECT FLOOR(@number) UNION ALL
-- 2620
SELECT CEILING(@number) UNION ALL
-- 2621
SELECT SQUARE(@number) UNION ALL
-- 6867002,142356937
SELECT SQRT(@number) UNION ALL
-- 51,19078574313936
SELECT LOG(@number) UNION ALL
-- 7,871119099790675
SELECT RAND() 
-- 0,9231973863380282
```

### Chaines de caractères

- [UPPER][12] : Transforme les caractères d'une expression en caractères majuscules
- [LOWER][13] : Transforme les caractères d'une expression en caractères minuscules
- [LEN][14] : Retourne le nombre de caractères d'une expression, à l’exception des espaces de fin
- [REVERSE][15] : Retourne l'ordre inverse d'une valeur de chaîne
- [CHARINDEX][16] : Retourne la position d'une expression de caractères dans une deuxième
- [LEFT][17] : Retourne la partie de gauche d'une chaîne de caractères avec le nombre spécifié de caractères
- [SUBSTRING][18] : Retourne une partie d'une expression de type caractère
- [REPLACE][19] : Replacer une valeur par une autre dans un chaîne de caractères

```sql
DECLARE @string VARCHAR = 'Burger King'
SELECT UPPER(@string) UNION ALL
-- BURGER KING
SELECT LOWER(@string) UNION ALL
-- burger king
SELECT CAST(LEN(@string) AS VARCHAR) UNION ALL
-- 11
SELECT REVERSE(@string) UNION ALL
-- gniK regruB
SELECT CAST(CHARINDEX(' ', @string) AS VARCHAR) UNION ALL
-- 7
SELECT LEFT(@string, 6) UNION ALL
-- Burger
SELECT SUBSTRING(@string, 8, 6) UNION ALL
-- King
SELECT REPLACE(@string, 'King', 'Queen')
-- Burger Queen
```

### Logiques

- [IIF][20] : Retourne l'une des deux valeurs possibles, selon l'expression booléenne
- [CHOOSE][21] : Retourne l'élèment à l'indice spécifié (base 1)

```sql
DECLARE @statusId INT = 3
SELECT 
  IIF(@statusId IS NULL, 'status not filled', 'status filled')
-- status filled
UNION ALL SELECT
  CHOOSE(@statusId, 'Active', 'Inactive', 'Expired')
-- Expired
```

## Classement

```sql
SELECT
    RANK() OVER(ORDER BY lc.MerchantName) AS RankByName,
    RANK() OVER(PARTITION BY lc.GroupId ORDER BY MerchantName) AS RankByNameByGroup,
    g.Name AS GroupName,
    lc.MerchantName
FROM [dbo].[LoyaltyCard] AS lc
JOIN [dbo].[Group] AS g ON lc.GroupId = g.Id
ORDER BY RankByName
```

| RankByName | RankByNameByGroup | GroupName   | MerchantName       |
| ---------- | ----------------- | ----------- | ------------------ |
| 1          | 1                 | Restaurant  | BurgerQueen        |
| 2          | 1                 | Supermarket | ChampionMarket     |
| 3          | 2                 | Supermarket | King Bazar         |
| 4          | 2                 | Restaurant  | Master of the seas |
| 5          | 1                 | Sport       | RoyalFitness       |

> [!note]
> D'autres fonctions de classement tel que `DENSE_RANK`, `NTILE` ou encore `ROW_NUMBER` sont referencées dans [cette documentation](https://learn.microsoft.com/en-us/sql/t-sql/functions/ranking-functions-transact-sql)

## Agrégation

- [COUNT][22] : Retourne le nombre d'éléments non `NULL` si un nom de colonne est renseigné
- [MIN][23] : Retourne la valeur minimale
- [MAX][24] : Retourne la valeur maximale
- [AVG][25] : Retourne la moyenne des valeurs en ignorant les valeurs `NULL`

```sql
DECLARE @table TABLE (Number FLOAT NULL)
INSERT INTO @table (Number) VALUES (1), (2), (3), (4), (NULL)

SELECT COUNT(Number) FROM @table UNION ALL
-- 4
SELECT COUNT(*) FROM @table UNION ALL
-- 5
SELECT COUNT(Number) FROM @table UNION ALL
-- 4
SELECT MIN(Number) FROM @table UNION ALL
-- 1
SELECT MAX(Number) FROM @table UNION ALL
-- 4
SELECT AVG(Number) FROM @table 
-- 2.5
```

## Groupement

Les colonnes de la clause `SELECT` doivent être utilisée dans une fonction d'agrégation ou être ajoutées dans la clause `GROUP BY`.

```sql
DECLARE @table TABLE (Letter char(1))
INSERT INTO @table (Letter) VALUES ('a'), ('b'), ('b'), ('c')

SELECT 
    Letter,
    COUNT(*) AS Total
FROM @table
GROUP BY Letter
ORDER BY Letter
```

| Letter | Total |
| ------ | ----- |
| a      | 1     |
| b      | 2     |
| c      | 1     |

[1]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/datename-transact-sql
[2]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/year-transact-sql
[3]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/datediff-transact-sql
[4]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/day-transact-sql
[5]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/round-transact-sql
[6]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/floor-transact-sql
[7]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/ceiling-transact-sql
[8]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/square-transact-sql
[9]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/sqrt-transact-sql
[10]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/log-transact-sql
[11]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/rand-transact-sql
[12]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/upper-transact-sql
[13]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/lower-transact-sql
[14]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/len-transact-sql
[15]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/reverse-transact-sql
[16]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/charindex-transact-sql
[17]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/left-transact-sql
[18]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/substring-transact-sql
[19]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/replace-transact-sql
[20]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/logical-functions-iif-transact-sql
[21]: https://learn.microsoft.com/en-us/sql/t-sql/functions/logical-functions-choose-transact-sql
[22]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/count-transact-sql
[23]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/min-transact-sql
[24]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/max-transact-sql
[25]: https://learn.microsoft.com/fr-fr/sql/t-sql/functions/avg-transact-sql
