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
