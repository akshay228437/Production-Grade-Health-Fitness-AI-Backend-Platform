No problem. Without wearable hardware, I would simplify the project while keeping the same Netrin-relevant engineering problems.

The key is: the application should simulate the data source, while your project focuses on production implementation. You can generate realistic workout/heart-rate session data through an ingestion client instead of building BLE/device integration.

Production-Grade Fitness Data Platform
Project Goal

Build a production-grade backend and AWS platform for a D2C fitness and performance product.

The project simulates a fitness application where users submit workout and physiological session data.

The application itself is intentionally minimal.

The primary objective is to demonstrate ownership of:

AWS infrastructure
Terraform
Containerized workloads
ECS Fargate
PostgreSQL/RDS
S3
SQS
IAM
KMS
CI/CD
Security
Authentication and RBAC
Event-driven processing
Data pipelines
Observability
Reliability
Failure recovery
Database design
LLM service integration
Production operations

The project should feel like the backend/platform of a real early-stage D2C health and fitness company.

1. Core User Journey

The core journey is:

User creates an account → submits a workout/session → backend accepts the data → asynchronous processing begins → fitness metrics are calculated → optional AI analysis is generated → results become available to the user.

The user-facing application can be extremely simple.

The important system is:

Client
  ↓
API
  ↓
PostgreSQL / S3
  ↓
SQS
  ↓
Worker
  ↓
Processing
  ↓
Metrics
  ↓
LLM / Embeddings
  ↓
Results


The project should focus on everything that makes this pipeline reliable and production-ready.

2. Data Source

There is no wearable.

Instead, create a small data ingestion client/simulator that behaves like a real mobile application sending workout data.

It should generate realistic sessions containing:

Workout ID
User ID
Session ID
Timestamp
Workout type
Duration
Heart-rate samples
Calories
Distance
Pace
Recovery-related metrics
Optional GPS points
Device/source metadata

Example:

{
  "session_id": "sess_123",
  "workout_type": "running",
  "started_at": "2026-08-13T06:30:00Z",
  "duration_seconds": 3600,
  "samples": [
    {
      "timestamp": "2026-08-13T06:30:01Z",
      "heart_rate": 132
    },
    {
      "timestamp": "2026-08-13T06:30:02Z",
      "heart_rate": 135
    }
  ]
}


The simulator exists only to produce realistic workload for the backend.

Do not spend significant time building the simulator.

3. Why This Data Source

The project deliberately separates:

Data acquisition

Simple client/simulator.

Data platform

The actual project.

This allows the engineering effort to focus on:

ingestion
reliability
asynchronous processing
infrastructure
security
deployment
observability
scalability

rather than spending most of the project on mobile development or hardware integration.

4. Initial Scale

Assume:

10,000 registered users
2,000 daily active users
Average 1–2 workouts per active user/day
Potential bursts around common workout times
Small engineering team
Cost-sensitive infrastructure
Initial AWS environment should be capable of growth without premature complexity

Do not design for millions of users.

Do not introduce Kubernetes, Kafka, or microservices simply to demonstrate knowledge.

The system should be:

Small enough for a founding team to operate, but structured enough to evolve into a larger production platform.

5. System Architecture

Target architecture:

                   Client / Simulator
                           │
                           ▼
                    Application Load
                       Balancer
                           │
                           ▼
                    ECS Fargate API
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
            RDS           S3            SQS
         PostgreSQL     Raw Data        Queue
                                         │
                                         ▼
                                   ECS Fargate
                                      Worker
                                         │
                            ┌────────────┼────────────┐
                            ▼            ▼            ▼
                           RDS          S3          LLM API
                         Metrics       Data


Terraform manages the infrastructure.

CI/CD manages application and infrastructure changes.

Observability covers the complete flow.

6. API Service

Use FastAPI.

But the API should remain intentionally small.

Example responsibilities:

POST /sessions
POST /sessions/{id}/data
POST /sessions/{id}/complete

GET /sessions/{id}
GET /sessions/{id}/status
GET /sessions/{id}/insights


The point is not to build dozens of endpoints.

The API's main responsibility is:

Accept, validate, authorize and persist incoming workload, then hand expensive processing to asynchronous infrastructure.

7. Ingestion Design

Do not send a huge workout payload blindly.

Implement chunked ingestion.

Example:

Workout Session
      │
      ├── Chunk 001
      ├── Chunk 002
      ├── Chunk 003
      └── Chunk 004


Each chunk contains:

session_id
chunk_id
sequence_number
schema_version
timestamps
measurements
checksum

This gives you realistic distributed-system problems.

8. Idempotency

This is mandatory.

Simulate:

Client
 ↓
Upload chunk
 ↓
Server processes
 ↓
Network timeout
 ↓
Client retries


The system must safely handle duplicate requests.

For example:

chunk_id = abc123


must represent one logical chunk even if the request arrives five times.

You should be able to explain:

idempotency key
unique constraints
transaction boundaries
retry behavior
duplicate processing
9. Asynchronous Architecture

The API should not perform expensive processing synchronously.

Instead:

API
 ↓
Persist ingestion
 ↓
SQS
 ↓
Worker
 ↓
Process


Use SQS for workloads such as:

Session processing
Metric calculation
AI analysis
Embedding generation

Keep the queue design simple.

Start with:

session-processing


and add additional queues only when there is a clear operational reason.

10. Worker Architecture

Run workers on ECS Fargate.

Worker responsibilities:

Consume SQS message.
Retrieve required data.
Validate processing state.
Process data.
Calculate deterministic metrics.
Persist results.
Trigger downstream processing if required.
Acknowledge the message.

The worker must be safe to retry.

11. Failure Semantics

Design around at-least-once delivery.

Assume:

A message can be delivered more than once.

Therefore processing must be idempotent.

Implement:

Visibility timeout
Retry policy
Dead-letter queue
Maximum receive count
Failure logging
Queue monitoring

Test:

Worker starts
 ↓
Processes session
 ↓
Crashes before acknowledgement
 ↓
Message becomes visible again
 ↓
Another worker processes it


The final result must remain correct.

12. PostgreSQL

Use Amazon RDS PostgreSQL.

Core tables:

users
workout_sessions
session_chunks
processing_jobs
metrics
insights


Potentially:

embeddings


if vector search is justified.

Focus on:

Constraints
Foreign keys
Indexes
Transactions
Query performance
Connection pooling
Migrations
Backup/recovery
13. Raw Data vs Queryable Data

Do not put everything into PostgreSQL.

Use:

S3

For:

Raw session payloads
Original imported data
Large historical datasets
Reprocessing inputs
PostgreSQL

For:

Users
Session metadata
Processing status
Aggregated metrics
Insights
Queryable application data

This gives the project a realistic data architecture.

14. PostgreSQL + pgvector

Introduce pgvector only after the core system works.

Potential use case:

Workout history
       ↓
Structured summary
       ↓
Embedding
       ↓
pgvector
       ↓
Semantic retrieval
       ↓
LLM


For example, the system could retrieve similar historical workouts when generating an insight.

But the project should explicitly justify:

Why do we need vector search?

If there isn't a real requirement, don't use it.

15. LLM Integration

LLM integration is a secondary dependency.

Example:

Workout
 ↓
Deterministic metrics
 ↓
Structured workout summary
 ↓
LLM
 ↓
Natural-language insight


The LLM should never be responsible for core numerical calculations.

The backend calculates metrics.

The LLM explains or summarizes them.

Design for:

Timeout
Retry
Rate limits
Provider failure
Cost limits
Structured output
Output validation
Prompt versioning
Logging without sensitive payloads
16. LLM Failure

If the LLM provider goes down:

Workout
 ↓
Metrics calculated
 ↓
LLM unavailable


The workout must remain successfully processed.

The AI job should become:

PENDING


or:

RETRYING


rather than causing the entire workout processing pipeline to fail.

This demonstrates dependency isolation.

17. AWS Infrastructure

Use:

VPC
ECS Fargate
ALB
RDS PostgreSQL
S3
SQS
IAM
KMS
CloudWatch
Secrets Manager
ECR


Use Lambda only where it genuinely simplifies a workload.

Do not force every AWS service into the architecture.

18. VPC Design

Use:

VPC
│
├── Public Subnets
│    └── ALB
│
├── Private App Subnets
│    └── ECS
│
└── Private DB Subnets
     └── RDS


Consider:

Availability zones
Route tables
NAT
Security groups
Database isolation
Egress
Load balancer access

RDS should not be publicly accessible.

19. ECS

Create two services:

api-service
worker-service


The services should be independently scalable.

For example:

API:
scale based on request load

Worker:
scale based on SQS backlog


This is more important than simply running everything in containers.

20. Terraform

Everything important must be provisioned through Terraform.

Suggested structure:

terraform/
├── modules/
│   ├── networking/
│   ├── ecs/
│   ├── rds/
│   ├── s3/
│   ├── sqs/
│   ├── iam/
│   ├── kms/
│   └── observability/
│
└── environments/
    ├── dev/
    ├── staging/
    └── prod/


Focus on:

State
Modules
Variables
Outputs
Environment separation
IAM
Dependency management
Drift
Safe changes
Plan/apply workflow
21. CI/CD

Build a complete pipeline:

Git push
   ↓
Lint
   ↓
Unit tests
   ↓
Security checks
   ↓
Docker build
   ↓
Container scan
   ↓
Push to ECR
   ↓
Terraform validation
   ↓
Terraform plan
   ↓
Deploy staging
   ↓
Smoke tests
   ↓
Production approval
   ↓
Production deployment


CI should have the minimum AWS permissions required.

Do not use:

AdministratorAccess


for the deployment pipeline.

22. Deployment

Use ECS rolling deployments initially.

Implement:

Health checks
Graceful shutdown
Deployment monitoring
Failed deployment detection
Rollback strategy

The application should be able to terminate safely without losing in-flight work.

For workers, explicitly handle shutdown signals so that messages are not incorrectly acknowledged.

23. Database Deployment

Use backward-compatible migrations.

Preferred strategy:

Expand
 ↓
Deploy
 ↓
Migrate data
 ↓
Switch application
 ↓
Contract


Do not couple destructive database changes directly to an application deployment.

24. Security

Treat fitness data as sensitive.

Examples:

Heart-rate data
Sleep data
Workout history
Recovery metrics
Location
Identity-linked performance data

Implement:

TLS
Encryption at rest
IAM
KMS
Secrets management
RBAC
Resource-level authorization
Private networking
Sensitive-data redaction
Auditability

Never log:

Passwords
JWTs
API keys
Secrets
Full sensitive workout payloads
25. Authentication & Authorization

Use an identity provider or OAuth2/JWT-based architecture.

Roles:

USER
COACH
ADMIN
SERVICE


But authorization must happen at the resource level.

For example:

GET /sessions/123


must verify that session 123 belongs to the authenticated user.

Knowing the UUID must not grant access.

26. Observability

Implement observability from the beginning.

API

Track:

Request rate
4xx
5xx
Latency
p95/p99
Dependency failures
ECS

Track:

CPU
Memory
Task restarts
Deployment failures
SQS

Track:

Queue depth
Message age
Processing failures
DLQ messages
RDS

Track:

CPU
Memory
Connections
Storage
Query latency
Slow queries
27. End-to-End Correlation

Every request/session should have identifiers.

Example:

request_id
session_id
chunk_id
job_id


Given:

session_id = sess_123


you should be able to trace:

API request
 ↓
database ingestion
 ↓
SQS message
 ↓
worker
 ↓
metrics processing
 ↓
LLM job
 ↓
final insight


This is one of the most important production engineering demonstrations in the project.

28. Alerting

Create actionable alerts.

Examples:

API 5xx rate > threshold
SQS oldest message age > threshold
DLQ message count > 0
ECS task restart spike
RDS storage low
RDS connection saturation
Deployment failure


Avoid alerting on every small anomaly.

An alert should mean:

An engineer should investigate this.

29. Disaster Recovery

Define:

RPO

How much data can be lost?

RTO

How quickly must the platform recover?

Then implement and test:

RDS backups
S3 protection
Database restore
Terraform recreation
Secret recovery
Queue recovery

Do not consider disaster recovery complete until you have performed an actual restore test.

30. Production Failure Scenarios

You must intentionally test:

API failure

ECS task crashes.

Worker failure

Worker dies during processing.

SQS failure

Messages accumulate.

Poison message

One malformed session repeatedly fails.

Duplicate request

Client sends the same chunk five times.

LLM failure

Provider becomes unavailable.

Database failure

RDS becomes unavailable.

Deployment failure

New version causes elevated 5xx.

Migration failure

Database migration fails during deployment.

Credential failure

A service loses an IAM permission.

Data deletion

User requests deletion while processing jobs remain queued.

For every scenario document:

Detection
Investigation
Mitigation
Recovery
Verification
Root Cause
Prevention

31. Cost Awareness

The system should be designed for a startup.

Track:

ECS costs
RDS costs
NAT costs
S3 storage
SQS
CloudWatch
LLM usage

Explicitly identify expensive components.

For example:

"NAT Gateway is operationally convenient but can become disproportionately expensive at small scale."

Then decide whether the trade-off is justified.

32. What You Actually Build

The implementation should contain approximately:

Application
Small FastAPI service
Small worker
Minimal authentication
Session ingestion
Session status
Result retrieval
Infrastructure
Terraform
VPC
ECS
RDS
S3
SQS
IAM
KMS
ECR
Observability
DevOps
Docker
CI/CD
Terraform pipeline
Deployment strategy
Rollback
Environment separation
Operations
Dashboards
Alerts
Logs
Failure drills
Runbooks
Backup/restore
AI
One LLM integration
Optional pgvector
Async AI processing
33. What You Do NOT Build

Do not spend time on:

Complex frontend
Social features
Chat
Payments
Marketplace
Complex workout recommendation algorithms
Wearable integration
BLE
Custom hardware
Kubernetes
Kafka
Microservices everywhere

The application is intentionally small.

The platform is the project.

34. Final Success Criteria

The project is complete when:

Client
 ↓
Submit realistic workout data
 ↓
API
 ↓
S3 + PostgreSQL
 ↓
SQS
 ↓
ECS worker
 ↓
Metrics
 ↓
LLM
 ↓
Result


works end-to-end.

But the real definition of done is that you can demonstrate:

Duplicate requests are safe.
Workers can fail without losing data.
SQS retries correctly.
Poison messages reach DLQ.
API and workers scale independently.
RDS is private.
IAM follows least privilege.
Sensitive data isn't leaked into logs.
CI/CD deploys repeatably.
Deployments can be rolled back.
Terraform can recreate infrastructure.
Database backups can actually be restored.
You can trace one session through the entire platform.
You can diagnose a production incident.
LLM failure does not destroy the core processing pipeline.

The final portfolio positioning should be:

Designed, provisioned, deployed and operated a production-grade AWS event-driven fitness data platform using Terraform, ECS Fargate, RDS PostgreSQL, S3, SQS, IAM/KMS and CI/CD, with asynchronous processing, observability, security and failure-recovery workflows.

This is the version I recommend you actually implement. It gives you a realistic workload without creating a second project around mobile/BLE/hardware, while still mapping very closely to the Netrin responsibilities: AWS → Terraform → ECS → RDS → SQS → PostgreSQL/pgvector → LLM dependencies → CI/CD → security → RBAC → observability → production operations.
