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
13. [Intermediate Cypher queries](#20-intermediate-cypher-queries)
14. [Cypher aggregations and lists](#21-cypher-aggregations-and-lists)
15. [Indexes and constraints in practice](#22-indexes-and-constraints-in-practice)

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

# 20. Intermediate Cypher queries

These notes extend the existing MATCH, WHERE, and RETURN examples with the techniques covered in GraphAcademy's **Intermediate Cypher Queries** course. The examples use the recommendations / movie graph: `Person`, `Movie`, `ACTED_IN`, `DIRECTED`, `RATED`, and `Genre`. Adjust labels and property names to match the dataset loaded in your database.

## 20.1 Advanced filtering

### Equality and multiple conditions

```cypher
MATCH (m:Movie)
WHERE m.released >= 2000
  AND m.released <= 2020
  AND m.title IS NOT NULL
RETURN m.title, m.released
ORDER BY m.released DESC;
```

Use `AND` when every condition must be true, `OR` when at least one condition is enough, and `NOT` to negate a condition.

### Range, list membership, and string predicates

```cypher
MATCH (m:Movie)
WHERE m.released IN [1999, 2000, 2001]
  AND m.title CONTAINS 'Star'
RETURN m.title, m.released;
```

Common predicates:

| Predicate | Example | Meaning |
|---|---|---|
| `IN` | `m.released IN [1999, 2000]` | Value appears in a list |
| `STARTS WITH` | `m.title STARTS WITH 'The'` | Prefix match |
| `ENDS WITH` | `m.title ENDS WITH 'Wars'` | Suffix match |
| `CONTAINS` | `m.title CONTAINS 'Star'` | Substring match |
| `IS NULL` | `m.tagline IS NULL` | Property is missing / null |
| `IS NOT NULL` | `m.tagline IS NOT NULL` | Property has a value |

For case-insensitive matching, normalize both sides:

```cypher
MATCH (m:Movie)
WHERE toLower(m.title) CONTAINS toLower($searchText)
RETURN m.title;
```

The parameter `$searchText` is supplied by the client; for example, `"matrix"`.

### Pattern predicates and EXISTS

Use an existence subquery when the question is whether a related pattern exists, rather than returning every matching related row.

```cypher
MATCH (p:Person)
WHERE EXISTS {
  MATCH (p)-[:ACTED_IN]->(:Movie)
}
RETURN p.name;
```

This returns people who have acted in at least one movie.

To find people who have not acted in any movie:

```cypher
MATCH (p:Person)
WHERE NOT EXISTS {
  MATCH (p)-[:ACTED_IN]->(:Movie)
}
RETURN p.name;
```

## 20.2 OPTIONAL MATCH

`MATCH` removes a row when its pattern cannot be found. `OPTIONAL MATCH` keeps the row and supplies `null` for variables in the missing part of the pattern.

```cypher
MATCH (p:Person)
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
ORDER BY p.name;
```

This is useful when a person may exist without a matching directed movie. Be careful when filtering optional variables: putting a condition in a later WHERE can remove rows whose optional value is null.

## 20.3 Ordering, pagination, and distinct results

```cypher
MATCH (m:Movie)
RETURN m.title, m.released
ORDER BY m.released DESC, m.title ASC
SKIP 10
LIMIT 5;
```

- `ORDER BY` sorts the rows.
- `SKIP` skips the first N rows.
- `LIMIT` returns at most N rows.
- Use a deterministic secondary sort key (such as title or id) for stable pagination.

Remove duplicate values with `DISTINCT`:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN DISTINCT p.name
ORDER BY p.name;
```

A person connected to multiple movies can produce multiple matched rows. `DISTINCT` removes duplicate returned values.

## 20.4 Working with maps and projections

A map is a collection of key-value pairs. A map projection builds a custom map from a node or relationship.

```cypher
MATCH (m:Movie)
RETURN m {
  .title,
  .released
} AS movie;
```

You can add computed values to a projection:

```cypher
MATCH (m:Movie)
RETURN m {
  .title,
  releaseDecade: (m.released / 10) * 10
} AS movie;
```

Map projections are useful when shaping query results for an API response. Ensure the source properties exist and have the expected types before doing arithmetic.

## 20.5 WITH: pipeline query stages

`WITH` passes selected variables and computed expressions from one stage of a query to the next. It also controls scope: variables not carried through a WITH are no longer available afterwards.

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WITH p, count(m) AS movieCount
WHERE movieCount >= 5
RETURN p.name, movieCount
ORDER BY movieCount DESC;
```

Step-by-step:
1. Match each person and their movies.
2. Group rows by person and calculate the number of movies.
3. Keep only people with at least five movies.
4. Return the people and counts in descending order.

A common mistake is trying to use a variable after a WITH that did not pass it forward.

## 20.6 UNWIND: turn a list into rows

`UNWIND` transforms each item in a list into a separate row.

```cypher
UNWIND ['Neo4j', 'Cypher', 'Graph'] AS topic
RETURN topic;
```

Example: create or match several genres:

```cypher
UNWIND ['Drama', 'Comedy', 'Thriller'] AS genreName
MERGE (g:Genre {name: genreName})
RETURN g.name;
```

Think of UNWIND as the reverse of collecting rows into a list with `collect()`.

## 20.7 CALL subqueries and UNION

A subquery can perform a separate query stage for each incoming row. Variables required inside the subquery are imported explicitly with a scope clause in current Cypher syntax:

```cypher
MATCH (p:Person)
CALL (p) {
  MATCH (p)-[:ACTED_IN]->(m:Movie)
  RETURN count(m) AS movieCount
}
RETURN p.name, movieCount
ORDER BY movieCount DESC;
```

For older Neo4j versions, check the version-specific syntax for importing variables into a CALL subquery.

Use `UNION` to combine results from compatible queries. `UNION` removes duplicate rows; `UNION ALL` preserves them.

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS person, m.title AS movie
UNION ALL
MATCH (p:Person)-[:DIRECTED]->(m:Movie)
RETURN p.name AS person, m.title AS movie;
```

Both sides must return the same number of columns with compatible names and types.

## 20.8 Variable-length traversal

A variable-length relationship pattern follows multiple hops:

```cypher
MATCH path = (start:Person {name: 'Tom Hanks'})-[:ACTED_IN*1..3]->(end)
RETURN path;
```

This legacy relationship-range form is widely encountered in existing Cypher material. In newer Cypher versions, quantified path patterns are also available; check the syntax supported by your Neo4j version.

For a practical query, keep the hop range bounded. The number of possible paths can grow quickly as the graph branches.

## 20.9 Parameters: separate data from query structure

Use parameters instead of concatenating user input into a query string.

```cypher
MATCH (m:Movie)
WHERE m.released >= $minYear
  AND m.released <= $maxYear
RETURN m.title, m.released
ORDER BY m.released DESC;
```

Example parameter values:

```json
{
  "minYear": 2000,
  "maxYear": 2020
}
```

Parameters:
- Make queries reusable with different values.
- Help prevent Cypher injection when values originate from users.
- Allow the database to reuse query plans in appropriate circumstances.
- Cannot be used as substitutes for labels, relationship types, or arbitrary query syntax.

In Neo4j Browser, parameters can be set with a command such as:

```cypher
:param minYear => 2000;
```

In application code, pass the parameter map through the Neo4j driver's query API.

---

# 21. Cypher aggregations and lists

Aggregation turns many matched rows into a summary. GraphAcademy's aggregation course covers grouping, list handling, and functions such as `count()`, `collect()`, `sum()`, `avg()`, `min()`, `max()`, and percentiles.

## 21.1 How grouping works in Cypher

In a RETURN or WITH clause, expressions that are not aggregated become grouping keys.

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, count(m) AS movieCount
ORDER BY movieCount DESC;
```

The result contains one row per actor because `p.name` is the grouping key. `count(m)` counts the matched movie values for each actor.

## 21.2 Common aggregation functions

| Function | Purpose | Example |
|---|---|---|
| `count(x)` | Count non-null values | `count(m)` |
| `collect(x)` | Collect values into a list | `collect(m.title)` |
| `sum(x)` | Add numeric values | `sum(r.rating)` |
| `avg(x)` | Calculate arithmetic mean | `avg(r.rating)` |
| `min(x)` | Smallest value | `min(r.rating)` |
| `max(x)` | Largest value | `max(r.rating)` |
| `percentileCont(x, p)` | Continuous percentile | `percentileCont(r.rating, 0.5)` |
| `percentileDisc(x, p)` | Discrete percentile | `percentileDisc(r.rating, 0.5)` |

## 21.3 Count related nodes

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, count(DISTINCT m) AS movies
ORDER BY movies DESC;
```

Use `DISTINCT` when multiple matched paths could otherwise count the same movie more than once.

## 21.4 Collect related values into a list

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, collect(DISTINCT m.title) AS movies;
```

This returns one row per actor and a list of their movie titles.

## 21.5 Calculate average ratings

Assuming users connect to movies through RATED relationships with a numeric `rating` property:

```cypher
MATCH (:User)-[r:RATED]->(m:Movie)
RETURN m.title AS movie,
       count(r) AS reviewCount,
       avg(r.rating) AS averageRating
ORDER BY averageRating DESC;
```

To avoid ranking a movie with only one review above well-reviewed movies, add a minimum review count:

```cypher
MATCH (:User)-[r:RATED]->(m:Movie)
WITH m, count(r) AS reviewCount, avg(r.rating) AS averageRating
WHERE reviewCount >= 10
RETURN m.title, reviewCount, averageRating
ORDER BY averageRating DESC;
```

## 21.6 List functions

Lists can be inspected and transformed using functions such as:

| Function | Purpose |
|---|---|
| `size(list)` | Number of elements in a list |
| `head(list)` | First element |
| `last(list)` | Last element |
| `tail(list)` | All elements except the first |
| `range(start, end)` | Generate a numeric list |
| `reverse(list)` | Reverse a list |

Example:

```cypher
WITH ['Neo4j', 'Cypher', 'Graph'] AS topics
RETURN size(topics) AS total,
       head(topics) AS firstTopic,
       last(topics) AS lastTopic,
       reverse(topics) AS reversed;
```

## 21.7 List comprehension

List comprehension creates a new list by iterating over an existing list and optionally filtering values.

```cypher
WITH [1990, 1995, 2000, 2005, 2010] AS years
RETURN [year IN years WHERE year >= 2000 | year] AS recentYears;
```

The general form is:

```cypher
[variable IN list WHERE condition | expression]
```

The WHERE and projection portions are optional depending on the operation.

## 21.8 Pattern comprehension

Pattern comprehension builds a list from a graph pattern without turning each match into a separate top-level result row.

```cypher
MATCH (p:Person {name: 'Tom Hanks'})
RETURN p.name,
       [(p)-[:ACTED_IN]->(m:Movie) | m.title] AS movies;
```

This returns the selected person's name and a list of titles connected through ACTED_IN.

## 21.9 Working with dates and durations

Neo4j has temporal types and functions. Use typed temporal values for date arithmetic rather than storing dates as arbitrary strings.

```cypher
RETURN date() AS today,
       date('2025-01-01') AS startDate,
       duration.between(date('2025-01-01'), date()).days AS daysElapsed;
```

For a stored date property:

```cypher
MATCH (r:Review)
WHERE r.createdAt IS NOT NULL
RETURN r.createdAt,
       date() - date(r.createdAt) AS age;
```

Choose a consistent property type during import. Convert strings to dates or datetimes at ingestion time where possible.

## 21.10 Aggregation pitfalls

- Aggregation changes the number of rows. Check which expressions are grouping keys.
- Use `count(*)` to count rows and `count(expression)` to count non-null values.
- Use `count(DISTINCT x)` when duplicate matches should count once.
- A missing optional match can yield null; aggregation functions generally ignore null values.
- Apply filters at the correct stage: WHERE before aggregation filters input rows; WHERE after WITH can filter aggregated groups.
- Use PROFILE on large queries to understand the cost of matching and aggregation.

---

# 22. Indexes and constraints in practice

Indexes and constraints help keep graph data reliable and make selective lookups more efficient. They solve related but different problems:

- **Constraint:** enforces a data-integrity rule.
- **Index:** helps the database find matching data efficiently.

## 22.1 Uniqueness constraint

A uniqueness constraint prevents two nodes with the same label from having the same value for the constrained property.

```cypher
CREATE CONSTRAINT person_id_unique
FOR (p:Person)
REQUIRE p.personId IS UNIQUE;
```

Now this identity-based merge is safer:

```cypher
MERGE (p:Person {personId: $personId})
ON CREATE SET p.name = $name
ON MATCH SET p.updatedAt = datetime()
RETURN p;
```

Prefer a stable identifier such as `personId` over a name, because names are not necessarily unique or immutable.

## 22.2 Node key and property existence constraints

A node key constraint combines existence and uniqueness for the constrained properties:

```cypher
CREATE CONSTRAINT person_key
FOR (p:Person)
REQUIRE (p.personId) IS NODE KEY;
```

Property existence constraint example:

```cypher
CREATE CONSTRAINT person_name_required
FOR (p:Person)
REQUIRE p.name IS NOT NULL;
```

**Version and edition note:** Constraint availability differs by Neo4j version and deployment edition. Check the documentation for the server you are using before running node key or property-existence examples. If a constraint already exists, inspect it before attempting to create another one with the same schema.

## 22.3 Range index

A range index is a common index type for equality, range, and ordering lookups on supported property types.

```cypher
CREATE RANGE INDEX movie_released_index
FOR (m:Movie)
ON (m.released);
```

Example query:

```cypher
MATCH (m:Movie)
WHERE m.released >= 2010
RETURN m.title, m.released
ORDER BY m.released;
```

The planner may use the index when the predicate and query plan are suitable. An index does not guarantee that every query will use it.

## 22.4 Composite index

A composite index covers more than one property on the same label:

```cypher
CREATE RANGE INDEX movie_year_title_index
FOR (m:Movie)
ON (m.released, m.title);
```

This can help queries that filter on the indexed property combination. Property order matters to index use, so design the index around the application's real predicates and inspect the plan.

## 22.5 Text index

A text index is intended for supported string search operations:

```cypher
CREATE TEXT INDEX person_name_text_index
FOR (p:Person)
ON (p.name);
```

Example:

```cypher
MATCH (p:Person)
WHERE p.name CONTAINS 'Tom'
RETURN p.name;
```

Use the appropriate index type for the search operation. A text index is not the same as a full-text index.

## 22.6 Full-text index

A full-text index supports tokenized text search across one or more properties:

```cypher
CREATE FULLTEXT INDEX person_fulltext_index
FOR (p:Person)
ON EACH [p.name, p.bio];
```

Query it with the full-text procedure:

```cypher
CALL db.index.fulltext.queryNodes(
  'person_fulltext_index',
  'tom hanks'
)
YIELD node, score
RETURN node.name AS name, score
ORDER BY score DESC;
```

Full-text search returns a relevance score. It is useful for text search across content, while a range or text index serves different predicate patterns.

## 22.7 Inspect indexes and constraints

List indexes:

```cypher
SHOW INDEXES;
```

List constraints:

```cypher
SHOW CONSTRAINTS;
```

Inspect query planning:

```cypher
EXPLAIN
MATCH (p:Person {personId: $personId})
RETURN p;
```

Run the query with profiling to see actual execution information:

```cypher
PROFILE
MATCH (p:Person {personId: $personId})
RETURN p;
```

Look for operators such as index seeks versus label scans, and compare estimated rows with actual rows. Use the plan to validate whether an index helps the specific query.

## 22.8 Drop an index or constraint

Only remove schema objects when you have confirmed they are no longer needed:

```cypher
DROP INDEX movie_released_index;
DROP CONSTRAINT person_id_unique;
```

Use the exact names shown by `SHOW INDEXES` and `SHOW CONSTRAINTS`. Do not drop an index backing a constraint independently; manage the constraint itself.

## 22.9 Practical index and constraint checklist

1. Identify the properties used frequently to find starting nodes.
2. Add uniqueness constraints to stable identity properties.
3. Add indexes for frequent, selective lookup patterns.
4. Avoid creating indexes for every property; indexes consume storage and add write overhead.
5. Use SHOW INDEXES and SHOW CONSTRAINTS to inspect schema state.
6. Use EXPLAIN and PROFILE to verify the planner's choices.
7. Revisit indexes as application query patterns and graph size evolve.

---

## 23. Quick revision: intermediate Cypher, aggregation, and schema

| Requirement | Cypher pattern |
|---|---|
| Filter with multiple conditions | `WHERE a AND b` |
| Check if a pattern exists | `WHERE EXISTS { MATCH ... }` |
| Keep rows without a related match | `OPTIONAL MATCH` |
| Sort and paginate | `ORDER BY ... SKIP ... LIMIT ...` |
| Remove duplicate rows | `RETURN DISTINCT` |
| Pass variables between stages | `WITH` |
| Convert list items into rows | `UNWIND` |
| Run a subquery | `CALL (variable) { ... }` |
| Combine query results | `UNION` / `UNION ALL` |
| Pass values safely from an app | `$parameterName` |
| Count matches | `count()` |
| Collect values | `collect()` |
| Calculate summary values | `sum()`, `avg()`, `min()`, `max()` |
| Build a list from another list | List comprehension |
| Build a list from a graph pattern | Pattern comprehension |
| Enforce unique identity | Uniqueness constraint |
| Index equality/range lookups | Range index |
| Index multiple properties | Composite range index |
| Search strings | Text index |
| Search tokenized text | Full-text index |
| Inspect schema | `SHOW INDEXES`, `SHOW CONSTRAINTS` |
| Inspect query plan | `EXPLAIN`, `PROFILE` |

---

## 24. Learning reference and acknowledgement

These additional notes were prepared with reference to the official **Neo4j GraphAcademy – Sharpen your Cypher** learning path:

- [Sharpen your Cypher – Learning Path](https://graphacademy.neo4j.com/categories/sharpen-cypher)
- [Intermediate Cypher Queries](https://graphacademy.neo4j.com/courses/cypher-intermediate-queries)
- [Cypher Aggregations](https://graphacademy.neo4j.com/courses/cypher-aggregation)
- [Cypher Indexes and Constraints](https://graphacademy.neo4j.com/courses/cypher-indexes-constraints)

**Thank you, Neo4j GraphAcademy team, for providing high-quality learning resources, hands-on exercises, and practical examples that make it easier for developers to understand Cypher and build better graph database applications.**
