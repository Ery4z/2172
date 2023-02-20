# 1. Querying in Rel and SQLite

### 1. List the names of all mountains.

SQL:

```sql
SELECT DISTINCT name FROM MOUNTAIN;
```

Rel:

```rel
Mountain {name}
```

---

### 2. List the codes of the countries with more than 10 000 cases of Covid in March 2021.

SQL:

```sql
SELECT DISTINCT c.code
FROM Country as c
JOIN CountryCovid as cc on c.code=cc.country
WHERE cc.total_cases>10000 AND cc.year=2021 AND cc.month=3;
```

Rel:

```rel
((Country JOIN (CountryCovid RENAME {country AS code})) WHERE year=2021 AND month=3 AND total_cases>10000) {code}
```

---

### 3. List all provinces in Europe whose area is less than 200. The results must be a relation composed of 2-tuples with attributes {province, country}, corresponding to the full province name and the full country name, respectively.

SQL:

```sql
SELECT P.name as province, C.name as country
FROM Province AS P JOIN Country AS C
WHERE P.country=C.code AND P.area<200
```

Rel:

```rel
((Province JOIN ((Country RENAME {code AS country, name AS cName}){country,cName})) WHERE area<200.) {name,cName} RENAME {name AS province, cName AS country}
```

---

### 4. List all countries in Europe whose total number of deaths in December 2022 exceeded 10 000.The result must be a relation composed of 3-tuples with attributes {country, population, total_deaths}, corresponding to the full country name, its population, and its number of deaths respectively.

SQL:

```sql
SELECT DISTINCT c.name as country, c.population as population, cc.total_deaths as total_deaths
FROM Country as c
JOIN CountryCovid as cc on c.code=cc.country
JOIN Encompasses as e on e.country=c.code
WHERE cc.total_deaths>10000 AND cc.year=2022 AND cc.month=12 AND e.continent="Europe";
```

Rel:

```rel
(((Country RENAME {code AS country}) JOIN CountryCovid JOIN Encompasses ) WHERE year=2022 AND month=12 AND total_deaths>10000 AND continent="Europe") {name,population,total_deaths} RENAME {name as country}
```

---

### 5. For each country that has declared independence, give its full name and its independence date.

TODO: Check if this is correct, maybe not aggregate

SQL:

```sql
SELECT DISTINCT c.name, MAX(i.independence) as independence
FROM Country as c RIGHT JOIN Independence as i
GROUP BY c.name;
```

Rel:

```rel
 ((Country RENAME {code AS country}) JOIN (SUMMARIZE (Independence) BY {country} : {independence := MAX(independence)})) {name,independence}
```

---

### 6. List country codes and full names of countries that share borders with both countries China and India. The result relation is composed of 2-tuples with attributes {code, name}.

SQL:

```sql
SELECT DISTINCT country1 as code,name
FROM (
    SELECT c.code, c.name, b.country1, b.country2
    FROM Country as c
    JOIN Borders as b ON c.code=b.country1
    WHERE b.country2="TJ" OR b.country2="IND"
) as sub
WHERE country2="IND" AND country1
IN (SELECT country1 FROM (
    SELECT c.code, c.name, b.country1, b.country2
    FROM Country as c
    JOIN Borders as b ON c.code=b.country1
    WHERE b.country2="TJ" OR b.country2="IND"
) as sub2 WHERE country2="TJ")
```

Rel:

```rel
(((Country RENAME {code AS country1}) JOIN Borders) WHERE country2="TJ") {country1,name} INTERSECT (((Country RENAME {code AS country1}) JOIN Borders) WHERE country2="IND") {country1,name}
```

---

### 7. List the capitals of countries who possess a mountain of height strictly superior to 4 000 in the Alps. The result must be a relation composed of 1-tuples with attribute {capital}.

SQL:

```sql
SELECT DISTINCT c.capital as capital
FROM Country as c
JOIN Province as p ON p.country=c.code
JOIN GeoMountain as g ON p.name=g.province
JOIN Mountain as m ON g.mountain=m.name
WHERE m.mountains="Alps" AND m.height>4000
```

Rel:

```rel
(((Mountain RENAME {name as mountain}) JOIN GeoMountain JOIN ((Province RENAME {name as province, country as code}) {province,code}) JOIN Country) WHERE height>4000. AND mountains="Alps") {capital}
```

---

### 8. List the codes and names of the countries without information about their languages.

SQL:

```sql
SELECT DISTINCT c.code, c.name
FROM Country as c
LEFT OUTER JOIN Language as l ON c.code=l.country
WHERE l.name is NULL
```

Rel:

```rel
(Country {code,name}) MINUS ((Language RENAME {country as code}){code} JOIN (Country {code,name}))
```

---

### 9. List the mountain ranges (as defined by the attribute mountains in Table Mountain) in which there is a mountain that stands on at least two countries. For example, Alps is a mountain range and it hosts the mountain Mont Blanc on the border of France and Italy.

SQL:

```sql
SELECT mountains
FROM (
SELECT DISTINCT m.mountains as mountains, g.country as country
FROM Mountain as m
JOIN GeoMountain as g ON m.name=g.mountain
)
GROUP BY mountains
HAVING COUNT(country)>1
```

Rel:

```rel

```

---

### 10. List all names shared by a province and a country’s capital whose country had more than 10 000 Covid deaths.

SQL:

```sql
SELECT capital
FROM(
SELECT DISTINCT c.capital as capital
FROM Country as c
JOIN CountryCovid as cc on c.code=cc.country
GROUP BY c.capital
HAVING SUM(cc.total_deaths)>10000
) as sub
WHERE capital IN (
SELECT name
FROM Province
)
```

Rel:

```rel

```

---

### 11. List the full names of the countries that border the United States, as well as the countries that border those bordering countries: i.e., the output should contain Mexico, but also Belize, which borders Mexico. The United States itself should not be part of the output.

SQL:

```sql
SELECT DISTINCT name
FROM (
    SELECT c.code, c.name, b.country1, b.country2
    FROM Country as c
    JOIN Borders as b ON c.code=b.country1
    WHERE b.country2="USA"
) as sub
UNION
SELECT DISTINCT name
FROM (
    SELECT b.country2,c.name
    FROM (
        SELECT c.code as usaBorder
        FROM Country as c
        JOIN Borders as b ON c.code=b.country1
        WHERE b.country2="USA"
        )
    INNER JOIN Borders as b ON usaBorder=b.country1
    INNER JOIN Country as c ON b.country2=c.code
    WHERE c.code<>'USA'
)
```

Rel:

```rel

```

---

### 12. Count the number of different countries whose Covid information was collected. The result must be a relation composed of 1-tuples with attribute {cnt}.

SQL:

```sql
SELECT COUNT(country) as cnt
FROM(
SELECT DISTINCT country
FROM CountryCovid
)
```

Rel:

```rel

```

---

### 13. List the name of all countries which have zero Covid cases until 12/2021 (included) together with their neighboring countries. The result must be a relation composed of tuples of 2 attributes {country1, country2} where the country1 is the zero-Covid country and country2 is its neighboring country. If a country has no neighboring country, it should not be included in the output.

SQL:

```sql
SELECT DISTINCT name as country1,c.name as country2
FROM (
    SELECT code, name
    FROM Country
    WHERE code NOT IN
    (SELECT DISTINCT country
    FROM CountryCovid
    WHERE total_cases>0 AND year<2022)
) as sub
LEFT JOIN Borders as b ON b.country1=sub.code
LEFT JOIN Country as c ON b.country2=c.code
WHERE c.name IS NOT NULL
```

Rel:

```rel

```

---

### 14. List the most prominent language(s) for each country; i.e., list for each country those languages for which the percentage is maximal. You are not allowed to use SUMMARIZE. The result must be a relation composed of 2-tuples with attributes {country, name}, where country is the code of the country and name is the name of the language.

SQL:

```sql
SELECT DISTINCT a.country  as country, a.name as name
FROM Language a
LEFT OUTER JOIN Language b
    ON a.country = b.country AND a.percentage < b.percentage
WHERE b.country IS NULL;
```

Rel:

```rel

```

---

### 15. In the given database the Borders relation is symmetric: if the country 1 (c1) borders country 2 (c2), country 2 also borders country 1. One may wish to check this property on a given database. Write a query that determines all tuples {c1, c2}, where c1 and c2 are the codes of the countries, for which the inverse direction is missing. (Note: on the given database, the output of this query should be empty; on a database in which the Border relation is not symmetric, however, it should return the violating tuples. Create a database yourself to check this.)

SQL:

```sql

```

Rel:

```rel

```

---

### 16. Some countries have incomplete information about their dialects and languages. List the name of countries for which the total percentage of reported languages does not reach 99% together with the proportion of the population for which this information is unknown. The result should have the form {name, unknown_lang_p}.

SQL:

```sql

```

Rel:

```rel

```
