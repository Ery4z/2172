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
SELECT DISTINCT P.name as province, C.name as country
FROM Province AS P JOIN Country AS C ON P.country=C.code JOIN Encompasses as e ON C.code=e.country
WHERE P.area<200 AND e.continent="Europe"
```

Rel:

```rel
((((Province JOIN ((Country RENAME {code AS country, name AS cName}){country,cName})) WHERE area<200.) JOIN Encompasses) WHERE continent="Europe" ){name,cName} RENAME {name AS province, cName AS country}
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
((SUMMARIZE (((Mountain RENAME {name as mountain}) JOIN GeoMountain) {mountains,country}) BY {mountains} : {count_country := COUNT() }) WHERE count_country>1) {mountains}
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
((((SUMMARIZE (((Country RENAME {code as country}) JOIN CountryCovid) {capital, total_deaths}) BY {capital} : {sum_total_deaths:=SUM(total_deaths)}) WHERE sum_total_deaths>10000) {capital}) RENAME {capital as name}) INTERSECT (Province {name})
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
(((((Borders WHERE country2="USA") {country1}) RENAME {country1 as code}) UNION ((((((Borders WHERE country2="USA") {country1}) JOIN Borders) {country2}) MINUS ((Borders {country2}) WHERE country2="USA")) RENAME {country2 as code})) JOIN Country) {name}
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
SUMMARIZE (CountryCovid {country}) BY {} : {cnt := COUNT()}
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
((((((((Country {code}) MINUS (((CountryCovid WHERE year<2022 AND total_cases > 0) {country}) RENAME {country as code})) JOIN Country) {code,name}) RENAME {code as country1,name as country1_name}) JOIN Borders) JOIN (Country RENAME {code as country2,name as country2_name})) {country1_name,country2_name}) RENAME {country1_name as country1, country2_name as country2}
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
(Language {country,name}) MINUS (((Language JOIN (Language RENAME {name as name_bis, percentage as percentage_bis}) WHERE percentage>percentage_bis) {name_bis,country}) RENAME {name_bis as name})
```

---

### 15. In the given database the Borders relation is symmetric: if the country 1 (c1) borders country 2 (c2), country 2 also borders country 1. One may wish to check this property on a given database. Write a query that determines all tuples {c1, c2}, where c1 and c2 are the codes of the countries, for which the inverse direction is missing. (Note: on the given database, the output of this query should be empty; on a database in which the Border relation is not symmetric, however, it should return the violating tuples. Create a database yourself to check this.)

SQL:

```sql
SELECT country1,country2
FROM Borders
WHERE (country2,country1) NOT IN (
    SELECT country1,country2
    FROM Borders
)
```

Rel:

```rel
(Borders MINUS (Borders RENAME {country1 as country2,country2 as country1}) JOIN Borders) {country1,country2}
```

---

### 16. Some countries have incomplete information about their dialects and languages. List the name of countries for which the total percentage of reported languages does not reach 99% together with the proportion of the population for which this information is unknown. The result should have the form {name, unknown_lang_p}.

SQL:

```sql
SELECT r.name as name, (100-known_lang)*c.population/100 as unknown_lang_p
FROM
(
    SELECT name, SUM(known_lang) as known_lang
    FROM
        (SELECT c.name as name, l.percentage as known_lang
        FROM Country as c
        JOIN Language as l ON c.code=l.country
        UNION
        SELECT name, 0 as known_lang
        FROM Country)
    GROUP BY name
HAVING SUM(known_lang)<99
) as r JOIN Country as c on c.name=r.name

```

TODO: Find how to cast to float in rel

Rel:

```rel
EXTEND ((SUMMARIZE Language BY {country} :{known_lang := SUM(percentage)}) WHERE known_lang<99.) JOIN ((Country {code,population }) RENAME {code as country}) : {missing_lang := (100 - known_lang) * population / 100}
```

(((Country RENAME {code AS country}) JOIN CountryCovid JOIN Encompasses ) WHERE year=2022 AND ((((((((Country {code}) MINUS (((CountryCovid WHERE year<2022 AND total_cases > 0) {country}) RENAME {country as code})) JOIN Country) {code,name}) RENAME {code as country1,name as country1_name}) JOIN Borders) JOIN (Country RENAME {code as country2,name as country2_name})) {country1_name,country2_name}) RENAME {country1_name as country1, country2_name as country2}

# 2. Optimization in rel

### 1. For each continent, count the number of independent countries and count the number of Covid cases in the year of 2021. The result must be a relation composed of 3 attributes {continent, countries, covid_cases}.

```rel
(Encompasses {country,continent}) JOIN (SUMMARIZE ((CountryCovid {country,year,month,total_cases}) WHERE year=2021) BY {country} : {sum_cases := SUM(total_cases)})


```

GET 2021 CASES

```rel
EXTEND ((((CountryCovid {country,year,month,total_cases}) WHERE year=2021 AND month=12) RENAME {total_cases as total_cases_2021}) {country,total_cases_2021} ) JOIN ((((CountryCovid {country,year,month,total_cases}) WHERE year=2020 AND month=12) RENAME {total_cases as total_cases_2020}){country,total_cases_2020}) : {case_during_2021 := total_cases_2021 - total_cases_2020}

```

```
SUMMARIZE (Encompasses {country,continent}) JOIN ((EXTEND (((((CountryCovid {country,year,month,total_cases}) WHERE year=2021 AND month=12) RENAME {total_cases as total_cases_2021}) {country,total_cases_2021} ) JOIN ((((CountryCovid {country,year,month,total_cases}) WHERE year=2020 AND month=12) RENAME {total_cases as total_cases_2020}){country,total_cases_2020}) JOIN (Independence {country}) ) : {case_during_2021 := total_cases_2021 - total_cases_2020}){country,case_during_2021}) BY {continent} : {countries := COUNT(), covid_cases := SUM(case_during_2021)}
```

---

### 2. List all mountains that are on a border. The result must be a relation composed of a 3-tuple with attributes {mountain, country1, country2} where country1 and country2 are the names of the countries for which a border exists and mountain is the mountain located on the border.

```rel
(((((((SUMMARIZE (GeoMountain {mountain,country}) BY {mountain} : {countries:= COUNT()}) WHERE countries=2) {mountain}) JOIN (GeoMountain {mountain,country}) RENAME {country as country_code1}) JOIN ((Country{code,name}) RENAME {code as country_code1,name as country1})) {mountain,country1} ) JOIN (((GeoMountain {mountain,country}) RENAME {country as country_code2}) JOIN ((Country{code,name}) RENAME {code as country_code2,name as country2})) {mountain,country2}) WHERE country1<country2
```

---

### 3. List all Asian countries with a larger population than 200 000 000 inhabitants, whose increase in total Covid cases is higher in 2021 than in 2022; here the increase in total cases of a year is defined as total cases in December minus total cases in December of the preceding year. The result must be a relation with one attribute {country}, where country is the full name of the country.

```rel
 (((EXTEND (
((((Country {code,name,population}) RENAME {code as country}) WHERE population > 200000000) JOIN (Encompasses {country,continent}) WHERE continent="Asia") JOIN (

(((CountryCovid {country,year,month,total_cases}) WHERE month=12 AND year=2020) RENAME {total_cases as total_cases_2020}) {country,total_cases_2020}

)  JOIN (

(((CountryCovid {country,year,month,total_cases}) WHERE month=12 AND year=2021) RENAME {total_cases as total_cases_2021} ) {country,total_cases_2021}

)   JOIN (

(((CountryCovid {country,year,month,total_cases}) WHERE month=12 AND year=2022) RENAME {total_cases as total_cases_2022} ) {country,total_cases_2022}

)
) : {increase_2021 :=total_cases_2021-total_cases_2020, increase_2022:=total_cases_2022-total_cases_2021}) {name,increase_2021,increase_2022}) WHERE increase_2022<increase_2021) {name} RENAME {name as country}


```

---

### 4. For all countries that have a bordering country with a province with an area of less than 5000.0, list the languages; provide the country code and the language name. The result must be a relation composed of a 2-tuple with attributes {country, language} where country is the code of the country and language is the name of the language.

```rel
((((((((Province {name,country,area}) WHERE area < 5000. ) {country}) RENAME {country as country2})
JOIN
(Borders {country1,country2})) {country1})RENAME {country1 as country})
JOIN(
Language {country,name}
) ) RENAME {name as language}

```

---

### 5. Luxembourg is in a specific topographic situation: it has exactly three neighboring countries (Belgium, Germany, and France), each of which are pairwise neighbors of each other as well. As a result, Luxembourg is surrounded by exactly three countries. List the abbreviations of all countries that are in a similar situation as Luxembourg. The result must be a relation composed by a 1-tuple attribute {country} where country is the code of the country.

```rel
((((
(((SUMMARIZE (Borders {country1,country2}) BY {country1} : {neighbor_count := COUNT()})  WHERE neighbor_count=3) RENAME {country1 as country}) {country}
) JOIN (

(Borders {country1,country2}) RENAME {country1 as country,country2 as country_a}

) JOIN (

(Borders {country1,country2}) RENAME {country1 as country,country2 as country_b}

) JOIN (

(Borders {country1,country2}) RENAME {country1 as country,country2 as country_c}

)) WHERE country_a <> country_b AND country_b <> country_c AND country_a <> country_c AND country_a < country_b AND country_b < country_c

) JOIN (

(Borders {country1,country2}) RENAME {country1 as country_a,country2 as country_b}

)  JOIN (

(Borders {country1,country2}) RENAME {country1 as country_b,country2 as country_c}

)  JOIN (

(Borders {country1,country2}) RENAME {country1 as country_a,country2 as country_c}

)) {country}
```

---

### 6. List all countries for which the number of covid cases by december 2022 per person in the population is higher than the number of covid cases by december 2022 for all neighboring countries. The result must be a relation composed by a 1-tuple attribute country where country is the code of the country.

```rel
( ((Borders {country1,country2} JOIN
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country1, covid_ratio as covid_ratio1}
)  JOIN (
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country2, covid_ratio as covid_ratio2}
)
)
)
WHERE covid_ratio1>covid_ratio2 ) {country1}  RENAME {country1 as country})
MINUS
(( (Borders {country1,country2} JOIN
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country1, covid_ratio as covid_ratio1}
)  JOIN (
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country2, covid_ratio as covid_ratio2}
)
)
)
WHERE covid_ratio1>covid_ratio2 ) {country2} RENAME {country2 as country} )

```

---

### 7. One way of evaluating the severity of Covid in a country is to calculate the number of Covid cases per 100.000 citizens of a country, which can be derived from the population size and the number of Covid cases. List all countries for which the number of Covid cases per 100.000 citizens by December 2022 is higher than the number of Covid cases per 100.000 citizens by December 2022 for all its neighboring countries. The result must be a relation composed of 1-tuples with attributes {country}.

```rel
( ((Borders {country1,country2} JOIN
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := 100000.*CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country1, covid_ratio as covid_ratio1}
)  JOIN (
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := 100000.*CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country2, covid_ratio as covid_ratio2}
)
)
)
WHERE covid_ratio1>covid_ratio2 ) {country1}  RENAME {country1 as country})
MINUS
(( (Borders {country1,country2} JOIN
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := 100000.*CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country1, covid_ratio as covid_ratio1}
)  JOIN (
(
EXTEND ((Country {code} RENAME {code as country}) MINUS (CountryCovid {country,year,month} WHERE month=12 AND year=2022) {country}) : {covid_ratio:=0.}
UNION(
EXTEND (CountryCovid {country,year,month,total_cases} WHERE month=12 AND year=2022) JOIN (Country {code,population} RENAME {code as country}) : {covid_ratio := 100000.*CAST_AS_RATIONAL(total_cases)/CAST_AS_RATIONAL(population)} {country,covid_ratio}
) RENAME {country as country2, covid_ratio as covid_ratio2}
)
)
)
WHERE covid_ratio1>covid_ratio2 ) {country2} RENAME {country2 as country} )
```
