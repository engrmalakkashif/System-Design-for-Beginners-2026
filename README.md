# System Design for Beginners 2026

A practical, beginner-to-production learning repository covering system design fundamentals, AWS architecture, DevOps implementation, hands-on labs, and interview preparation.

## What You Will Learn

- Design scalable, highly available, and fault-tolerant systems
- Connect application architecture with AWS, Linux, Docker, Jenkins, networking, and databases
- Compare monoliths, microservices, SQL, NoSQL, replication, caching, and sharding
- Understand request flows, resource limits, deployment, scaling, security, and observability
- Practice architecture decisions through production-style scenarios and labs
- Prepare for technical interviews with focused questions and revision challenges

## Learning Path

| No. | Topic | Notes |
|---:|---|---|
| 01 | Introduction to System Design | [Read notes](notes/01-introduction.md) |
| 02 | What Is a Server and Where Does Code Live? | [Read notes](notes/02-server-and-code-lifecycle.md) |
| 03 | CPU and RAM Limitations | [Read notes](notes/03-cpu-and-ram.md) |
| 04 | Load Balancing | [Read notes](notes/04-load-balancing.md) |
| 05 | Database Overloading | [Read notes](notes/05-database-overloading.md) |
| 06 | Caching | [Read notes](notes/06-caching.md) |
| 07 | Replication and Backups | [Read notes](notes/07-replication-and-backups.md) |
| 08 | High-Level Design vs. Low-Level Design | [Read notes](notes/08-hld-vs-lld.md) |
| 09 | Functional vs. Non-Functional Requirements | [Read notes](notes/09-functional-vs-nonfunctional.md) |
| 10 | Five Questions Before Designing a System | [Read notes](notes/10-five-questions.md) |
| 11 | Monolithic vs. Microservice Architecture | [Read notes](notes/11-monolith-vs-microservices.md) |
| 12 | Vertical vs. Horizontal Scaling | [Read notes](notes/12-scaling.md) |
| 13 | How a Request Travels Through a System | [Read notes](notes/13-request-flow.md) |
| 14 | Health Checks and Traffic Distribution | [Read notes](notes/14-health-checks-and-distribution.md) |
| 15 | Stateless vs. Stateful Architecture | [Read notes](notes/15-stateless-vs-stateful.md) |
| 16 | Data Modeling and Access Patterns | [Read notes](notes/16-data-modeling.md) |
| 17 | SQL vs. NoSQL Databases | [Read notes](notes/17-sql-vs-nosql.md) |
| 18 | Database Indexes | [Read notes](notes/18-database-indexes.md) |
| 19 | Caching with Redis | [Read notes](notes/19-redis-caching.md) |
| 20 | Database Replication | [Read notes](notes/20-database-replication.md) |
| 21 | Database Sharding | [Read notes](notes/21-database-sharding.md) |
| 22 | Putting Everything Together | [Read notes](notes/22-production-architecture.md) |

## Recommended Study Method

1. Read the fundamentals and study the everyday-life analogy.
2. Draw the architecture and request flow from memory.
3. Complete the local Docker or AWS lab when applicable.
4. Explain the topic aloud in 60 seconds.
5. Answer the interview questions without referring to the notes.
6. Complete the final revision challenge before moving to the next topic.

## Repository Structure

```text
.
├── README.md
└── notes/
    ├── 01-introduction.md
    ├── 02-server-and-code-lifecycle.md
    ├── ...
    └── 22-production-architecture.md
```

Each topic follows a consistent structure:

- Concept and fundamentals
- Real-world analogy
- Technical architecture and request flow
- Practical use cases
- AWS and DevOps implementation
- Production scenario
- Hands-on lab
- Advantages, disadvantages, and best practices
- Interview preparation
- Revision notes and practical challenge

## Suggested Prerequisites

Basic familiarity with any of the following is helpful:

- HTTP, DNS, TCP, IP, and TLS
- Linux commands and file permissions
- Git and Docker
- SQL databases
- EC2, networking, or one cloud provider

No prior distributed-systems experience is required.

## Start Learning

Begin with [Introduction to System Design](notes/01-introduction.md), then continue through the topics in numerical order.

## Contribution

Corrections, diagrams, labs, and improvements are welcome. Keep contributions beginner-friendly, technically accurate, and consistent with the repository's established topic structure.
