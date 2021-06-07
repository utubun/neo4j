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