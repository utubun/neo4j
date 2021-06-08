Querying with Cypher in Neo4j 4.x
===

# Introduction to Cypher

# Using WHERE to Filter Queries

# Working with Patterns in Queries

## Objectives

- Specify multiple `MATCH` patterns;
- Specify multiple `MATCH` clauses;
- Specify varying length paths;
- Return a subgraph;
- Specify `OPTIONAL` in a query.

## Traversl in MATCH clause

> In computer science, graph traversal (also known as graph search) refers to the process of visiting (checking and/or updating) each vertex in a graph. Such traversals are classified by the order in which the vertices are visited. Tree traversal is a special case of graph traversal ([Wikipedia](https://en.wikipedia.org/wiki/Graph_traversal)).

Imagine you want to find all followers of the people reviewed the movie *The Replacements*. 

### Query

```cypher
MATCH (follower:Person)-[:FOLLOWS:]->(reviewer:Person)-[:REVIEWED]->(m:Movie)
WHERE m.title = 'The Replacements'
RETURN follower.name AS follower, reviewer.name AS reviewer
```

### Query mechanism

1. Find the *Movie* node with the *title* *The Replacements*;
2. Find all *Person* nodes with relationship *REVIEWED*;
3. Find all incoming *Person* nodes for each *Person* node from step 2.

## Specifying multiple patterns in `MATCH`

Find movies released in the year 2000.

### Query

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie),
      (m)<-[:DIRECTED]-(d:Person)
WHERE m.released = 2000
RETURN a.name, m.title, d.name
```

>It returns all Person nodes for people who acted in these three movies and using that same movie node, m it retrieves the Person node who is the director for that movie, m.

## Specifying a single pattern

A better way to do the same query as above, is to use single pattern.

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
WHERE m.released = 2000
RETURN a.name, m.title, d.name
```
But in some cases, where two or more patterns are needed. This is the case when the query is too complex and cannot be satisfied with single pattern. E.g. when you are looking for the specific node in the graph, and want to connect it to a different node.

### Example: Using two patterns in a `MATCH`

Find all actors who acted in the same movies as *Keanu Reeves* but not when *Hugo Weaving* acted in the same movie.

```cypher
MATCH (keanu:Person)-[:ACTED_IN]->(movie:Movie)<-[:ACTED_IN]-(n:Person),
      (hugo:Person)
WHERE keanu.name = 'Keanu Reeves' AND
      hugo.name  = 'Hugo Weaving'
AND NOT (hugo)-[:ACTED_IN]->(movie)
RETURN n.name
```

### Example: Using two patterns in a `MATCH`

Retrieve the movies that *Meg Ryan* acted in and their respective directors, and all other actors played in the same movies.

```cypher
MATCH (meg:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person),
      (other:Person)-[:ACTED_IN]->(m)
WHERE meg.name = 'Meg Ryan'
RETURN m.title AS movie, d.name AS director, other.name AS `co-actor`
```

## Specifying varying path

Find the person connected to another person through the following relationship (following person who follows the person) which is exactly two hoops away in the graph.

```cypher
MATCH (follower:Person)-[:FOLLOWS*2]->(p:Person)
WHERE follower.name = 'Paul Blythe'
RETURN p.name
```
If, instead of `[:FOLLOWS*2]`  we specify `[:FOLLOWS*]` we retreive *all* nodes that are in the `:FOLLOWS` path from *Paul Blythe*.

### Simplified syntax

```cypher
(nodeA)-[:RELTYPE*]->(nodeB)
# Retrieve all paths from nodeA binded by RELTYPE to nodeB
(nodeA)-[:RELTYPE*]-(nodeB)
# Retrieve all path between nodeA and nodeB irrelevant of direction
```

Alternatives:

```cypher
(nodeA)-[:RELA*3]->(nodeB)
# Retrieve all paths from NodeA to nodeB of length 3 and relationship RELA
(nodeA)-[:RELA*1..3]->(nodeB)
# Retrieve all paths from NodeA to NodeB of length from 1 to 3 with relationship RELA
```

## Shortest path

Built-in function `shortestPath()`. Improves the performance of the query.

Find the shortest path between *The Matrix* and *A Few Good Men*. Notice how we specify `*` as a relationship of the query, which means *any relationship*:

```cypher
MATCH (p = shortestPath(m1:Movie)-[*]-(m2:Movie))
WHERE m1.title = 'The Matrix' AND m2.title = 'A Few Good Men'
RETURN p
```

>When you use shortestPath(), you can specify a upper limits for the shortest path. In addition, aim to provide the patterns for the from and to nodes that execute efficiently. For example, use labels and indexes.

## Returning a subgraph

>In using shortestPath(), the return type is a path. A subgraph is essentially as set of paths derived from your MATCH clause.

```cypher
MATCH paths = (m:Movie)--(p:Person)
WHERE m.title = 'The Replacements'
RETURN paths
```
>The APOC library is very useful if you want to query the graph to obtain subgraphs.

## Optional match

`OPTIONAL MATCH` matches the query with the graph, and return `null` if match is not found. It's an equivalent of `OUTER JOIN` in SQL.

Find a subgraph of Movies graph with all people named James:

```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH 'James'
OPTIONAL MATCH (p)-[r:REVIEWED]->(m:Movie)
RETURN p.name, type(r), m.title
```

# Working with Cypher Data

## Objectives

- Aggregate data into lists;
- Work with lists;
- Count results returned;
- Work with maps;
- Work with dates;

## Aggregation in Cypher

>When you return results as values, Cypher automatically returns the values grouped by a common value.

```cypher
MATCH (p:Person)-[:REVIEWED]->(m:Movie)
RETURN  p.name, m.title
```
>Cypher orders values returned by default will depend on the number of nodes of each type in the graph. 

### Using `count()` to aggregate

`count()` performs count of nodes, relationships, paths, rows during query processing.

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
RETURN a.name, d.name, count(m)
```

>The query engine processed all nodes and relationships in the pattern so that it could perform a count of all movies for a particular actor/director pair in the graph. Then, it returned the results grouped by the name of the director.

>In Cypher, you need not specify a grouping key. As soon as an aggregation function is used, all non-aggregated result columns become grouping keys. The grouping is implicitly done, based upon the fields in the RETURN clause.

### Collecting results

Built-in function `collect()` aggregates a value into a list. The value can be a property value, a node, a relationship, or a path.

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Cruise'
RETURN collect(m.title) AS `movies for Tom Cruise`
```

### Collecting nodes

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie) 
WHERE p.name = 'Tom Cruise'
RETURN collect(m) AS `movies for Tom Cruise`
```

### Counting and collecting

- `count(*)` number of rows retrieved including those with the `null` values;
- `count()`  counts using *group by* based upon the aggregation;
- `count(x)` number of occurence of `n`;

```cypher
MATCH (actor:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person)
RETURN actor.name, director.name, count(m) AS collaborations, collect(m.title) AS movies
```

>Notice here that the director.name value is a grouping key for the results returned.

### Using `collect()` and `size()`

>There are more aggregating functions such as min() or max() that you can also use in your queries. These are described in the Aggregating Functions section of the Neo4j Cypher Manual.

`size()` function returns a number elements in a list. However `count()` is more efficient because it gets its values from the internal count store of the graph.

```cypher
MATCH (actor:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(director:Person)
RETURN actor.name, director.name, size(collect(m)) AS collaborations, collect(m.title) AS movies
```

## Working with cypher data

### Lists

Collect lists of movies casts along with the size of the cast

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN m.title, collect(a) as cast, size(collect(a)) AS castSize
```

Modified query returns actor names as a list, instead of list of actores represented by objects

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN m.title, collect(p.name) AS cast, size(collect(a)) AS castSize
```
Using brackets notation and zero-based indexing, we can access elements of the list

```cypher
MATCH (actor:Person)-[:ACTED_IN]->(m:Movie)
RETURN m.title, collect(actor.name)[0] AS `A cast member`, count(a)
```

## Working with maps

>A Cypher map is list of key/value pairs where each element of the list is of the format 'key': value. For example, a map of months and the number of days per month could be:
`{Jan: 31, Feb: 28, Mar: 31, Apr: 30 , May: 31, Jun: 30 , Jul: 31, Aug: 31, Sep: 30, Oct: 31, Nov: 30, Dec: 31}`. 

We can access elements of *map* by the *key* of the element:

```cypher
RETURN {Jan: 31, Feb: 28, Mar: 31, Apr: 30 , May: 31, Jun: 30 , Jul: 31, Aug: 31, Sep: 30, Oct: 31, Nov: 30, Dec: 31}['Feb'] AS DaysInFeb
```
In *neo4j* nodes of the graph are *maps*.

### Map projections

>Map projections are when you can use retrieved nodes to create or return some of the information in the nodes... Suppose we want to return the *Movie* node information, but without the *tagline* property?

```cypher
MATCH (m:Movie)
WHERE m.title CONTAINS 'Matrix'
RETURN m { .title, .released } AS movie
```

>You can learn more about map projections in the Cypher Reference Manual.

## Working with dates

```cypher
RETURN date(), datetime(), time(), timestamp()
```

>Since both date() and datetime() store their values as strings, you can use properties such as day, year, time to extract the values...

```cypher
RETURN date().day, date.year(), datetime().year, datetime().hour, datetime().minute
```

>Working with timestamp() is different as its value is a long integer that represents time. The value of datetime().epochmillis is the same as timestamp().

```cypher
RETURN datetime({epochmillis:timestamp()}).day,
       datetime({epochmillis:timestamp()}).year,
       datetime({epochmillis:timestamp()}).month
```

## Type and data conversions

- `toInteger()`;
- `toLower()`;
- `toUpper()`;
- `toString()`;