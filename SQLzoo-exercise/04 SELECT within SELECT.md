# SELECT WITHIN SELECT Tutorial :snowflake:
**table `world`**

name | continent |	area |	population |	gdp
------ |	------ |	------ |	------ |	------
Afghanistan |	Asia |	652230 |	25500100 |	20343000000
Albania |	Europe |	28748 |	2831741 |	12960000000
Algeria |	Africa |	2381741 |	37100000 |	188681000000
Andorra |	Europe |	468 |	78115 |	3712000000
Angola |	Africa |	1246700 |	20609294 |	100990000000
... |	... |	... |	... |	...


## SELECT .. SELECT :cloud:
### 1. Subquery with FROM
**You may use a `SELECT` statement in the `FROM` line.  
In this case the derived table `X` has columns `name` and `gdp_per_capita`. The calculated values in the inner `SELECT` can be used in the outer `SELECT`.**

Notice that
*  the inner table is given an alias X
*  the first column in the inner query keeps its name
*  the second column in the inner query has an alias

```sql
SELECT name, ROUND(gdp_per_capita)
  FROM  (SELECT name,
                gdp/population AS gdp_per_capita
         FROM bbc) AS X
 WHERE gdp_per_capita>20000
```

### 2. Subquery with IN
**Find the countries in the same region as Bhutan**

You may use a `SELECT` statement in the `WHERE` line - this returns a list of regions.
```sql
SELECT name
FROM   bbc
WHERE  region IN  (SELECT region FROM bbc
                   WHERE name='Bhutan')
```

### 3. Correlated Subquery
If a value from the outer query appears in the inner query this is "correlated subquery".

**Show the countries where the population is greater than 5 times the average for its region**

```sql
SELECT name
FROM   bbc b1
WHERE  population > 5*( SELECT AVG(population)
                        FROM bbc
                        WHERE region=b1.region)
```

## Exercise :snowman:

### 1. Bigger than Russia
**List each country name where the population is larger than that of 'Russia'.**

```SQL
SELECT name
FROM   world
WHERE  Population > (SELECT population
                    FROM world
                    WHERE name = 'Russia')
AND    name != 'Russia'
```

### 2. Richer than UK
**Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.**

```SQL
select name
from   world
where  continent = 'Europe'
and    gdp/population > (select gdp/population
                         from world
                         where name = 'United Kingdom')
```
### 3. Neighbours of Argentina and Australia
**List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.**

```SQL
select name, continent
from   world
where  continent IN (select continent
                     from   world
                     where  name='Argentina'  
                     or     name='Australia')
order by name
```

### 4. Between Canada and Poland
**Which country has a population that is more than Canada but less than Poland? Show the name and the population.**

```SQL
select name , population
from   world
where  population < (select population from world where name ='Poland' )
and    population > (select population from world where name ='Canada' )
```

### 5. Percentages of Germany
Germany (population 80 million) has the largest population of the countries in Europe. Austria (population 8.5 million) has 11% of the population of Germany.  
**Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.**

```SQL
SELECT name, CONCAT(ROUND(100 * population / (SELECT population
                                              FROM world
                                              WHERE name = 'Germany'),0),'%')
FROM   world
WHERE  continent = 'Europe'
```
> :rose:**CONCAT**
`CONCAT` allows you to stick two or more strings together.  
This operation is **concatenation**.  
Syntax : `CONCAT(s1, s2 ...) `  


### 6. Bigger than every country in Europe
**Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)**

```SQL
select name
from world
where gdp > ALL(select gdp
                from   world
                where  continent =  'Europe'
                and    gdp is not null)
```

### 7. Largest in each continent
**Find the largest country (by area) in each continent, show the continent, the name and the area:**  
>We can refer to values in the outer SELECT within the inner SELECT. We can name the tables so that we can tell the difference between the inner and outer versions.

```SQL
select  a.continent, a.name , a.area
from    world a
where   a.area >= all(select area
                      from world b
                      where a.continent = b.continent)
```

### 8. First country of each continent (alphabetically)
**List each continent and the name of the country that comes first alphabetically.**  

```SQL
select   continent, min(name) as name
from     world
group by continent
order by name
```
OR   
```SQL
select continent, name
from   world a
where  name <= all( select name
                    from world b  
                    where a.continent = b.continent)

```

## Difficult Questions That Utilize Techniques Not Covered In Prior Sections
### 9.
**Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.**  

```SQL
select name, continent , population
from world
where continent in ( select continent
                     from world
                     group by continent
                     having count(name) = sum( if(population <= 25000000 ,1,0) ))
```
OR

```SQL
select name, continent , population
from world
where continent in ( select continent
                     from world
                     group by continent
                     having count(name) = sum( case when population <= 25000000 then 1 else 0 end ))
```
> :mushroom:**`CASE WHEN`**
Syntax:
```sql
  CASE
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  WHEN conditionN THEN resultN
  ELSE result
  END;
  # DO NOT ignore `END`
```

> :maple_leaf:**`GROUP BY`**
The general syntax with `ORDER BY` is:
```SQL
  SELECT column-names
  FROM table-name
  WHERE condition
  GROUP BY column-names
  HAVING condition
  ORDER BY column-names
```

### 10.  
**Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.**  

```SQL
SELECT name, continent
FROM   world x
WHERE  population > all ( SELECT 3*population
                          # '3*' 不能放在`all`之前 -- syntax error
                          FROM   world y
                          WHERE  y.continent=x.continent
                          AND    population IS NOT NULL
                          AND    x.name != y.name)
```
