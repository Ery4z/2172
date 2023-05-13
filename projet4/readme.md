1. List the full names of countries that border Greece and are not 100% located in Europe. Make
sure to include every country in your output only once.

```neo4j
MATCH (c:Country { name: 'Greece' })-[:Borders]->(borderingCountry:Country), (cn:Continent {name: 'Europe'})-[e:Encompasses]->(borderingCountry)
WHERE e.percentage<>"100.0"
RETURN DISTINCT borderingCountry.name;
```


2. List the pairs of countries that neighbor China, and that are also neighbors of each other.
Provide the full names of both countries; only list the pairs in which the first name of the pair
is strictly lower than the second name in the pair.

```neo4j
MATCH (china:Country { name: 'China' })-[:Borders]->(neighbor:Country)-[:Borders]->(mutualNeighbor:Country)
WHERE (china)-->(mutualNeighbor) AND neighbor.name < mutualNeighbor.name
RETURN neighbor.name AS FirstCountry, mutualNeighbor.name AS SecondCountry;
```
3. Provide the full names of all countries that are 100% located in Europe, but that border a
country not located, or not entirely located, in Europe.

```neo4j

MATCH (c:Country)-[:Borders]->(borderingCountry:Country), 
      (c)<-[e:Encompasses {percentage: "100.0"}]-(:Continent {name: 'Europe'}),
      (borderingCountry)<-[e2:Encompasses]-(:Continent {name: 'Europe'})
WHERE e2.percentage <> "100.0"
RETURN DISTINCT c.name;
```

4. Provide the full names of all countries in Asia that can be reached in at most two steps from
Turkey, excluding Turkey itself. Make sure to include every country in the output only once.

```neo4j

MATCH (turkey:Country { name: 'Turkey' })-[:Borders*1..2]-(countryInTwoSteps:Country)<-[:Encompasses]-(:Continent {name: 'Asia'})
WHERE NOT countryInTwoSteps.name = 'Turkey'
RETURN DISTINCT countryInTwoSteps.name;

```
5. List the number of neighboring countries for every country located in Europe (i.e., this is the
degree of every country in Europe, when only considering the Borders relation). Provide for
each country the full name, and its degree. Sort the output in decreasing order of degree.

```neo4j

MATCH (c:Country)<-[e:Encompasses {percentage: "100.0"}]-(:Continent {name: 'Europe'})
WITH c, size((c)-[:Borders]-()) AS degree
RETURN c.name AS Country, degree AS Degree
ORDER BY degree DESC;

or


MATCH (c:Country)<-[e:Encompasses {percentage: "100.0"}]-(:Continent {name: 'Europe'})
WITH c, SIZE([(c)-[:Borders]-(neighbor) | neighbor]) AS degree
RETURN c.name AS Country, degree AS Degree
ORDER BY degree DESC;



```


6. Find the country with the largest number of neighbors; if there are multiple such countries, it
is allowed to break ties arbitrarily; provide the full name and the number of neighbors.

```neo4j

MATCH (c:Country)-[:Borders]-()
WITH c, SIZE([(c)-[:Borders]-(neighbor) | neighbor]) AS degree
ORDER BY degree DESC
RETURN c.name AS Country, degree AS NumberOfNeighbors
LIMIT 1;

```
7. Determine the shortest path from Belgium to China; provide the complete path, that is, all
nodes and edges on this path, starting from Belgium and ending in China.

```neo4j

MATCH path=shortestPath((belgium:Country { name: 'Belgium' })-[:Borders*]-(china:Country { name: 'China' }))
RETURN path;

```

8. Find the country which has the longest shortest path to Belgium; provide the full name of this
country

```neo4j

MATCH (belgium:Country { name: 'Belgium' }), (c:Country)
WHERE belgium <> c
WITH belgium, c, shortestPath((belgium)-[:Borders*]-(c)) AS path
WHERE path IS NOT NULL
with belgium, c, length(path) AS Distance
RETURN c.name AS Country
ORDER BY Distance DESC
LIMIT 1;

```


9. Luxembourg is in a specific topographic situation: it has exactly three neighboring countries
(Belgium, Germany, and France), each of which are pairwise neighbors of each other as well.
As a result, Luxembourg is surrounded by exactly three countries. List the full names of all
countries that are in a similar situation as Luxembourg.

```neo4j

MATCH (c:Country)-[:Borders]->(n1:Country), (c)-[:Borders]->(n2:Country), (c)-[:Borders]->(n3:Country)
WHERE n1 <> n2 AND n1 <> n3 AND n2 <> n3 AND n1.name<n2.name AND n2.name<n3.name
AND (n1)-[:Borders]-(n2) AND (n1)-[:Borders]-(n3) AND (n2)-[:Borders]-(n3)
WITH c, count(*) AS lines
WHERE lines = 1
RETURN  c.name;

```


10. List all countries in Europe for which there is no path of length in between 1 and 3 to a country
(entirely or partially) located in Asia.


```neo4j

MATCH (eCountry:Country)<-[:Encompasses]-(europe:Continent {name:'Europe'})
WHERE NOT EXISTS {
  MATCH (eCountry)-[:Borders*1..3]->(aCountry:Country)<-[:Encompasses]-(asia:Continent {name:'Asia'})
}
RETURN eCountry.name AS Country;

```
