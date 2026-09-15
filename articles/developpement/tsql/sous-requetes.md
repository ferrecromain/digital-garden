---
title: Sous-requêtes
---

Une sous requête, également appellée **requête interne**, est une instruction `SELECT` retournant un ou plusieurs resultats, et est imbriquée dans une autre requête, appellée **requête externe**.

La table de données ci dessous est utilisée dans les différents exemples :

```sql
DECLARE @employees TABLE (
	FirstName NVARCHAR(50), 
	LastName NVARCHAR(50), 
	Position NVARCHAR(50), 
	Salary FLOAT);

INSERT INTO @employees 
	(FirstName, LastName, Position, Salary) 
VALUES
	('John', 'Doe', 'Developer', 2500),
	('Jane', 'Smith', 'Designer', 1900),
	('Bill', 'Gates', 'Manager', 10000),
	('Lisa', 'White', 'Analyst', 3000),
	('Mary', 'Brown', 'Developer', 2700),
    ('James', 'Johnson', 'Designer', 2200),
    ('Patricia', 'Williams', 'Manager', 9500),
    ('Michael', 'Jones', 'Analyst', 3200)
```
## Autonomes

### Scalaire

La *requête interne* renvoie une valeur unique et la *requête externe* traite un résultat unique

```sql
-- Liste des employés ayant un salaire supérieur à la moyenne (4375)
SELECT FirstName, LastName FROM @employees
WHERE Salary > (SELECT AVG(Salary) FROM @employees)
```

| FirstName | LastName | Salary |
| --------- | -------- | ------ |
| Bill      | Gates    | 10000  |
| Patricia  | Williams | 9500   |

### Valeurs multiples

La *requête interne* renvoie plusieurs valeurs, et la *requête externe* traite plusieurs valeurs.

```sql
-- Moyenne des masses salariales par postes
DECLARE @averageTotalSalary FLOAT;

-- Calcul de la moyenne des masses salariales par postes (8750)
SELECT @averageTotalSalary = AVG(TotalSalary)
FROM (
	-- Sous requête à valeurs multiple
    SELECT SUM(Salary) AS TotalSalary
    FROM @employees
    GROUP BY Position
) AS PositionSalaries; -- Alias de la table dérivée

-- Liste des employés appartenant à des postes dont la masse salariale est supèrieure à la masse salariable moyenne de tous les postes
SELECT FirstName, LastName, Salary
FROM @employees e
WHERE e.Position IN (
	-- Sous requête à valeurs multiples
    SELECT Position
    FROM @employees
    GROUP BY Position
    HAVING SUM(Salary) > @averageTotalSalary
);
```

| FirstName | LastName | Salary |
| --------- | -------- | ------ |
| Bill      | Gates    | 10000  |
| Patricia  | Williams | 9500   |

> [!note]
> Les sous requêtes à valeurs multiples peuvent généralement être réécrites à l'aide de jointures.

## Corrélées

Une valeur de la *requête interne* est comparée avec une valeur de la *requête externe*. Cette dernière ne peut pas être executée toute seule.

```sql
-- Liste des employés dont le salaire dépasse le salaire moyen de leur département
SELECT e.nom, e.salaire, e.dept_id
FROM Employes e
WHERE e.salaire > (
    SELECT AVG(e2.salaire)
    FROM Employes e2
    WHERE e2.dept_id = e.dept_id
);
```

| FirstName | LastName | Position  | Salary |
| --------- | -------- | --------  | ------ |
| Michael   | Jones    | Analyst   | 3200   |
| James     | Johnson  | Designer  | 2200   |
| Mary      | Brown    | Developer | 2700   |
| Bill      | Gates    | Manager   | 10000  |