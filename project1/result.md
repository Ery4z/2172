# 1. Querying in Rel and SQLite

### 1. List the names of all mountains.

SQL:

```sql
SELECT DISTINCT name FROM MOUNTAIN;
```

Rel:

```rel

```

---

### 2. List the codes of the countries with more than 10 000 cases of Covid in March 2021.

SQL:

```sql

```

Rel:

```rel

```

---

### 3. List all provinces in Europe whose area is less than 200. The results must be a relation composed of 2-tuples with attributes {province, country}, corresponding to the full province name and the full country name, respectively.

SQL:

```sql

```

Rel:

```rel

```

---

### 4. List all countries in Europe whose total number of deaths in December 2022 exceeded 10 000.The result must be a relation composed of 3-tuples with attributes {country, population, total_deaths}, corresponding to the full country name, its population, and its number of deaths respectively.

SQL:

```sql

```

Rel:

```rel

```

---

### 5. For each country that has declared independence, give its full name and its independence date.

SQL:

```sql

```

Rel:

```rel

```

---

### 6. List country codes and full names of countries that share borders with both countries China and India. The result relation is composed of 2-tuples with attributes {code, name}.

SQL:

```sql

```

Rel:

```rel

```

---

### 7. List the capitals of countries who possess a mountain of height strictly superior to 4 000 in the Alps. The result must be a relation composed of 1-tuples with attribute {capital}.

SQL:

```sql

```

Rel:

```rel

```

---

### 8. List the codes and names of the countries without information about their languages.

SQL:

```sql

```

Rel:

```rel

```

---

### 9. List the mountain ranges (as defined by the attribute mountains in Table Mountain) in which there is a mountain that stands on at least two countries. For example, Alps is a mountain range and it hosts the mountain Mont Blanc on the border of France and Italy.

SQL:

```sql

```

Rel:

```rel

```

---

### 10. List all names shared by a province and a country’s capital whose country had more than 10 000 Covid deaths.

SQL:

```sql

```

Rel:

```rel

```

---

### 11. List the full names of the countries that border the United States, as well as the countries that border those bordering countries: i.e., the output should contain Mexico, but also Belize, which borders Mexico. The United States itself should not be part of the output.

SQL:

```sql

```

Rel:

```rel

```

---

### 12. Count the number of different countries whose Covid information was collected. The resul must be a relation composed of 1-tuples with attribute {cnt}.

SQL:

```sql

```

Rel:

```rel

```

---

### 13. List the name of all countries which have zero Covid cases until 12/2021 (included) together with their neighboring countries. The result must be a relation composed of tuples of 2 attributes {country1, country2} where the country1 is the zero-Covid country and country2 is its neighboring country. If a country has no neighboring country, it should not be included in the

    output.

SQL:

```sql

```

Rel:

```rel

```

---

### 14. List the most prominent language(s) for each country; i.e., list for each country those languages for which the percentage is maximal. You are not allowed to use SUMMARIZE. The result must be a relation composed of 2-tuples with attributes {country, name}, where country is the code of the country and name is the name of the language.

SQL:

```sql

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
