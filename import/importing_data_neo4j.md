Importing Data with neo4j 4.x
===

# Using LOAD CSV for import

## Objective

- Describe te steps for importing data with Cypher
- Prepare the graph and data for import
    - Inspect data;
    - Determine if data needs to be transformed;
    - Create the constraints in the graph;
- Import the data with LOAD CSV;
- Create indexes for newly-loaded data;

## Loading data with Cypher

In Cypher it is possible to:

- Load data from a URL (http(s) or file);
- Process data as a stream of records;
- Create or update the graph with the data being loaded;
- Use transactions during the load;
- Transform and convert values from the load stream;
- Load up to 10M nodes and relationships;

## Steps for loading data with Cypher

CSV import is commonly used to import data into a graph which requires to consider how your data maps into the data in your graph.

Scenario:

1. Determine how the CSV file will be structured;
2. Determine if normalized or denormalized data;
3. Ensure IDS to be used in the data are unique;
4. Ensure data in CSV files is 'clean';
5. Execute cypher code to inspect the data;
6. Determine if data needs to be transformed;
7. Ensure constraints are created in the graph;
8. Determine the size of the data to be loaded;
9. Execute Cypher code to load the data;
10. Add indexes to the graph;

## CSV file structure

- Determine whether the CSV file will have header information, describing the names of the fields;
- What the delimiter will be for the fields in each row;

Including headers reduces syncing issues. If the size of the files is extremely large, it is sometimes better to separate the headers from the data, especially if multiple files will be split to use the same set of headers.

`,` is the default delimiter for Cypher. Specify alternative delimiter with `FIELDTERMINATOR` symbol.

## Normalized vs Denormalized data

Normalized data represented by several tables with *unique* IDs for a choosen column, these tables are connected through these IDs (unique for one, and present in another). 

Denormalized data represented by table(s) with several columns and IDs which are not unique for each particular column.

## IDs must be unique

Non-unique IDs will cause a problem during import.

## Is the data clean

- Check for headers that do not match;
- Are quotes used correctly?;
- If an element has no value will and empty string be used?
- Are UTF-8 prefixes used?
- Do some fields have trailing spaces?
- Do the fields contain binary zeros?
- How lists are formed? Default is to use colon `:` as a separator;
- Is `,` the delimiter;
- Any obvious typos?

### Example: Inspect the data at a URL

```Cypher
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/v4.0-intro-neo4j/people.csv'
AS line
RETURN line LIMIT 10
```

The example shows how the data will be interpreted during the load.

### Example: Inspect the data stored locally

```Cypher
LOAD CSV WITH HEADERS
FROM 'file:///people.csv'
AS line
RETURN line LIMIT 10
```

### Loading and viewing subset of data

```Cypher
LOAD CSV WITH HEADERS
AS line
WHITH line WHERE line.birthYear > '1999'
RETURN line LIMIT 10
```

## Determine if data need transformation

Transform data with built-ins such as:

- `toInteger()`
- `toFloat()`

```Cypher
LOAD CSV WITH HEADERS
AS line
RETURN toFloat(line.avgVote), line.genres, toInteger(line.movieId),
       line.title, toInteger(line.releaseYear) LIMIT 10
```

## Transforming lists

For splitting strings into lists use following:

- `split()`
- `coalesce()`

```Cypher
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/v4.0-intro-neo4j/movies1.csv'
AS line
RETURN toFloat(line.avgVote0), split(coalesce(line.genres, ""), ":"),
       toInteger(line.movieId), line.title, toInteger(line.releaseYear)
       LIMIT 10
```

> If all fields have data, then `split()` alone will work. If, however, some fields may have no values and you want an empty list created for the property, then use `split()` together with `coalesce()`.

## Create constraints before loading the data

Need to be implemented to ensure uniquness of the nodes.

```Cypher
CREATE CONSTRAINT UniqueMovieIdConstraint ON (m:Movie) ASSERT m.id IS UNIQUE
CREATE CONSTRAINT UniquePersonIdConstraint ON (p:Person) ASSERT p.id IS UNIQUE
```

>If the load process uses `MERGE` instead of `CREATE` to create nodes, the load wil be *very slow* if constraints are not defined first, because `MERGE` needs to determine if the node already exists. The uniquness constraint is itself an index, which makes a lookup fast.

## Determine the size of the data to be loaded

```Cypher
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/v4.0-intro-neo4j/people.csv'
AS line
RETURN count(line)
```

## Loading a large CSV file

If the number of rows exceeds 100K you have two options:

- Use `:auto USING PERIODIC COMMIT LOAD CSV`;
- Use *APOC*;

`:auto USING PERIODIC COMMIT LOAD CSV`  enables the load, by default, to commit it's transactions every 1000 rows. However, Cypher statements which use *eager* operators will prevent you from using `:auto USING PERIODIC COMMIT`, which simply be ignored. This operators include:

- `collect()`;
- `count()`;
- `ORDER BY`;
- `DISTINCT`;

### Importing nodes

```Cypher
:auto USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM
    'https://data.neo4j.com/v4.0-intro-neo4j/movies1.csv AS row
MERGE (m:Movie {id:toInteger(row.movieId)})
    ON CREATE SET
    m.title = row.title,
    m.avgVote = toFloat(row.avgVote),
    m.releaseYear = toInteger(row.releaseYear),
    m.genres = split(row.genres, ':')
```

We read each line as a `row`. Then we use the row field names to assign values to a new *Movie* node. We use built-in functions to transform the string data in the row and assigne them to the properties of the *Movie* node. `MERGE` is the best choice because we have our uniqueness constraint defined for the `id` property of the *Movie* node.

### Importing relationships

Creating relationships between *Movie* and *Person* nodes.

```Cypher
LOAD CSV WITH HEADERS FROM
'https://data.neo4j.com/v4.0-intro-neo4j/directors.csv' 
AS row
MATCH (movie:Movie {id:toInteger(row.movieId)})
MATCH (person:Person {id: toInteger(row.personId)})
MERGE (person)-[:DIRECTED]->(movie)
ON CREATE SET person:Director
```

In last line we add label *Director* to the person node.

## Add indexes

The final step after all nodes and relationships have been created in the graph is to create additional indexes. These indexes are based upon th most important queries for the graph.

```Cypher
CREATE INDEX MovieTitleIndex ON (m:Movie) FOR (m.title)
CREATE INDEX PersonNameIndex ON (p:Person) FOR (p.name)
```

> These indeces will make lookup of a Movie by title as well as lookup of a Person by name fast. These indeces are not unique indexes.

# Using APOC for Import

