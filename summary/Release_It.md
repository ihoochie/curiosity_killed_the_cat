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
