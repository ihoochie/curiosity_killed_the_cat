## Release It!: Design and Deploy Production-Ready Software (Michael T. Nygard)

### Chapter 1: Living in Production
* Unplanned things will happen anyway
* Design for manufacturability analogy (Design for production)
* We need to design software to operate at low cost and high quality
* Two types of architects: "ivory tower" and pragmatic
* Software delivers its value in production.

### Chapter 2: 
* A better question to ask: "How do we prevent bugs in one system from affecting everything else?"

### Chapter 3: Stabilize Your System
* The high stable design usually costs the same to implement as the unstable one
* A transaction is an abstract unit of work processed by the system (not the same as a database transaction)
* Transactions are the reason that the system exists
* One type transaction - a dedicated system
* Mixed workload - combination of different types of transactions
* A robust system keeps processing transactions even when transient impulses, persistent stresses, or component failures occur
* Impulse - a short-lived disturbance, stress - force that acts over a long period of time
* The major dangers to the system's longevity are memory leaks and data growth
* Applications never run long enough in the development environment to reveal their longevity bugs
* The only way to catch them is ro run longevity tests
* "Cracks in the system" - they propagate through the system
* Trigger + the way it spreads through the system + result = **_failure mode_**
* Like _crumple zones_ in cars, the system should have a few places where it can absorb the impact of a failure
* The more tightly coupled the architecture, the greater the chance the coding error can propagate.
* The combination of events that caused the failure is not independent - because layers are coupled
* Triggering a fault opens the crack, _faults_ become _errors_, and errors provoke _failures_
* One way to prepare is to look at every external call, I/O, use of resources, and so on, and ask, "What happens if this fails?"
* Faults will happen: they can never be completely prevented. And we must keep faults from becoming errors.

### Chapter 4: Stability Antipatterns

#### Integration Points
* Two main (anti)patterns: butterfly and spider
* Butterfly - a system that has a single point of integration with the outside world
* Spider - a system that has many points of integration with the outside world
* Integration points - number-one killer of systems
* The most effective pattens to defend against integration points are Circuit Breakers and Decoupling Middleware
* Cynical software should handle violations of form and function. Use tests
* Use automated tests to make considerable load on the system
* Every integration point will fail at some point
* Prepare for the many forms of failure - most likely, you will not get nice error responses
* Failures propagate quickly through the system, usually in a cascading fashion
* Patterns to avert integration points problems:
  * Circuit Breakers
  * Decoupling Middleware
  * Timeouts
  * Handshaking

1. **Socket-Based Protocols**
* Set the socket time-out to break out of the blocking call
* Fast and slow network failures
* The listen queue of the port - the worst place to be
* Packet capture tools - tcpdump, Wireshark
* Firewalls can drop packets "on the floor"
* Main lesson - not every problem can be solved at the level of abstraction where it occurs

2. **HTTP Protocols**
* Examples of what can go wrong:
  * The provider accepts the TCP connection, but never sends an HTTP response
  * The provider accepts the connection but not reads the request
  * Sends response status or content type the caller doesn't understand
  * Claims to be sending JSON but sends HTML/plain text
* Use client libraries with control over timeouts - connection and read
* Treat a response as data until you've confirmed it meets your expectations

3. Vendor API Libraries
* Usually, the vendor provides a libraries that have a lot of problems and often with stability risk. They are not as stable as the main product.
* The main stability killer with vendor API libraries is about blocking

#### Chain Reactions
* Chain reactions are a type of failure mode when an application has some defect - usually a resource leak or a load-related crash.
* Other nodes pick up the slack, but the load on each node increases, so they are more likely to crash.
* Other dependant layers must protect themselves
* Most of the time a chain reaction is caused by a memory leak
* Race conditions can also cause chain reactions
* Use autoscaling, but scalers should react faster than the chain reaction propagates
* To prevent: Bulkheads - for server side, Circuit Breakers - for caller side

#### Cascading Failures
* Cascading Failures - when a crack in one layer triggers a crack in a calling layer
* Crucial services with high demand are worth extra scrutiny
* CF requires some mechanism to transmit the failure to the caller ("jump the gap") - bad behavior in the caller is triggered by th failure in the provider
* CF is number-one accelerator of failure propagation
* Make sure the system can stay up when the provider is down
* Resource pools often cause cascading failures - when all the threads get blocked waiting for connections
* Circuit Breakers - protects the system by avoiding calls to the troubled integration points.
* Timeouts ensure that you can come back from a call to the troubled point

#### Users

* Users - the reason systems exist, but the can be the source of trouble

1. Traffic
* Sooner or later, the traffic growth will exceed the system's capacity
* So how would the system react?
* Capacity - the max throughput of the system under a given workload with acceptable performance
* Autoscaling is a friend, but buggy apps would cost you a lot of money

2. Heap Memory
* One of the hard limits - memory available (especially in interpreted languages)
* Sessions occupy memory (they live for a while after last request)
* It's a bad idea to keep big search results in memory, it's better to ask the search engine to do it
* User Weak References - to help Garbage Collector to free memory
* Since GC can remove objects, callers should be prepared for nulls

3. Off-Heap Memory, Off-Host Memory
* Another effective way - store data in different process (Memcached, Redis) - we can put it on a different machine
* It's a trade-off between total addressable memory size and latency to access it

4. Sockets
* One host can serve up to 64511 connections. How to handle more?
* Use virtual IP addresses - we need about 16 IP addresses to handle 1 million connections
* Learn your operating system's TCP tuning parameters

5. Closed Sockets
* After closing a socket, the OS will keep it in the TIME_WAIT state for a while (waiting for Bogots)
* We can turn the TIME_WAIT interaval down to get ports back faster

6. Expensive to Serve
* Some users are more demanding than others - usually those who want to buy something
* The best thing to do - to test aggressively most expensive transactions
* But not test against 100% of workload - it's too expensive
* Instead, double/triple the load on the most expensive transactions

7. Unwanted Users
* Sessions are the Achilles' heel of web applications
* But this is an abstractions that makes building web applications easier
* Sessions are about caching data in memory.
* We need some way to identify a request as part of the same session
* Competitive intelligence - industry that monitors the activities of competitors. They produce parasitic traffic (crawler bots)
* robots.txt - a file that tells search engines which pages not to index
* Bad robots don't respect robots.txt
* Two ways to deal with bad robots:
  * Technical - robots.txt, CAPTCHA, rate limiting, ARIN
  * Legal
* To test random actions - fuzzing toolkits, property-based testing, simulation testing

8. Malicious Users
* Truly talented crackers are rare
* The majority is "script kiddies"
* The primary risk - Distributed Denial of Service (DDoS) attacks
* Nearly all attacks vector in against the applications rather than the network
* Network vendors have products that detect and mitigate DDoS attacks

#### Blocked Threads
* Interpreted languages - the interpreter can be running, and the app can be totally deadlocked
* The most common failure mode - navel-gazing
* We need external monitoring tools - log file scraping, process monitoring, network monitoring, metrics
* Counters like "successful logins" and "failed credit cards" will show problems quickly, before alerts are triggered
* Blocked thread can happen anytime, but the problems are:
  * Error conditions and exceptions create too many permutations to test
  * Unexpected interactions can introduce problems in previously safe code
  * Timing is crucial. The probability of app hang goes up with the number of concurrent requests
  * Developers never hit their application with 10000 concurrent requests
* If possible don't create your own connection pools - use a library, they are usually better
* Don't use synchronized methods:
  * Application will break when running on multiple machines (for data integrity)
  * The app will scale better if request-handling threads never block each other
* Use Command Query Responsibility Segregation (CQRS) - separate the command and query paths
* Beware the code you cannot see

1. Use Caching Carefully
* Keeping something in cache - the cost of generating once + caching and lookups is less than the cost of generating every time
* It's wise to avoid caching things that are cheap to generate
* Every cache should have invalidation strategy

2. Libraries
* Libraries are notorious sources of blocking threads
* If you use vendore code, it may be worth some time beating them up for a better client library
* A blocking thread is often found near an integration point

#### Self-Denial Attacks
* A self-denial attack is situations in which the system - or the extended system that include humans conspires against itself

1. Avoiding Self-Denial
* Shared nothing architecture - the system is built from independent components that don't share state
* Decoupling middleware
* Make shared resource horizontally scalable
* Pre-autoscale before the marketing event goes out
* Keep the lines of communication open in the organization

#### Scaling Effects
* When scaling one part of the system, we need to consider the impact on other parts of the system

1. Point-to-Point Communications
2. PtP Communications scale badly
* The total number of connections goes up as square of the number of instances
* We can replace point-to-point communications with:
  * UDP broadcast
  * TCP or UDP multicast
  * Publish/subscribe messaging
  * Message queues

2. Shared Resources
* Shared resources are the most common scaling bottleneck
* When the SR is redundant, we can scale it horizontally (scaling the bottleneck)
* The most scaleble architecture is shared nothing

#### Unbalanced Capacity
* It's mismatched ratios between different layers
* Circuit Breakers will help by relieving the pressure on downstream services

1. Drive Out Though Testing
* UC is rarely observed during QA
* Be ready for anything:
  * Use capacity modeling
  * Don't test your system only with usual load
  * Use autoscaling to react to surging demand (but impose financial constraints)
* Stress both side of the interface in testing

#### Dogpile
* Can occur when:
  * booting up several servers, after a code upgrade and restart
  * a cron job triggers at midnight or every hour
  * the configuration management system pushes out a change
* It forces the system to spend too much to handle peak demand
* Use random clock slew to diffuse the demand
* Use increasing backoff times to avoid pulsing

#### Force Multiplier
* A force multiplier is a system that amplifies the effect of a small change

1. Outage Amplification
* Control plane can cause a problem if it has incorrect state

2. Controls and Safeguards
Examples:
* If observations report that more than 80% of the system is unavailable - more likely the broblem is in observer
* Start machines quickly, but shut them down slowly
* When the gap between expected state and observed state is large, ask for confirmation
* Build in deceleration zones - control plane could ask every second, but it take a few minutes to start up new resources

#### Slow Responses
* Generating a slow response is worse than refusing a connection or return an error
* SR tend to propagate through the system upwards (Cascade Failure)
* Give your system the ability to monitor its own performance (it helps to  meet SLAs). Consider Fail Fast

#### Unbound Result Sets
* Design with scepticism, and you will achieve resilience
* Ask "What can system X do to hurt me?"
* URS are a common cause of slow responses
* Use realistic data volumes in testing
* Paginate at the front end
* Don't rely on the data producers
* Put limits into other application-level protocols
* Just make sure your system can handle accidental huge collections of data

### Chapter 5: Stability Patterns

#### Timeouts
* Networks are fallible
* Resource pools and thread blocks should have timeouts to ensure that calling threads will be unblocked
* Group sets of operations into query objects (see EAA patterns)
* Use generic gateway to provide template for connection handling, error handling, query execution and result processing (one place setup)
* Amazon API Gateway can handle details
* Don't retry immediately - it's a waste of resources. Fast retries are very likely to fail again
* From client's perspective, waiting longer is a bad thing. It keeps resources busy longer than needed.
* Use queue-and-retry instead if possible
* Timeouts have synergy with circuit breakers
* Timeouts and Fail Fast address latency problems
* Fail Fast - for incoming requests
* Timeouts - for outbound requests
* Apply for: Integration Points, Blocked Threads, and Slow Responses

#### Circuit Breakers
* It's like circuit breakers in electrical systems
* Use it to wrap dangerous operations (Don't call a provider if it hurts)
* States: Closed, Open, Half-Open
* It's a way to automatically degrade the system when it's under stress
* We usually more interested in the fault density than the total count
* Leaky Bucket - a pattern for smarter CB
* Changes in CB should be logged for alerts and monitoring
* There should be some way to manually reset the CB
* !!! CB should be built at the scope of the process (each instance of the service discover problems independently - to avoid out-process communication)
* Use together with Timeouts

#### Bulkheads
* Bulkheads are a way to isolate the failure of one part of the system from the rest of the system
* Redundant VMs are not quite as robust as redundant physical machines - they do not ensure physical isolation
* At large scale, a mission-critical service might be done as several independent farms of servers where some servers reserved by critical apps
* In the cloud - run instances in different availability zones
* Hidden dependencies - two services that are not directly connected, but they share a common resource
* Useful to partition the threads inside a process (e.g. reserve threads in the pool for admin use)
* Pick a useful granularity: threads in the connection pool, CPUs in a server, servers in a cluster, etc.

#### Steady State
* Keep people off production systems (avoid fiddling)
* Computing resources are always finite
* The Steady State pattern says that for every mechanism that accumulates a resource, some other mechanism must recycle that resource

1. Data Purging
* Symptom of data growth: increasing I/O rates on the database, increasing latency
* Purge data with app logic (will it be functional after data is purged?)
2. Log Files
* Don't keep logs forever
* Use rotation based on size
* If it required by compliance, use a separate system for storing logs data
3. In-Memory Caching
* Is the space of possible cache keys finite or infinite? (user limits of keys)
* Do the cached items ever change? (they can be huge)
* Simple mechanism - time-based expiration (fits 99% of cases)

#### Fail Fast
* Slow responses are worse than no response. The worst - slow failure response
* If LB has a queue for connections to execute late (in hope of recovery), it's a violation of the Fail Fast principle
* Check availability of resources before you start using them
* Report a system failures differently than an application failure (for the upstream systems)
* Use simple input validation before validating the business logic and starting the business transaction

#### Let It Crash
* Sometimes the best thing to create system stability is to abandon the component-level stability
* The Let It Crash pattern says that error recovery id difficult and unreliable, so our goal is to get back to a known state as quickly as possible
* Conditions
  * Limited Granularity
  * Fast Replacement
  * Supervision
  * Reintegration
* Restart and reintegrate
* Isolate components to crash independently
* Don't crash monoliths (heavy runtimes or long startup)

#### Handshaking
* Handshaking - signals between devices that regulate communication between them
* It rarely exists at the application level
* It's most valuable when unbalanced capacity are leading to slow responses
* Build clients and servers to perfom handshaking (cooperative demand control)
* Consider health checks in clustered or load-balanced services

#### Test Harnesses
* Build a test harness that substitutes for the remote end of every web service call
* Hint: different ports for different sets of misbehavior
* Emulate out-of-spec failures
* Stress the caller: slow responses, no responses, garbage responses
* Leverage shared harnesses for common failures for different integration points (subclass the harness)
* It's a supplement to other testing techniques

#### Decoupling Middleware
* Done well, it integrates and decouples services at the same time
* It's architectural decision - hard to change later
* It allows to avoid most of the problems of integration points
* Choose proper architectural style

#### Shed Load
* The world can always make more load than you can handle
* Model TCP's approach - when load gets too high, start to refuse requests for work
* Make the app to monitor its own performance
* It's related to the Fail Fast
* Whit LB instance can use 503 status code (shock absorber)
* Inside the boundaries of the system or enterprise, it's more efficient to use Back Pressure

#### Create Back Pressure
* If a queue is unbounded, in can consume all available memory
* BP creates safety by slowing down consumers to not crash the provider
* Apply within a system boundary
* Apply Shed Load inside the system
* Queues must be finite for response times to be finite
* We don't want unbounded queues in our system.
* We have to decide what to do when the queue is full
  * Accept new item but drop it on th floor
  * Accept new item but drop the oldest item
  * Refuse new item
  * Block the caller until there is space in the queue
* If data value decreases with age, dropping oldest ones is a good idea
* It's important do distinguish back pressure due to load and back pressure due to failure

#### Governor
* The Governor limits the speed of an "engine"
* Create one to slow th rate of actions (e.g. autoscaler can only shut down a certain number of instances per minute)
* A Governor is stateful and time-aware
* The whole point is to slow things down to give humans time to get involved
* Apply resistance to the unsafe directions (U-shaped curve)


### Part 2: Design for Production
* Operations:
  * security, availability, capacity, status, communication
* Control Plane:
  * system monitoring, deployment, anomaly detection, features
* Interconnect:
  * routing, load balancing, failover, traffic management
* Instances:
  * services, processes, components, instance monitoring
* Foundation:
  * hardware, VMs, IP addresses, physical network

### Chapter 6: 
* The ability to restart components, instead of entire servers, is a key concept of recovery-oriented computing

### Chapter 7: Foundations
* Design for production means thinking about prod issues as first-class concerns
* It also means designing for people who do operations
* Considerations:
  * How is the network structured?
  * Is there on ore several of them?
  * Will a machine has NICs on multiple networks for different purposes?
  * Do machines have long-lasting identities?
  * Are machines automatically provisioned?
  * How do we manage images for them?

1. Networking in the Data Center and the Cloud
* NICs and the Names: there different NICs in the same server
* DNS name to IP address ia a Many-to-Many relationship
* Use different networks for different purposes: admin, production, etc.
* Programming for Multiple Networks: server apps that need to listen on sockets must add configurable properties to define to which interfaces the server should bind

2. Physical Hosts, Virtual Machines, and Containers
* Physical Hosts:
  * Philosophy now: load-balance services across enough hosts that the loss of a single host is not a problem.
  * Each host is as cheap as possible
  * Exceptions: workloads where large amount of RAM is required, GPU computing
* Virtual Machines in the Data Center
  * Almost any resource on the host can be oversubscribed
  * Make sure apps are not sensitive to the loss or slowdown of any one host
  * Don't trust the OS clock, use external services if needed
* Containers in the Data Center
  * Apps should rely on external storage files, data, or cache
  * We use special control plane software to operate containers
  * To handle problems with containers, follow https://12factor.net/
  * Containers are meant to start and stop rapidly.
  * Aim for a total startup time of one second
* VMs in the Cloud
  * Have worse availability than physical hosts
  * Ephemeral nature
  * Use bastion and jumphosts to access individual VMs
* Containers in the Cloud
  * The biggest challenge is to connect all the containers together

### Chapter 8: Processes on Machines
* Every machine needs the right code, configuration, and network connections
* Vocab:
  * Service
  * Instance
  * Executable
  * Process
  * Installation
  * Deployment

1. Code
* Developers must be able to build the system, run test, and run the system locally (at least a portion)
* Consider moving dependencies to the private registry
* Beware of the plugins to the build system (security concerns)
* Developers should not do production builds locally. Local environments are usually contaminated
* Only make production builds on a CI server

2. Immutable and Disposable Infrastructure
* It's more reliable to always start from known base image, apply a fixed set of changes, and then never attempt to patch the image/machine

3. Configuration
* Every piece of production software requires configurable properties
* Some of them changes (even from instance to instance), some of them don't - keep separately
* The code should look outside the deployment directory for configuration
* Sensitive information should be protected and not stored in the version control system
* Usage with disposable infrastructure:
  * Inject configuration into the container at startup
  * User configuration server
* Configuration server should by highly available
* Name properties obvious way

4. Transparency
* Transparency allows operators, developers, business users to gain understanding of the system's historical trends, current state, and future behavior
* Debugging a transparent system is easier
* A system wihtout it cannot survive long

1. Designing for Transparency
* It's architecture decision - harder to implement later
* The monitoring and reporting systems should be like an exoskeleton around the system

2. Enabling Technologies
* The process tells nothing about itself
* Black Box: observer is outside the system
* White Box: observer is inside the process (often like an agent delivered in a language-specific library)
* It couples the code to the library, but it's a small price to pay for the transparency

3. Logging
* Log files are still on of the most valuable tools - it's loosely coupled
* For physical machines - keep logs in a separate drive
* Apps in containers usually log to stdout (see 12 factor)
* Logs train people on what is normal app behavior
* Admins and operations will use logs more frequently than developers
* If something logged in ERROR/SEVERE required actions - serious system problems (e.g. circuit breaker change)
* Errors in business logic or user input should be logged as WARNING (or not at all)
* It's better not to log debug in production (method traces, checkpoints, etc.)
* Logs should be whitten in a structured way - humans should be able to read and understand them easily
* Messages should include some identifiers (e.g. request ID)
* Interesting state transitions should be logged

4. Instance Metrics
* Instance should emit metrics for later analysis
* Use metric collection libraries

5. Health Checks
* Reveals the application's internal view or its health (a page or API call)
* What could be reported in the healthcheck:
  * The host ip addresses
  * The version of interpreter
  * The app version
  * Whether the instance is accepting work
  * Statuses of connection pools, caches, and circuit breakers

### Chapter 9: Interconnect
* Traffic management (routing), load balancing, load shedding, and discovery
* Consul or other discovery service - for large teams
1. DNS
* DNS entries - for small teams
* DNS - just put hostnames in the config
* Use logical names for services, not physical ones
* Load balancing with DNS: round robin - the simplest, low-cost
* Flaws:
  * if the one of the instances is dwon, it send traffic to it anyway
  * inappropiate for long-lived connections (apps can cache the IP address)
* DNS is good at global server load balancing (GSLB)
* DNS outage can have a massive impact
* Diversity: don't host them on the same infrastructure
* Use DNS for services that don't change often
* "Discovery" is a human process, DNS names supplied in configs
2. Load Balancing
* We need LB for horizontally scaled services
* It's about distributing traffic across multiple instances
* LB listens on one or more sockets accrose one or more IP addresses - Virtual IPs
* A pool defines IP addresses of the instances and policies for distributing traffic:
  * LB algorithm
  * What health checks to perform
  * What kind of stickiness to apply
  * What to do when all instances are down
* The client shouldn't know about the load balancer involved
* Underlying services should generate URLs with the DNS name of the VIP rather than the IP address of the instance
* LB could be software or hardware
* Squid, HAProxy, Nginx, httpd, etc. - software
* X-Forwarded-For header - to get the real IP address of the client
* Hardware LBs provide better capacity and throughput
* Health checks - on of the most important features
* Stickiness - to keep the client on the same instance (fits well for stateful services)
* Stickiness options:
  * Cookie-based
  * IP-based
  * Session-based
* Content-based routing - to route traffic based on the content of the request (e.g. URL for signups and search requests)
* LB are integral to the delivery of the system
3. Demand Control
* There is no natural protection against a sudden surge in traffic
* Two strategies: refuse work or scale out
* Every failing system starts with a queue backing up somewhere
* Possible queues:
  * Socket
  * NIC buffer (read/write)
  * Listen queue
  * Provider listen queue
* So the best thing to do is to refuse work the system can't handle
* Shed load as early as possible - LB is the ideal place
* Services can measure their own response time to help with this
* Also, response time of the downstream services
* Services should have relatively short listen queues
* Learn to define the right queue length
* [PDQ](http://www.perfdynamics.com/Tools/PDQ.html) - a tool for queue length estimation
4. Network routing
* Sometimes we need to define preferred routes for traffic
5. Discovering Services
* The discovery service is a central repository of information about the services in the system
* It practical when:
  * Too many services for DNS management
  * Highly dynamic environment
* Two parts of Service Discovery:
  * Service registration
  * Service lookup
* SD is another service, so it's a good idea for clients to cache the results
* It's better to not use your own discovery service, but use a third-party one - it's more reliable
6. Migratory Virtual IP Addresses
* If the app call the service by the VIP, it should be prepared for the case that next TCP packet isn't going to the same interface

### Chapter 10: Control Plane
* Every part of the CP is optional - build as much as you need
* Be careful with automation - it amplifies mistakes (automation goes really fast)
* System failure, not human error
1. Development Is Production
* Be as close to production as possible (e.g. fresh environment for QA from image)
* Tools and services for development should be treated with production-level SLAs
2. System-Wide Transparency
* Start with visibility, use logging, tracing, and metrics to create transparency
* Collect and index logs to look for general patterns
* Use configuration, provisioning, and deployment tools to gain leverage
* Once the system is stabilized and problems are visible, build control mechanisms
* Two questions to ask:
  * Are users receiving a good experience?
  * Is the system creating the economic value we want?
* Apply RUM ([Real User Monitoring](https://en.wikipedia.org/wiki/Real_user_monitoring)) to the whole system
* Economic Value: Top line and bottom line
* Top line: revenue (watch for problems in service related to revenue generation, exceptions, queue length, etc.)
* Bottom line: cost (infrastructure, operations, etc.)
* Also questions to ask:
  * Are there opportunities to increase the top line by improving performance or reducing queues?
  * Are we going to hit a bottleneck that will prevent us from increasing the top line?
  * Are the opportunities to increase the bottom line by optimizing services? Can we see places thar are overscaled?
  * Can we replace slow-performing or large-footprint services with more efficient ones?
* All the information (graphs, metrics, etc.) should be built from the same data source, but they can be presented from different perspectives
* Each group can have their own favorite dashboards
* Logs and stats:
  * Gather all the data - log and metrics collectors
  * Push and pull models
  * Indexing logs for search and analysis
  * Some information about instances are generated from dedicated programs running on the instances
  * Metrics can be time-grouped (e.g. 1 minute, 5 minutes, 1 hour, etc.)
  * Key metrics can change over time
* What to expose:
  * Traffic indicators: page requests. transaction counts, concurrent sessions, etc.
  * Business transactions, for each type: number processed, conversion rate, completion rate
  * Users: number of users, usage patterns, successful logins
  * Resource pool health: 
  * Database connection health
  * Data consumption
  * Integration points health: state of circuit breakers, timeouts, response times, number of networks/app errors
  * Cache health: items in cache, hit rate, cache size
* All the counters have an implied time component (e.g. number of requests per minute)
* There could be second range counters (caution signal, the param is approaching the threshold)
3. Configuration Services
* They are also services, so they also bound to the constraints of the CAP theorem
* Make sure instances can start without the configuration service
* Make sure instances don't stop if the configuration service is down
* Make sure the configuration service doesn't brake "the world" if it's down
* Replicate across geographic regions
4. Provisioning and Deployment Services
* The deployment tool should be with a package repository
* Have locations for artifacts that built not from developers' machines
* Repeatable builds are important so code on local machine works in production, too
* Canary deployments - to test new versions of the service
5. Command and Control
* It's simpler to kill the instance and run a new one, when the instance is needed to be modified
* But it's not always possible
* Then we need to have a way to modify the running instance
  * Reset circuit breakers
  * Adjust connection pool size and timeouts
  * Disable a specific outbound integrations
  * Reload configuration
  * Start and stop accepting load
  * Feature toggles
* Don't build a self-destruct button into your production code
* The simplest way to give those capabilities - API over HTTP (on different port)
* But the best interface for long-term operation is command line
6. The Platform Players
* It manages resurces and schedules tasks accross multiple computes.
* It abstracts the underlying infrastructure and presents a friendlier programming model.
7. The Shopping List
* Log collection and search
* Metrics collection and visualization
* Deployment
* Configuration service
* Instance placement
* Instance and system visualization
* Scheduling
* IP, overlay network, firewall, and load route management
* Autoscaler
* Alerting and notification

### Chapter 11: Security
* It's a complex topic that should be studied separately
* Look at the [OWASP Top 10](https://owasp.org/www-project-top-ten/) for more information and cheat sheets 
* Security is an ongoing process, not a one-time event
* Use the principle of least privilege
