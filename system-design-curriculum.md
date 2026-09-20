# System Design Curriculum: Fresher to Senior

Each concept below is layered into three tiers so you can track your own growth against it:

- **Fresher** — what you should be able to explain and implement with guidance in your first 0-2 years.
- **Mid** — what an engineer with 2-5 years owns end-to-end: trade-offs, common failure modes, when to reach for the pattern.
- **Senior/Staff** — what you're expected to reason about at 5+ years: cross-cutting trade-offs, scale limits, org-level and cost implications, and knowing when *not* to use the pattern.

The 24 modules and case studies below match the original syllabus, in order.

## Module 1: System Design Thinking & Requirement Analysis

**What is HLD? Scope, goals, functional vs. non-functional requirements**
- Fresher: Define functional requirements (what the system does) vs. non-functional (latency, availability, durability, scalability). Write a one-page scope doc for a simple feature.
- Mid: Push back on vague requirements; turn ambiguous asks into measurable NFRs (e.g. "fast" → "p99 < 200ms"). Know which NFRs conflict with each other.
- Senior/Staff: Set scope boundaries across teams, defend against scope creep, and decide which NFRs are load-bearing for the business (e.g. durability over latency for a ledger) versus nice-to-have.

**PEDALS / RESHADED interview frameworks**
- Fresher: Memorize the checklist (Requirements, Estimation, Storage, High-level design, API, Detailed design, Evaluation, Distributed aspects) and use it to structure an answer.
- Mid: Apply the framework adaptively — skip or compress steps based on what the interviewer/stakeholder actually cares about, rather than reciting all of it.
- Senior/Staff: Use the framework as a scaffold for real design docs and RFCs, not just interviews; know when the framework itself is the wrong tool (e.g. incident postmortems need a different structure).

**Capacity estimation - DAU, QPS, storage back-of-envelope calculations**
- Fresher: Convert DAU to QPS (DAU × actions/day ÷ 86400), estimate storage with average row/object size × count.
- Mid: Model peak-to-average ratios, growth over 1-3 years, and read/write ratios to size caches and replicas correctly.
- Senior/Staff: Use estimation to drive cost projections and capacity planning across multi-year roadmaps, and know when precise estimation matters less than building for elastic scaling.

**Bottleneck analysis and identifying system constraints**
- Fresher: Identify the obvious bottleneck (usually the database) in a simple design.
- Mid: Profile a system to find the *actual* constraint (CPU vs I/O vs network vs lock contention) rather than assuming.
- Senior/Staff: Reason about bottlenecks that only appear at scale (e.g. thundering herd, hot partitions, GC pauses) and design headroom into the architecture proactively.

**API contracts and interface design principles**
- Fresher: Design a clean CRUD API with sensible resource naming and status codes.
- Mid: Design contracts that are backward compatible, version-aware, and resilient to partial failures (idempotency keys, pagination contracts).
- Senior/Staff: Own API contracts as a long-term liability — think about deprecation policy, contract testing, and how a bad contract decision constrains future architecture for years.

**Client-server architecture fundamentals**
- Fresher: Understand request/response cycle, stateless HTTP, client vs server responsibilities.
- Mid: Decide what logic belongs on client vs server (validation, caching, rendering) based on trust boundaries and latency.
- Senior/Staff: Architect for multiple client types (web, mobile, third-party) sharing one backend surface without coupling client release cycles to backend deploys.

**Trade-off decision framework (CAP, latency vs consistency)**
- Fresher: State the CAP theorem and give an example of a CP vs AP system.
- Mid: Apply CAP/PACELC trade-offs to real feature decisions (e.g. "can this counter be eventually consistent?").
- Senior/Staff: Make and defend irreversible trade-off calls with full awareness of downstream cost — e.g. choosing eventual consistency for a payments-adjacent feature and building compensating controls around it.

**Case Study: Bitly URL Shortener**
- Fresher: Design the read/write path, choose an ID generation scheme, sketch the schema.
- Mid: Add caching, handle custom aliases, collision handling, and analytics without blocking the redirect path.
- Senior/Staff: Design for redirect latency SLAs at global scale, abuse/spam prevention, and multi-region consistency of the ID-to-URL mapping.

## Module 2: Scalability Fundamentals

**Vertical vs. horizontal scaling - trade-offs and use cases**
- Fresher: Explain the difference; know vertical scaling hits a hardware ceiling.
- Mid: Decide per-component whether to scale up or out based on statefulness and cost curves.
- Senior/Staff: Architect systems so horizontal scaling is the default path, and know the few components (e.g. a single-writer leader) where vertical scaling is still the pragmatic answer.

**Stateless architecture for horizontal scalability**
- Fresher: Move session state out of app servers into a shared store so any instance can serve any request.
- Mid: Design services so scaling out requires zero coordination — no local caches that create correctness bugs, no in-memory locks.
- Senior/Staff: Enforce statelessness as an architectural invariant across teams; catch subtle state leaks (local file writes, in-process queues) before they become outage triggers.

**Load balancing - round-robin, least connections, IP hash**
- Fresher: Know the three algorithms and when each is used.
- Mid: Choose LB algorithm based on request cost variance (least-connections for uneven workloads) and configure health checks correctly.
- Senior/Staff: Design multi-tier load balancing (L4 + L7, global + regional) and reason about LB failure as a single point of failure.

**Sticky sessions - challenges, failure modes, and alternatives**
- Fresher: Understand what a sticky session is and why it's used.
- Mid: Identify failure modes — uneven load distribution, lost sessions on instance death — and mitigate with shorter TTLs or session replication.
- Senior/Staff: Eliminate sticky sessions where possible by externalizing state (Redis-backed sessions, JWTs) and only accept them as a deliberate trade-off for latency-sensitive stateful protocols (e.g. WebSockets).

**Auto-scaling fundamentals and scaling policies**
- Fresher: Set up a basic CPU-based auto-scaling policy.
- Mid: Tune scaling policies using the right signal (queue depth, latency, custom metrics) and account for scale-up lag time.
- Senior/Staff: Design predictive/scheduled scaling for known traffic patterns and prevent scaling flapping/cost blowouts under adversarial or bursty load.

**Geographic distribution and multi-region deployments**
- Fresher: Understand why latency improves when serving users from a nearby region.
- Mid: Implement active-passive multi-region with failover; handle data replication lag.
- Senior/Staff: Design active-active multi-region architectures with conflict resolution, data residency compliance, and region-failure runbooks.

**Cost vs performance trade-offs**
- Fresher: Recognize that more replicas/instances cost more money.
- Mid: Right-size infrastructure against actual traffic, using reserved/spot capacity where appropriate.
- Senior/Staff: Own the cost-performance curve at the architecture level — decide where to over-provision for reliability vs where lower cost is an acceptable trade for degraded (but bounded) performance.

**Case Study: Dropbox**
- Fresher: Design file upload/download, chunking, and basic metadata storage.
- Mid: Add deduplication (block-level hashing), delta sync, and conflict resolution for concurrent edits.
- Senior/Staff: Design for exabyte-scale storage, cross-datacenter sync consistency, and the block storage/metadata split that lets Dropbox scale storage and metadata independently.

## Module 3: Database Fundamentals

**RDBMS vs. NoSQL - when to choose which**
- Fresher: Know RDBMS gives you joins/transactions, NoSQL gives you flexible schema and horizontal scale.
- Mid: Pick the right store per access pattern (document store for nested data, wide-column for time-series, graph for relationships) rather than defaulting to one.
- Senior/Staff: Design polyglot persistence architectures, own the migration cost of switching stores later, and resist NoSQL hype when strong consistency is actually required.

**ACID properties and BASE model**
- Fresher: Define Atomicity, Consistency, Isolation, Durability and contrast with Basically Available, Soft state, Eventual consistency.
- Mid: Know which isolation level (read committed, repeatable read, serializable) a given feature actually needs, and the anomalies each permits.
- Senior/Staff: Design systems that mix ACID (payments) and BASE (activity feeds) subsystems coherently, with clear boundaries and compensating logic where consistency is relaxed.

**CAP theorem and PACELC theorem**
- Fresher: State CAP: pick 2 of Consistency, Availability, Partition tolerance.
- Mid: Apply PACELC (even without a partition, trade Latency vs Consistency) to real database configuration choices (sync vs async replication).
- Senior/Staff: Make deliberate PACELC trade-offs per data domain and document them so on-call engineers understand what breaks under partition.

**Database indexing strategies - B-tree, hash, composite indexes**
- Fresher: Add an index on a frequently-queried column; understand index speeds reads but slows writes.
- Mid: Design composite indexes matching query predicate order, understand covering indexes, and diagnose missing-index slow queries via EXPLAIN.
- Senior/Staff: Own indexing strategy at scale — index bloat, write amplification, and when to denormalize or use a different index type (GIN, hash, partial) entirely.

**Schema design and query optimization fundamentals**
- Fresher: Normalize a schema to 3NF; write basic joins.
- Mid: Deliberately denormalize for read-heavy paths; optimize N+1 queries and slow joins.
- Senior/Staff: Design schemas that anticipate sharding and multi-tenant isolation from day one, since schema mistakes are the most expensive to fix later.

**Database performance bottlenecks**
- Fresher: Recognize a missing index or a full table scan as a performance issue.
- Mid: Diagnose lock contention, connection exhaustion, and slow query plans under load.
- Senior/Staff: Model bottlenecks before they occur via load testing and capacity planning; design the escape hatch (read replicas, caching, sharding) ahead of the wall being hit.

**Case Study: Local Delivery Service**
- Fresher: Design core tables for orders, drivers, restaurants; basic matching query.
- Mid: Add geo-indexing for driver-order matching, handle order state transitions reliably.
- Senior/Staff: Design for real-time geo-matching at scale, surge pricing consistency, and multi-region delivery zones with strict SLA guarantees.

## Module 4: Database Scaling

**Database sharding - horizontal partitioning strategies**
- Fresher: Understand sharding splits data across multiple database instances by a shard key.
- Mid: Choose shard keys that avoid hotspots, implement range vs hash sharding, and handle cross-shard queries.
- Senior/Staff: Design resharding strategies for growth, own the operational cost of shard rebalancing, and prevent shard-key mistakes that are near-impossible to undo later.

**Consistent hashing for shard routing**
- Fresher: Explain why consistent hashing minimizes remapping when nodes are added/removed vs. modulo hashing.
- Mid: Implement virtual nodes for even load distribution; handle node addition/removal gracefully.
- Senior/Staff: Design hash ring topologies for multi-tenant systems, reasoning about replication factor and quorum alongside the ring.

**Read/write separation - primary replica pattern**
- Fresher: Route writes to primary, reads to replicas.
- Mid: Handle replication lag correctly — read-your-writes consistency, routing logic for freshness-sensitive reads.
- Senior/Staff: Design read/write splitting transparently at the data-access layer, with automatic failover promotion and lag-aware routing.

**Read replicas and scaling read-heavy workloads**
- Fresher: Add a read replica to offload reporting queries.
- Mid: Scale replicas horizontally with a load balancer in front, monitor replication lag as an SLO.
- Senior/Staff: Architect for replica lag spikes under load, cascading replica failure, and multi-region replica topologies.

**Multi-leader replication and conflict resolution**
- Fresher: Understand multi-leader allows writes at multiple nodes, unlike single-leader.
- Mid: Implement conflict resolution strategies (last-write-wins, application-level merge) for concurrent writes.
- Senior/Staff: Design CRDTs or vector-clock-based conflict resolution for systems requiring multi-region write availability without data loss.

**Connection pooling and database optimization**
- Fresher: Use a connection pool instead of opening a new connection per request.
- Mid: Tune pool size against database max connections and application concurrency; diagnose pool exhaustion.
- Senior/Staff: Design connection management across a fleet of stateless services hitting a shared database tier (e.g. PgBouncer/ProxySQL layers) to prevent connection storms.

**Case Study: News Aggregator**
- Fresher: Design schema for articles, sources, and user feeds; basic pagination.
- Mid: Add sharding by source or user, caching for trending articles, read replicas for feed queries.
- Senior/Staff: Design ranking/deduplication pipelines at scale, handle write-heavy ingestion spikes from many sources, and balance freshness vs. read latency.

## Module 5: Caching Fundamentals

**Cache-aside, write-through, write-behind, read-through patterns**
- Fresher: Implement cache-aside (check cache, miss → load from DB → populate cache).
- Mid: Choose the right pattern per workload — write-through for consistency-sensitive data, write-behind for write-heavy paths that tolerate lag.
- Senior/Staff: Combine patterns across a system's data domains and own the consistency/latency trade-off each introduces under partial failure.

**Cache eviction policies - LRU, LFU, FIFO**
- Fresher: Explain LRU evicts least-recently-used items.
- Mid: Choose eviction policy based on access pattern (LFU for stable hot sets, LRU for recency-biased access) and size caches accordingly.
- Senior/Staff: Design tiered eviction (e.g. TinyLFU, ARC) for workloads where naive LRU causes cache pollution from scans.

**Cache stampede / thundering herd - prevention patterns**
- Fresher: Understand what happens when a hot key expires and many requests hit the DB simultaneously.
- Mid: Implement mitigations — request coalescing, locking, probabilistic early expiration.
- Senior/Staff: Design stampede protection as a systemic property (jittered TTLs, background refresh) so no single hot key can take down the origin store.

**Cache invalidation taxonomy (TTL, event-based, tag-based)**
- Fresher: Set a TTL on cached values.
- Mid: Implement event-based invalidation (invalidate on write) and understand its consistency guarantees vs TTL.
- Senior/Staff: Design tag-based invalidation for complex object graphs, and reason about invalidation storms when a single event fans out to millions of cache keys.

**When NOT to cache - anti-patterns & failure modes**
- Fresher: Recognize caching adds complexity and can serve stale data.
- Mid: Avoid caching data with tight consistency requirements or low reuse (cache miss rate too high to be worth it).
- Senior/Staff: Recognize when caching masks an underlying performance problem instead of fixing it, and when a cache becomes a second source of truth that causes production incidents.

**Case Study: Ticketmaster**
- Fresher: Design event/seat listing with basic caching for popular events.
- Mid: Handle seat-locking during checkout, prevent overselling, cache invalidation on purchase.
- Senior/Staff: Design for flash-sale traffic spikes (queueing, waiting rooms), strong consistency for seat inventory despite heavy caching elsewhere, and fairness under contention.

## Module 6: Distributed Caching & CDN Deep Dive

**Redis Cluster - data structures, consistency guarantees**
- Fresher: Use basic Redis data types (string, hash, list, set, sorted set).
- Mid: Deploy Redis Cluster with hash-slot sharding, understand replication and failover.
- Senior/Staff: Reason about Redis Cluster's weak consistency during failover (potential data loss on async replication) and design around it for critical use cases.

**Memcached vs Redis - when each wins**
- Fresher: Know Memcached is simpler/pure cache; Redis has richer data structures and persistence.
- Mid: Choose Memcached for pure high-throughput ephemeral caching, Redis when you need pub/sub, sorted sets, or durability.
- Senior/Staff: Make infra-wide caching tier decisions factoring in operational maturity, memory efficiency, and multi-threading differences (Memcached's multi-threaded architecture vs Redis single-threaded event loop).

**CDN cache-control headers - Cache-Control, ETag, Vary**
- Fresher: Set Cache-Control: max-age on static assets.
- Mid: Use ETag/If-None-Match for revalidation, Vary header to avoid serving wrong variants (e.g. by Accept-Encoding).
- Senior/Staff: Design cache header strategy across an entire site/API to maximize CDN offload while avoiding stale-content incidents, including cache-busting on deploy.

**Edge caching and latency optimization**
- Fresher: Understand edge nodes cache content closer to users.
- Mid: Configure edge caching for dynamic-ish content (personalized fragments, API responses) with short TTLs.
- Senior/Staff: Design edge compute + caching together (e.g. edge-side includes, stale-while-revalidate) to hide origin latency almost entirely.

**Multi-tier caching (L1/L2/L3) & geo-distributed replication**
- Fresher: Understand there can be a local in-process cache plus a shared distributed cache.
- Mid: Design L1 (in-process) + L2 (Redis) tiers with consistent invalidation between them.
- Senior/Staff: Architect L1/L2/L3 (edge/CDN) tiers with geo-replication, accepting bounded staleness at each tier deliberately.

**Cache warming strategies for cold-start problems**
- Fresher: Understand a cold cache causes a burst of DB load after restart/deploy.
- Mid: Pre-populate caches from a snapshot or replay recent traffic before cutting over.
- Senior/Staff: Design zero-downtime cache warming for large fleets (shadow traffic, blue-green cache pools) so deploys never cause a stampede.

**Case Study: Facebook News Feed**
- Fresher: Design a basic feed generation and caching model (fan-out on write).
- Mid: Handle fan-out on read for celebrity/high-follower accounts to avoid write amplification.
- Senior/Staff: Design hybrid fan-out strategies, ranking pipelines, and multi-region feed caching consistent enough to feel real-time at billions-of-users scale.

## Module 7: Messaging Systems

**Synchronous vs asynchronous communication**
- Fresher: Know sync (REST call, wait for response) vs async (publish and move on).
- Mid: Decide per-interaction which model fits based on latency tolerance and coupling requirements.
- Senior/Staff: Architect systems that mix both deliberately, with clear boundaries where sync calls must never be allowed to cascade into async-only domains (and vice versa).

**RabbitMQ / AMQP - routing, exchanges, bindings**
- Fresher: Publish to a queue, consume from it.
- Mid: Use exchanges (direct, topic, fanout) and bindings to route messages to multiple consumers by pattern.
- Senior/Staff: Design RabbitMQ topologies for multi-tenant routing, prefetch tuning, and cluster/mirrored-queue HA trade-offs.

**Apache Kafka - partitions, offsets, consumer groups**
- Fresher: Understand a topic has partitions, consumers track an offset, consumer groups share the load.
- Mid: Choose partition keys for ordering guarantees, handle consumer rebalancing, and size partitions for throughput.
- Senior/Staff: Design Kafka-based architectures for multi-datacenter replication (MirrorMaker), exactly-once semantics via transactions, and partition count as a long-term scaling lever that's expensive to change.

**Delivery guarantees - at-most-once, at-least-once, exactly-once**
- Fresher: Know at-least-once means possible duplicates, at-most-once means possible loss.
- Mid: Implement idempotent consumers to make at-least-once behave like exactly-once in practice.
- Senior/Staff: Design true exactly-once pipelines (transactional outbox + idempotent sinks) where duplicate processing has real business cost (billing, inventory).

**Dead letter queues - handling failed messages**
- Fresher: Route messages that fail processing to a DLQ instead of dropping them.
- Mid: Build DLQ replay tooling and alerting so failed messages don't silently pile up.
- Senior/Staff: Design poison-message detection (max retry with backoff before DLQ) and treat DLQ depth as a first-class operational SLO.

**Kafka vs RabbitMQ decision framework; idempotency patterns**
- Fresher: Know Kafka is a log for high-throughput streaming, RabbitMQ is a traditional broker for task queues.
- Mid: Choose based on replay needs (Kafka retains history) vs. complex routing needs (RabbitMQ's exchange model).
- Senior/Staff: Run both where appropriate in one architecture, and design idempotency keys/deduplication tables that work consistently regardless of which broker is behind a given event.

**Case Study: Tinder**
- Fresher: Design match creation and basic notification on match.
- Mid: Use async messaging for match/notification fan-out to avoid blocking the swipe path.
- Senior/Staff: Design real-time matching pipelines at scale with geo-sharding, low-latency swipe processing, and reliable exactly-once match notification delivery.

## Module 8: Event-Driven Architecture & CQRS

**Event sourcing deep dive**
- Fresher: Understand storing state changes as an append-only log of events instead of current state.
- Mid: Implement event replay to rebuild state, and snapshotting to avoid replaying the entire log.
- Senior/Staff: Design event schemas and stores that support decades of replay, versioned event upcasting, and audit/compliance requirements as a side benefit.

**CQRS - Command Query Responsibility Segregation**
- Fresher: Understand separating write model (commands) from read model (queries).
- Mid: Implement separate read-optimized projections updated asynchronously from the write model.
- Senior/Staff: Design CQRS across service boundaries, reasoning about eventual consistency between write and read sides and how to communicate that to product/UX.

**Saga pattern for distributed transactions**
- Fresher: Understand a saga is a sequence of local transactions with compensating actions on failure.
- Mid: Implement a saga for a multi-step business process (e.g. order → payment → inventory) with compensation logic.
- Senior/Staff: Design saga orchestration/choreography for complex multi-service workflows, including partial-failure recovery and idempotent compensation across retries.

**Choreography vs orchestration in event-driven systems**
- Fresher: Know choreography = services react to each other's events; orchestration = a central coordinator drives the flow.
- Mid: Choose based on process complexity — orchestration for complex/visible workflows, choreography for simple decoupled reactions.
- Senior/Staff: Recognize choreography sprawl (untraceable implicit workflows) as a scaling failure mode and know when to refactor into orchestration.

**Outbox pattern - reliable event publishing**
- Fresher: Understand the dual-write problem (DB write + message publish can fail independently).
- Mid: Implement the transactional outbox pattern — write event to an outbox table in the same DB transaction, publish via a separate relay process.
- Senior/Staff: Design outbox relay infrastructure (polling vs CDC-based) that scales across hundreds of services without becoming a bottleneck itself.

**Event schema evolution; Change Data Capture (CDC) with Debezium**
- Fresher: Add optional fields to an event schema in a backward-compatible way.
- Mid: Use a schema registry with compatibility checks; use Debezium to stream DB changes as events without app-level dual writes.
- Senior/Staff: Own schema governance across an org's event bus, and design CDC pipelines that don't leak internal DB schema as a public contract.

**Case Study: LeetCode**
- Fresher: Design submission flow: code submit → queue → judge → result.
- Mid: Use async event-driven judging with a worker pool, handle timeouts and sandboxing per submission.
- Senior/Staff: Design a globally scaled judge system with fair scheduling across contest traffic spikes, secure sandboxed execution, and real-time leaderboard updates via CQRS-style read projections.

## Module 9: API Architectures - REST, GraphQL & gRPC

**REST - resource modelling, statelessness, Richardson Maturity Model**
- Fresher: Model resources as nouns, use HTTP verbs correctly, build stateless endpoints.
- Mid: Apply HATEOAS where it adds value, understand the maturity levels (0-3) and where most "REST" APIs actually sit (level 2).
- Senior/Staff: Decide when full REST maturity is worth the complexity vs. a pragmatic RPC-over-HTTP style, based on client ecosystem needs.

**API versioning strategies - URL, header, content negotiation**
- Fresher: Version via URL path (/v1/, /v2/).
- Mid: Choose between URL, header, and content-negotiation versioning based on client control and caching implications.
- Senior/Staff: Own a versioning policy and deprecation timeline across dozens of consumers, minimizing breaking changes via additive evolution instead of version bumps.

**GraphQL - schema, resolvers, queries, mutations, subscriptions**
- Fresher: Write a basic schema, query, and resolver.
- Mid: Solve N+1 resolver problems with DataLoader batching; design mutations and subscriptions correctly.
- Senior/Staff: Design GraphQL federation across microservices, enforce query complexity limits to prevent abuse, and decide when GraphQL's flexibility isn't worth its caching/ops complexity.

**gRPC - Protocol Buffers, service definition, streaming**
- Fresher: Define a simple service and message in .proto, generate client/server stubs.
- Mid: Use client/server/bidi streaming appropriately; manage proto schema evolution (field numbering rules).
- Senior/Staff: Architect internal service-to-service communication on gRPC for performance, with deadline propagation, interceptors, and multi-language codegen governance.

**API gateway - routing, auth, rate limiting, transformation**
- Fresher: Route requests through a gateway to backend services.
- Mid: Implement auth, rate limiting, and request/response transformation at the gateway layer.
- Senior/Staff: Design the gateway as the org's API control plane — canary routing, protocol translation (REST↔gRPC), and avoiding it becoming a monolithic bottleneck itself.

**API versioning strategies and best practices**
- Fresher: Avoid breaking changes to a live API; add fields, don't remove them.
- Mid: Run contract tests against consumers before deploying breaking changes.
- Senior/Staff: Establish org-wide API design guidelines and review processes so versioning discipline scales beyond one team's judgment.

**Case Study: WhatsApp**
- Fresher: Design 1:1 messaging with basic send/receive over a persistent connection.
- Mid: Handle message delivery status (sent/delivered/read), offline message queuing, and end-to-end encryption basics.
- Senior/Staff: Design for billions of concurrent connections, multi-device sync, and message ordering/delivery guarantees across unreliable mobile networks at global scale.

## Module 10: Real-Time & Async API Patterns

**WebSockets - bidirectional communication, scaling challenges**
- Fresher: Open a WebSocket connection and send/receive messages.
- Mid: Handle reconnection logic, heartbeats, and horizontal scaling via a shared pub/sub backend (since connections are sticky per server).
- Senior/Staff: Design connection-management infrastructure (gateway servers, connection registries) for millions of concurrent WebSocket connections with graceful failover.

**Server-Sent Events (SSE) - unidirectional streaming**
- Fresher: Stream server-to-client updates over a single HTTP connection.
- Mid: Handle reconnection with Last-Event-ID for resuming streams; know SSE's browser connection limits.
- Senior/Staff: Choose SSE over WebSockets for one-way feeds specifically because it's simpler to scale and debug over standard HTTP infra (proxies, load balancers).

**Long polling - when and why**
- Fresher: Understand long polling holds a request open until new data or timeout.
- Mid: Use long polling as a fallback for clients/networks that don't support WebSockets/SSE.
- Senior/Staff: Decide when long polling's simplicity (works through any HTTP infra) outweighs its resource cost at scale, vs migrating to a push-based model.

**Webhooks - event delivery, retry, signature verification**
- Fresher: Send an HTTP POST to a configured URL when an event occurs.
- Mid: Implement retry with backoff, and HMAC signature verification so receivers can trust payload authenticity.
- Senior/Staff: Design webhook infrastructure with delivery guarantees, dead-lettering, per-tenant rate limiting, and replay tooling for customers who missed events.

**BFF (Backend for Frontend) - tailored APIs per client**
- Fresher: Understand a BFF is an API layer tailored to one client type (web vs mobile).
- Mid: Implement a BFF that aggregates/reshapes calls to backend services for a specific client's needs.
- Senior/Staff: Decide when BFF proliferation becomes its own maintenance burden, and design shared libraries/schemas to avoid duplicated logic across BFFs.

**Fan-out strategies at scale; webhook reliability guarantees**
- Fresher: Understand fan-out means delivering one event to many subscribers.
- Mid: Implement fan-out via a queue per subscriber to isolate slow consumers from fast ones.
- Senior/Staff: Design fan-out systems that scale to millions of subscribers without a slow/dead subscriber degrading delivery to the rest, with SLA-backed reliability guarantees.

**Case Study: Yelp**
- Fresher: Design business search with basic geo + category filtering.
- Mid: Add real-time review/rating updates, ranking signals, and caching for popular searches.
- Senior/Staff: Design geo-indexed search at scale (geohash/quadtree), real-time inventory of business updates via event streams, and personalized ranking pipelines.

## Module 11: Microservices Fundamentals

**Monolith vs microservices - when to break apart**
- Fresher: Know microservices trade simplicity for independent scalability/deployability.
- Mid: Recognize the signals that justify a split (team size, deploy coupling, differing scaling needs) vs premature decomposition.
- Senior/Staff: Make and own the build-vs-split call at the org level, including the cost of distributed systems complexity a split introduces.

**Bounded contexts - domain-driven design basics**
- Fresher: Understand a bounded context is a boundary where a domain model is consistent.
- Mid: Draw service boundaries along bounded contexts rather than technical layers (e.g. not a "database service").
- Senior/Staff: Facilitate cross-team domain modeling (event storming) to define bounded contexts that minimize coupling and align with team ownership (Conway's Law).

**Service communication - synchronous (REST/gRPC) vs async (queue)**
- Fresher: Call another service via REST for a simple request.
- Mid: Choose async messaging when the caller shouldn't block on the callee's availability/latency.
- Senior/Staff: Design communication topology across dozens of services to avoid synchronous call chains that create cascading latency/failure.

**Service discovery - client-side vs server-side**
- Fresher: Understand services need to find each other's network location dynamically.
- Mid: Implement client-side discovery (via a registry like Consul/Eureka) or server-side (via a load balancer).
- Senior/Staff: Design service discovery + health checking as core platform infrastructure that must never become a single point of failure.

**Data ownership boundaries; shared database anti-patterns**
- Fresher: Know each service should own its own data.
- Mid: Refactor a shared-database dependency into an API call or event, even when it's slower short-term.
- Senior/Staff: Own the org-wide policy against shared databases, and design data-sharing patterns (events, API composition) that preserve service autonomy at scale.

**Case Study: Strava**
- Fresher: Design activity upload and basic feed/leaderboard.
- Mid: Split into services (activity ingestion, segments, leaderboards) with async processing of GPS data.
- Senior/Staff: Design segment-matching and leaderboard computation at scale (geo-matching against millions of segments), with eventual consistency between ingestion and leaderboard services.

## Module 12: Advanced Microservices

**Circuit breaker pattern - closed, open, half-open states**
- Fresher: Know a circuit breaker stops calling a failing downstream service after repeated failures.
- Mid: Configure failure thresholds, timeouts, and half-open probing correctly for a given service's traffic pattern.
- Senior/Staff: Design circuit-breaker policy consistently across a service mesh, and reason about cascading breaker trips across dependent services during an incident.

**Retry with exponential backoff and jitter**
- Fresher: Add a retry loop with increasing delay between attempts.
- Mid: Add jitter to avoid synchronized retry storms; cap max retries and total time budget.
- Senior/Staff: Design retry budgets across a call chain so retries at each hop don't multiply into an amplification attack on the origin service.

**Strangler fig pattern - incremental monolith migration**
- Fresher: Understand routing some traffic to a new service while the monolith still handles the rest.
- Mid: Implement the routing layer (proxy/facade) that incrementally shifts traffic as functionality is migrated.
- Senior/Staff: Own a multi-year strangler migration plan, sequencing which capabilities to extract first to minimize risk and maximize independent value delivery.

**Sidecar pattern - logging, monitoring, auth proxies**
- Fresher: Understand a sidecar is a helper container deployed alongside the main service.
- Mid: Use sidecars for cross-cutting concerns (mTLS termination, log shipping) without modifying application code.
- Senior/Staff: Design sidecar-based platform capabilities (service mesh data plane) that every team gets for free, and own the resource/latency overhead trade-off at fleet scale.

**Service mesh - Istio, Linkerd overview**
- Fresher: Know a service mesh handles service-to-service traffic (mTLS, retries, observability) outside app code.
- Mid: Configure traffic policies (canary weights, retries, timeouts) via the mesh control plane.
- Senior/Staff: Decide whether a service mesh's operational complexity is justified for the org's scale, and own mesh upgrades/security posture across hundreds of services.

**Saga pattern; canary releases & feature flags**
- Fresher: Roll out a change to a small percentage of traffic before full rollout.
- Mid: Implement canary analysis (compare error rates/latency between canary and baseline) to automate rollback decisions.
- Senior/Staff: Design progressive delivery infrastructure (canary + feature flags + saga-based rollback) as a unified release safety system across the org.

**Case Study: Design a Distributed Rate Limiter**
- Fresher: Implement a fixed-window or token-bucket limiter for a single server.
- Mid: Make the limiter distributed using Redis with atomic operations (Lua scripts) for correctness across multiple app instances.
- Senior/Staff: Design a rate limiter that scales globally with minimal added latency, handles clock skew, and supports per-tenant/per-endpoint policies without a single Redis instance becoming a bottleneck.

## Module 13: Fault Tolerance & Resilience

**High availability - redundancy, failover, SLA targets**
- Fresher: Know redundancy means no single point of failure; understand what "99.9% uptime" means in downtime minutes.
- Mid: Design automated failover (health checks + leader election) that meets a stated SLA.
- Senior/Staff: Own SLA commitments end-to-end — translate them into architecture decisions, and negotiate realistic SLAs based on what the architecture can actually guarantee.

**Bulkhead pattern - isolating failures between services**
- Fresher: Understand isolating resources (thread pools, connections) per dependency so one slow dependency can't exhaust shared resources.
- Mid: Implement bulkheads (separate connection/thread pools per downstream) in a service with multiple dependencies.
- Senior/Staff: Design fleet-wide resource isolation policies (per-tenant, per-dependency quotas) so a single noisy neighbor can't take down shared infrastructure.

**Timeout strategies - fail fast vs wait and retry**
- Fresher: Set a reasonable timeout on outbound calls instead of waiting indefinitely.
- Mid: Set timeouts based on the callee's actual p99 latency, and propagate deadlines through the call chain.
- Senior/Staff: Design deadline propagation across a distributed call graph so no request holds resources past its useful budget, especially in fan-out scenarios.

**Graceful degradation - fallback responses**
- Fresher: Return a cached/default response when a downstream call fails.
- Mid: Design fallback logic per feature (e.g. show cached recommendations if the live ranking service is down) rather than failing the whole request.
- Senior/Staff: Architect degradation tiers across an entire product so, under partial outage, the highest-value functionality survives while lower-priority features shed load first.

**Disaster recovery planning**
- Fresher: Know backups exist and can be restored.
- Mid: Test restore procedures regularly; define and validate RTO/RPO for critical systems.
- Senior/Staff: Own org-wide DR strategy (multi-region failover runbooks, regular game days) and the cost/complexity trade-off of different RTO/RPO tiers per system criticality.

**Chaos engineering; disaster recovery - RTO and RPO**
- Fresher: Know RTO (time to recover) and RPO (acceptable data loss window) as concepts.
- Mid: Run controlled chaos experiments (kill a pod, inject latency) in staging to validate resilience assumptions.
- Senior/Staff: Run chaos engineering in production safely (with blast-radius controls) as a continuous practice, using findings to drive architecture investment.

**Case Study: Online Auction Platform**
- Fresher: Design bid placement and basic auction-close logic.
- Mid: Handle concurrent bid race conditions, last-second bid extensions, and notification of outcome.
- Senior/Staff: Design for fairness and consistency under extreme end-of-auction bid bursts, with strict ordering guarantees and fault-tolerant auction-close finalization.

## Module 14: Observability & Monitoring

**The three pillars - metrics, logs, traces**
- Fresher: Emit basic metrics (counters, gauges) and structured logs from a service.
- Mid: Correlate metrics, logs, and traces for a single request to diagnose an issue end-to-end.
- Senior/Staff: Design an observability strategy that answers "why" not just "what" — balancing cardinality, cost, and retention across all three pillars org-wide.

**Structured logging - JSON logs, correlation IDs**
- Fresher: Log in JSON with a consistent schema instead of free text.
- Mid: Propagate a correlation/request ID across service boundaries so a single request can be traced through logs.
- Senior/Staff: Own logging standards across the org (PII scrubbing, log levels, sampling) so logs stay useful and compliant at high volume.

**Distributed tracing - OpenTelemetry**
- Fresher: Add a tracing SDK and see a single service's spans.
- Mid: Propagate trace context across service calls (including through async/queue boundaries) to get full end-to-end traces.
- Senior/Staff: Own trace sampling strategy (head-based vs tail-based) to control cost while still catching rare slow/error traces, and instrument the platform so every new service gets tracing for free.

**Prometheus + Grafana - dashboards & alerting**
- Fresher: Build a basic dashboard showing request rate, errors, latency.
- Mid: Write alerting rules with sensible thresholds and reduce alert noise (avoid flapping/duplicate pages).
- Senior/Staff: Design a metrics platform that scales (cardinality limits, federation/remote-write) and build alerting around symptoms (user-facing impact) not causes.

**SLI, SLO, SLA - defining & measuring reliability**
- Fresher: Know SLI is a measured indicator, SLO is an internal target, SLA is an external contractual commitment.
- Mid: Define meaningful SLIs (e.g. "% of requests under 300ms") and set SLOs that reflect real user experience.
- Senior/Staff: Negotiate SLOs across teams so upstream/downstream dependencies compose correctly, and own the process of setting SLAs that the architecture can actually sustain.

**Error budgets; post-mortem process; FinOps basics**
- Fresher: Understand an error budget is the allowed amount of unreliability before an SLO is breached.
- Mid: Use error budget burn rate to decide whether to freeze feature launches and focus on reliability.
- Senior/Staff: Run blameless postmortems that drive systemic fixes, and factor FinOps (cost per request, cost of over-provisioning for reliability) into reliability trade-off decisions.

**Case Study: Facebook Live Comments**
- Fresher: Design real-time comment posting and display for a live video.
- Mid: Handle fan-out of comments to many viewers with acceptable latency, using pub/sub.
- Senior/Staff: Design for viral live-video spikes (millions of concurrent viewers on one stream), with observability to detect and shed load before a hot video takes down the comment pipeline.

## Module 15: Security Architecture

**Defence in depth - layered security model**
- Fresher: Know security should have multiple layers (network, app, data) so no single control failure is fatal.
- Mid: Implement layered controls in a service (input validation + auth + least-privilege DB access).
- Senior/Staff: Design defence-in-depth as an architectural review checklist across the org, ensuring no team relies on a single control as their only defense.

**Zero trust architecture - never trust, always verify**
- Fresher: Understand zero trust means no implicit trust based on network location.
- Mid: Implement service-to-service auth even inside a "trusted" internal network.
- Senior/Staff: Own a zero-trust migration across a legacy perimeter-based network, including identity-based access for every service and human.

**OAuth 2.0 and OIDC - authorisation code flow, PKCE**
- Fresher: Implement login via an OAuth provider using the authorization code flow.
- Mid: Add PKCE for public clients (mobile/SPA), handle token refresh and scopes correctly.
- Senior/Staff: Design an org-wide identity architecture (OIDC provider, token issuance policy, session/refresh token lifecycle) that balances security and UX across many client types.

**API gateway authentication - JWT validation at the edge**
- Fresher: Validate a JWT's signature and expiry before allowing a request through.
- Mid: Implement JWT validation at the gateway so downstream services trust a verified identity header instead of re-validating.
- Senior/Staff: Design key rotation, token revocation strategy (since JWTs can't be easily revoked), and short-lived token + refresh patterns at scale.

**Secrets management - HashiCorp Vault, AWS Secrets Manager**
- Fresher: Avoid hardcoding secrets; use environment variables at minimum.
- Mid: Use a secrets manager with automatic rotation and audit logging.
- Senior/Staff: Design secrets lifecycle management across the org — dynamic short-lived credentials, least-privilege access policies, and incident response for a leaked secret.

**mTLS for service-to-service auth; OWASP API Top 10**
- Fresher: Know mTLS means both client and server present certificates.
- Mid: Deploy mTLS via a service mesh sidecar without app code changes; know the OWASP API Top 10 categories.
- Senior/Staff: Own certificate issuance/rotation infrastructure at fleet scale, and run OWASP API Top 10 as a standing architecture review gate for new services.

**Case Study: Facebook Post Search**
- Fresher: Design a basic search index over post text.
- Mid: Add privacy-aware filtering (only show posts the searcher is authorized to see) and relevance ranking.
- Senior/Staff: Design a search system enforcing complex, dynamic privacy/ACL rules at query time across billions of documents without leaking unauthorized content, while staying fast.

## Module 16: Compliance & Protection

**Web Application Firewall (WAF) concepts**
- Fresher: Know a WAF filters malicious HTTP traffic (SQLi, XSS patterns) before it reaches the app.
- Mid: Configure WAF rules for the app's specific attack surface and tune to avoid false-positive blocking of legit traffic.
- Senior/Staff: Own WAF strategy as one layer of defence-in-depth, integrating it with bot detection and DDoS mitigation at the edge.

**Rate limiting and abuse prevention**
- Fresher: Add per-IP or per-user rate limits on an API.
- Mid: Implement tiered rate limits (per-endpoint, per-tenant) and graceful 429 responses with retry-after headers.
- Senior/Staff: Design abuse-prevention systems combining rate limiting, anomaly detection, and progressive friction (CAPTCHAs, step-up auth) without harming legitimate power users.

**RBAC vs ABAC - attribute-based access control**
- Fresher: Implement role-based access control (admin, user, viewer roles).
- Mid: Recognize RBAC's limits for fine-grained rules and implement ABAC (policies based on user/resource/context attributes) where needed.
- Senior/Staff: Design a unified authorization system (e.g. policy-as-code with OPA) that scales from simple roles to complex attribute-based rules across the org.

**OWASP Top 10 - mapping to architectural decisions**
- Fresher: Know the OWASP Top 10 categories (injection, broken auth, etc.).
- Mid: Map specific architectural decisions (parameterized queries, output encoding) to the relevant OWASP category they mitigate.
- Senior/Staff: Run OWASP Top 10 as a standing architecture and code review lens across the org, and track how new categories (e.g. SSRF, insecure deserialization) get addressed systemically.

**Data residency & compliance (GDPR, SOC 2, HIPAA)**
- Fresher: Know some data must stay in specific geographic regions or be handled specially (PII, health data).
- Mid: Implement data residency (region-pinned storage) and data subject rights (deletion, export) for a service.
- Senior/Staff: Own compliance architecture across the org — audit trails, encryption-at-rest/in-transit policy, and vendor/sub-processor risk for SOC 2 / HIPAA / GDPR obligations.

**Supply chain security - SBOM, dependency scanning**
- Fresher: Run a dependency vulnerability scanner in CI.
- Mid: Generate and maintain a Software Bill of Materials (SBOM); gate builds on critical vulnerability findings.
- Senior/Staff: Own supply-chain security policy org-wide (signed artifacts, provenance attestation, dependency pinning) in response to real-world attacks like compromised packages.

**Case Study: Price Tracking Service**
- Fresher: Design periodic price scraping/polling and storing price history.
- Mid: Add change-detection alerts, rate-limited scraping to avoid abuse/blocking, and deduplication of near-identical prices.
- Senior/Staff: Design a compliant, respectful large-scale scraping/ingestion pipeline (robots.txt adherence, distributed rate limiting per source) with anomaly detection for price-manipulation or scraper-blocking evasion concerns.

## Module 17: Cloud Architecture (AWS + Multi-Cloud)

**Core AWS services - EC2, ECS, Lambda, S3, RDS, ElastiCache**
- Fresher: Deploy an app on EC2, store files in S3, use RDS for a database.
- Mid: Choose the right compute model (EC2 vs ECS vs Lambda) per workload shape, and use ElastiCache for caching.
- Senior/Staff: Architect multi-service AWS solutions optimizing for cost, operational overhead, and blast radius, knowing when managed services trade control for velocity.

**VPC - subnets, security groups, NACLs, NAT gateway**
- Fresher: Launch resources into a VPC with public/private subnets.
- Mid: Design security group and NACL rules following least privilege; use a NAT gateway for private subnet egress.
- Senior/Staff: Design multi-account VPC architectures (hub-and-spoke, transit gateway) balancing network isolation, cost, and cross-account access needs.

**Load balancers - ALB vs NLB (L4 vs L7)**
- Fresher: Put an ALB in front of a web app.
- Mid: Choose NLB for raw TCP/UDP performance or static IPs, ALB for path/host-based routing and WebSocket support.
- Senior/Staff: Design multi-tier load balancing (NLB → ALB → service mesh) for workloads with mixed protocol and performance needs.

**SQS and asynchronous communication patterns**
- Fresher: Send/receive messages via a standard SQS queue.
- Mid: Choose FIFO vs standard queues, implement visibility timeout tuning and DLQ redrive policies.
- Senior/Staff: Design SQS-based architectures at scale, including fan-out via SNS+SQS and cost/throughput trade-offs vs Kafka for the same use case.

**Infrastructure as Code - Terraform fundamentals**
- Fresher: Write basic Terraform to provision a resource.
- Mid: Structure Terraform with modules, remote state, and workspaces for multiple environments.
- Senior/Staff: Own IaC governance across the org — module registries, drift detection, policy-as-code (Sentinel/OPA) gating risky changes.

**Cloud architecture best practices and cost optimization**
- Fresher: Turn off unused resources; pick appropriately sized instances.
- Mid: Use auto-scaling, spot/reserved instances, and right-sizing tools to cut cost without hurting reliability.
- Senior/Staff: Own FinOps practice org-wide — cost allocation tagging, showback/chargeback, and architecture reviews that weigh cost as a first-class NFR.

**Case Study: Instagram**
- Fresher: Design photo upload, storage, and basic feed retrieval.
- Mid: Add CDN-backed media delivery, feed fan-out, and caching for popular content.
- Senior/Staff: Design globally distributed media storage/serving at exabyte scale, with feed ranking, celebrity fan-out handling, and multi-region consistency for likes/comments counters.

## Module 18: Containers & Kubernetes

**Docker - images, layers, multi-stage builds**
- Fresher: Write a Dockerfile, build and run a container.
- Mid: Use multi-stage builds to shrink image size; understand layer caching for faster builds.
- Senior/Staff: Own base-image and build standards org-wide (minimal/distroless images, reproducible builds) for security and cost at fleet scale.

**Kubernetes architecture - nodes, pods, deployments, services**
- Fresher: Deploy a pod via a Deployment, expose it with a Service.
- Mid: Understand scheduling, resource requests/limits, and how Services/Endpoints route traffic to pods.
- Senior/Staff: Design multi-tenant cluster architecture (namespaces, node pools, taints/tolerations) balancing isolation and resource utilization across many teams.

**Kubernetes - ConfigMaps, Secrets, HPA, networking, Helm**
- Fresher: Use a ConfigMap/Secret to inject config into a pod; package a chart with Helm.
- Mid: Configure Horizontal Pod Autoscaler with the right metric; understand cluster networking (CNI, network policies).
- Senior/Staff: Design platform-level Helm chart/library standards and network policy defaults so every team's workloads are secure and scalable by default.

**ECS on AWS - Fargate vs EC2 launch type**
- Fresher: Deploy a container on ECS.
- Mid: Choose Fargate (no server management) vs EC2 launch type (more control/cheaper at scale) per workload.
- Senior/Staff: Own the ECS vs Kubernetes decision org-wide, weighing operational maturity against the flexibility Kubernetes provides.

**GitOps - ArgoCD & Flux for declarative deployments**
- Fresher: Understand GitOps means the Git repo is the source of truth for deployed state.
- Mid: Set up ArgoCD/Flux to sync a cluster's state to a Git repo automatically.
- Senior/Staff: Design multi-cluster, multi-environment GitOps pipelines with progressive sync policies and drift-detection alerting as core platform infrastructure.

**K8s rolling updates, rollback, and container image scanning**
- Fresher: Trigger a rolling update by changing an image tag; roll back on failure.
- Mid: Tune rolling update parameters (maxSurge/maxUnavailable) and integrate image vulnerability scanning into the pipeline.
- Senior/Staff: Design deployment safety nets (automated rollback on SLO breach, mandatory scan gates) as non-bypassable platform policy.

**Case Study: YouTube Top-K System**
- Fresher: Design a basic "most viewed" counter per video with periodic aggregation.
- Mid: Use approximate counting (Count-Min Sketch) and windowed aggregation for trending/top-K at scale.
- Senior/Staff: Design a real-time, globally distributed top-K/trending system handling billions of events/day with bounded error and low staleness, containerized and auto-scaled across regions.

## Module 19: CI/CD, Platform Engineering

**CI/CD pipeline - GitHub Actions - Docker - K8s**
- Fresher: Set up a pipeline that builds, tests, and deploys on push.
- Mid: Add build caching, parallel test stages, and environment promotion (dev → staging → prod).
- Senior/Staff: Design CI/CD as a platform capability — reusable pipeline templates, security scanning gates, and deployment approval workflows used by every team.

**Rolling updates, rollback strategies**
- Fresher: Trigger a deployment and manually roll back on failure.
- Mid: Automate rollback on failed health checks/metrics during a rolling update.
- Senior/Staff: Design automated, metric-driven rollback across the org's deployment platform so a bad deploy self-heals within minutes without human intervention.

**Blue-green and canary deployments**
- Fresher: Understand blue-green swaps traffic between two full environments; canary shifts a small percentage first.
- Mid: Implement canary analysis with automated promotion/rollback based on error rate/latency comparisons.
- Senior/Staff: Own progressive delivery infrastructure org-wide, choosing blue-green vs canary per service based on cost (2x infra) vs risk tolerance.

**Feature flags - LaunchDarkly, AWS AppConfig patterns**
- Fresher: Gate a new feature behind a boolean flag.
- Mid: Use percentage rollouts and targeting rules (by user segment) for gradual exposure.
- Senior/Staff: Own feature-flag governance (flag debt cleanup, kill-switch runbooks for incidents) as a first-class release-safety mechanism across the org.

**Internal developer platforms (IDPs) - Backstage, Port**
- Fresher: Use a service catalog to find ownership/docs for a service.
- Mid: Onboard a new service to the IDP with scaffolding templates and standard CI/CD wiring.
- Senior/Staff: Own the IDP strategy — golden paths, self-service infrastructure provisioning, and measuring developer experience/productivity impact.

**Case Study: Uber**
- Fresher: Design ride request/matching and basic driver location tracking.
- Mid: Add real-time geo-matching, surge pricing calculation, and ETA computation as separate scalable services.
- Senior/Staff: Design a globally distributed dispatch system balancing supply/demand in real time, with resilient deployment pipelines (canary rollouts across regions) given the safety-criticality of the dispatch/pricing logic.

## Module 20: Serverless, Edge Computing & AI-Integrated Systems

**Lambda - serverless for event-driven workloads; cold starts & optimisation**
- Fresher: Write a Lambda triggered by an event (S3 upload, API Gateway request).
- Mid: Optimize cold starts (runtime choice, package size, provisioned concurrency) for latency-sensitive paths.
- Senior/Staff: Decide where serverless fits the org's workload mix vs containers, factoring in cost-at-scale (Lambda gets expensive at high sustained throughput) and vendor lock-in.

**S3 - presigned URLs, versioning, lifecycle policies**
- Fresher: Upload/download objects to S3; generate a presigned URL for temporary client access.
- Mid: Configure versioning and lifecycle policies (transition to cheaper storage tiers, expiration) for cost management.
- Senior/Staff: Design S3-based architectures for compliance (object lock, cross-region replication) and cost at exabyte scale, including intelligent tiering strategy.

**RDS - Multi-AZ, read replicas, automated backups**
- Fresher: Enable automated backups and point-in-time recovery on RDS.
- Mid: Configure Multi-AZ for automatic failover and read replicas for read scaling.
- Senior/Staff: Design RDS architecture for cross-region DR, and know when RDS's limits (vertical scaling ceiling) mean it's time to move to a sharded/managed distributed database.

**Edge computing - Cloudflare Workers, Lambda@Edge**
- Fresher: Run a simple request-transform function at the edge.
- Mid: Move latency-sensitive logic (auth checks, A/B routing, personalization) to the edge to cut round-trip time.
- Senior/Staff: Architect edge-first systems where the edge runtime's constraints (limited CPU/memory, no persistent state) shape the whole request-handling design.

**AI-integrated system design - model serving, inference at scale**
- Fresher: Call a hosted LLM/model API from a backend service.
- Mid: Design model-serving infrastructure with batching, autoscaling GPU pools, and latency/cost trade-offs (model size vs quality).
- Senior/Staff: Architect multi-model inference platforms with request routing, fallback models, and cost governance across many product teams consuming shared inference infrastructure.

**RAG architecture - vector stores, embedding pipelines, LLM APIs**
- Fresher: Embed documents, store in a vector DB, retrieve top-k for a query, pass to an LLM prompt.
- Mid: Design chunking/embedding pipelines that keep the index fresh, and tune retrieval (hybrid search, reranking) for relevance.
- Senior/Staff: Architect RAG systems at scale — incremental reindexing, multi-tenant data isolation in the vector store, and evaluation pipelines to catch retrieval/generation regressions.

**Case Study: Robinhood**
- Fresher: Design order placement and basic portfolio/balance display.
- Mid: Handle real-time price streaming, order matching/execution status updates, and idempotent order processing.
- Senior/Staff: Design a financial-grade system with strict consistency for balances/orders, real-time market data fan-out at scale, and regulatory-compliant audit trails for every trade.

## Module 21: URL Shortener - Backend & Infrastructure

**Design a URL shortening service - API design, redirection flow, URL mapping**
- Fresher: Build a POST /shorten and GET /{code} redirect endpoint backed by a key-value mapping.
- Mid: Handle custom aliases, expiration, and 301 vs 302 redirect trade-offs (caching implications).
- Senior/Staff: Design the API contract to support future needs (analytics, bulk creation, API-key tenants) without breaking existing clients.

**Implement unique ID generation strategies - auto-increment, hashing, Snowflake IDs**
- Fresher: Use an auto-incrementing DB ID encoded to base62.
- Mid: Compare hashing (MD5/hash of URL, truncated) vs counter-based vs Snowflake ID generation for collision risk and distributed generation needs.
- Senior/Staff: Design a globally distributed, collision-free ID generation service that doesn't become a single point of contention as write volume grows.

**Database design using PostgreSQL - schema design and optimization**
- Fresher: Create a table with short_code, long_url, created_at, indexed on short_code.
- Mid: Add expiration/TTL handling, click-count tracking without hurting redirect latency, and connection pooling.
- Senior/Staff: Design the schema to support sharding by short_code hash as volume grows, and separate hot redirect-path data from analytics data.

**Caching architecture using Redis - reducing latency and database load**
- Fresher: Cache short_code → long_url lookups in Redis with a TTL.
- Mid: Implement cache-aside with stampede protection for popular links; warm cache for known-hot codes.
- Senior/Staff: Design multi-region cache replication so redirect latency stays low globally without every region hitting a central database.

**Containerization with Docker - packaging and deployment readiness**
- Fresher: Write a Dockerfile for the service and run it locally.
- Mid: Optimize the image (multi-stage build, non-root user) and add health check endpoints.
- Senior/Staff: Own container build/security standards (base image policy, scanning) applied consistently across services in the org.

**Deploy the application on AWS infrastructure**
- Fresher: Deploy the container to a single EC2 instance or basic ECS service.
- Mid: Deploy behind an ALB with auto-scaling and RDS/ElastiCache managed services.
- Senior/Staff: Design the full production topology (VPC, multi-AZ, IaC via Terraform) as a repeatable, reviewable deployment, ready for the productionization work in Module 22.

## Module 22: Productionizing URL Shortener

**Load balancing and traffic distribution strategies**
- Fresher: Put an ALB in front of multiple app instances.
- Mid: Tune health checks and connection draining so deploys don't drop in-flight redirects.
- Senior/Staff: Design multi-region load balancing (Route53/global accelerator) routing users to the nearest healthy region.

**Auto-scaling architecture for handling peak traffic**
- Fresher: Set a target-tracking auto-scaling policy on CPU.
- Mid: Scale on request count/queue depth, and pre-warm capacity ahead of known traffic spikes (marketing campaigns).
- Senior/Staff: Design predictive scaling and load-shedding so a viral link spike degrades gracefully instead of taking the service down.

**CDN integration for global content delivery**
- Fresher: Cache redirect responses at the CDN edge with an appropriate TTL.
- Mid: Balance CDN caching against the need for fresh click-analytics and expiration correctness.
- Senior/Staff: Design edge-based redirect resolution (e.g. via Lambda@Edge/Workers) so most redirects never hit origin at all.

**Monitoring and observability using industry-standard tools**
- Fresher: Set up basic dashboards for request rate, latency, error rate.
- Mid: Add distributed tracing across the redirect path and alerting on SLO burn.
- Senior/Staff: Define SLOs for redirect latency/availability and build the error-budget process around them.

**CI/CD pipeline implementation for automated deployments**
- Fresher: Auto-deploy on merge to main via GitHub Actions.
- Mid: Add canary deployment with automated rollback on error-rate regression.
- Senior/Staff: Own the deployment safety system (progressive delivery, feature flags) so releases never risk redirect availability.

**Rate limiting, logging, and production readiness practices**
- Fresher: Add per-IP rate limiting on the shorten endpoint to prevent abuse.
- Mid: Add structured logging with correlation IDs and abuse detection (spam link creation, malicious URL scanning).
- Senior/Staff: Own a production-readiness checklist (rate limiting, WAF, DR plan, runbooks) applied consistently before any new service ships.

## Module 23: Real-Time Chat System

**Design a chat application - WebSockets, message delivery, and persistence**
- Fresher: Build a WebSocket server that relays messages between two connected clients and stores them in a DB.
- Mid: Handle delivery acknowledgement (sent/delivered/read), message ordering, and reconnection with message replay.
- Senior/Staff: Design guaranteed message delivery semantics across unreliable networks, with ordering guarantees per conversation even under server failover.

**User presence and connection management architecture**
- Fresher: Track online/offline status in a simple in-memory map per server.
- Mid: Externalize presence to a shared store (Redis) so status is consistent across multiple chat servers.
- Senior/Staff: Design presence at scale (heartbeat-based TTL expiry, fan-out of presence changes to relevant contacts only) without it becoming a write-amplification problem.

**Message storage and retrieval strategies**
- Fresher: Store messages in a table keyed by conversation ID with pagination.
- Mid: Design for efficient recent-message retrieval (e.g. time-bucketed partitions) and handle large group conversations.
- Senior/Staff: Design storage that scales to billions of messages/day, balancing hot recent-data access against cold long-tail history, with data retention/compliance policies.

**Kafka-based asynchronous message fan-out**
- Fresher: Understand why writing to N recipients synchronously doesn't scale for group chats.
- Mid: Use Kafka to decouple message ingestion from fan-out to recipient inboxes/notification services.
- Senior/Staff: Design fan-out architecture that handles both small 1:1 chats and huge broadcast/group scenarios without one path starving the other.

**Redis Pub/Sub for real-time communication workflows**
- Fresher: Use Redis Pub/Sub to relay a message from the server that received it to the server holding the recipient's WebSocket connection.
- Mid: Understand Redis Pub/Sub's at-most-once delivery (no persistence) and design around message loss on subscriber disconnect.
- Senior/Staff: Decide when Pub/Sub's simplicity is enough vs when a durable log (Kafka) is required for correctness, and design a hybrid where presence/typing-indicators use Pub/Sub but messages use a durable path.

**Cloud deployment and distributed system considerations**
- Fresher: Deploy chat servers behind a load balancer with sticky sessions for WebSocket affinity.
- Mid: Design a connection-routing layer so any server can reach any user regardless of which server holds their socket.
- Senior/Staff: Architect the full distributed chat backend (gateway servers, presence service, message store, fan-out pipeline) as independently scalable components, ready for the multi-region scaling covered in Module 24.

## Module 24: Scaling Chat Systems for Production

**Multi-region deployment architecture and traffic routing**
- Fresher: Understand why users should connect to the nearest region for lowest latency.
- Mid: Implement geo-DNS/anycast routing to the nearest region with cross-region message delivery for conversations spanning regions.
- Senior/Staff: Design active-active multi-region chat infrastructure with conflict-free message ordering across regions and region-failure failover without losing in-flight messages.

**Presence scaling and session management strategies**
- Fresher: Scale the presence store (Redis) as a single cluster.
- Mid: Shard presence data by user ID and batch presence-change notifications to reduce fan-out cost.
- Senior/Staff: Design presence systems for hundreds of millions of concurrent users, with regional presence stores synced globally at bounded staleness.

**Sticky sessions and real-time workload optimization**
- Fresher: Use sticky sessions so a user's WebSocket always reconnects to a consistent server.
- Mid: Combine sticky sessions with a connection registry so messages can still be routed correctly if a user reconnects to a different server.
- Senior/Staff: Design connection-management infrastructure that minimizes reliance on stickiness altogether, using a routing layer that decouples message delivery from any specific server holding the socket.

**File uploads and cloud storage integration using Amazon S3**
- Fresher: Upload chat attachments directly to S3 via a presigned URL from the client.
- Mid: Add virus scanning, thumbnail generation (async pipeline), and access-controlled retrieval URLs.
- Senior/Staff: Design media pipelines that handle huge attachment volume without blocking message delivery, with lifecycle policies and CDN-backed retrieval at global scale.

**Notification pipelines and event-driven processing**
- Fresher: Send a push notification when a message arrives and the recipient is offline.
- Mid: Batch/debounce notifications to avoid spamming a user with many messages in quick succession; integrate with APNs/FCM.
- Senior/Staff: Design a notification platform shared across features (chat, mentions, activity) with per-user preference rules and delivery-guarantee SLAs.

**Monitoring, chaos testing, and production readiness review**
- Fresher: Monitor connection count, message latency, and error rates per server.
- Mid: Run chaos tests (kill a chat server, partition Redis) to validate failover doesn't drop messages.
- Senior/Staff: Own the production-readiness review for the entire chat platform — SLOs, DR runbooks, and chaos-engineering cadence — as the closing capstone across everything from Module 1 through Module 24.
