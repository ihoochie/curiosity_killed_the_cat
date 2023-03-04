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
