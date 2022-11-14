# Lesson 5: Distributed Systems (27 points)

## 1. Lamport's Logical Clock (8 points)

A student has implemented a distributed algorithm using Lamport’s happened-before relationship to timestamp the events. She is in the middle of debugging the program. She observes the following events for each processor. Each event is tagged with the local timestamp recorded for the event on that processor following Lamport’s logical clock.

| P1                        | P2                       | P3                       |
|---------------------------|--------------------------|--------------------------|
| 1: msg-send (to ???)      |                          | 1: local event           |
|                           | 2: msg-receive (from ??) | 2: msg-send (to ??)      |
|                           | 3: msg-receive (from ??) | 3: local event           |
|                           | 4: msg-send (to ??)      | 4: local event           |
| 5: msg-receive (from ???) | 5: msg-send (to ??)      |                          |
|                           |                          | 6: msg-receive (from ??) |

Please help her by identifying who is the sender/receiver for each message (namely, the sender, receiver, and the logical timestamp associated with that event.

In answering the question use the following format: `<processor-number>: <local-time>: msg-<send/receive> (<from>/<to>)`

(e.g., `P1: 20: msg-receive(P3)` would mean P1 received a message from P3 at local time 20)

---

| P1                      | P2                      | P3                      |
|-------------------------|-------------------------|-------------------------|
| 1: msg-send (to 2)      |                         | 1: local event          |
|                         | 2: msg-receive (from 1) | 2: msg-send (to 2)      |
|                         | 3: msg-receive (from 3) | 3: local event          |
|                         | 4: msg-send (to 1)      | 4: local event          |
| 5: msg-receive (from 2) | 5: msg-send (to 3)      |                         |
|                         |                         | 6: msg-receive (from 2) |

## 2. Lamport's M.E. Algorithm (10 points)

https://en.wikipedia.org/wiki/Lamport%27s_distributed_mutual_exclusion_algorithm

https://drive.google.com/file/d/1HkVo68Ykq-zsA2pXVF_34NqCXaq4HHvV/view

// TODO

## 3. Latency Reduction in RPC (9 points) (hidden)

https://drive.google.com/file/d/18Oiq99TQYNg-D0h7Oyu9D9V2VEyxV-h0/view

// TODO

# Lesson 6: Distributed Objects and Middleware (26 points) 

## 4. Spring OS (10 points)

You are managing a subset of nodes in a cluster. You have chosen to use Spring as the network OS for the cluster. You must host the following services:

1. A Postgres database server which is replicated on 3 nodes
1. A web server on 3 nodes
1. A web server load balancer that does ensures equitable CPU utilization on all the servers for the client requests.
1. A Postgres load balancer that does round-robin allocation of the servers for the client requests.

Each of the above servers and the load balancers are hosted on distinct nodes on the LAN. The clients are all on the same LAN and are expected to make requests to both the Postgres and Web service.

### 4.1 (6 points)

#### 4.1a

List the subcontracts needed on the client machines.

---

The clients will need a subcontract with the webserver load balancer and the Postgres load balancer.

#### 4.1b

List the subcontracts needed on the web server load balancer.

---

The web server load balancer will need one subcontract for communication with the three webservers. This subcontract will need to be able to determine the CPU utilization of each web server and route a invocation to the web server with the lowest utilization. The subcontract will dynamically discover web servers on the network. The load balancer will also need a subcontract with the clients.

#### 4.1c

List the subcontracts needed on the Postgres load balancer.

---

The Postgres load balancer will need one subcontract for communication with the three Postgres servers. The subcontract will perform round-robin distribution of requests, so it will need to maintain state to know which server to contact next. The subcontract will dynamically discover Postgres instances on the network. The load balancer will also need a subcontract with the clients.

### 4.2 (2 points)

You decide to beef up your web server with 2 more nodes. What changes will you need to make to ensure that the client requests can utilize the two new nodes?

---

No change is needed. The client contacts the load balancers which dynamically discover their respective instances.

### 4.3 (2 points)

You now need to access the Postgres database from the web service. What changes do you need to make to the system?

---

The web servers will need a subcontract with the Postgres load balancer. The Postgres load balancer will need a corresponding subcontract.

## 5. EJB (10 points)

It is circa 2002. Yelp and Google Reviews don’t exist yet. You’re a developer and a foodie. You decide to build a restaurant review website that has the following functionalities:

1. Accept a restaurant name or a cuisine as input and display a list of restaurants with their ratings.
1. If a user clicks on a restaurant, they will be shown the reviews for the restaurant.
1. The user should be able to sort restaurants by distance from their location, and average review score.

1. Allow a user to post a review about a restaurant and store it in the database, along with some keywords (e.g., cuisine, ambience, etc.), which may or may not explicitly be provided by the user.

Now you realize that restaurant searches are hyper-local, so you only need to show the user the restaurants which are within a 15-mile radius. So, you decide to use the user’s GPS location to filter results.

You decide to implement the system with the state of the art, i.e., EJB entity beans as shown below:

![EJB](ejb.png)

### 5.1 (6 points)

For the functionality that you need on the website, describe concisely what components go into the presentation logic, business logic and the entity beans. 

---

Presentation logic:

* Interacts with the client
* Controls (e.g. HTML, CSS) for searching, sorting, filtering
* Controls for publishing reviews
* Logic for rendering reviews and restaurants
* On search the presentation logic will call the business logic with the desired searching, sorting, and filtering parameters
* When posting a review the presentation logic will pass the review information to the business logic
* GPS data is passed from the client to the business logic for additional filtering

Business logic:

* Validation of requests
* Bridge between presentation logic and entity beans
* Forwards searches from the presentation logic to entity beans
* Could potentially perform searching, filtering, and sorting, but it would be more efficient to do this in the database
* When posting reviews the business logic interacts with the entity beans to create a new database record to persist the review information. It may also perform any special logic to handle optional/missing fields

Entity beans:

* Interacts with a database to persist and query data such as restaurants and reviews
* Provides mechanisms to search for restaurants and reviews based on some criteria
* Acts as the bridge between the database and Java objects

### 5.2 (2 points)

How would you optimize for latency for concurrent requests from users in the same location?

---

// TODO

### 5.3 (2 points)

You decide to open shop in India, where there are 22 official languages. You realize that restricting yourself to English might be a problem. Where in this framework would you add the functionality to render content in different languages? Justify your answer (No points without justification).

---

All layers will need to support a string encoding format that supports India's 22 languages. Java uses UTF-8 by default. Let's assume that all layers were designed to support UTF-8, and that UTF-8 is a good fit for the languages we're displaying.

* The presentation layer will need to be updated so that users can select their preferred language(s)
  * The presentation layer could optionally be updated to automatically translate user-submitted text (e.g. reviews) with a technology like Google translate
  * Otherwise we should provide a filter so that we only show reviews that the user can understand
  * We may also want to allow restaurants to display which languages they speak, so that a customer knows they will be well served there
* The business logic will need to support the new language filter
* The entity beans will need to be updated to store our new languages fields, and expose methods for searching/filtering
* The database schema will likewise need to be updated for the new fields

## 6. Java RMI (6 points)

### 6.1 (4 points)

Java RMI evolved from the Spring Subcontract mechanism. Name one similarity and one difference in the implementation of the two systems.

---

Method invocation from the client is impervious to the location of the server

Subcontracts rely on IDL to create strong interface; whereas RMI uses Java objects via extending the object or using a remote object interface

### 6.2 (2 points)

Java allows object references to be passed as parameters during object invocation. What is the difference in parameter passing (when a local object reference is passed as a parameter) while invoking a remote object using Java RMI?

// TODO

# Lesson 7: Distributed Subsystems (46 points)

## 7. GMS (16 points)

### 7.1 (2 points)

Upon a page fault, GMS converts the VPN to a UID. The UID includes the IP-ADDR of an NFS server. Why?

---

// TODO

### 7.2 (2 points)

Your friend suggests that the UID could be generated by simply prefixing the faulting VPN with the IP-ADDR of the source node and the PID of the faulting process. Will that work? Credit only if there is justification for your answer.

---

// TODO

### 7.3 (2 points)

For the geriatrics algorithm, it is straightforward to record the time of access to the virtual pages visited by a process during its execution. Answer True/False with justification.

---

// TODO

### 7.4 (6 points)

Assume that there is a designated node (which never fails) in charge of additions/churns of the nodes participating in GMS. A new node joins the GMS.

#### 7.4a

List the actions that ensue.

---

// TODO

#### 7.4b

What data structures of GMS will get modified as a result? Why?

---

// TODO

#### 7.4c

What data structures of GMS will not get modified as a result? Why?

---

// TODO

### 7.5 (4 points)

Your friend suggests implementing GMS with a replicated table in each node that gives the mapping `UID -> <nodeid, pframe>`.

#### 7.5a

What work would need to be done on each page fault?

---

// TODO

#### 7.5b

What work would need to be done on each page eviction from a node.

---

// TODO

## 8. DSM (18 points)

### 8.1 (2 points)

For correctness of a multiprocess application using DSM running on a LAN cluster, a given virtual page should be mapped to the same physical frame number in each node. Answer True/False with justification.

---

// TODO

### 8.2 (2 points)

For correctness of a multiprocess application using DSM running on a LAN cluster, the size of physical memory on each node of the cluster should be the same. Answer True/False with justification).

---

// TODO

### 8.3 (2 points)

Consider a DSM application that has no data races. It properly uses synchronization with mutual exclusion locks to safeguard access to shared data structures. The underlying DSM uses SC memory model with page level granularity for coherence. The application experiences poor performance. Explain why.

---

// TODO

### 8.4 (4 points)

The data structures protected by a lock is in the purview of the application and the DSM system has no knowledge of this association. LRC ensures the coherence maintenance of the data structures protected by a lock to the point of lock acquisition. How is coherence maintenance achieved in LRC?

---

// TODO

### 8.5 (8 points)

In Treadmarks DSM system the following critical section is executed at a node N1:

```
Lock(L1);
 Write to Page X;
 // Assume that the page is not present at this node;
 // Assume that there are three diff files for page X
 // named, Xd2, Xd3, and Xd4 in nodes N2, N3, and N4,
 // respectively.
 // Assume the sync causality for the lock L1 is
 // N3 -> N2 -> N1 (i.e, this is the order of lock acquisition).
Unlock (L1);
```

---

// TODO

#### 8.5a (2 points)

What actions would be carried out by Treadmarks at Node 1 before the
critical section above is executed by N1?

---

// TODO

#### 8.5b (2 points)

Upon exiting the critical section what action would be carried out by
Treadmarks at Node N1?

---

// TODO

---

A little while later, Node N1 executes the following critical section. Assume that no other node acquired the lock L1 in the interim.

```
Lock(L1);
 Read page X;
 do some computation without changes to any pages;
Unlock(L1);
```

#### 8.5c (2 points)

What would be the action carried out by Treadmarks at Node 1 before
the critical section above is executed by N1?

---

// TODO

#### 8.5d (2 points)

Upon exiting the critical section what would be the action by
Treadmarks at Node N1?

---

// TODO

## 9. DFS (12 points)

Inspired by xFS, you and your classmate decide to implement a true distributed FS. In your implementation similar to xFS, the location of the files on the disks remain fixed (i.e., they are never migrated). However, you periodically assess the meta-data server activity on each node and redistribute the meta-data management to balance the load in the entire cluster. In your implementation you are using the same data structures as in the original xFS.

### 9.1 (4 points)

You observe that there is a load imbalance in the system. You carry out a load re-distribution.
List the steps in your meta-data load re-distribution algorithm.

---

// TODO

### 9.2 (4 points)

What data structures change as a result of the load re-distribution
algorithm? Why?

---

// TODO

### 9.3 (4 points)

What data structures do not change as a result of the load redistribution algorithm? Why?

---

// TODO

