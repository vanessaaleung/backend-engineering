# Backend Engineering
- [Monolith vs Microservices Architecture](#monolith-vs-microservices-architecture)
- [Service to Service communication](#service-to-service-communication)
- [Scaling](#scaling)
- [Monitoring](#monitoring)
- [Functional Programming](#function-programming)

## Monolith vs Microservices Architecture
- Monolith: User Interface + Business Logic + Data Access Layer in a single code base
- Microservices: Split up Monolith, have a dedicated system and databse for each business logic
  - Benefits: independently scale a system which is heavily used
<img src="https://d1.awsstatic.com/Developer%20Marketing/containers/monolith_1-monolith-microservices.70b547e30e30b013051d58a93a6e35e77408a2a8.png" height="300px">
  
## Service to Service communication
- HTTP
- Hermes: more high performance than HTTP
- gRPC: remote procedure call

## Scaling
- Vertical scaling (bigger machines)
- Horizontal scaling (more machine)
- Virtual Machines: virtualizes the hardware
- Kubenetes: virtualizes the operating system

## Monitoring
- Logging: e.g. Google Cloud Logging
- Metrics: time-series data showing how things changed, e.g. Grafana (dashboard), pager duty (alerting)
- Distributed request tracing: shows how a request goes through different systems/services, how long does it take, e.g. Lightstep
<img src="https://images.saasworthy.com/lightstep_4904_screenshot_1574232008_0sov2.png" height="300px">

## Functional Programming
- Everything is immutable, avoids mutability
