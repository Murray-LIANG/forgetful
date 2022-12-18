# Designing Data Intensive Applications


## Data intensive application

Common seen functionalities:
- Store data so that they, or another application, can find it again later (_databases_)
- Remember the result of an expensive operation, to speed up reads (_caches_)
- Allow users to search data by keyword or filter it in various ways (_search indexes_)
- Send a message to another process, to be handled asynchronously (_stream processing_)
- Periodically crunch a large amount of accumulated data (_batch processing_)

## Reliability

Continuing to work correctly, even when things go wrong.

**A fault is not the same as a failure.** A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures.


## Scalability

- "If the system grows in a particular way, what are our options for coping with the growth?"
- "How can we add computing
resources to handle the additional load?"

### Load parameters
The best choice of parameters
depends on the architecture of your system: it may be requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, or something else.

#### Twitter example

Use cases:
1. Post tweet: A user can publish a new message to their followers (4.6k requests/sec on average, over 12k requests/sec at peak).
2. Home timeline: A user can view tweets posted by the people they follow (300k requests/sec).

Two ways to implement these use cases:
1. Posting a tweet simply inserts the new tweet into a global collection of tweets. When a user requests their home timeline, look up all the people they follow, find all the tweets for each of those users, and merge them (sorted by time).

![](../../static/images/ddia-figure-1-2.png)

2. Maintain a cache for each user’s home timeline—like a mailbox of tweets for each recipient user (see Figure 1-3). When a user posts a tweet, look up all the people who follow that user, and insert the new tweet into each of their home timeline caches. The request to read the home timeline is then cheap, because its result has been computed ahead of time.

![](../../static/images/ddia-figure-1-3.png)

The first version of Twitter used approach 1, but the systems struggled to keep up with the load of home timeline queries, so the company switched to approach 2. This works better because the average rate of published tweets is almost two orders of magnitude lower than the rate of home timeline reads, and so in this case it’s preferable to do more work at write time and less at read time.

However, the downside of approach 2 is that posting a tweet now requires a lot of extra work. On average, a tweet is delivered to about 75 followers, so 4.6k tweets per second become 345k writes per second to the home timeline caches. But this average hides the fact that the number of followers per user varies wildly, and some users have over 30 million followers. This means that a single tweet may result in over 30 million writes to home timelines! Doing this in a timely manner—Twitter tries to deliver tweets to followers within five seconds—is a significant challenge.

In the example of Twitter, the distribution of followers per user (maybe weighted by how often those users tweet) is a key load parameter for discussing scalability, since it determines the fan-out load. Your application may have very different characteristics, but you can apply similar principles to reasoning about its load.

The final twist of the Twitter anecdote: now that approach 2 is robustly implemented, Twitter is moving to a hybrid of both approaches. Most users’ tweets continue to be fanned out to home timelines at the time when they are posted, but a small number of users with a very large number of followers (i.e., celebrities) are excepted from this fan-out. Tweets from any celebrities that a user may follow are fetched separately and merged with that user’s home timeline when it is read, like in approach 1. This hybrid approach is able to deliver consistently good performance.

### Describing performance

- When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
- When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

#### Percentiles
The median is also known as the 50th percentile, and sometimes abbreviated as p50.

#### SLO and SLA
For example, percentiles are often used in service level objectives (SLOs) and service level agreements (SLAs), contracts that define the expected performance and availability of a service. An SLA may state that the service is considered to be up if it has a median response time of less than 200 ms and a 99th percentile under 1 s (if the response time is longer, it might as well be down), and the service may be required to be up at least 99.9% of the time. These metrics set expectations for clients of the ser‐
vice and allow customers to demand a refund if the SLA is not met.

### Scaling up vs. scaling out

Distributing load across multiple machines is also known as a **shared-nothing** architecture.

## Maintainability

### 1. Operability
Make it easy for operations teams to keep the system running smoothly.

### 2. Simplicity
Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.)

### 3. Evolvability
Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as _extensibility_, _modifiability_, or _plasticity_.

## Simplicity

There are various possible symptoms of complexity: **explosion of the state space**, **tight coupling of modules**, **tangled dependencies**, **inconsistent naming and terminology**, **hacks aimed at solving performance problems**, **special-casing to work around issues elsewhere**, and many more.

## Data models

### Hierarchical model

All data are in one big tree. Not good for representing many-to-many relationships.

### Relational model

#### One-to-many relationship
1. Foreign key reference.
2. XML/JSON datatype column.
3. Encode XML/JSON document and store it on a text column.

#2 has better **locality** than #1.

#### Many-to-many relationship
Normal to refer to rows in other tables by ID, because joins are easy.

### NoSQL model

#### One-to-many relationship
Not needed for one-to-many.

#### Many-to-many relationship
You could store a text string directly but it will introduce duplication.

And if you store ID but the database itself doesn't support joins, you have to emulate a join in application code by making multiple queries to the database.

1. Document databases: data comes in self-contained documents and relationships between one document and another are rare.
2. Graph databases go in the opposite direction, targeting use cases where anything is potentially related to everything.

### Relational vs. Document databases

- Document data model: schema flexibility, better performance due to locality, and that for some applications it is closer to the data structures used by the application.
- Relational model: provides better support for joins, and many-to-one and many-to-many relationships.

- Document data model: if your application does use many-to-many relationships, the document model becomes less appealing. It’s possible to reduce the need for joins by denormalizing, but then the application code needs to do additional work to keep the denormalized data consistent. Joins can be emulated in application code by making multiple requests to the database, but that also moves complexity into the application and is usually slower than a join performed by specialized code inside the database.  In such cases, using a document model can lead to significantly more complex application code and worse performance.

- Document data model: schema-on-read (the structure of the data is implicit, and only interpreted when the data is read)
- Relational model: schema-on-write (the schema is explicit and the database ensures all written data conforms to it)

- Document data model: the schema-on-read approach is advantageous if the items in the collection don’t all have the same structure for some reason (i.e., the data is heterogeneous) - for example, because:
  - There are many different types of objects, and it is not practical to put each type of object in its own table.
  - The structure of the data is determined by external systems over which you have no control and which may change at any time.
- Relational model: with "statically typed" database schema, you would typically perform a **migration**.
```sql
ALTER TABLE users ADD COLUMN first_name text;
UPDATE users SET first_name = split_part(name, ' ', 1); -- PostgreSQL
```
Schema changes have a bad reputation of being slow and requiring downtime.

- Document data model: A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB’s BSON). If your application often needs to access the entire document (for example, to render it on a web page), there is a performance advantage to this **storage locality**.
- Relational model: If data is split across multiple tables, multiple index lookups are required to retrieve it all, which may require more disk seeks and take more time.

## Query languages

### Imperative vs. declarative
An _imperative_ language tells the computer to perform certain operations in a certain order. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.
In a _declarative_ query language, like SQL or relational algebra, you just specify the pattern of the data you want - what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated) - but not how to achieve that goal. It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query.
A declarative query language is attractive because it is typically more concise and easier to work with than an imperative API. But **more importantly**, it also hides implementation details of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries.

## Graph-like data models

### Property graph
Each **vertex** consists of:
- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)

Each **edge** consists of:
- A unique identifier
- The vertex at which the edge starts (the tail vertex)
- The vertex at which the edge ends (the head vertex)
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)


You can think of a graph store as consisting of two relational tables, one for vertices and one for edges, as shown in _Example 2-2_ (this schema uses the PostgreSQL json datatype to store the properties of each vertex or edge). The head and tail vertex are stored for each edge; if you want the set of incoming or outgoing edges for a vertex, you can query the edges table by `head_vertex` or `tail_vertex`, respectively.

_Example 2-2. Representing a property graph using a relational schema_
```sql
CREATE TABLE vertices (
vertex_id integer PRIMARY KEY,
properties json
);
CREATE TABLE edges (
edge_id integer PRIMARY KEY,
tail_vertex integer REFERENCES vertices (vertex_id),
head_vertex integer REFERENCES vertices (vertex_id),
label text,
properties json
);
CREATE INDEX edges_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```
Some important aspects of this model are:
1. Any vertex can have an edge connecting it with any other vertex. There is no schema that restricts which kinds of things can or cannot be associated.
2. Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus traverse the graph - i.e., follow a path through a chain of vertices - both forward and backward. (That's why Example 2-2 has indexes on both the `tail_vertex` and `head_vertex` columns.)
3. By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph, while still maintaining a clean data model.

### Triple-store
All information is stored in the form of very simple three-part statements: `(subject, predicate, object)`. For example, in the triple `(Jim, likes, bananas)`, `Jim` is the subject, `likes` is the predicate (verb), and `bananas` is the object.
