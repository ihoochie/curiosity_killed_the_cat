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
