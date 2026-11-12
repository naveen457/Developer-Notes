# Neo4j & Graph Database Notes

### A practical guide to graph concepts, Neo4j's data model, Cypher, and query execution

> **Purpose:** Understand how graph databases represent connected information, why relationships are first-class data, and how to model, query, and create data in Neo4j. These notes combine the concepts and examples from the handwritten study pages with clearer explanations and runnable Cypher patterns.

---

## Table of contents

1. [Taxonomy vs ontology](#1-taxonomy-vs-ontology)
2. [What is a graph database?](#2-what-is-a-graph-database)
3. [Neo4j's property graph model](#3-neo4js-property-graph-model)
4. [Graph databases vs relational databases](#4-graph-databases-vs-relational-databases)
5. [Graph theory and knowledge graphs](#5-graph-theory-and-knowledge-graphs)
6. [Cypher: querying a graph](#6-cypher-querying-a-graph)
7. [Reading a graph: the Movie database](#7-reading-a-graph-the-movie-database)
8. [How Neo4j executes a Cypher query](#8-how-neo4j-executes-a-cypher-query)
9. [Pattern matching](#9-pattern-matching)
10. [Creating nodes and relationships](#10-creating-nodes-and-relationships)
11. [Indexes, constraints, and performance](#11-indexes-constraints-and-performance)
12. [Quick revision](#12-quick-revision)
13. [Course-by-course practical coverage](#20-course-by-course-practical-coverage)

---

# 1. Taxonomy vs ontology

Both taxonomy and ontology organize concepts, but they describe different levels of meaning.

## 1.1 Taxonomy

A **taxonomy** is a structured way of organizing concepts into categories and subcategories. It commonly follows a hierarchy, similar to a folder structure.

Example:

\`\`\`text
Food
├── Vegetables
│   ├── Brinjal
│   └── Potato
└── Fruits
    ├── Apple
    └── Banana
\`\`\`

- A parent category groups one or more child categories.
- It helps classify and navigate information.
- A taxonomy primarily tells us how concepts are grouped.
- By itself, it does not fully describe the meaning or semantics of relationships.

## 1.2 Ontology

An **ontology** describes concepts, their meanings, properties, and the relationships that can exist between them. It captures more semantics than a simple hierarchy.

Example:

\`\`\`text
(Naveen:Student) ──LEARNs──> (AI:Subject)
(AI:Subject) ──USES──> (Python:ProgrammingLanguage)
\`\`\`

This graph expresses facts about entities and how they are related.

## 1.3 Key differences

| Aspect | Taxonomy | Ontology |
|---|---|---|
| Main purpose | Classification | Representing meaning and knowledge |
| Structure | Usually hierarchical | Can be hierarchical and network-shaped |
| Relationships | Commonly parent-child | Can represent many semantic relationship types |
| Meaning | Groups concepts | Defines concepts, properties, and relationships |
| Example | Fruit → Apple | Person → WORKS_AT → Company |

**Remember:** A taxonomy answers *"What category does this belong to?"* An ontology can also answer *"What is it, what properties does it have, and how is it related to other things?"*

---

# 2. What is a graph database?

A **graph database** stores data as entities and the relationships connecting them. It is especially useful when the connections between records are as important as the records themselves.

Neo4j is a graph database. It uses a property graph model and the Cypher query language.

A graph can be represented as:

\`\`\`text
        relationship
(Node A) ─────────────> (Node B)
\`\`\`

Unlike a model in which connections must be reconstructed through multiple joins, a graph database represents relationships directly.

## 2.1 Why use a graph database?

Consider a social network:

- People are entities.
- A person can follow another person.
- A person can work at a company.
- A person can know another person.

The relationships are meaningful data, not merely implementation details.

Graph databases are useful for:

- Social networks and recommendation systems.
- Fraud detection and suspicious-connection analysis.
- Knowledge graphs and entity linking.
- Dependency and network analysis.
- Multi-hop questions such as finding friends-of-friends or connected organizations.

---

# 3. Neo4j's property graph model

Neo4j organizes graph data using four core concepts.

## 3.1 Nodes

A **node** represents an entity or object.

Examples: Person, Employee, Company, Movie, Product, or Location.

A node can have one or more labels and a set of properties.

\`\`\`cypher
CREATE (p:Person {
    name: 'Michael',
    dateOfBirth: '1990-09-22'
})
RETURN p;
\`\`\`

Here:

- \`p\` is a variable used within the query.
- \`Person\` is a label.
- \`name\` and \`dateOfBirth\` are properties.

## 3.2 Relationships

A **relationship** is a directed connection between two nodes. It has a relationship type and can also contain properties.

\`\`\`text
(Person) ──WORKS_AT──> (Company)
\`\`\`

Example:

\`\`\`cypher
MATCH (p:Person {name: 'Michael'})
MATCH (c:Company {name: 'Neo4j'})
CREATE (p)-[:WORKS_AT {role: 'Engineer'}]->(c);
\`\`\`

The direction is part of the stored relationship pattern. A query can match a relationship in either direction when direction is not specified.

## 3.3 Labels

A **label** classifies a node. It helps identify groups of nodes that represent the same kind of entity.

Examples:

- \`Person\`
- \`Employee\`
- \`Company\`
- \`Movie\`

A node may have multiple labels:

\`\`\`cypher
CREATE (p:Person:Employee {name: 'Michael'})
RETURN p;
\`\`\`

Labels apply to nodes. Relationship types identify the kind of connection between nodes.

## 3.4 Properties

**Properties** are key-value attributes stored on nodes or relationships.

Examples:

\`\`\`text
Person node:
{
  name: "Michael",
  age: 30
}

WORKS_AT relationship:
{
  role: "Engineer",
  since: 2024
}
\`\`\`

Properties provide details about an entity or about the connection itself.

## 3.5 Putting the model together

\`\`\`text
(:Person {name: "Michael"})
       |
       | [:WORKS_AT {role: "Engineer"}]
       v
(:Company {name: "Neo4j"})
       |
       | [:FOUNDED_IN]
       v
(:Location {name: "Sweden"})
\`\`\`

- Nodes represent entities.
- Labels classify nodes.
- Relationships connect nodes.
- Properties describe nodes and relationships.

---

# 4. Graph databases vs relational databases

A relational database stores data in tables. Relationships are commonly represented through primary keys and foreign keys, then traversed using joins.

A graph database stores relationships as explicit connections between nodes.

| Aspect | Relational database | Graph database |
|---|---|---|
| Main structure | Tables, rows, columns | Nodes and relationships |
| Connections | Foreign keys and joins | Explicit relationships |
| Query approach | SQL | Cypher in Neo4j |
| Relationship traversal | Often requires joins | Follows graph connections |
| Typical strength | Structured tabular data and transactions | Highly connected data and traversals |

## 4.1 Why graph traversal can be efficient

In Neo4j's native graph storage, relationships connect records directly. This is often described as **index-free adjacency**: once a starting node is found, traversal can follow its stored relationships rather than repeatedly resolving foreign-key joins.

This does not mean every graph query is automatically constant time. The cost depends on:

- How many starting nodes are selected.
- How many relationships are traversed.
- The degree of each node.
- The depth and shape of the traversal.
- Whether indexes help locate starting nodes.
- The amount of data returned.

A query that expands through a huge part of a graph can still be expensive.

## 4.2 When relational databases can be a better fit

Relational databases are often a natural choice for strongly tabular data, well-defined schemas, and workloads dominated by structured transactions and aggregations. Graph databases are especially attractive when the application frequently asks questions about connections and paths.

Choose based on the access patterns and workload, not simply because one database model is newer.

---

# 5. Graph theory and knowledge graphs

## 5.1 Graph theory

Graph theory is the mathematical study of graphs: collections of vertices (nodes) and edges (connections).

A classic origin story is the **Seven Bridges of Königsberg** problem from the 18th century. The problem asked whether a route could cross each of the city's seven bridges exactly once. Euler's analysis helped establish foundational ideas in graph theory.

A graph can represent:

- Entities as vertices.
- Connections as edges.
- Directed or undirected relationships.
- Paths between entities.

## 5.2 Self-referential and hierarchical data

Graphs naturally represent data that refers to other data of the same kind.

For example, employees may report to other employees:

\`\`\`text
(Manager) ──MANAGES──> (Employee)
                         |
                         └──MANAGES──> (AnotherEmployee)
\`\`\`

The depth of a hierarchy does not need to be fixed in the data model. A query can traverse as far as required, subject to query limits and practical performance constraints.

## 5.3 Knowledge graphs

A **knowledge graph** represents entities and the meaningful relationships between them, often combining facts from multiple sources.

For an AI agent, a knowledge graph can help retain and retrieve connected information instead of treating every fact as an isolated text fragment.

### Three useful memory categories

The study notes distinguish three kinds of memory:

1. **Short-term memory:** A chain of messages within the current conversation. It preserves immediate conversational context.
2. **Long-term memory:** A persistent knowledge graph that can retain facts, people, entities, and context across sessions.
3. **Reasoning memory:** An audit trail of how an agent reached a conclusion, including reasoning steps, linked entities, tool calls, and beliefs.

These are conceptual memory categories. A particular AI application may implement them using different storage systems and data structures.

---

# 6. Cypher: querying a graph

**Cypher** is Neo4j's declarative query language. It lets a user describe a graph pattern to find or modify data.

Instead of specifying every low-level traversal step, a query describes the pattern of nodes and relationships that should match.

## 6.1 Basic syntax

\`\`\`cypher
MATCH (p:Person)
WHERE p.name = 'Naveen'
RETURN p;
\`\`\`

- \`MATCH\`: finds graph patterns.
- \`WHERE\`: filters matched results.
- \`RETURN\`: specifies what to return.

## 6.2 Pattern notation

Cypher uses ASCII-art-like syntax.

| Syntax | Meaning |
|---|---|
| \`(p)\` | A node bound to variable \`p\` |
| \`(p:Person)\` | A node with label \`Person\` |
| \`{name: 'Naveen'}\` | A property map |
| \`-[r]->\` | A directed relationship bound to \`r\` |
| \`-[:ACTED_IN]->\` | A directed relationship of type \`ACTED_IN\` |
| \`(a)--(b)\` | A relationship between nodes, with direction/type unspecified |
| \`RETURN p\` | Return the variable \`p\` |

Example:

\`\`\`cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN p.name AS actor, m.title AS movie;
\`\`\`

This finds movies connected to the matching person through an \`ACTED_IN\` relationship.

## 6.3 Common clauses

| Clause | Purpose |
|---|---|
| \`MATCH\` | Find graph patterns |
| \`OPTIONAL MATCH\` | Match a pattern when possible, retaining rows when it is absent |
| \`WHERE\` | Filter rows or matched patterns |
| \`RETURN\` | Return results |
| \`WITH\` | Pass and transform intermediate results |
| \`CREATE\` | Create new nodes and relationships |
| \`MERGE\` | Match a pattern or create it when it does not exist |
| \`SET\` | Add or update properties and labels |
| \`REMOVE\` | Remove properties or labels |
| \`DELETE\` | Delete nodes or relationships |
| \`ORDER BY\` | Sort results |
| \`LIMIT\` | Restrict the number of returned rows |

---

# 7. Reading a graph: the Movie database

Neo4j's Movie graph is a useful example for understanding relationships.

A simplified model:

\`\`\`text
(Person) ──DIRECTED──> (Movie)
(Person) ──ACTED_IN──> (Movie)
(Movie)  ──IN_GENRE──> (Genre)
(User)   ──RATED─────> (Movie)
\`\`\`

## 7.1 Find a person by name

\`\`\`cypher
MATCH (p:Person)
WHERE p.name = 'Naveen'
RETURN p;
\`\`\`

This matches a Person node whose name property is Naveen.

## 7.2 Find a movie and the people who acted in it

\`\`\`cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE m.title = 'RRR'
RETURN p, r, m;
\`\`\`

The pattern connects a Person to a Movie through an \`ACTED_IN\` relationship. The query returns the person, relationship, and movie for every matching row.

## 7.3 Return selected properties

\`\`\`cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN p.name AS actor, m.title AS movie
ORDER BY movie;
\`\`\`

Returning selected properties is often more useful than returning entire nodes when building an application response.

---

# 8. How Neo4j executes a Cypher query

A Cypher query is declarative: you describe the result pattern, and Neo4j plans how to retrieve it.

A simplified query lifecycle:

\`\`\`text
Cypher query string
        |
        v
Parsing and semantic analysis
        |
        v
Abstract Syntax Tree (AST)
        |
        v
Logical planning and optimization
        |
        v
Physical execution plan
        |
        v
Execution pipeline
        |
        v
Result records streamed to the client
\`\`\`

## 8.1 Parsing and semantic analysis

Neo4j parses the query and builds an internal representation, commonly described as an abstract syntax tree (AST).

It checks items such as:

- Whether the query follows Cypher syntax.
- Whether variables and identifiers are used consistently.
- Whether referenced labels, properties, and clauses are valid in context.

## 8.2 Logical planning and optimization

The planner determines a logical way to satisfy the requested pattern.

An important concept is the **anchor** or starting point: the planner chooses where to begin finding a pattern. A selective starting point can reduce the number of records that need to be explored.

For example, locating one Person by an indexed property may be a better starting point than scanning every Person node.

## 8.3 Cost-based optimization (CBO)

The cost-based optimizer estimates the cost of alternative plans. Estimates can consider factors such as:

- Expected number of rows (cardinality).
- Available indexes and constraints.
- Database statistics.
- Expected work and memory use of plan operators.

The optimizer chooses a plan based on estimated cost. Estimates are not guarantees of actual runtime.

## 8.4 Execution and result streaming

The chosen plan is executed by the database engine. Results are returned as records to the client. Depending on the driver and result consumption, records can be consumed incrementally rather than requiring the application to materialize every result at once.

Useful commands for inspecting a plan:

\`\`\`cypher
EXPLAIN
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
\`\`\`

\`\`\`cypher
PROFILE
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, m.title;
\`\`\`

- \`EXPLAIN\` shows the planned operations without running the query.
- \`PROFILE\` runs the query and reports actual execution measurements, such as rows and database hits.

---

# 9. Pattern matching

Pattern matching is the central way to describe graph traversals in Cypher.

Example:

\`\`\`cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p, m;
\`\`\`

This pattern describes:

1. A node labelled \`Person\`, bound to \`p\`.
2. An outgoing relationship of type \`ACTED_IN\`.
3. A node labelled \`Movie\`, bound to \`m\`.

## 9.1 Variables

Variables such as \`p\`, \`m\`, and \`r\` bind matched graph elements so they can be referenced in later clauses.

\`\`\`cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
RETURN p.name, type(r), m.title;
\`\`\`

## 9.2 Variable-length paths

A relationship pattern can specify a range of traversal lengths. This is useful when the number of hops is not fixed.

For example, in current Cypher syntax:

\`\`\`cypher
MATCH path = (p:Person)-[:KNOWS*1..3]->(other:Person)
WHERE p.name = 'Naveen'
RETURN path;
\`\`\`

This searches paths of one to three KNOWS relationships. Variable-length traversal should be bounded when possible because the number of possible paths can grow quickly.

---

# 10. Creating nodes and relationships

Cypher supports both creating new graph elements and matching existing ones.

## 10.1 Create a node with CREATE

\`\`\`cypher
CREATE (m:Movie {
    title: 'Inception',
    released: 2010
})
RETURN m;
\`\`\`

\`CREATE\` creates a new pattern each time it runs. Running the same query again can create another node with the same property values.

## 10.2 Create multiple nodes and a relationship

\`\`\`cypher
CREATE (m:Movie {title: 'Inception'})
CREATE (u:User {name: 'Adam'})
CREATE (u)-[:RATED {rating: 5}]->(m)
RETURN u, m;
\`\`\`

This creates two nodes and connects them with a relationship that has a \`rating\` property.

## 10.3 Use MERGE to match or create

\`\`\`cypher
MERGE (m:Movie {title: 'Inception'})
RETURN m;
\`\`\`

- If the specified pattern matches, \`MERGE\` uses the existing match.
- If it does not match, \`MERGE\` creates the pattern.

For predictable identity and uniqueness under concurrent writes, define an appropriate uniqueness constraint on the identifying property. Also remember that MERGE applies to the whole pattern supplied to it; splitting node and relationship merges can make intended matching behavior clearer.

## 10.4 Set properties

\`\`\`cypher
MATCH (m:Movie {title: 'Inception'})
SET m.rating = 8.8
RETURN m;
\`\`\`

## 10.5 Delete a relationship

\`\`\`cypher
MATCH (u:User {name: 'Adam'})-[r:RATED]->(m:Movie)
DELETE r;
\`\`\`

To delete a node, its relationships must first be removed, or use \`DETACH DELETE\` when deleting the node and its relationships is intended:

\`\`\`cypher
MATCH (u:User {name: 'Adam'})
DETACH DELETE u;
\`\`\`

Be careful with deletion queries: inspect the matched pattern before executing destructive operations.

---

# 11. Indexes, constraints, and performance

## 11.1 Indexes

An index helps Neo4j find starting nodes by property values without scanning every node with a label.

Example:

\`\`\`cypher
CREATE INDEX person_name_index
FOR (p:Person)
ON (p.name);
\`\`\`

A query can then use the index when its pattern and predicates are suitable:

\`\`\`cypher
MATCH (p:Person {name: 'Naveen'})
RETURN p;
\`\`\`

Indexes are most useful for selective lookups and anchoring a graph traversal. They do not automatically make every part of every query faster.

## 11.2 Uniqueness constraints

A uniqueness constraint prevents duplicate values for a property within a label.

\`\`\`cypher
CREATE CONSTRAINT person_name_unique
FOR (p:Person)
REQUIRE p.name IS UNIQUE;
\`\`\`

Use a stable identifier when possible. A person's name may not be unique in a real application, so an application-specific ID is often a better identity key.

## 11.3 Understanding database hits

When inspecting a query with \`PROFILE\`, database hits provide an indication of the work performed by plan operators. They are not the same thing as elapsed time, and should be interpreted alongside rows, memory, and runtime.

Practical performance habits:

- Start from selective indexed lookups when appropriate.
- Avoid unbounded variable-length traversals unless required.
- Return only the properties and records the application needs.
- Filter early when it reduces downstream work.
- Use \`EXPLAIN\` and \`PROFILE\` to inspect actual plans and measurements.
- Be mindful of high-degree nodes and patterns that expand into many paths.

---

# 12. Quick revision

## 12.1 Core concepts

| Term | Meaning |
|---|---|
| Taxonomy | Hierarchical classification of concepts |
| Ontology | Formal representation of concepts, meaning, properties, and relationships |
| Graph database | Database model built around entities and their connections |
| Node | An entity in the graph |
| Label | A category assigned to a node |
| Relationship | A directed, typed connection between two nodes |
| Property | A key-value attribute on a node or relationship |
| Cypher | Neo4j's declarative graph query language |
| Pattern matching | Describing the nodes and relationships to find |
| Index-free adjacency | Native graph traversal by following stored relationships |
| Knowledge graph | Connected representation of entities and meaningful facts |
| CBO | Cost-based optimization of query plans |

## 12.2 Essential Cypher patterns

\`\`\`cypher
// Find nodes
MATCH (p:Person)
RETURN p;

// Filter by a property
MATCH (p:Person)
WHERE p.name = 'Naveen'
RETURN p;

// Match a relationship
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p, m;

// Create a node
CREATE (p:Person {name: 'Naveen'})
RETURN p;

// Match or create a node
MERGE (p:Person {id: 'person-001'})
RETURN p;

// Update a property
MATCH (p:Person {id: 'person-001'})
SET p.name = 'Naveen'
RETURN p;

// Inspect a query plan
EXPLAIN
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p, m;
\`\`\`

## 12.3 Mental model to remember

- **Taxonomy** organizes concepts into categories.
- **Ontology** describes concepts and the meaning of their relationships.
- **Graph database** stores entities and connections directly.
- **Neo4j** uses nodes, labels, relationships, and properties.
- **Cypher** describes graph patterns declaratively.
- **The query planner** chooses an execution strategy; \`PROFILE\` helps inspect what actually happened.

---

*These notes are a learning reference. Check the Neo4j documentation for version-specific Cypher syntax and database behavior before applying commands in production.*


---

# 13. Cypher fundamentals, filtering, and data modification

This section adds the Cypher topics from the handwritten study notes: naming conventions, filtering with `WHERE`, relationship traversal, graph metadata, and creating, updating, and deleting graph data.

## 13.1 Cypher naming and case conventions

The study notes use these conventions:

| Element | Recommended style | Example |
|---|---|---|
| Node labels | PascalCase / CamelCase | `Person`, `Movie` |
| Property keys | camelCase | `firstName`, `releasedYear` |
| Variables | camelCase | `person`, `movie` |
| Cypher keywords | UPPERCASE | `MATCH`, `WHERE`, `RETURN` |

Cypher keywords are case-insensitive, but using uppercase keywords makes queries easier to scan. Labels, property keys, and variables should be written consistently because their identifiers are case-sensitive.

## 13.2 Filtering with WHERE

The `WHERE` clause filters the rows produced by a `MATCH` pattern. It is generally clearer to keep filtering conditions in `WHERE` rather than trying to put every condition directly into the pattern.

```cypher
MATCH (p:Person)
WHERE p.name = 'Tom Hanks'
RETURN p;
```

Multiple conditions can be combined with `AND`, `OR`, and `NOT`:

```cypher
MATCH (m:Movie)
WHERE m.released >= 2000
  AND m.released <= 2023
RETURN m.title, m.released;
```

The same range can also be expressed using a comparison chain:

```cypher
MATCH (m:Movie)
WHERE 2000 <= m.released <= 2023
RETURN m.title, m.released;
```

Check whether a property exists and is not null:

```cypher
MATCH (m:Movie)
WHERE m.tagline IS NOT NULL
RETURN m.title, m.tagline;
```

## 13.3 Partial-string filtering

Cypher provides operators for matching part of a string:

| Operator | Meaning | Example |
|---|---|---|
| `STARTS WITH` | String begins with the given text | `p.name STARTS WITH 'Mich'` |
| `ENDS WITH` | String ends with the given text | `p.name ENDS WITH 'Hanks'` |
| `CONTAINS` | String contains the given text | `p.name CONTAINS 'cha'` |

Example:

```cypher
MATCH (p:Person)
WHERE p.name STARTS WITH 'Michael'
RETURN p.name;
```

For case-insensitive matching, normalize the property and comparison value to the same case:

```cypher
MATCH (p:Person)
WHERE toLower(p.name) = toLower('MICHAEL')
RETURN p;
```

Other useful string functions include `toUpper()` and `toLower()`.

## 13.4 Filtering with lists and NOT

Use `IN` to check whether a value occurs in a list:

```cypher
MATCH (p:Person)
WHERE p.born IN [1995, 1965, 1970]
RETURN p.name, p.born;
```

Use `NOT` to exclude a condition or graph pattern. For example, find people who wrote a movie but did not direct it:

```cypher
MATCH (p:Person)-[:WROTE]->(m:Movie)
WHERE NOT (p)-[:DIRECTED]->(m)
RETURN p.name, m.title;
```

The negated pattern is evaluated for the same person and movie matched by the main pattern.

## 13.5 Traversing relationships in either direction

A relationship in the stored graph has a direction. A query can explicitly match that direction:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p, m;
```

To traverse from the movie back to the person, reverse the pattern:

```cypher
MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)
RETURN m, p;
```

If direction is not important for a particular match, an undirected pattern can be used:

```cypher
MATCH (p:Person)-[:ACTED_IN]-(m:Movie)
RETURN p, m;
```

## 13.6 Filter using relationship properties

A relationship can contain properties just like a node. For example, a person's role in a movie can be stored on the `ACTED_IN` relationship:

```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE 'Neo' IN r.roles
RETURN p.name, m.title, r.roles;
```

This finds people whose `ACTED_IN` relationship contains the role `Neo`.

## 13.7 Inspecting properties and graph metadata

Use `keys()` to retrieve the property names present on a particular node or relationship:

```cypher
MATCH (p:Person)
RETURN p.name, keys(p);
```

Different nodes with the same label can have different property sets. For example, one Person node may have `name` and `born`, while another may also have `bio`.

To retrieve property keys used in the graph, the study notes use:

```cypher
CALL db.propertyKeys();
```

To retrieve labels present in the database:

```cypher
CALL db.labels();
```

These procedures report graph-wide metadata rather than the properties of only one matched node.

## 13.8 Creating nodes: CREATE versus MERGE

Use `CREATE` when a new node or relationship should be created every time the query runs:

```cypher
CREATE (p:Person {name: 'Naveen'});
```

Running this query repeatedly can create duplicate Person nodes with the same name.

Use `MERGE` when you want to match the specified pattern if it exists, or create it if it does not:

```cypher
MERGE (p:Person {id: 'person-001'})
RETURN p;
```

A `MERGE` pattern should contain a reliable identifying property. The study notes emphasize specifying a unique primary-key-like property when creating a node with `MERGE`. In a real database, enforce the intended uniqueness with a uniqueness constraint where appropriate.

**Choosing between them:**

- Use `MERGE` for identity-based, match-or-create operations.
- Use `CREATE` when every execution is intended to create a new entity or event, such as a new sensor reading or transaction record.
- Do not use `CREATE` as a substitute for checking identity when duplicates are not wanted.

## 13.9 Creating a relationship with MERGE

A relationship can be merged between already matched nodes:

```cypher
MATCH (p:Person {name: 'Naveen'}),
      (m:Movie {title: 'The Matrix'})
MERGE (p)-[:ACTED_IN]->(m);
```

This matches or creates the directed `ACTED_IN` relationship from the Person to the Movie. The direction in the pattern matters.

## 13.10 Updating properties with SET and REMOVE

Use `SET` to add a property or update its value:

```cypher
MATCH (m:Movie {title: 'The Matrix'})
SET m.released = 2008
RETURN m;
```

Multiple properties can be set in one clause:

```cypher
MATCH (m:Movie {title: 'The Matrix'})
SET m.tagline = 'A world within a world',
    m.released = 1999
RETURN m;
```

Use `REMOVE` to remove a property:

```cypher
MATCH (m:Movie {title: 'The Matrix'})
REMOVE m.tagline
RETURN m;
```

The property is removed from the node; `REMOVE` is not used to assign a null value.

## 13.11 Conditional behavior in MERGE

`ON CREATE SET` applies only when `MERGE` creates the pattern. `ON MATCH SET` applies when the pattern already exists.

```cypher
MERGE (p:Person {name: 'Mckenna Grace'})
ON CREATE SET p.createdAt = datetime()
ON MATCH SET p.updatedAt = datetime()
RETURN p;
```

This is useful when tracking when an entity was first created and when an existing entity was most recently encountered.

## 13.12 Deleting nodes and relationships

First match the element that should be deleted. Use `DELETE` to delete a relationship:

```cypher
MATCH (p:Person {name: 'Jane Doe'})-[r:ACTED_IN]->()
DELETE r;
```

Use `DELETE` to delete a node only when it has no relationships:

```cypher
MATCH (p:Person {name: 'Jane Doe'})
DELETE p;
```

A node that still has relationships cannot be deleted with plain `DELETE`. Use `DETACH DELETE` when the intention is to delete the node and all relationships connected to it:

```cypher
MATCH (p:Person {name: 'John Doe'})
DETACH DELETE p;
```

To delete every node and relationship in the database:

```cypher
MATCH (n)
DETACH DELETE n;
```

**Warning:** The last query is destructive and removes the entire graph. Always inspect the matched data and confirm the target database before running deletion queries.

## 13.13 Adding and removing labels

Use `SET` to add a label to a node:

```cypher
MATCH (p:Person {name: 'Naveen'})
SET p:Developer
RETURN p;
```

Use `REMOVE` to remove a label:

```cypher
MATCH (p:Person {name: 'Naveen'})
REMOVE p:Developer
RETURN p;
```

The node remains in the graph; only the specified label is changed.

---

## 14. Quick revision: filtering and data modification

| Requirement | Cypher pattern |
|---|---|
| Filter rows | `WHERE condition` |
| Check a range | `WHERE 2000 <= m.released <= 2023` |
| Check property presence | `WHERE m.tagline IS NOT NULL` |
| Match a prefix | `STARTS WITH` |
| Match a suffix | `ENDS WITH` |
| Match a substring | `CONTAINS` |
| Match any value in a list | `IN [value1, value2]` |
| Exclude a pattern | `WHERE NOT (a)-[:TYPE]->(b)` |
| Get property names | `keys(n)` |
| Get graph property keys | `CALL db.propertyKeys()` |
| Get graph labels | `CALL db.labels()` |
| Create unconditionally | `CREATE` |
| Match or create | `MERGE` |
| Update or add a property | `SET` |
| Remove a property or label | `REMOVE` |
| Delete a relationship | `DELETE r` |
| Delete a node and its relationships | `DETACH DELETE n` |
| Set values only on creation | `ON CREATE SET` |
| Set values only on a match | `ON MATCH SET` |

*Reminder: run destructive queries only after checking the matched records. Use a stable unique identifier for identity-based MERGE operations.*


---

# 15. Building a domain graph

A **domain graph** models the important concepts in a particular application domain and the relationships between them. The goal is to translate the application's needs and the questions users ask into nodes, relationships, and properties.

Domain graph modelling is iterative rather than a one-pass exercise:

1. Start with the domain and the questions the application needs to answer.
2. Identify the important concepts and represent them as nodes.
3. Connect the nodes using meaningful, named relationships.
4. Check the model against real questions.
5. If the model cannot answer a question clearly, refine it by adding or changing nodes, relationships, or properties.
6. Repeat with the next question and refine the domain scope as understanding improves.

The central modelling test is practical: **does the graph answer the real questions the application needs to answer?**

## 15.1 A useful modelling rule

The handwritten notes capture a simple naming guideline:

> (Noun) — [VERB] → (Noun)

Use nouns for the entities and verbs for the relationships between them.

For example:

```text
(Tom Hanks:Person) ──ACTED_IN──> (Forrest Gump:Movie)
(Forrest Gump:Movie) ──IN_GENRE──> (Drama:Genre)
(User) ──RATED──> (Forrest Gump:Movie)
```

This makes the graph readable: the nodes represent things, while the relationship types describe what connects them.

## 15.2 Anchor nodes

Not every noun in a domain needs to become a node. An **anchor node** is an important entity through which many of the application's questions and graph traversals begin.

When choosing anchors, ask:

- Which entities do users commonly start their questions with?
- Which nodes are likely to be revisited by many query paths?
- Which entities provide a useful starting point for traversing the model?

For example, in a movie domain, Person and Movie can be useful anchors because questions often begin with an actor, director, movie, or genre. The best anchors depend on the application's actual query patterns.

## 15.3 Properties versus relationships

When modelling a fact, decide whether it should be represented as a property or as its own relationship or node.

- Use a **property** for a value that describes an entity or a relationship, such as a movie's release year or a rating value.
- Use a **relationship** when the fact connects two entities and that connection is meaningful to traverse.
- Consider a separate **node** when the concept has its own identity, attributes, or connections to other entities.

For example, a user's rating can be represented as a relationship with a rating property:

```cypher
MATCH (u:User {id: 'user-1'})
MATCH (m:Movie {title: 'Forrest Gump'})
MERGE (u)-[r:RATED]->(m)
SET r.rating = 5;
```

Whether a connection deserves its own relationship or a richer intermediate node depends on how the application needs to query and describe it.

---

# 16. Domain graphs, lexical graphs, entity graphs, and ontologies

Unstructured content can be transformed into connected representations in more than one way. The notes distinguish lexical graphs and entity graphs as two approaches.

## 16.1 Lexical graphs

A **lexical graph** represents the content and structure of a document itself. It focuses on what the document says and how its pieces are arranged, rather than only extracting real-world entities.

A basic approach is to split a document into chunks and preserve their order:

```text
(Review)
   ├──HAS_CHUNK──> (Chunk 1)
   │                  |
   │              NEXT_CHUNK
   │                  |
   └──HAS_CHUNK──> (Chunk 2)
```

For longer documents, a flat list of chunks may lose useful structure. Documents can contain sections and subsections, so model those levels too:

```text
(Production Report)
   ├──HAS_SECTION──> (Filming Section)
   │                     └──HAS_CHUNK──> (Chunk)
   └──HAS_SECTION──> (Reception Section)
                         └──HAS_CHUNK──> (...)
```

This preserves document hierarchy and chunk relationships. It can be useful when an application needs to retrieve a passage together with its surrounding section or document context.

## 16.2 Entity graphs

An **entity graph** represents the things mentioned in content and the connections between those things.

For example, two chapters might mention people, locations, or themes:

```text
(Chapter 1) ──MENTIONS──> (Tom Hanks:Person)
(Chapter 1) ──MENTIONS──> (Savannah:Location)

(Chapter 2) ──MENTIONS──> (Tom Hanks:Person)
(Chapter 2) ──MENTIONS──> (Redemption:Theme)
(Chapter 2) ──MENTIONS──> (Robin Wright:Person)
```

The same entity can be connected to multiple chunks or chapters. This links references across the document collection instead of keeping every mention isolated.

## 16.3 How the representations work together

Lexical graphs and entity graphs are not mutually exclusive models. They are different layers of representation that can be connected:

- The lexical layer preserves the document, section, chunk, and ordering structure.
- The entity layer captures people, places, organizations, concepts, and other extracted entities.
- Relationships between the layers connect a source passage to the entities it mentions.

Together, they allow an application to move from a retrieved chunk to the entities mentioned in it, and from an entity to other chunks where it appears.

## 16.4 Ontology: rules for the graph

An **ontology** is not the data itself. It defines the rules and meaning of the model: what kinds of things are allowed to exist and how they fit together.

For example, an ontology can specify that an Actor is a kind of Person and a Director is also a kind of Person:

```text
             (Person)
              /    \
         (Actor)  (Director)
```

This is a type hierarchy: it moves from a general type to more specific types.

An ontology helps establish:

- Which labels or entity types should be used.
- Which relationships are meaningful or permitted.
- How entity types relate to one another.
- Consistent naming and classification rules for future extracted entities.

For example, if an extraction system identifies someone as an Actor, the ontology provides the type vocabulary and hierarchy that the application should follow. It also helps resolve naming ambiguity, such as whether a domain concept should be called Customer or Client.

**Key distinction:** the graph stores the entities and facts; the ontology describes the vocabulary, types, and rules used to model those facts.

---

# 17. Importing data into Neo4j: choosing an approach

There is no single import method that fits every dataset. Choose an ingestion path based on the kind of data, its volume, whether the load is one-off or continuous, the required load speed, and the amount of application-specific business logic.

The decision flow in the study notes can be summarized as follows.

## 17.1 Start with the kind of data

| Data category | Approach noted in the study material |
|---|---|
| Semantic data / RDF | neosemantics (n10s) |
| Application configuration or DB-form changes | Neo4j migrations / Liquibase |
| Ordinary data | Continue by considering volume, load requirements, and ingestion pattern |

## 17.2 For ordinary data, consider scale and purpose

Ask these questions in order:

1. **How large is the data?** Is it a small proof of concept, or a large dataset?
2. **Are very fast initial load times required?** If so, consider a bulk-import route.
3. **Is the load batch-based or streaming?** A one-off import and a continuous event stream have different requirements.
4. **Does the ingestion require application-specific business logic?** Simple transformations may fit an ETL tool; complex logic may need a custom application using the Neo4j driver.
5. **What is the source format?** CSV and other formats may have different import paths.

## 17.3 A practical selection guide

| Situation | Candidate approach from the notes |
|---|---|
| Very large initial import where load speed is important | `neo4j-admin database import` |
| Streaming ingestion | Neo4j Kafka Connector |
| One-off or batch CSV import | Cypher `LOAD CSV` or Neo4j Data Importer |
| Simple transformation workflow | An available ETL tool |
| Complex application-specific ingestion logic | Custom application using a Neo4j driver |
| RDF / semantic data | neosemantics (n10s) |
| Application schema/configuration migrations | Migration tooling such as Liquibase |

The choice should be driven by the ingestion requirements rather than by the availability of a particular tool.

---

# 18. Neo4j data-import tools

The study notes list several tools and workflows for moving data into Neo4j.

## 18.1 Neo4j Data Importer

Neo4j Data Importer is a visual, low-code tool for mapping source data into a graph model. It is useful when the import can be described through a mapping workflow rather than custom application code.

Use it when the source data and mapping requirements fit the tool's capabilities.

## 18.2 Cypher LOAD CSV

Cypher provides built-in support for loading CSV files. The source notes highlight using a CSV header and reading the file from a file path.

Example:

```cypher
LOAD CSV WITH HEADERS FROM 'file:///transactions.csv' AS row
MERGE (t:Transaction {id: row.id})
SET t.reference = row.reference,
    t.amount = toInteger(row.amount),
    t.timestamp = datetime(row.timestamp)
RETURN t;
```

Important points:

- `LOAD CSV WITH HEADERS` exposes each column through the row variable.
- `row.id`, `row.reference`, and similar expressions access CSV columns.
- Convert values to the intended data types when loading them.
- `MERGE` can be used with a stable identifier to avoid creating the same logical entity repeatedly.
- File access depends on the database's import configuration and deployment environment.

## 18.3 neo4j-admin import

For large initial datasets, the notes identify `neo4j-admin` import as a high-performance bulk-loading option. It is intended for importing large datasets and can be preferable to inserting a very large number of records through ordinary transactional queries.

Use this path when the dataset size and loading requirements justify a bulk import workflow. Check the Neo4j version's command syntax and operational requirements before running it.

## 18.4 Neo4j Kafka Connector

For streaming data, the notes identify the Neo4j Kafka Connector as an ingestion option.

A streaming workflow is appropriate when new events or records need to be processed continuously rather than loaded only once. The connector helps integrate Kafka-based event streams with Neo4j.

## 18.5 ETL tools and custom applications

ETL means **Extract, Transform, Load**:

1. **Extract** data from one or more source systems.
2. **Transform** it into the desired structure and types.
3. **Load** the transformed data into Neo4j.

The notes mention Apache Hop as an ETL option. An ETL tool can be suitable when the workflow consists of supported source connections and straightforward transformations.

When the ingestion process contains complex business rules or application-specific behavior, a custom application using a Neo4j driver provides more control.

## 18.6 RDF and semantic data: neosemantics

The notes identify **neosemantics (n10s)** for working with semantic data and RDF in Neo4j.

This is a different import concern from loading ordinary CSV rows: RDF represents information using semantic graph statements, so an RDF-aware integration is appropriate when the source data is already expressed in that model.

## 18.7 Migration tooling

Application configuration and schema changes should be managed deliberately as an application evolves. The notes mention Neo4j migrations and Liquibase as migration-oriented options.

Migration tooling is useful for keeping database changes organized and repeatable across development and deployment environments. It serves a different purpose from bulk-loading a large dataset.

## 18.8 Aura Document Intelligence

The notes also mention **Aura Document Intelligence** as a way to build a knowledge graph from unstructured documents.

The intended workflow is:

1. Provide unstructured documents.
2. Use the document-intelligence workflow to design or extract a knowledge graph.
3. Query the resulting graph using Cypher.

This can reduce the amount of manual entity and relationship modelling needed for document-driven graph use cases. Review the product's current capabilities and availability before selecting it for a project.

---

## 19. Quick revision: graph modelling and data ingestion

| Concept / tool | Remember |
|---|---|
| Domain graph | Models the concepts and relationships needed to answer domain questions |
| Iterative modelling | Test the graph against real questions and refine it |
| Anchor node | A useful starting entity for common query paths |
| Noun–verb–noun | A practical naming pattern: node, relationship, node |
| Lexical graph | Preserves document, section, chunk, and ordering structure |
| Entity graph | Connects extracted entities and their mentions |
| Ontology | Defines entity types, relationship vocabulary, and modelling rules |
| Neo4j Data Importer | Visual / low-code graph data mapping |
| `LOAD CSV` | Load CSV rows through Cypher |
| `neo4j-admin` import | Bulk import for large initial datasets |
| Neo4j Kafka Connector | Streaming ingestion |
| ETL | Extract, transform, and load data |
| Apache Hop | ETL tool mentioned in the study notes |
| neosemantics (n10s) | RDF / semantic data integration |
| Liquibase / migrations | Manage repeatable database changes |
| Aura Document Intelligence | Document-driven knowledge graph workflow |

**Mental model:** first decide what the graph should represent; then decide how to ingest the data. A well-designed domain model, lexical structure, entity layer, and ontology make the resulting graph easier to query and evolve.



---

# 20. Learning reference and acknowledgement

These notes cover the learning path content completed so far: the first four courses in the official **Neo4j GraphAcademy – Sharpen your Cypher** pathway, ending with **Importing Data Fundamentals**.

1. [Neo4j Fundamentals](https://graphacademy.neo4j.com/courses/neo4j-fundamentals)
2. [Cypher Fundamentals](https://graphacademy.neo4j.com/courses/cypher-fundamentals)
3. [Graph Data Modeling Fundamentals](https://graphacademy.neo4j.com/courses/modeling-fundamentals)
4. [Importing Data Fundamentals](https://graphacademy.neo4j.com/courses/importing-fundamentals)

The next courses in the pathway — Intermediate Cypher Queries, Cypher Aggregations, and Cypher Indexes and Constraints — are intentionally not added here because they are beyond the current learning progress.

**Thank you, Neo4j GraphAcademy team, for providing high-quality learning resources, hands-on exercises, and practical examples that make learning graph databases, Cypher, graph data modeling, and data importing clear and engaging.**


---

# 20. Course-by-course practical coverage

This section fills in practical topics from the first four courses in the GraphAcademy **Sharpen your Cypher** path. It is intentionally limited to:

1. Neo4j Fundamentals
2. Cypher Fundamentals
3. Graph Data Modeling Fundamentals
4. Importing Data Fundamentals

The examples use a small movie graph so that the same entities and relationships can be followed from modelling, to querying, to importing.

## 20.1 Neo4j Fundamentals: graph thinking and tools

### When is a graph database a good fit?

Use a graph model when the important questions involve connections and paths between entities.

| Use case | Example graph question |
|---|---|
| Recommendations | Which movies are liked by users with similar tastes? |
| Fraud detection | Are two accounts connected through shared devices or identities? |
| Knowledge graph | Which people, companies, and documents are connected to a topic? |
| Network management | Which services depend on a service that is failing? |
| Social network | Who follows the same people as a given user? |

A relational database can also represent connected data, but graph queries express traversals directly through relationships. Choose the database based on the shape of the questions and workload.

### Graph vocabulary

- **Vertex / node:** an entity such as a person, movie, or user.
- **Edge / relationship:** a named connection between two nodes.
- **Path:** a sequence of nodes and relationships.
- **Directed graph:** relationships have a start node and an end node.
- **Property graph:** nodes and relationships can both store key-value properties.

Example:

```text
(:Person {name: 'Tom Hanks'})
       -[:ACTED_IN]->
(:Movie {title: 'Forrest Gump', released: 1994})
```

### Neo4j tools: what each is for

| Tool | Main purpose |
|---|---|
| Neo4j Browser | Run Cypher, inspect query results, and visualize graph patterns |
| Neo4j Aura Console | Create and manage cloud-hosted Neo4j database instances |
| Neo4j Desktop | Manage local projects and database instances (availability/product packaging can vary by release) |
| Neo4j Data Importer | Visually map CSV columns to nodes, relationships, and properties |
| Neo4j Bloom | Explore and visualize graph data through a search-oriented interface |

The tool you choose depends on the task: Browser for querying, Console for Aura management, and Data Importer for a visual CSV-to-graph workflow.

## 20.2 Cypher Fundamentals: read and write graph data

### Reading nodes

Return every node with the Person label:

```cypher
MATCH (p:Person)
RETURN p;
```

Return only selected properties:

```cypher
MATCH (p:Person)
RETURN p.name AS personName, p.born AS birthYear;
```

### Finding relationships

Find all people who acted in a movie:

```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
RETURN p, r, m;
```

The pattern has three parts:
1. `(p:Person)` matches a Person node.
2. `-[r:ACTED_IN]->` matches an outgoing ACTED_IN relationship.
3. `(m:Movie)` matches the connected Movie node.

To find the movies acted in by a particular person:

```cypher
MATCH (p:Person {name: 'Tom Hanks'})-[:ACTED_IN]->(m:Movie)
RETURN m.title, m.released
ORDER BY m.released;
```

### Filtering query results

```cypher
MATCH (m:Movie)
WHERE m.released >= 1990
  AND m.released <= 2000
RETURN m.title, m.released
ORDER BY m.released;
```

A compact equivalent for a property equality lookup is:

```cypher
MATCH (m:Movie {title: 'Forrest Gump'})
RETURN m;
```

### Creating nodes

```cypher
CREATE (m:Movie {
  title: 'The Matrix',
  released: 1999
})
RETURN m;
```

This creates a new node every time it runs. If you execute it repeatedly, duplicate Movie nodes can be created.

### Creating relationships

Match the two endpoints first, then create the relationship:

```cypher
MATCH (p:Person {name: 'Keanu Reeves'})
MATCH (m:Movie {title: 'The Matrix'})
CREATE (p)-[:ACTED_IN]->(m);
```

The relationship direction is from Person to Movie. Direction is part of the stored relationship pattern and should represent the domain meaning consistently.

### Updating properties

```cypher
MATCH (m:Movie {title: 'The Matrix'})
SET m.tagline = 'Welcome to the Real World'
RETURN m;
```

Update several properties in one SET clause:

```cypher
MATCH (m:Movie {title: 'The Matrix'})
SET m.released = 1999,
    m.tagline = 'Welcome to the Real World'
RETURN m;
```

### MERGE: match or create

```cypher
MERGE (m:Movie {title: 'The Matrix'})
ON CREATE SET m.createdAt = datetime()
ON MATCH SET m.lastSeenAt = datetime()
RETURN m;
```

- If the pattern is not found, MERGE creates it and ON CREATE SET runs.
- If the pattern already exists, MERGE matches it and ON MATCH SET runs.
- A property such as title is only a safe identity when the domain guarantees it is unique. Prefer a stable ID and enforce uniqueness with a constraint.

### Deleting data safely

Delete a relationship while keeping its endpoint nodes:

```cypher
MATCH (p:Person {name: 'Keanu Reeves'})-[r:ACTED_IN]->(m:Movie {title: 'The Matrix'})
DELETE r;
```

Delete a node that has no relationships:

```cypher
MATCH (m:Movie {title: 'Temporary Movie'})
DELETE m;
```

Delete a node and all of its relationships only when that is intended:

```cypher
MATCH (m:Movie {title: 'Temporary Movie'})
DETACH DELETE m;
```

**Safety check:** run the MATCH portion with RETURN first to verify exactly which nodes or relationships will be affected. Avoid broad deletion patterns unless you intentionally want to clear the graph.

## 20.3 Graph Data Modeling Fundamentals: design, test, and refactor

Graph modelling is not simply converting every CSV column into a property. Start from the questions the application must answer, then choose nodes, relationships, and properties that make those questions natural to query.

### Step 1: define the domain and questions

For a movie application, example questions might be:

- Which movies did a person act in?
- Who directed a movie?
- Which users rated a movie?
- Which genres does a movie belong to?
- What is the average rating for each movie?

These questions reveal the entities and connections the model needs.

### Step 2: apply the modelling rule

A useful starting rule is:

**Nouns become candidate nodes; verbs become candidate relationships; descriptive values become properties.**

Example:

```text
(Person) -[:ACTED_IN]-> (Movie)
(Person) -[:DIRECTED]-> (Movie)
(User)   -[:RATED {rating: 5}]-> (Movie)
(Movie)  -[:IN_GENRE]-> (Genre)
```

This is a starting point, not an automatic rule. Promote a property to a node when it has its own identity, is shared by many entities, or needs its own relationships.

### Step 3: distinguish nodes, relationships, and properties

| Domain fact | Suggested representation | Why |
|---|---|---|
| Movie title | Movie property | Describes one movie |
| Movie release year | Movie property | Descriptive value |
| Person acts in movie | ACTED_IN relationship | Connects two entities |
| User rates movie | RATED relationship | Rating belongs to the user–movie connection |
| Genre shared by many movies | Genre node | Can be reused and queried as an entity |
| Actor's role in a movie | Relationship property or Role node | Depends on whether the role needs its own identity/details |

### Step 4: create an initial model

```cypher
CREATE (p:Person {personId: 'p-101', name: 'Tom Hanks'})
CREATE (m:Movie {movieId: 'm-101', title: 'Forrest Gump', released: 1994})
CREATE (g:Genre {name: 'Drama'})
CREATE (p)-[:ACTED_IN]->(m)
CREATE (m)-[:IN_GENRE]->(g);
```

For repeatable setup scripts, use MERGE with stable IDs and suitable uniqueness constraints instead of blindly running CREATE again.

### Step 5: test the model with real questions

A model is not complete just because the nodes and relationships look sensible. Test it by writing queries for the questions identified earlier.

```cypher
// Which movies did this person act in?
MATCH (p:Person {personId: 'p-101'})-[:ACTED_IN]->(m:Movie)
RETURN m.title, m.released;
```

```cypher
// Which movies belong to the Drama genre?
MATCH (m:Movie)-[:IN_GENRE]->(g:Genre {name: 'Drama'})
RETURN m.title;
```

If a required question is awkward or impossible to answer, revisit the model. Testing against a small instance graph helps expose missing labels, relationships, and properties before importing a large dataset.

### Step 6: refactor the graph when the model becomes clearer

Refactoring means changing the graph structure to represent the domain more accurately or make common queries easier.

#### Use specific labels thoughtfully

A person can have a general Person label and additional labels when the distinction is meaningful:

```cypher
MATCH (p:Person {personId: 'p-101'})
SET p:Actor;
```

Do not create a new label for every descriptive value or one-off category. Labels should represent useful entity types that help organize and query the graph.

#### Avoid storing repeated shared values everywhere

If many movies share a genre, model Genre as a node rather than copying a genre string onto every movie:

```cypher
MERGE (g:Genre {name: 'Drama'})
MATCH (m:Movie {movieId: 'm-101'})
MERGE (m)-[:IN_GENRE]->(g);
```

This reduces duplicated descriptive data and makes genre-based traversal straightforward.

#### Use meaningful relationship types

If an application frequently needs to distinguish acting from directing, separate relationship types make the graph explicit:

```text
(Person)-[:ACTED_IN]->(Movie)
(Person)-[:DIRECTED]->(Movie)
```

Do not use one vague relationship type for unrelated meanings if queries need to distinguish them.

#### Consider an intermediate node for complex facts

A simple role can be stored as a relationship property:

```cypher
MATCH (p:Person {personId: 'p-101'})
MATCH (m:Movie {movieId: 'm-101'})
MERGE (p)-[r:ACTED_IN]->(m)
SET r.role = 'Forrest';
```

If the role itself needs more details or relationships (for example, screen time, character identity, or links to other story entities), represent it as a node:

```cypher
MATCH (p:Person {personId: 'p-101'})
MATCH (m:Movie {movieId: 'm-101'})
MERGE (role:Role {name: 'Forrest'})
MERGE (p)-[:PLAYED]->(role)
MERGE (role)-[:IN_MOVIE]->(m);
```

Use an intermediate node only when the concept has enough identity or behavior to justify it. Otherwise, a relationship property is simpler.

## 20.4 From domain graph to knowledge graph

A knowledge graph can connect structured entities with information extracted from documents.

### Lexical layer: preserve document structure

```text
(Document)
   -[:HAS_SECTION]-> (Section)
   -[:HAS_CHUNK]-> (Chunk)
(Chunk)-[:NEXT_CHUNK]->(NextChunk)
```

The lexical layer keeps track of where information came from and how chunks relate to their original document and section.

### Entity layer: connect mentioned things

```text
(Chunk)-[:MENTIONS]->(Person)
(Chunk)-[:MENTIONS]->(Company)
(Chunk)-[:MENTIONS]->(Location)
```

The same Person node can be mentioned by multiple chunks. This creates links across documents rather than storing every mention as an isolated string.

### Ontology: keep the vocabulary consistent

An ontology defines the kinds of entities and relationships the application recognizes. For example, Actor and Director can be modeled as more specific kinds of Person. It provides shared modelling rules; it is not a replacement for the actual graph data.

## 20.5 Importing Data Fundamentals: understand the import options

The course's hands-on workflow uses CSV movie and user-rating data and the Neo4j Data Importer. The source files are flat tables; the goal is to map them into connected graph elements.

### Choose an import method

| Method | When it is useful |
|---|---|
| Neo4j Data Importer | Visual mapping of CSV columns to nodes and relationships |
| Cypher `LOAD CSV` | Scriptable CSV imports with custom Cypher transformations |
| Official Neo4j drivers | Application-controlled ingestion and custom business logic |
| `neo4j-admin database import` | High-volume initial import where bulk-loading is appropriate |

The right method depends on dataset size, source format, repeatability, transformation needs, and whether the database is already running. The Data Importer is the visual workflow taught in this course.

### Understand the source files before importing

Before mapping CSV data, inspect:

1. What does each row represent?
2. Which columns identify a record uniquely?
3. Which columns describe the entity?
4. Which columns refer to another file's entity?
5. Are values missing, duplicated, or inconsistent?
6. Which values should be strings, integers, floats, booleans, dates, or datetimes?
7. Does the source contain multiple rows for one entity?

Example source files:

**movies.csv**

```csv
movieId,title,released
m-101,Forrest Gump,1994
m-102,Toy Story,1995
```

**people.csv**

```csv
personId,name
p-101,Tom Hanks
p-102,Tim Allen
```

**acted_in.csv**

```csv
personId,movieId,role
p-101,m-101,Forrest
p-102,m-102,Buzz Lightyear
```

**ratings.csv**

```csv
userId,movieId,rating
u-1,m-101,5
u-1,m-102,4
```

The ID columns connect rows across files. They should be stable and consistently formatted.

## 20.6 Neo4j Data Importer: end-to-end workflow

The Data Importer provides a visual workflow to map source files to a graph model.

1. Open Neo4j Data Importer from the Neo4j tools / Console workflow available in your environment.
2. Create a new import project.
3. Add the CSV source files.
4. Inspect the columns and sample rows.
5. Create node mappings for entities such as Movie, Person, and User.
6. Assign a label to each node mapping.
7. Choose a unique ID field for each entity mapping.
8. Set the correct property types.
9. Add relationship mappings between the appropriate node mappings.
10. Choose the relationship type and direction.
11. Map source columns to relationship properties when needed.
12. Review the model and run the import.
13. Query the database to verify that nodes, relationships, and properties were created correctly.

The exact screen labels can vary as the product evolves, but the mapping concepts remain the same.

### Map CSV columns to node properties

For the movies.csv example:

| CSV column | Graph mapping |
|---|---|
| `movieId` | Movie unique identifier |
| `title` | `Movie.title` string |
| `released` | `Movie.released` integer |

For the people.csv example:

| CSV column | Graph mapping |
|---|---|
| `personId` | Person unique identifier |
| `name` | `Person.name` string |

### Choose property types deliberately

CSV files store textual representations. The importer must convert values to the intended graph property types.

| Source value | Intended type | Example |
|---|---|---|
| `Forrest Gump` | String | `title` |
| `1994` | Integer | `released` |
| `4.5` | Float | `rating` |
| `true` | Boolean | `active` |
| `2026-11-12` | Date | `createdOn` |

If a numeric value is imported as a string, numeric comparisons and calculations may not behave as intended. Check conversion failures and missing values before importing.

### Unique IDs and constraints

Every entity mapping should use a stable unique identifier. For example, Movie should use movieId rather than title.

A uniqueness constraint can enforce this rule in Neo4j:

```cypher
CREATE CONSTRAINT movie_id_unique
FOR (m:Movie)
REQUIRE m.movieId IS UNIQUE;
```

Similarly:

```cypher
CREATE CONSTRAINT person_id_unique
FOR (p:Person)
REQUIRE p.personId IS UNIQUE;
```

A constraint protects data integrity. It also creates or uses an index to support the constrained lookup, depending on the constraint type and Neo4j version.

### Create directed relationships from source IDs

The acted_in.csv file connects people to movies. The relationship should be directed from Person to Movie:

```text
(Person)-[:ACTED_IN {role: 'Forrest'}]->(Movie)
```

The personId column identifies the start node; movieId identifies the end node. The role column becomes a relationship property.

### Import user ratings as relationships

A rating belongs to a particular user–movie pair, so model it on the relationship:

```text
(User)-[:RATED {rating: 5}]->(Movie)
```

Example Cypher equivalent:

```cypher
MATCH (u:User {userId: 'u-1'})
MATCH (m:Movie {movieId: 'm-101'})
MERGE (u)-[r:RATED]->(m)
SET r.rating = 5;
```

If the same user can rate the same movie more than once over time, consider whether a single RATED relationship is sufficient. If rating history matters, an intermediate Rating node with timestamp and other metadata may be a better model.

## 20.7 Verify the imported graph

After importing, do not assume that a successful import means the model is correct. Run checks.

Count nodes by label:

```cypher
MATCH (m:Movie)
RETURN count(m) AS movieCount;
```

Inspect a relationship:

```cypher
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
RETURN p.name, r.role, m.title
LIMIT 10;
```

Inspect user ratings:

```cypher
MATCH (u:User)-[r:RATED]->(m:Movie)
RETURN u.userId, m.title, r.rating
LIMIT 10;
```

Check for nodes missing identifiers:

```cypher
MATCH (m:Movie)
WHERE m.movieId IS NULL
RETURN m;
```

Check for duplicate IDs before adding a uniqueness constraint:

```cypher
MATCH (m:Movie)
WITH m.movieId AS id, count(*) AS occurrences
WHERE id IS NOT NULL AND occurrences > 1
RETURN id, occurrences;
```

Compare imported counts with source-file row counts, inspect a few sample records, and test the domain questions the model was designed to answer.

## 20.8 Import checklist

- [ ] Source files and headers have been inspected.
- [ ] Each entity has a stable unique identifier.
- [ ] Duplicate and missing identifiers have been checked.
- [ ] Property types are correct.
- [ ] Node labels match the domain model.
- [ ] Relationship types and directions are correct.
- [ ] Relationship properties are mapped to the connection they describe.
- [ ] Constraints and indexes are considered for important lookup keys.
- [ ] Import counts are compared with source counts.
- [ ] Sample traversals and application questions return expected results.

---

# 21. Learning reference and acknowledgement

These notes cover the first four courses in the official **Neo4j GraphAcademy – Sharpen your Cypher** learning path:

1. [Neo4j Fundamentals](https://graphacademy.neo4j.com/courses/neo4j-fundamentals)
2. [Cypher Fundamentals](https://graphacademy.neo4j.com/courses/cypher-fundamentals)
3. [Graph Data Modeling Fundamentals](https://graphacademy.neo4j.com/courses/modeling-fundamentals)
4. [Importing Data Fundamentals](https://graphacademy.neo4j.com/courses/importing-fundamentals)

**Thank you, Neo4j GraphAcademy team, for providing high-quality learning resources, hands-on exercises, and practical examples that make learning graph databases, Cypher, graph data modeling, and data importing clear and engaging.**
