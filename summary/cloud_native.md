## Cloud Native: Using Containers, Functions, and Data to Build Next-Generation Applications (Boris Scholl, Trent Swanson, Peter Jausovec)

### Chapter 1: Introduction to Cloud Native

#### Fallacies of Distributed Systems
* The network is reliable
* Latency is zero
* Bandwidth is infinite
* The network is secure
* Topology doesn't change
* There is one administrator
* Transport cost is zero
* The network is homogeneous

#### CAP Theorem
The CAP theorem states that it is impossible for a distributed data store to simultaneously provide more than two out of the following three guarantees:
* Consistency: equivalent to having a single up-to-date copy of the data
* Availability: high availability of the data (for updates)
* Partition tolerance: tolerance to network partitions

#### The Twelve-Factor App
See [12factor.net](https://12factor.net/)

#### Availability and Service-Level Agreements
The compound SLA will never be as high as the highest availability of an individual service.


### Chapter 2: Fundamentals

#### Containers
* Containers are a way to package an application and its dependencies into a single unit of service
* Encapsulated, individually deployable components running as isolated instances on the same kernel with virtualization happening on the OS level
* The required isolation is accomplished by using kernel features such as namespaces and cgroups (Linux features)
* Containers are fast at starting up and shutting down - they are good for scale-out scenarios.

* Downsides of using VMs:
    * VMs are slow to start up and shut down
    * VMs are heavy and require a lot of resources
    * VMs are not as portable as containers
    * VMs are not as easy to scale as containers

* We need a container orchestrator to manage the life cycle of containers at scale
* Tasks for a container orchestrator:
    * Scheduling
    * Scaling
    * Health monitoring
    * Load balancing
    * Rolling updates
    * Service discovery
    * Logging
    * Monitoring
    * Security
    * Networking
    * Storage

#### Functions
* It's a unit of work that is triggered by an event

#### Microservices
* Benefits of microservices:
    * Agility
    * Continuous innovation
    * Evolutionary design
    * Small, focused teams
    * Fault isolation
    * Improved scale and resource usage
    * Improved observability

* Challenges with a Microservices Architecture:
    * Complexity
    * Data integrity and consistency
    * Performance
    * Development and testing
    * Versioning and integration
    * Monitoring and logging
    * Service dependency management
    * Availability

### Chapter 3: Designing Cloud Native Applications
* What to consider choosing a cloud provider:
    * Operational excellence (Build, measure, learn):
  * Principles:
      * Automate everything
      * Monitor everything
      * Document everything
      * Make incremental changes
      * Design for failure
  * Security considerations (Kubernetes):
    * Source code - check for vulnerabilities as part of the CI/CD pipeline
    * Container images - add only the required packages to the image
    * Container registry - use a private registry
    * Pod - use a pod security policy
    * Cluster and orchestrator - network policies, RBAC, and audit logging
  * Reliability and Availability
    * Reliability - the ability of a system to recover from failures
    * Availability - the app is available for certain amount of time
    * Consider retries and circuit breakers
  * Scalability and Cost 

#### Cloud Native vs. Traditional Architectures
* Cloud native apps are stateless by nature
* State is usually externalized
* Cloud native apps often use event-driven patterns for communication
* They expect failures and are designed to handle them

#### API Design and Versioning
* Three strategies for API versioning:
    * The knot - when the API changes, all clients must be updated at the same time
    * Point-to-point - all API versions are supported at the same time
    * Compatible versioning - all clients use the same version. Current version is backwards compatible with previous versions
* Compatible versioning is the most effective strategy for API versioning
* Three approaches to API versioning:
    * global versioning - /api/v1, or subdomain versioning - v1.api.example.com
    * resource versioning - /api/v2/users (other resources are still on v1)
    * mime-based approach (version in headers) - application/vnd.company.app-v1+json

#### API Backward and Forward Compatibility
* Best practices:
  * Provide defaults or optional values for new APIs. If not possible, use a new endpoint
  * Never rename existing fields or remove them
  * Never make optional things required
  * Mark old endpoints as obsolete if not used anymore
  * Test the combination of old and new API versions by passing old messages between them

#### Service Communication
* External (North-South) vs. Internal (East-West) Communication
* Some popular protocols for internal communication:
    * WebSockets
    * HTTP/2 (binary)
    * gRPC
* Most popular protocols for messaging:
    * AMQP
    * MQTT
* Keep in mind serialization and deserialization costs when choosing a protocol
* Idempotency - make sure that a request can be repeated without side effects (unique IDs, timestamps, etc.)
* Request/Response - can be synchronous or asynchronous, use CID for correlation
* Publish/Subscribe:
  * Enable loose coupling between services
  * Enable event-driven design
* Ensure message ordering when using publish/subscribe, if needed
* Put the message into poison queue if it can't be processed

* Synchronous vs. Asynchronous
    * Synchronous concerns:
        * Exhaustion of resources
        * Response latency
        * Cascading failures
    * Asynchronous: event and queue-based asynchronous messaging - most popular patterns for IPC in cloud native apps

#### Gateways
* A gateway is a single entry point for all external requests
* Functions of gateways:
  * Routing (based on URL, headers, etc. to backend services)
  * Aggregation (takes one request from the client and makes multiple requests to backend services)
  * Offloading (offload some of the work to the gateway: authentication, rate limiting, retries, circuit breakers, caching, compression, logging, monitoring, etc.)
* Popular tools:
  * NGINX
  * Envoy
  * HAProxy
* Egress Gateways - a single entry point for all outbound requests

#### Service Mesh
* A service mesh is a dedicated infrastructure layer for handling service-to-service communication
* The main building block of a service mesh is a sidecar proxy
* Or one proxy per host (we can use DaemonSet in Kubernetes)
* Groups of main features of a service mesh:
  * Traffic management
  * Failure handling
  * Security
  * Tracing and monitoring

### Chapter 4: Working with Data
* trend - separate datastore for each service
* Cloud natiive apps characteristics for data:
  * prefer managed data storages and analytics services
  * use polyglot persistence, data partitioning, and caching
  * embrace eventual consistency and use strong consistency only when needed
  * prefer cloud native databases that scale out, tolerate failures, and optimized for cloud storage
  * deal with data distributed across multiple datastores

#### Data Storage Systems
##### Objects, Files, and Disks
* Prefer object storage for storing file data
* Object/blob storage:
  * use it with files whe apps support cloud provider API
  * It is inexpensive and can store large amounts of data

* File storage:
  * use it when apps designed to support Network Attached Storage (NAS)
  * Use it when shares access to files is required
  * it is more expensive than object storage

* Disk(block) storage:
  * use it for apps that assume persistent local storage disks, like MongoDB or MySQL

##### Databases
* Key/value
  * can be viewed as a very large hash table
  * generally inexpensive and very scalable
  * capable or partitioning
* Document
  * need to conform to some defined structure
  * can be queried using SQL-like language
  * generally don't enforce schema (schema on read)
* Relational
  * most popular type of database
  * enforce schema (schema on write)
  * support ACID transactions
  * support SQL
  * not very scalable
* Graph
  * store data in nodes and edges
  * work well at analyzing relationships between entities
* Column family
* Time series
  * optimized for storing time series data
  * can be used for storing metrics, logs, and events
  * commonly used to collect lagre amounts of data in real time from large number of sources
  * record are small but the number of records is large
* Search
  * optimized for searching text
  * can be used for full-text search, autocomplete, and faceted search
  * can be used for analytics
  * near-real-time indexing

##### Streams and Queues
* store events and messages
* in an event stream, data is stored as immutable stream of events
* messaging queues - mutable messages, it's possible to remove messages from the queue

##### Blockchain
* a distributed ledger that stores transactions in a chain of blocks
* each block contains a hash of the previous block
* each block is cryptographically signed

##### Selecting a Datastore
* Functional requirements:
  * Data format
  * Read/write ratio
  * Data size
  * Scale and structure (capacity, need of partitioning)
  * Data relationships (need of complex relationships?)
  * Consistency model (strong or eventual consistency?)
  * Schema flexibility
  * Concurrency (need of pessimistic or optimistic locking?)
  * Data movement (need to move data between datastores?)
  * Data life cycle
  * Change streams (need of CDC?)
  * Other supported features (need of specific features?)
* Nonfunctional requirements:
  * Team experience
  * Support
  * Performance and scalability
  * Reliability (backup and restore features)
  * Replication
  * Limits (on size and scale)
  * Portability
* Management and cost:
  * Managed service (use when possible)
  * Region or cloud provider availability
  * Licensing
  * Overall cost
Track of database popularity: https://db-engines.com/

#### Data in Multiple Datastores
* Challenges of distributed data:
  * Data consistency across datastores
  * Analysis of data in multiple datastores
  * Backup and restore of the datastores

##### Change Data Capture (CDC)
* Many databeses offer stream of data change events (change log) with easy-to-consume API.
* Common use cases:
  * Data replication
  * Notifications
  * Materialized views
  * Cache invalidation
  * Auditing
  * Search
  * Analytics
  * Change analytics (near-real-time analytics of data changes)
  * Legacy data integration
* Concern - either record the event that something changed when it didn't or miss the event when something did change
* By using DBs, we can write change or mutation and the log of changes using transaction

##### Write Changes as an Event to a Change log
* Write changes to a change log as an event and then read the change log and apply changes to the other datastores

##### Transaction Supervisor
* Use supervisor service to ensure that a transaction is successfully completed or is compensated
* Useful with external services (eg. writing an order to the system and processing payment)
* To solve - save the order or cart with status of processing, then make the call to the payment service, and then update the status of the order or cart
* Could be a simple function that's triggered at a specific time interval

##### Compensating Transactions
* A logical set of operations need to complete
* If one fails, the others need to be rolled back
* Example: save the file and update status in the database

##### Extract, Transform, and Load (ETL) page 89