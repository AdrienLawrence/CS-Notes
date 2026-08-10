# Overview

- NoSQL means "Not Only SQL" and refers to databases whose main model is not the traditional relational table model

- NoSQL is an umbrella term, not one database design. Document, key-value, wide-column, graph, time-series, and vector databases solve different problems

- NoSQL does not automatically mean schemaless, faster, or more scalable. Every database has a data model, consistency behavior, indexes, limits, and operational tradeoffs

- Choose a database from the application's access patterns, correctness rules, scale, team knowledge, and operations instead of choosing from popularity

- Main families:

    Document: JSON-like records, such as MongoDB

    Key-value: Values accessed by key, such as Redis or DynamoDB key-value patterns

    Wide-column: Sparse distributed rows grouped by column families, such as Cassandra

    Graph: Nodes and edges optimized for relationship traversal, such as Neo4j

    Vector: Embeddings searched by similarity, such as Pinecone or pgvector inside PostgreSQL

# Relational vs NoSQL

- Relational databases organize normalized tables connected by foreign keys and queried with SQL

- NoSQL databases often place data needed together into one record or partition to reduce joins and distribute work

- Relational strengths include constraints, joins, ad hoc querying, mature transactions, and a standardized language

- NoSQL strengths may include flexible documents, specialized access, geographic distribution, horizontal partitioning, or a model matching the domain

- Polyglot persistence means using more than one storage technology where each has a clear responsibility

- Adding a second database also adds backups, monitoring, security, migrations, consistency boundaries, cost, and failure modes

# Distributed Data Concepts

- Replication stores copies of data on multiple nodes for availability, locality, and read scaling

- Partitioning or sharding divides data across nodes. A partition key decides where a record belongs

- A good partition key spreads load while keeping related access together. A hot key sends too much work to one partition

- Consistency describes what a read may observe after a write

    Strong consistency: A completed write is immediately visible under the system's guarantee

    Eventual consistency: Replicas may temporarily disagree but converge without new writes

    Read-your-writes: A client sees its own completed updates

- CAP states that during a network partition, a distributed system must choose which operations preserve consistency or availability. It does not mean every database is permanently only "CP" or "AP"

- PACELC adds the normal-operation tradeoff: if there is no partition, systems may still trade latency against consistency

- Quorum-like systems reason about replica count `N`, write acknowledgements `W`, and read responses `R`

- Multi-record transactions exist in several NoSQL systems but may cost more than operations designed around one document or partition

# Modeling Principles

- Model from the queries and mutations the application must perform

- Questions to ask:

    Which records are always read together?

    Which fields update together?

    How large can one record or embedded array become?

    Which fields require uniqueness or validation?

    Which queries need ordering, filtering, text search, or similarity search?

    What is the tenant boundary?

    What happens during retry or partial failure?

- Denormalization duplicates data to make reads local and fast. It requires a strategy for updating every copy

- Embedding stores related values inside a parent record. Referencing stores another record's identifier

- Prefer embedding when related data is bounded, owned by the parent, and normally read or updated with it

- Prefer referencing when related data is large, shared, independently queried, many-to-many, or frequently updated on its own

- Schema validation remains valuable in flexible databases. Flexibility should support evolution, not accidental inconsistent shapes

# MongoDB Fundamentals

- MongoDB is a document database storing BSON documents in collections

- BSON supports JSON-like objects plus types such as dates, binary data, decimals, and ObjectIds

- Hierarchy:

    Deployment/cluster -> Database -> Collection -> Document -> Field

- Every normal document has a unique `_id`

    ```javascript
    {
        _id: ObjectId("..."),
        email: "alice@example.com",
        profile: {
            displayName: "Alice",
            timezone: "America/New_York"
        },
        roles: ["student"],
        createdAt: ISODate("2026-08-10T12:00:00Z")
    }
    ```

- A collection does not require every document to contain identical fields, but application validation and collection schema validation should define supported shapes

- MongoDB operations are atomic at the single-document level, including changes to embedded fields and arrays

# MongoDB CRUD

    ```javascript
    // Create
    db.users.insertOne({
        email: "alice@example.com",
        active: true,
        createdAt: new Date()
    })

    // Read
    db.users.findOne({ email: "alice@example.com" })

    db.users.find(
        { active: true, age: { $gte: 18 } },
        { email: 1, active: 1 }
    ).sort({ createdAt: -1 }).limit(20)

    // Update
    db.users.updateOne(
        { email: "alice@example.com" },
        { $set: { active: false }, $currentDate: { updatedAt: true } }
    )

    // Delete
    db.users.deleteOne({ email: "alice@example.com" })
    ```

- Query operators include `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`, `$exists`, `$regex`, `$and`, `$or`, and `$not`

- Array operators include `$all`, `$elemMatch`, `$size`, `$push`, `$addToSet`, `$pull`, and positional update operators

- Update operators modify selected fields without replacing the full document

- `replaceOne` replaces the document while preserving `_id`

- Upsert updates a match or inserts a new document when none exists

- A cursor streams query results. Apply filtering and projection on the server rather than loading a collection into application memory

# MongoDB Indexes

- An index stores ordered or specialized keys which allow queries to avoid scanning every document

    ```javascript
    db.users.createIndex({ email: 1 }, { unique: true })
    db.events.createIndex({ tenantId: 1, createdAt: -1 })
    ```

- Compound index order matters. It should match common filters and sorting patterns

- The equality, sort, range guideline is useful when ordering compound keys

- Index types include single-field, compound, multikey for arrays, text, geospatial, hashed, wildcard, TTL, and partial indexes

- A unique index enforces uniqueness. Application checks alone have a race condition

- TTL indexes automatically remove eligible documents after a time but deletion is asynchronous, not exact to the second

- Partial indexes only include matching documents and can reduce size

- Covered queries return fields directly from an index without reading full documents

- Every index consumes storage and memory and adds write cost. Create indexes from observed query patterns

- Use `explain("executionStats")` to inspect index use, documents examined, keys examined, and execution plan

# MongoDB Aggregation

- The aggregation pipeline passes documents through ordered stages

    ```javascript
    db.orders.aggregate([
        { $match: { status: "PAID" } },
        { $unwind: "$items" },
        {
            $group: {
                _id: "$items.productId",
                units: { $sum: "$items.quantity" },
                revenue: { $sum: { $multiply: ["$items.price", "$items.quantity"] } }
            }
        },
        { $sort: { revenue: -1 } },
        { $limit: 10 }
    ])
    ```

- Common stages:

    `$match`: Filter

    `$project` / `$set`: Shape or compute fields

    `$group`: Aggregate by key

    `$sort`, `$skip`, `$limit`: Order and paginate

    `$unwind`: Expand array elements

    `$lookup`: Join another collection

    `$facet`: Run multiple sub-pipelines

    `$setWindowFields`: Window calculations

    `$merge` / `$out`: Write pipeline results

- Place selective `$match` stages early and support them with indexes

# MongoDB Relationships, Transactions, and Scaling

- Embedding gives one-read access and atomic parent updates but is limited by document growth and duplication

- References keep shared data separate and may be resolved with multiple queries or `$lookup`

- MongoDB supports multi-document ACID transactions. Use them when correctness crosses documents, while still modeling common operations to remain local when possible

- Replica sets provide redundancy and automatic primary election

- Read preference chooses which members may serve reads. Read concern and write concern control consistency/durability behavior

- Sharding distributes collections using a shard key

- A shard key should have useful cardinality, balanced frequency, and alignment with query targeting

- A monotonically increasing key may concentrate new writes unless the distribution strategy accounts for it

- MongoDB Atlas is the managed cloud service and adds managed operations, search, vector search, backups, and related platform tools

# MongoDB Application Practices

- Use the official driver for the language and maintain a reusable client/connection pool

- Validate input separately from trusting database document flexibility

- Use projections to return only required fields

- Store timestamps as date types, money as appropriate decimal/integer representations, and identifiers consistently

- Paginate large collections with stable indexed keys when offset pagination becomes expensive

- Protect against NoSQL injection by constructing typed query objects and never accepting raw client operators

- Back up and test restoration. Replication is not a backup against accidental deletion or bad writes

# Vector Database Fundamentals

- An embedding is a numeric vector produced by a model to represent semantic or structural features

- Similar items should occupy nearby regions in the model's vector space

- Vector search returns approximate or exact nearest neighbors under a distance/similarity function

- Common metrics:

    Cosine similarity: Compares direction and is common for text embeddings

    Dot product/inner product: Direction and magnitude; often used by normalized model outputs

    Euclidean/L2 distance: Geometric straight-line distance

- Use the metric expected by the embedding model. Distance and similarity may reverse ordering: smaller distance is better, while larger similarity is better

- Dense vectors contain a value for every dimension. Sparse vectors store mostly zero dimensions and often represent lexical/token signals

- Exact search checks all eligible vectors and gives perfect recall. Approximate Nearest Neighbor search trades recall for speed and scale

- HNSW builds a navigable graph. IVF groups vectors into partitions/lists. Product quantization compresses vectors at some accuracy cost

- Evaluate recall, latency, throughput, memory/storage, ingestion delay, filtering behavior, and cost using realistic data

# Pinecone

- Pinecone is a managed vector database designed for semantic, lexical, full-text, and hybrid retrieval

- Core model:

    Index: Searchable vector/document collection with a dimension, metric, and deployment configuration

    Namespace: Partition inside an index targeted by each data operation

    Record: ID, dense/sparse vector or integrated text, and optional metadata

    Metadata: Flat JSON values used for filtering and returning source context

- Namespaces are the strongest normal tool for tenant isolation and reduce the amount of data scanned per query

- Metadata filters narrow eligible records by fields such as user, notebook, source, date, or access level

    ```python
    from pinecone import Pinecone

    pc = Pinecone(api_key="...")
    index = pc.Index(host="INDEX_HOST")

    index.upsert(
        namespace="tenant-123",
        vectors=[{
            "id": "chunk-1",
            "values": embedding,
            "metadata": {
                "document_id": "doc-9",
                "chunk_text": "Source text",
                "visibility": "private",
            },
        }],
    )

    results = index.query(
        namespace="tenant-123",
        vector=query_embedding,
        top_k=8,
        filter={"visibility": {"$eq": "private"}},
        include_metadata=True,
        include_values=False,
    )
    ```

- Upsert inserts or replaces records by ID. Use stable deterministic IDs to make retries idempotent

- Fetch retrieves known IDs. Query/search ranks neighbors. Delete removes IDs, filters, or namespaces depending on the API

- Integrated embedding lets supported indexes accept raw text. External embedding gives direct control over the model and vectors

- Pinecone supports dense semantic search, sparse lexical search, full-text search, and hybrid approaches

- Hybrid retrieval combines semantic meaning with exact token signals. It is useful for identifiers, names, error messages, and domain vocabulary

- Reranking applies a more expensive model to a small candidate set for better final relevance

- Pinecone is eventually consistent, so new writes may not immediately appear in every query path

- Dimension and metric must match the embedding model. Changing the model commonly requires a new index and complete re-embedding

- Do not place all tenants in one namespace and depend only on a high-cardinality user filter when namespaces fit the tenancy model

- Never trust a namespace or tenant filter sent by the client. Derive it from the authenticated server-side identity

# pgvector vs Pinecone vs MongoDB Vector Search

- pgvector keeps vectors beside relational source data inside PostgreSQL

- Pinecone provides a specialized managed vector system with namespaces, metadata filtering, sparse/dense retrieval, and managed scaling

- MongoDB Vector Search keeps search close to document data in MongoDB Atlas

- Prefer pgvector when relational filters, transactions, joins, and one operational database are important and scale fits PostgreSQL

- Prefer Pinecone when vector search is a dedicated large-scale service and managed vector operations are more valuable than relational locality

- Prefer MongoDB Vector Search when application data already belongs in MongoDB documents and one platform should serve document and vector access

- The right choice depends on data ownership, scale, update pattern, query filters, latency, consistency, cost, portability, and team experience

# RAG Storage Pattern

- A retrieval record should usually include:

    Stable chunk ID

    Embedding

    Source document ID

    Tenant/user/notebook authorization scope

    Chunk position and text or a pointer to it

    Content hash

    Embedding model/version

    Source version and timestamps

- Ingestion flow:

    Authorize source -> Extract text -> Normalize -> Chunk -> Hash -> Skip unchanged chunks -> Embed -> Upsert -> Remove stale chunks

- Query flow:

    Authenticate -> Build server-side scope -> Embed query -> Retrieve candidates -> Rerank/filter -> Build cited context -> Generate -> Validate/log

- Similarity is not authorization. Apply tenant and ownership restrictions before or during retrieval, never only after generation

- Content hashes such as SHA-256 prevent re-embedding unchanged chunks and support idempotent ingestion

- Store model versions because embeddings from different models or dimensions generally do not share one comparable vector space

- Measure retrieval with labeled queries using recall@k, precision@k, mean reciprocal rank, normalized discounted cumulative gain, latency, and grounded answer quality

# Other NoSQL Tools

- Redis is an in-memory data structure server used for caching, rate limits, distributed coordination, streams, queues, and short-lived state

- DynamoDB is a managed key-value/document database built around partition/sort keys, secondary indexes, and explicit capacity/access patterns

- Cassandra is a wide-column distributed database designed for high write availability and query-driven denormalized tables

- Neo4j is a property graph database using nodes, relationships, and graph traversal queries

- Elasticsearch/OpenSearch are distributed search engines for inverted-index full-text search, aggregations, logs, and vector capabilities

- These systems overlap but are not interchangeable. Learn the primary data model and failure behavior before adopting one as a general database

# Quick Reference

- JSON-like owned aggregate -> Document database

- Fast lookup by known key -> Key-value database

- Relationship traversal -> Graph database

- Massive write-oriented partitioned tables -> Wide-column database

- Semantic similarity -> Vector search

- Related bounded data read together -> Embed

- Shared or independently queried data -> Reference

- Query is slow -> Inspect access pattern and execution plan before adding indexes

- Multi-tenant Pinecone -> Namespace per tenant when appropriate

- Relational records plus embeddings -> pgvector

- MongoDB relationship query -> Reference plus `$lookup`, or remodel for embedding

- Durable correctness across writes -> Use supported transactions/conditional writes and idempotency

- Replicas exist -> Still create and test backups
