# Microservices Architecture


Service oriented architecture (SOA) resulted in monolithic large services.

Microservices knows how to size a service.

## Design Principles

### High Cohesion
- Single thing done well
- Single focus

**Identify a single focus**. It could be:
- Business function which has its own clear inputs and outputs.
- Business domain where the microservice focus is in the form of creating, retrieving, updating, deleting (CRUD) data related to a specific part of the organization.

Overall to ensure your microservices have high cohesion, continuously question the design of microservices if a new microservice **has one reason to change**.
  
### Autonomous
- Independently changeable
- Independently deployable

**Loosely coupled.** Below things could help achieve this:
- Communication by network
  - Synchronous (client waiting for a reply)
  - Asynchronous (via message broker distributing messages)
- Technology agnostic API (i.e REST API with JSON data)
- Contracts between services (including API ans shared models)
- Avoid sharing between services (including databases, data should be shared by microservices calling each other and sending the data)
- Microservice ownership by team
  - responsibility to make autonomous
  - agreeing contracts between teams
  - responsible for long-term maintenance
  - collaborative development
  - concurrent development
- Versioning
  - Avoid breaking changes
  - Backwards compatibility
  - Integration tests
  - Have a versioning strategy
    - Concurrent versions if breaking changes cannot avoid
    - Coexisting endpoints (`/v1/xxx` and `/v2/xxx`)

### Business Domain Centric
- Represent business function/domain

### Resilience
- Embrace failure
- Default or degrade functionality

**Design system to fail fast and recover fast.** A hanging transaction or a delayed transaction might be as bad as seeing an error in the screen.

**Use timeouts to fail fast**

### Observable
- See system health
- Centralized logging and monitoring

### Automation
- Tools for test and feedback
- Tools for deployment

## Technology

### Synchronous Communication

#### Communication Protocols
- Remote procedure call (RPC)
- HTTP
- REST. CRUD using HTTP verbs (GET/POST/PUT/DELETE)

**NOTE: you could implement RPC over HTTP. REST works with HTTP.**

#### Payload Protocols
- JSON
- XML
- ProtocolBuffers

#### Synchronous Issues
- Both parties have to be available
- Performance issue due to network

### Asynchronous Communication

**Event Based Communication**
- Mitigate the need of client and service availability
- Decouple client and service

It normally uses **Message Queueing Protocol**, where:
- Events == Messages
- Who generates events == Publisher
- Who processes events == Subscriber

And messages are buffered by a **message broker**.

#### Asynchronous Issues
- Complicated
- Message broker could be another single point of failure
- Manage the messaging queue
  

