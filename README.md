Absolutely. For this role, I’d avoid building a generic “FastAPI CRUD + Docker” project. The strongest portfolio project would look like a mini production-grade health/fitness AI platform that demonstrates almost every requirement in the JD.

Here’s a high-end prompt you can give to ChatGPT/Cursor/Claude to generate the project architecture and implementation plan:

Build a Production-Grade Health & Fitness AI Backend Platform

Act as a Staff Backend Engineer + AWS Solutions Architect + DevOps Engineer + Security Engineer + AI Platform Engineer.

I am preparing for a Backend & DevOps Engineer role at a sports technology company building a D2C health, fitness, lifestyle, and AI product.

I want to build a high-end, production-grade portfolio project that demonstrates the exact engineering capabilities expected from this role.

Do NOT build a simple CRUD application.

Build something that looks like an early-stage startup's real production backend, with strong architecture, security, observability, asynchronous processing, infrastructure-as-code, CI/CD, PostgreSQL optimization, vector search, authentication, RBAC, and LLM integration.

PROJECT: NEXUS — AI-Powered Fitness & Health Intelligence Platform

Build a cloud-native backend platform where users can:

Create an account and authenticate securely.
Maintain their fitness profile.
Record workouts, activities, health metrics, goals, and lifestyle data.
Upload health/fitness documents or data files.
Ask an AI fitness/health assistant questions.
Receive personalized insights generated from their data.
Search their historical fitness information using semantic/vector search.
Trigger asynchronous processing jobs.
Receive AI-generated recommendations.
View activity and AI processing history.
Allow administrators/operators to manage users, jobs, and platform health.

The system should be architected as if it will eventually serve 100,000+ users, even though the initial deployment can remain small and cost-conscious.

CORE TECHNOLOGY STACK

Backend:

Python 3.12+
FastAPI
Pydantic v2
SQLAlchemy 2.x
Alembic
PostgreSQL
pgvector
PostGIS where useful
Redis only where genuinely justified
boto3
pytest
httpx
Ruff
MyPy

AWS:

ECS Fargate
Application Load Balancer
ECR
RDS PostgreSQL
S3
SQS
Lambda where appropriate
CloudWatch
CloudWatch Logs
CloudWatch Alarms
IAM
KMS
Secrets Manager
VPC
Private/Public subnets
NAT Gateway
Security Groups
Route 53 where appropriate
AWS WAF where appropriate

Infrastructure:

Terraform
Remote Terraform state
Modular Terraform architecture
Environment separation: dev / staging / production

CI/CD:

GitHub Actions
Docker
Multi-stage Docker builds
Automated tests
Linting
Type checking
Security scanning
Terraform validation
Terraform plan
Deployment pipeline
Database migration strategy
Rollback strategy

AI:

OpenAI-compatible LLM API abstraction
Embeddings
RAG
pgvector
Prompt versioning
Retrieval pipeline
AI response validation
Guardrails
Token/cost tracking
LLM observability
ARCHITECTURE

Use a modular monolith initially rather than prematurely creating dozens of microservices.

The architecture should be designed so that high-load components can later be extracted into independent services.

Recommended high-level architecture:

Internet
↓
Route 53
↓
AWS WAF
↓
Application Load Balancer
↓
ECS Fargate
↓
FastAPI Backend
↓
├── PostgreSQL / RDS
├── S3
├── SQS
├── KMS
├── Secrets Manager
├── CloudWatch
└── External LLM Provider

Asynchronous processing:

FastAPI
↓
SQS
↓
Worker
↓
Processing Pipeline
↓
PostgreSQL / S3 / Vector Store

Design the system around API + asynchronous workers + persistent data + AI pipeline.

BACKEND ARCHITECTURE

Use clean architecture principles without unnecessary abstraction.

Suggested structure:

app/
├── api/
│ ├── routes/
│ ├── dependencies/
│ └── middleware/
├── core/
│ ├── config.py
│ ├── security.py
│ ├── logging.py
│ └── telemetry.py
├── domain/
│ ├── users/
│ ├── workouts/
│ ├── health/
│ ├── documents/
│ ├── ai/
│ └── jobs/
├── services/
├── repositories/
├── models/
├── schemas/
├── workers/
├── integrations/
│ ├── aws/
│ ├── llm/
│ └── embeddings/
├── db/
│ ├── session.py
│ ├── models.py
│ └── migrations/
└── main.py

tests/
├── unit/
├── integration/
├── api/
├── workers/
└── security/

terraform/
├── modules/
│ ├── vpc/
│ ├── ecs/
│ ├── rds/
│ ├── s3/
│ ├── sqs/
│ ├── iam/
│ ├── kms/
│ ├── alb/
│ ├── cloudwatch/
│ └── secrets/
├── environments/
│ ├── dev/
│ ├── staging/
│ └── production/
└── backend.tf

.github/
└── workflows/

docs/
├── architecture.md
├── api.md
├── security.md
├── deployment.md
├── database.md
├── observability.md
└── decisions/

DATABASE DESIGN

Design a serious PostgreSQL schema.

Include entities such as:

users
roles
permissions
user_roles
fitness_profiles
goals
workouts
workout_exercises
health_metrics
activity_events
documents
document_chunks
embeddings
ai_conversations
ai_messages
ai_requests
recommendations
background_jobs
audit_logs

Use:

UUID primary keys
created_at / updated_at
appropriate foreign keys
CHECK constraints
unique constraints
indexes
composite indexes
partial indexes where useful
JSONB for genuinely semi-structured data
normalized relational data where appropriate
database-level constraints

Explain every important schema decision.

Demonstrate PostgreSQL optimization.

Include examples of:

EXPLAIN ANALYZE
index selection
query optimization
pagination
cursor-based pagination
connection pooling
transaction boundaries
VECTOR SEARCH

Use PostgreSQL + pgvector rather than immediately introducing another vector database.

Build:

Document
→ Chunk
→ Embedding
→ pgvector

Implement:

embedding generation
vector storage
cosine similarity search
metadata filtering
hybrid retrieval where appropriate
top-K retrieval
similarity thresholds

Example:

User question
↓
Generate embedding
↓
pgvector similarity search
↓
Retrieve relevant fitness/health context
↓
Build grounded prompt
↓
LLM
↓
Validate response
↓
Return answer

Do NOT allow the LLM to blindly generate health claims.

Implement guardrails and clearly separate:

user-provided facts
retrieved context
model-generated suggestions
ASYNCHRONOUS PROCESSING

Use SQS extensively where asynchronous processing makes sense.

Example:

POST /documents
↓
Store metadata
↓
Generate presigned S3 upload URL
↓
Client uploads document
↓
API publishes SQS message
↓
Worker consumes message
↓
Extract content
↓
Chunk content
↓
Generate embeddings
↓
Store vectors
↓
Mark processing complete

Implement:

idempotency
visibility timeout
retry handling
dead-letter queue
exponential backoff
job status tracking
correlation IDs
structured logging

Workers must be safe to retry.

Demonstrate how duplicate SQS deliveries are handled.

LLM INTEGRATION

Create an LLM provider abstraction.

For example:

LLMProvider
├── OpenAIProvider
└── MockLLMProvider

Do not tightly couple business logic to one provider.

Implement:

prompt templates
prompt versioning
structured outputs
timeout handling
retry strategy
rate limiting
token tracking
estimated cost tracking
request/response metadata
failure handling
fallback behavior

AI endpoint example:

POST /api/v1/ai/ask

Flow:

Authentication
↓
Authorization
↓
Input validation
↓
Rate limiting
↓
Retrieve relevant user context
↓
Vector search
↓
Construct prompt
↓
LLM request
↓
Structured output validation
↓
Safety/guardrail validation
↓
Persist AI request
↓
Return response

SECURITY

Treat security as a first-class architectural concern.

Implement:

OAuth2/JWT authentication
short-lived access tokens
refresh token strategy
password hashing if local authentication is included
RBAC
resource-level authorization
least-privilege IAM
KMS encryption
Secrets Manager
encrypted S3
encrypted RDS
TLS
secure HTTP headers
CORS configuration
request validation
rate limiting
audit logs
PII protection
no secrets in Git
no secrets in Docker images
no credentials in Terraform source
private RDS subnet
ECS task roles
security groups with minimal access
S3 bucket policies
database credential rotation strategy

Explicitly document the threat model.

Include protection against:

IDOR
privilege escalation
JWT misuse
SQL injection
SSRF
insecure file uploads
prompt injection
sensitive data leakage
excessive API requests
OBSERVABILITY

Implement production-grade observability.

Every request should have:

request ID
correlation ID
structured JSON logs
timestamp
endpoint
HTTP status
latency
user ID where safe
error information

Add:

application metrics
request latency
request count
error rate
SQS queue depth
worker failures
database connection health
LLM latency
LLM failures
token usage
estimated AI cost

Use CloudWatch.

Create alarms for:

elevated 5xx rate
high API latency
unhealthy ECS tasks
high SQS backlog
DLQ messages
RDS CPU
RDS storage
worker failures

Include health endpoints:

GET /health/live
GET /health/ready

API DESIGN

Build versioned REST APIs.

Example:

/api/v1/auth
/api/v1/users
/api/v1/profile
/api/v1/workouts
/api/v1/health
/api/v1/documents
/api/v1/jobs
/api/v1/ai
/api/v1/admin

Implement:

pagination
filtering
sorting
consistent error format
HTTP status correctness
idempotency keys for important POST operations
request validation
OpenAPI documentation
API versioning

Do not expose internal database models directly.

TERRAFORM

Infrastructure must be fully reproducible.

Terraform should provision:

VPC
subnets
route tables
NAT
security groups
ECS cluster
ECS service
ECS task definition
ALB
ECR
RDS
S3
SQS
DLQ
IAM roles
KMS keys
Secrets Manager
CloudWatch logs
CloudWatch alarms

Use reusable modules.

Avoid overly broad IAM policies such as AdministratorAccess.

Every IAM policy should demonstrate least privilege.

CI/CD

Create a professional GitHub Actions pipeline.

Pull Request:

Ruff
MyPy
pytest
coverage
dependency vulnerability scan
Docker build
Terraform fmt
Terraform validate
Terraform plan

Main branch:

Run tests
Build Docker image
Scan image
Push image to ECR
Deploy infrastructure
Run database migrations safely
Deploy ECS service
Run smoke tests
Verify health
Roll back if deployment fails

Use GitHub OIDC to authenticate to AWS instead of long-lived AWS credentials.

DOCKER

Create a secure multi-stage Dockerfile.

Requirements:

slim Python base image
non-root user
minimal packages
dependency caching
deterministic builds
health check
no secrets
optimized image size

Also create:

docker-compose.yml

for local development with:

API
PostgreSQL
LocalStack if useful
worker
TESTING

Target high-quality automated tests.

Include:

Unit tests:

services
repositories
authorization
validation
AI orchestration

Integration tests:

PostgreSQL
SQS
S3
worker processing

API tests:

authentication
RBAC
CRUD
pagination
error handling

Security tests:

unauthorized access
IDOR
role escalation
invalid JWT
malformed input
rate limiting

Test asynchronous behavior.

Target 80%+ meaningful coverage, not artificial coverage.

RESILIENCY

Design for failure.

Explicitly handle:

database unavailable
SQS unavailable
LLM timeout
LLM rate limit
duplicate messages
worker crash
S3 failure
ECS task replacement
network failures
partial processing
deployment failure

Explain:

retry policy
exponential backoff
circuit breaker considerations
dead-letter queues
idempotency
graceful degradation
API EXAMPLE

Create an endpoint:

POST /api/v1/ai/ask

Request:

{
"question": "Why has my running performance decreased recently?"
}

Backend:

Authenticate user.
Authorize access.
Load relevant fitness profile.
Retrieve recent workouts.
Retrieve relevant health metrics.
Perform vector search.
Construct grounded context.
Send request to LLM.
Validate structured response.
Apply safety guardrails.
Persist AI request metadata.
Return response.

The response should contain:

{
"answer": "...",
"sources": [],
"confidence": "...",
"disclaimer": "...",
"request_id": "..."
}

Do not pretend the system is providing medical diagnosis.

ADMIN / OPERATIONS

Create internal APIs for operators:

user lookup
job inspection
failed job retry
DLQ inspection
AI request inspection
audit log search
system health

Protect all admin APIs with strict RBAC.

FRONTEND

Create a minimal but polished frontend only if useful.

Use:

Next.js
TypeScript
Tailwind

It should demonstrate:

login
dashboard
workout history
health metrics
AI assistant
document upload
job processing status

The backend remains the primary focus.

DOCUMENTATION

Documentation should look like a real engineering project.

Create:

README
Architecture diagram
AWS architecture explanation
Database ERD
API documentation
Security model
Threat model
Terraform documentation
CI/CD documentation
Observability guide
Local development guide
Deployment guide
Disaster recovery strategy
Database migration strategy
AI/RAG architecture
ADRs

Include Architecture Decision Records for decisions such as:

Why FastAPI?
Why ECS Fargate?
Why PostgreSQL?
Why pgvector instead of a dedicated vector DB?
Why SQS?
Why modular monolith?
When should services be extracted?
Why Terraform?
Why GitHub OIDC?
SYSTEM DESIGN

Include an architecture section explaining:

Scalability

How the platform scales from:

100 users
→ 1,000 users
→ 10,000 users
→ 100,000+ users

Discuss:

horizontal ECS scaling
database scaling
read replicas
caching
SQS worker scaling
S3
connection pooling
async processing
Reliability

Explain:

availability
fault isolation
retries
DLQs
health checks
deployment strategy
backups
recovery
Security

Explain:

IAM
network isolation
encryption
authentication
authorization
secrets
auditability
Cost

Explain approximate cost drivers and how to keep the MVP inexpensive.

DEVELOPMENT APPROACH

Do NOT dump the entire project at once.

Build it in professional engineering phases.

Phase 1:
Architecture + requirements + repository structure.

Phase 2:
FastAPI foundation + PostgreSQL + migrations.

Phase 3:
Authentication + RBAC.

Phase 4:
Core fitness APIs.

Phase 5:
SQS + workers.

Phase 6:
S3 document processing.

Phase 7:
pgvector + embeddings + RAG.

Phase 8:
LLM integration + guardrails.

Phase 9:
Terraform AWS infrastructure.

Phase 10:
Docker + ECS deployment.

Phase 11:
CI/CD.

Phase 12:
Observability + alarms.

Phase 13:
Security hardening.

Phase 14:
Testing + load testing.

Phase 15:
Documentation + architecture diagrams.

At every phase:

explain the engineering decisions
produce production-quality code
explain tradeoffs
show how to test it
identify failure scenarios
identify security concerns
identify scalability concerns
ENGINEERING QUALITY BAR

Pretend this repository will be reviewed by:

Staff Backend Engineer
AWS Solutions Architect
DevOps Lead
Security Engineer
CTO

Avoid:

tutorial-quality code
excessive abstractions
unnecessary microservices
hardcoded credentials
broad IAM permissions
giant functions
duplicated business logic
untyped code
fake observability
fake infrastructure
meaningless tests
over-engineering

Prioritize:

simplicity
correctness
security
maintainability
observability
testability
reliability
scalability
IMPORTANT

I want this project to be resume-worthy and interview-worthy, not merely functional.

For every major architectural decision, explain:

Problem → Options → Decision → Tradeoff → Why this is appropriate for an early-stage production system.

Also provide realistic interview questions based on the project, including:

AWS
ECS
VPC
IAM
Terraform
PostgreSQL
SQS
FastAPI
distributed systems
security
observability
CI/CD
LLM/RAG
system design

For each question provide a strong engineering answer.

At the end, generate a section called:

"How this project maps to the Netrin Backend & DevOps Engineer JD"

Create a table mapping every responsibility and requirement from the job description to a concrete feature in this project.

The final project should make it obvious that I understand:

Python + FastAPI + AWS + Terraform + PostgreSQL + SQS + ECS + Docker + CI/CD + IAM + KMS + VPC + observability + security + RBAC + LLM APIs + RAG + pgvector + production backend engineering.

Start with the system architecture, repository structure, database design, AWS architecture, and implementation roadmap before writing the application code.

Why this is a strong choice

This project deliberately hits almost the entire JD:

Netrin requirement	Project evidence
Python/FastAPI	Core REST backend
AWS	ECS, RDS, S3, SQS, IAM, KMS, VPC
Terraform	Entire infrastructure
PostgreSQL	Relational + JSONB + optimization
Event-driven systems	SQS + workers
ECS Fargate	Containerized production deployment
CI/CD	GitHub Actions + ECR + ECS
Security	JWT/OAuth2, RBAC, IAM, KMS, WAF
Observability	CloudWatch + structured logs + alarms
LLM APIs	AI service abstraction
RAG	pgvector + embeddings
Vector DB	PostgreSQL/pgvector
Health/fitness	Domain itself
Async processing	Document/embedding pipeline
Production engineering	Retries, DLQs, idempotency, migrations
System design	Scalability + reliability + tradeoffs

One important recommendation: don't try to make every component enormous. For a 2–4 year backend candidate, a well-designed modular monolith with genuinely working AWS infrastructure is more impressive than pretending to have 15 microservices. The interview value comes from being able to explain why you chose each component and what you'd change at 10× scale.
