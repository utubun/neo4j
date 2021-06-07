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