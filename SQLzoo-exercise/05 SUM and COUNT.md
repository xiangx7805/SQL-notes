# SUM and COUNT :musical_note:
## Syntax intro :zzz:
### *  Aggregates
The functions `SUM`, `COUNT`, `MAX` and `AVG` are "aggregates", each may be applied to a numeric attribute resulting in a single row being returned by the query. (These functions are even more useful when used with the `GROUP BY` clause.)

### *  `Distinct`
By default the result of a `SELECT` may contain duplicate rows. We can remove these duplicates using the `DISTINCT` key word.

### *  `ORDER BY`
permits us to see the result of a `SELECT` in any particular order. We may indicate `ASC` or `DESC` for ascending (smallest first, largest last) or descending order.

### *  `GROUP BY` and `HAVING`

By including a `GROUP BY` clause functions such as `SUM` and `COUNT` are applied to groups of items sharing values. When you specify `GROUP BY continent` the result is that you get only one row for each different value of `continent`. All the other columns must be "aggregated" by one of `SUM`, `COUNT` ...

The `HAVING` clause allows use to filter the groups which are displayed. The `WHERE` clause filters rows before the aggregation, the `HAVING` clause filters after the aggregation.

If a `ORDER BY` clause is included we can refer to columns by their position.

```SQL
SELECT column_name(s)
FROM   table_name
WHERE  condition
GROUP BY column_name(s)
HAVING condition
ORDER BY column_name(s);
```

## World Country Profile :notes:

**table `world`**

name | continent |	area |	population |	gdp
------ |	------ |	------ |	------ |	------
Afghanistan |	Asia |	652230 |	25500100 |	20343000000
Albania |	Europe |	28748 |	2831741 |	12960000000
Algeria |	Africa |	2381741 |	37100000 |	188681000000
Andorra |	Europe |	468 |	78115 |	3712000000
Angola |	Africa |	1246700 |	20609294 |	100990000000
... |	... |	... |	... |	...

### 1. Total world population
**Show the total population of the world.**

```sql
SELECT SUM(population)
FROM   world
```

### 2. List of continents
**List all the continents - just once each.**  

```sql
select distinct continent
from   world
```

### 3. GDP of Africa
**Give the total GDP of Africa**  

```sql
select sum(gdp)
from   world
where  continent = 'Africa'
```

### 4. Count the big countries
**How many countries have an area of at least 1000000**

```sql
select count(name)
from   world
where  area >= 1000000
```

### 5. Baltic states population
**What is the total population of ('Estonia', 'Latvia', 'Lithuania')**

```sql
select sum(population)
from   world
where  name in ('Estonia', 'Latvia', 'Lithuania')
```

### 6. Counting the countries of each continent
**For each continent show the continent and number of countries.**
```sql
select continent , count(name)
from   world
group by continent
```

### 7. Counting big countries in each continent
**For each continent show the continent and number of countries with populations of at least 10 million.**

```sql
select continent , count(*)
from   world
where  population > 10000000
group by continent
```

### 8. Counting big continents
**List the continents that have a total population of at least 100 million.**

```sql
select continent
from world
group by continent
having sum(population) > 100000000
```
## Noble Prize Profile:ocean:

**table `nobel`**

yr |	subject	 | winner
-------- |	-------- |	-----------
2015 |		Chemistry	 |	Aziz Sancar
2015 |		Chemistry |		Paul L. Modrich
2015 |		Chemistry |		Tomas Lindahl
2015 |		Economics |		Angus Deaton
2015 |		Literature |		Svetlana Alexievich
2015 |		Medicine |		Tu Youyou
... |	... |	...

### 1.
**Show the total number of prizes awarded.**

```sql
SELECT COUNT(winner) FROM nobel
```

### 2.
**List each subject - just once**

```sql
select distinct subject
from   nobel
```

### 3.
**Show the total number of prizes awarded for Physics.**

```sql
select count(winner)
from   nobel
where  subject = 'Physics'
```

### 4.
**For each subject show the subject and the number of prizes.**

```sql
select subject , count(winner)
from   nobel
group by subject
```

### 5.
**For each subject show the first year that the prize was awarded.**

```sql
select subject , min(yr)
from   nobel
group by subject
```

### 6.
**For each subject show the number of prizes awarded in the year 2000.**  
```sql
select subject ,count(winner)
from   nobel
where  yr =2000
group by subject
```

### 7.
**Show the number of different winners for each subject.**
```sql
select subject , count(distinct winner)
from   nobel
group by subject
```

### 8.
**For each subject show how many years have had prizes awarded.**

```sql
select subject , count(distinct yr)
from   nobel
group by subject
```

### 9.
**Show the years in which three prizes were given for Physics.**
```sql
select yr
from   nobel
where  subject = 'Physics'
group by yr
having count(winner) =3
```

### 10.
**Show winners who have won more than once.**
```sql
select winner
from   nobel
group by winner
having count(*) >1
```

### 11.
**Show winners who have won more than one subject.**
```sql
select winner
from   nobel
group by winner
having count(distinct subject) >1
```

### 12.
**Show the year and subject where 3 prizes were given. Show only years 2000 onwards.**
```sql
select yr,subject
from   nobel
where  yr >= 2000
group by yr,subject
having count(*) =3
```
