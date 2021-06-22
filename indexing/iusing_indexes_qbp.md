Using Indexes and Query Best Practices in Neo4j 4.x
===

# Objectives

- Defining constraints in the database;
- Creating and using indexes;
- Using query best practices;

# Defining constraints for your data

> In most graphs, you will want to provide *uniqueness* for some properties of the nodes in the graph. In addition, you may want to enforce the *creation of mandatory properties* for selected nodes. These are called *constraints* in a Neo4j graph data model and are typically defined during the graph data modeling phase of your application developement.

## Objectives

- Create a uniquness constraint for a node property in the graph;
- Create an existence constraint for a node property in the graph;
- Create a uniquness constraint for a set of node properties in the graph;
- Manage constraints in the graph;

## Uniqueness and existence

There is a possibility to create *duplicated* nodes, that is the reason why `MERGE` is preffered over `CREATE` because `MERGE` does use *locks*. In addition, if you want to *ensure* that node or relationship has particular *property* you can use constraints to do that. Another scenario: you want to ensure that for the nodes of the same type set of properties have *unique* values. 

- Add a *uniqueness constraint* that ensures that a value for a property is unique for all nodes of that type.
- Add an *existence constraint* that ensures that when a node or relationship is created or modified, it must have certain properties set.
- Add a *node key* that ensures that a set of values for properties of a node of a given type is unique.

>Existence constraints and node keys are only available in Enterprise Edition of Neo4j.

### Uniqueness

>You add a uniqueness constraint to the graph by creating a constraint that asserts that a particular node property is unique in the graph for a particular type of node.

#### Example

```cypher
CREATE CONSTRAINT UniqueMovieTitleConstraint ON (m:Movie) ASSERT m.title IS UNIQUE
```

>Although the name of the constraint, UniqueMovieTitleConstraint is optional, Neo4j recommends that you name it. Otherwise, it will be given an auto-generated name. This Cypher statement will fail if the graph already has multiple Movie nodes with the same value for the title property. Note that you can create a uniqueness constraint, even if some Movie nodes do not have a title property.

After constraint creation, this will fail:

```cypher
CREATE (:Movie {title: 'The Matrix'})
```

The same will happen if you are trying to modify the value of a node's property violating uniqueness constrain:

```cypher
MATCH (m:Movie) {title: 'The Matrix})
SET m.title = 'Top Gun'
```

### Ensuring that properties exist

>You add an existence constraint to the graph by creating a constraint that asserts that a particular type of node or relationship property must exist in the graph when a node or relationship of that type is created or updated.

#### Example

This will fail because for some nodes constrainted property does not exist:

```cypher
CREATE CONSTRAINT ExistsMovieTagline ON (m:Movie) ASSERT exists(m.tagline)
```

#### Example

If we know that for all instances property does exist, we can add constraint directly:

```cypher
CREATE CONSTRAINT ExistsREVIEWEDRating
    ON ()-[rel:REVIEWED]-() ASSERT exists(rel.rating)
```

>Notice that when you create the constraint on a relationship, you need not specify the direction of the relationship.

Attempt to add relationship *without required property*, or attempt to *remove property with constraint* will fail as well:

```cypher
MATCH (p:Person)-[rel:REVIEWED]-(m:Movie)
WHERE p.name = 'Jessica Thompson'
REMOVE rel.rating
```

Attempting to remove property from relationship or node where the existence constraint has been created in the graph will fail:

```Cypher
MATCH (p:Person)-[rel:REVIEWED]-(m:Movie)
WHERE p.name = 'Jessica Thompson'
REMOVE rel.rating
```

## Retrieving constraints defined for the graph

```cypher
CALL db.constraints()
```

## Dropping constraints

```cypher
DROP CONSTRAINT ExistsREVIEWEDRating
```

## Creating multi-property uniqueness/existence constraint: node key

>Suppose that in our Movie graph, we will not allow a Person node to be created where both the name and born properties are the same. We can create a constraint that will be a node key to ensure that this uniqueness for the set of properties is asserted.

```cypher
CREATE CONSTRAINT UniqueNameBornConstraint 
    ON (p:Person) ASSERT (p.name, p.born) IS NODE KEY
```

If `born` property is not defined on *Person* nodes in the graph, this attempt will fail. It can be fixed by assigning the property to the nodes:

```Cypher
MATCH (p:Person)
WHERE NOT exists(p.born)
SET p.born = 0
```
After constraints are set, any attempt to modify an existing *Person* nod with *name* or *born* values that violate the uniquness constraint as a node key will fail. The attempt to remove the property which used as a node key will fail as well.

