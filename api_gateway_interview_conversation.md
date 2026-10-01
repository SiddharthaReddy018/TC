# API Gateway / Load Balancer --- Complete Interview Prep Conversation

This document contains the question-and-answer chain from our
interview-preparation discussion, with internal links between each
follow-up question.

## Table of Contents

1.  [Project overview](#q01)
2.  [Purpose of each component](#q02)
3.  [Route matching and backend selection](#q03)
4.  [Why the monitor exists](#q04)
5.  [Backend pools](#q05)
6.  [Why multiple pools can be useful](#q06)
7.  [Gradual rollout / migration](#q07)
8.  [Data migration between versions](#q08)
9.  [Monitor and log rotation](#q09)
10. [The problem after fork](#q10)
11. [Locks and their purpose](#q11)
12. [Examples of fcntl and config_rwlock](#q12)
13. [What a route means](#q13)
14. [active_connections and volatile](#q14)
15. [Workers vs backends and full request flow](#q15)
16. [Authentication and hashing](#q16)
17. [Statistics](#q17)

------------------------------------------------------------------------

# 1. Project overview {#q01}

### Question

> so I have an interview , I need to master this project for the
> interview , so explain me this project so that I can do well in it ,
> lets do it section by section

### Answer

This project is a **multi-process API Gateway and Load Balancer written
in C for Linux/POSIX**.

At a high level:

``` text
Clients
   |
   v
API Gateway :8080
   |
   +--> Authentication
   |
   +--> Load Balancer
            |
            +--> Backend 1 :9001
            +--> Backend 2 :9002
            +--> Backend 3 :9003
```

The gateway:

1.  Accepts client TCP connections.
2.  Reads HTTP requests.
3.  Matches routes.
4.  Authenticates when required.
5.  Selects a backend.
6.  Connects to the backend.
7.  Proxies data in both directions.
8.  Maintains statistics.
9.  Logs requests.
10. Does this concurrently using multiple worker processes.

### Master

The master process initializes configuration, shared memory,
synchronization, the listening socket, workers, CLI, monitoring and
shutdown.

### Workers

Workers are created using `fork()` and handle actual client traffic:

``` text
accept()
 -> read request
 -> parse HTTP
 -> route match
 -> authentication
 -> load balancing
 -> connect backend
 -> proxy
 -> update statistics
 -> close
```

### Monitor

Workers send log messages through a named pipe/FIFO to a monitor:

``` text
Worker -> FIFO -> Monitor -> gateway.log
```

### Why multiple processes?

The project uses a pre-fork architecture. Multiple workers can execute
concurrently and a worker failure does not necessarily take down all
workers.

### Shared-memory problem

After `fork()`, ordinary variables are not shared between processes.
Therefore the project uses System V shared memory for common state such
as:

-   backends
-   routes
-   statistics
-   round-robin index
-   configuration state
-   shutdown state

Synchronization is needed because multiple workers can access the shared
state simultaneously.

Important synchronization areas include:

-   `lb_mutex` --- load-balancing state
-   `backend_mutex` --- backend-pool modifications
-   `accept_mutex` --- `accept()`
-   `config_rwlock` --- shared configuration/routes
-   `fcntl` lock --- configuration file

### Load balancing

Round Robin:

``` text
R1 -> Backend 1
R2 -> Backend 2
R3 -> Backend 3
R4 -> Backend 1
```

Least Connections:

``` text
B1 = 5
B2 = 2
B3 = 7

new request -> B2
```

### Reverse proxying

The worker has two connections:

``` text
Client <---- TCP connection 1 ----> Worker <---- TCP connection 2 ----> Backend
```

### Interview answer

> This project is a multi-process API Gateway and Load Balancer
> implemented in C using POSIX system calls. The master initializes the
> gateway and forks workers. Workers accept client connections, parse
> HTTP, match routes, authenticate requests, select a backend using
> Round Robin or Least Connections, and proxy traffic to the backend.
> Because workers are separate processes, shared state is kept in System
> V shared memory and protected by process-shared synchronization. The
> project also supports configuration reload, role-based CLI
> authorization, worker respawning, and centralized logging through a
> named pipe and monitor process.

**Next:** [Question 2 --- Purpose of each component](#q02)

------------------------------------------------------------------------

# 2. Purpose of each component {#q02}

### Question

> explain what's the purpose of each component

### Answer

The three main process types are:

### Master

Think of it as the manager/coordinator.

``` text
Master
  |
  +--> Worker 1
  +--> Worker 2
  +--> Worker 3
  +--> Worker 4
```

It:

-   reads configuration
-   creates shared memory
-   initializes locks
-   creates the listening socket
-   creates workers
-   starts CLI
-   monitors workers
-   restarts workers if needed
-   handles shutdown

### Worker

The worker handles the actual request:

``` text
accept
 -> read
 -> parse
 -> route
 -> authentication
 -> load balancing
 -> backend connection
 -> proxy
 -> statistics
 -> close
```

Interview answer:

> The worker handles the actual client request lifecycle: accepting the
> connection, parsing HTTP, matching routes, authenticating, selecting a
> backend, proxying the request/response, updating statistics, and
> logging.

### Monitor

The monitor collects logs:

``` text
Workers -> FIFO -> Monitor -> gateway.log
```

### Load balancer

The load balancer is **not a separate process**. It is functionality
used by workers.

``` text
Worker
  |
  +--> Round Robin
  |
  +--> Least Connections
```

### Shared memory

Shared memory is not a process. It is the common state used by
workers/master.

### Simplest memory aid

``` text
Master  -> Manage the system
Worker  -> Serve traffic
Monitor -> Collect logs
```

**Next:** [Question 3 --- Route matching and backend selection](#q03)

------------------------------------------------------------------------

# 3. What is route matching? What does "select backend" mean? {#q03}

### Question

> what is this and also what does select backend mean ? , dont we have
> the asame backend for all ?

### Answer

There are two separate questions.

### Route matching

If the client sends:

``` http
GET /api/v1/users
```

the gateway checks which configured rule matches the path:

``` text
/api/v1/users
      |
      v
matches /api/v1
```

### Backend selection

Suppose there are:

``` text
Backend 1 -> :9001
Backend 2 -> :9002
Backend 3 -> :9003
```

After determining the route, the gateway asks:

> Which actual server should handle this request?

That is backend selection.

Example:

``` text
/api/v1/users
      |
      v
route = /api/v1
      |
      v
select Backend 2 :9002
```

### Why multiple backends?

One backend can be a bottleneck or a single point of failure. Multiple
backend instances provide capacity and redundancy.

### Important distinction

``` text
Route matching
  = What kind of request/routing rule is this?

Backend selection
  = Which actual server should receive it?
```

**Next:** [Question 4 --- Why the monitor?](#q04)

------------------------------------------------------------------------

# 4. Why do we need the monitor in the middle? {#q04}

### Question

> why do we need the monitor in the midway can't we have pipe directly
> to gateway.log

### Answer

Yes, workers could technically write directly to `gateway.log`.

Without monitor:

``` text
Worker 1 ----Worker 2 -----+--> gateway.log
Worker 3 ----/
```

With monitor:

``` text
Worker 1 ----Worker 2 -----+--> FIFO --> Monitor --> gateway.log
Worker 3 ----/
```

The monitor becomes a **single writer** and centralizes log-file
management.

This makes it easier to implement:

-   formatting
-   serialization
-   rotation
-   filtering
-   compression
-   forwarding

The key interview point:

> The monitor is not strictly required for multiple processes to write a
> file. It is an architectural choice that centralizes logging and keeps
> file management out of the workers.

A FIFO can still block if its buffer fills; the architecture does not
automatically make logging non-blocking.

**Next:** [Question 5 --- Backend pools](#q05)

------------------------------------------------------------------------

# 5. How can a pool have different backends? {#q05}

### Question

> how can an pool have diffrent backends , dont the pool have all the
> backends servers ? s when route matching we should be able all the
> live servers right?

### Answer

For **this project**, the understanding is correct: there is effectively
one common backend pool containing the configured backend servers.

``` text
backend_pool
  |
  +--> 127.0.0.1:9001
  +--> 127.0.0.1:9002
  +--> 127.0.0.1:9003
```

Routes point to that common pool.

So:

``` text
request
  |
  v
route matching
  |
  v
matched route
  |
  v
common backend_pool
  |
  v
load balancer
  |
  v
one backend
```

Route matching does not itself select the server.

Also distinguish **route matching** from **backend health**.

``` text
9001 -> alive
9002 -> dead
9003 -> alive
```

The load balancer should select from the eligible/live servers.

A more sophisticated gateway could support multiple pools, but that is
an extension rather than the core architecture of this project.

**Next:** [Question 6 --- Why multiple pools can be useful](#q06)

------------------------------------------------------------------------

# 6. Is having two or more groups useful? {#q06}

### Question

> is having two or more groups of same thing useful ? how ?

### Answer

Yes, when the groups represent different services, versions,
environments, regions, or other eligibility boundaries.

Example:

``` text
Gateway
  |
  +--> User Pool
  |      +--> U1
  |      +--> U2
  |      +--> U3
  |
  +--> Payment Pool
         +--> P1
         +--> P2
         +--> P3
```

Then:

``` text
/api/users    -> User Pool
/api/payments -> Payment Pool
```

This prevents a payment request from being sent to a user-service
server.

Another useful case is API versions:

``` text
/api/v1 -> V1 pool
/api/v2 -> V2 pool
```

This supports gradual migration.

Other examples:

-   production vs canary
-   India vs US
-   different services

### Key idea

The pool answers:

> Which servers are eligible for this request?

Then the load balancer answers:

> Which eligible server should I choose?

**Next:** [Question 7 --- Gradual rollout](#q07)

------------------------------------------------------------------------

# 7. What does gradual rollout/migration mean? {#q07}

### Question

> explain this

### Answer

Suppose the company has V1:

``` text
/api/v1/users
  |
  +--> V1 server 1
  +--> V1 server 2
  +--> V1 server 3
```

and a new V2:

``` text
/api/v2/users
  |
  +--> V2 server 1
  +--> V2 server 2
  +--> V2 server 3
```

Instead of switching everyone immediately, traffic can be moved
gradually:

``` text
100% V1
 -> 90% V1 / 10% V2
 -> 50% V1 / 50% V2
 -> 10% V1 / 90% V2
 -> 100% V2
```

This reduces the risk of moving all traffic to a new version at once.

Even with the same public URL, the gateway could internally control
which version receives traffic.

Separate pools make that control explicit.

**Next:** [Question 8 --- Moving stored data between versions](#q08)

------------------------------------------------------------------------

# 8. How do we move stored data between versions? Same database? {#q08}

### Question

> so in gradual roll out how do we send the stored data from one version
> to another ? do we use solve this by having same database ?

### Answer

The gateway normally does **not** move the data. It moves **traffic**.

A common setup is:

``` text
          Database
          /              V1        V2
```

Both application versions use the same database.

During rollout:

``` text
90% -> V1
10% -> V2
```

then gradually increase V2.

The data remains in the same database.

### If V2 changes the schema

A common approach is:

> Expand -\> Migrate -\> Contract

Example.

Old:

``` text
users
id
name
email
```

Expand:

``` text
users
id
name
email
first_name
last_name
```

Backfill:

``` text
"Alice Smith"
    ->
first_name = "Alice"
last_name = "Smith"
```

Then V2 can use the new fields while V1 still works.

Once V1 is gone, old fields can be removed.

### If V1 and V2 use different databases

Then data migration/synchronization is needed, which is more
complicated.

Key distinction:

``` text
Gateway
  -> traffic migration

Application/database layer
  -> data/schema migration
```

**Next:** [Question 9 --- Monitor and log rotation](#q09)

------------------------------------------------------------------------

# 9. Is the monitor mainly for log rotation? Do workers need old logs? {#q09}

### Question

> we dont want to have the log rotation right ? that why we will have it
> through monitor right ? as each worker should access the older logs
> right ?

### Answer

Not quite.

Log rotation means:

``` text
gateway.log
gateway.log.1
gateway.log.2
```

The monitor is a convenient place to implement it because it owns the
log file.

But workers do **not** need to access older logs.

Workers simply produce new log messages:

``` text
Worker -> "GET /api/v1 -> 200"
```

The monitor manages:

``` text
current gateway.log
old rotated logs
rotation policy
```

So the design principle is:

``` text
Workers = log producers
FIFO    = communication channel
Monitor = log consumer/manager
```

The monitor could later handle rotation, formatting, filtering,
compression, etc.

**Next:** [Question 10 --- The problem after fork](#q10)

------------------------------------------------------------------------

# 10. When does the "problem" after fork happen? {#q10}

### Question

> you have said "But then we have a problem" but when ?I didn't
> understand which would cause this problem

### Answer

The problem begins when the master uses `fork()` to create workers, but
those workers need common state.

Before fork:

``` text
Master
counter = 10
```

After:

``` text
fork()
  |
  +--> Master: counter = 10
  |
  +--> Worker: counter = 10
```

They initially have the same value, but they have separate memory.

If Worker increments:

``` text
Worker -> counter = 11
Master -> counter = 10
```

With multiple workers:

``` text
Master   -> 10
Worker 1 -> 11
Worker 2 -> 10
```

This is a problem for global things such as:

-   active connections
-   request counters
-   round-robin state
-   backend state

Therefore the project uses shared memory.

Then, because multiple workers access the same shared memory
concurrently, synchronization is also required.

Sequence:

``` text
fork()
 -> separate processes
 -> ordinary variables are separate
 -> workers need common state
 -> shared memory
 -> concurrent access
 -> locks/semaphores
```

**Next:** [Question 11 --- Locks](#q11)

------------------------------------------------------------------------

# 11. What are these locks, where are they used, and what do they mean? {#q11}

### Question

> explain where and how do we use these and what these mean ?

### Answer

Two important mechanisms discussed here are:

-   `config_rwlock`
-   `fcntl` file lock

They protect different things.

## `config_rwlock`

A read-write lock protecting configuration/routes in shared memory.

Multiple workers can read:

``` text
Worker 1 ---Worker 2 ----+--> routes[] READ
Worker 3 ---/
```

During reload, a writer gets the write lock:

``` text
Workers -> WAIT

Reload -> WRITE routes
```

After the update, workers can read again.

Conceptually:

``` c
pthread_rwlock_rdlock(&state->config_rwlock);
/* read routes */
pthread_rwlock_unlock(&state->config_rwlock);
```

and reload:

``` c
pthread_rwlock_wrlock(&state->config_rwlock);
/* replace routes */
pthread_rwlock_unlock(&state->config_rwlock);
```

## `fcntl` file lock

Protects the physical configuration file:

``` text
gateway.conf
```

It coordinates file access while the gateway reads/uses the
configuration file.

### Difference

``` text
fcntl lock
  -> gateway.conf on disk

config_rwlock
  -> loaded routes/config in shared memory
```

**Next:** [Question 12 --- Examples of both](#q12)

------------------------------------------------------------------------

# 12. Examples for `fcntl` and `config_rwlock` {#q12}

### Question

> give me examples for the above two

### Answer

## `fcntl` example

Suppose:

``` text
server0 = 127.0.0.1:9001
server1 = 127.0.0.1:9002
```

An edit could temporarily leave the file incomplete:

``` text
server0 = 127.0.0.1:9001
server1 = 127.0.
```

If the gateway reads at that moment, it could see incomplete
configuration.

A file lock coordinates access:

``` text
Gateway
  |
  +--> acquire fcntl lock
  +--> read config
  +--> release lock
```

It protects the **file**.

## `config_rwlock` example

Once loaded:

``` text
SharedState
  |
  +--> routes[]
       +--> /api/v1
       +--> /api/v2
       +--> /health
```

Many workers can read simultaneously:

``` text
Worker 1 -> READ
Worker 2 -> READ
Worker 3 -> READ
```

During reload:

``` text
Worker 1 -> WAIT
Worker 2 -> WAIT
Worker 3 -> WAIT
Reload   -> WRITE
```

It protects the **loaded in-memory configuration**.

Memory aid:

> `fcntl` -\> file.

> `config_rwlock` -\> loaded configuration in shared memory.

**Next:** [Question 13 --- What is a route?](#q13)

------------------------------------------------------------------------

# 13. What does a route even mean? {#q13}

### Question

> what does an route even mean ?

### Answer

A route is simply a **configured rule that matches an incoming request
path and determines how the gateway should handle it**.

Example request:

``` http
GET /api/v1/users
```

The gateway sees:

``` text
/api/v1/users
```

and checks:

``` text
/api/v1
/api/v2
/health
/
```

It can match:

``` text
/api/v1
```

A route can carry rules such as:

-   authentication required
-   which backend/pool applies
-   other request-handling rules

Think of routes as road signs:

``` text
Incoming traffic
      |
      v
   Gateway
      |
      +--> /users
      +--> /orders
      +--> /health
```

Interview definition:

> A route is a configured rule that matches an incoming request path and
> determines how that request should be handled by the gateway.

**Next:** [Question 14 --- active_connections and volatile](#q14)

------------------------------------------------------------------------

# 14. What are `active_connections` and `volatile`? {#q14}

### Question

> explain these two , what's the purpose of, what is volatile ? and use
> of the two

### Answer

## `active_connections`

For each backend:

``` text
Backend 1 -> 5 active
Backend 2 -> 2 active
Backend 3 -> 8 active
```

It means how many active connections are currently associated with that
backend.

Least Connections uses it:

``` text
5, 2, 8
  |
  v
choose Backend 2
```

When a request starts:

``` text
2 -> 3
```

When it ends:

``` text
3 -> 2
```

Because workers are separate processes, the value is shared through
shared memory.

## `volatile`

For:

``` c
volatile int active_connections;
```

`volatile` tells the compiler:

> This value may change outside the normal execution flow of this code,
> so do not optimize away accesses based on assuming it stays unchanged.

Another process can modify shared memory.

### Very important

`volatile` is **not a lock**.

It does not make this safe:

``` text
Worker 1 -> modify active_connections
Worker 2 -> modify active_connections
```

simultaneously.

Synchronization is still needed.

Example race:

``` text
active = 5

Worker 1 reads 5
Worker 2 reads 5

Worker 1 writes 4
Worker 2 writes 4
```

Expected after two decrements:

``` text
3
```

This is why a semaphore/mutex is needed.

### Memory aid

``` text
volatile -> compiler visibility/optimization concern
lock     -> concurrency/synchronization concern
```

**Next:** [Question 15 --- Workers vs backends and full request
flow](#q15)

------------------------------------------------------------------------

# 15. What is the difference between workers and backends? Full flow with 4 workers and 3 servers {#q15}

### Question

> what is the difference between the workers , backends ? should both be
> the same , when we send an requisition from client how does it may
> look -- for this case as it has 4 workers , 3 servers -- show for
> multiple client multiple requests in detail everything mentioned

### Answer

They are not the same.

## Workers

Workers are part of the gateway:

``` text
API Gateway
  |
  +--> Worker 1
  +--> Worker 2
  +--> Worker 3
  +--> Worker 4
```

They receive requests from clients.

## Backends

Backends are the application servers behind the gateway:

``` text
Gateway
  |
  +--> Backend :9001
  +--> Backend :9002
  +--> Backend :9003
```

They receive requests from workers and perform the application logic.

### Configuration

``` text
Gateway :8080
  |
  +--> Worker 1
  +--> Worker 2
  +--> Worker 3
  +--> Worker 4
  |
  +--> Load balancer
          |
          +--> :9001
          +--> :9002
          +--> :9003
```

Thus:

``` text
4 workers = gateway processes
3 servers = backend application instances
```

## Multiple clients

Suppose:

``` text
A -> GET /api/v1/users
B -> GET /api/v1/products
C -> GET /health
D -> POST /api/v2/orders
E -> GET /api/v1/users/10
F -> GET /api/v2/orders/20
```

An illustrative worker assignment:

``` text
A -> Worker 1
B -> Worker 2
C -> Worker 3
D -> Worker 4
E -> Worker 1
F -> Worker 2
```

The exact worker assignment is controlled by the OS/socket behavior, so
this is illustrative.

## Client A

Request:

``` http
GET /api/v1/users
```

Flow:

``` text
Client A
  |
  | TCP connection
  v
Gateway :8080
  |
  v
Worker 1
```

Worker calls `accept()` and reads the request.

Route matching:

``` text
/api/v1/users
     |
     v
/api/v1
```

Authentication is checked if required.

Then the load balancer selects a backend, say:

``` text
Backend 0 :9001
```

There are now two TCP connections:

``` text
Client A <-- connection 1 --> Worker 1 <-- connection 2 --> Backend 0
```

The worker forwards the request to the backend.

The backend performs the actual application logic and sends a response.

The response returns:

``` text
Backend
  |
  v
Worker
  |
  v
Client
```

## Client B

For example:

``` text
Client B -> Worker 2 -> Backend :9002
```

## Client C

``` text
Client C -> Worker 3 -> Backend :9003
```

## Client D

``` text
Client D -> Worker 4 -> Backend :9001
```

The exact mapping is illustrative; the important point is that **workers
are not permanently tied to backends**.

Worker 1 can connect to any backend:

``` text
Worker 1 -> :9001
Worker 1 -> :9002
Worker 1 -> :9003
```

Worker 2 can also connect to any backend.

The load balancer decides.

## Why 4 workers and 3 backends?

Workers provide concurrency at the **gateway**.

Backends provide capacity/redundancy for the **application**.

``` text
Clients
  |
  v
4 gateway workers
  |
  v
load balancing
  |
  v
3 application backends
```

## Active connections

For example:

``` text
Backend 1 -> 0
Backend 2 -> 0
Backend 3 -> 0
```

After requests:

``` text
Backend 1 -> 2
Backend 2 -> 1
Backend 3 -> 1
```

Least Connections can use those values to select a backend.

## Locks

If two workers update shared backend state simultaneously, a lock such
as `lb_mutex` prevents a race.

### Core flow

``` text
Client
  ↓
Worker
  ↓
Route matching
  ↓
Authentication
  ↓
Load balancer
  ↓
One of 3 backends
  ↓
Response
  ↓
Worker
  ↓
Client
```

**Next:** [Question 16 --- Authentication/hashing](#q16)

------------------------------------------------------------------------

# 16. How does password hashing work? Entire authentication flow {#q16}

### Question

> explain how does this hating work , explain the entire flow

The intended concept was password **hashing**.

### Answer

Hashing is not encryption.

You do not hash a password so that you can later decrypt it.

Instead:

``` text
password
  |
  v
hash function
  |
  v
hash
```

Then compare the hash of the supplied password to the stored hash.

Suppose:

``` text
admin = admin123:admin
```

Conceptually:

``` text
username = admin
stored password hash = H("admin123")
role = admin
```

## Full flow

Client sends:

``` http
GET /api/v1/users
Authorization: Basic ...
```

The worker:

1.  Parses the request.
2.  Matches `/api/v1`.
3.  Determines authentication is required.
4.  Extracts credentials.
5.  Hashes the supplied password.
6.  Compares it with the stored hash.
7.  If it matches, identifies the role.
8.  Checks permissions if needed.
9.  Continues to backend selection.
10. Proxies the request.

Conceptually:

``` text
Client
  |
  v
Worker
  |
  v
Route match
  |
  v
Authentication required?
  |
  v
Extract credentials
  |
  v
auth_hash(password)
  |
  v
Compare stored hash
  |
  +--> FAIL -> reject
  |
  +--> SUCCESS
          |
          v
       role
          |
          v
    authorization
          |
          v
    backend selection
          |
          v
        proxy
          |
          v
       backend
```

## Authentication vs authorization

Authentication:

> Who are you?

Authorization:

> What are you allowed to do?

## Project's hashing limitation

The project uses djb2. It is a fast general-purpose hash, not a
password-specific cryptographic password hashing algorithm.

For production, use something like:

-   Argon2
-   bcrypt
-   scrypt

with a unique salt.

Without salt:

``` text
Alice: H("hello123") -> ABC
Bob:   H("hello123") -> ABC
```

With different salts:

``` text
Alice: H("hello123" + saltA) -> hashA
Bob:   H("hello123" + saltB) -> hashB
```

### Interview answer

> When a request reaches a worker, the worker determines whether the
> route requires authentication. It extracts the credentials, hashes the
> supplied password using the project's auth hash function, and compares
> the result with the stored password hash. If they match, the user is
> authenticated and the role can be checked for authorization. In this
> project the hash is djb2, which is suitable for a demonstration but
> not for production password storage; a production system should use a
> password-specific algorithm such as Argon2 with a unique salt.

**Next:** [Question 17 --- Statistics](#q17)

------------------------------------------------------------------------

# 17. What are the statistics? Explain all {#q17}

### Question

> what are the statistics ? explain all

### Answer

Statistics are the numbers the gateway keeps about traffic and backend
activity.

Example:

``` text
Total requests:      1,250
Failed requests:        70
Bytes proxied:      15.2 MB
```

Per backend:

``` text
Backend :9001
  active connections: 3

Backend :9002
  active connections: 1

Backend :9003
  active connections: 4
```

Statistics are useful for:

1.  Monitoring
2.  Load-balancing decisions

## `total_requests`

How many requests the gateway has processed.

If three requests arrive:

``` text
total_requests = 3
```

Because there are multiple workers, the global counter is shared.

## `failed_requests`

Counts failed requests.

Example:

``` text
2 successes
2 failures

total_requests  = 4
failed_requests = 2
```

A simple failure rate:

``` text
failed_requests / total_requests
```

## `bytes_proxied`

Measures data transferred through the gateway.

Example:

``` text
5 KB + 10 KB + 2 KB = 17 KB
```

## `active_connections`

How many connections are currently active for a backend.

Example:

``` text
Backend 1 -> 5
Backend 2 -> 2
Backend 3 -> 7
```

Least Connections chooses Backend 2.

When a request starts:

``` text
2 -> 3
```

When it ends:

``` text
3 -> 2
```

## `alive`

Whether a backend is considered available.

Example:

``` text
Backend 1 -> alive
Backend 2 -> dead
Backend 3 -> alive
```

The load balancer should avoid dead backends.

## Why shared memory?

Suppose:

``` text
Worker 1 -> 100 requests
Worker 2 -> 150 requests
Worker 3 -> 80 requests
Worker 4 -> 70 requests
```

Global total:

``` text
400
```

A shared counter is needed for all workers to contribute to the same
total.

## Why locks?

If:

``` text
total_requests = 100
```

and two workers execute `total_requests++` simultaneously, both could
read 100 and write 101.

Expected:

``` text
102
```

Possible result:

``` text
101
```

Synchronization prevents this race.

### Memory aid

``` text
total_requests      -> how much traffic?
failed_requests     -> how many failures?
bytes_proxied       -> how much data?
active_connections   -> how busy is each backend?
alive               -> is backend available?
```

Special point:

> `active_connections` is both a statistic and an input to the Least
> Connections algorithm.

------------------------------------------------------------------------

# Final Interview Mental Model

``` text
                           CLIENTS
                              |
                              v
                     API GATEWAY :8080
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           Worker 1        Worker 2        Worker 3 ...
              |               |               |
              +---------------+---------------+
                              |
                        Shared State
                   +----------+----------+
                   |          |          |
                 Routes    Backends   Statistics
                              |
                        Load Balancer
                              |
                  +-----------+-----------+
                  |           |           |
                  v           v           v
                :9001       :9002       :9003
                              |
                           Backend
```

Logging:

``` text
Workers -> FIFO -> Monitor -> gateway.log
```

Configuration:

``` text
gateway.conf -> config manager -> SharedState
                      |              |
                 fcntl lock     config_rwlock
```

Request:

``` text
Client
  ↓
Worker
  ↓
Parse HTTP
  ↓
Route matching
  ↓
Authentication
  ↓
Load balancing
  ↓
Backend
  ↓
Response
  ↓
Worker
  ↓
Client
```

The core interview concepts to master are:

1.  Master vs worker
2.  Worker vs backend
3.  Route vs backend
4.  Round Robin vs Least Connections
5.  `fork()` and process memory
6.  System V shared memory
7.  Semaphores/mutexes
8.  `config_rwlock`
9.  `fcntl` file locking
10. FIFO + monitor
11. Reverse proxying / two TCP connections
12. Authentication vs authorization
13. Password hashing
14. Statistics
15. `active_connections`
16. `volatile`
17. Hot configuration reload
18. Worker respawning
19. Backend pools / gradual rollout
20. Database/schema migration

------------------------------------------------------------------------

## Navigation chain

The discussion evolved as a chain of doubts:

**[Project overview](#q01)**\
→ [Purpose of components](#q02)\
→ [Route matching/backend selection](#q03)\
→ [Why monitor](#q04)\
→ [Backend pools](#q05)\
→ [Why multiple pools](#q06)\
→ [Gradual rollout](#q07)\
→ [Data migration](#q08)\
→ [Log rotation](#q09)\
→ [Problem after fork](#q10)\
→ [Locks](#q11)\
→ [Lock examples](#q12)\
→ [Routes](#q13)\
→ [active_connections/volatile](#q14)\
→ [Workers vs backends](#q15)\
→ [Authentication](#q16)\
→ [Statistics](#q17)
