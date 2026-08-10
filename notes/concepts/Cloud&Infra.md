# Overview

- Infrastructure is the compute, networking, storage, databases, identity, delivery, and operational systems which run an application

- Cloud computing rents programmable infrastructure and managed services from a provider instead of owning every physical system

- Main service models:

    IaaS: Virtual machines, networks, and disks managed mostly by the customer

    PaaS: Application platform manages more runtime and deployment details

    SaaS: Complete software consumed by users

    FaaS/Serverless: Run functions or request handlers without managing long-lived servers directly

- "Serverless" still uses servers. The provider controls capacity, patching, and lifecycle while billing by usage or allocation

- Main goals are reliability, security, performance, scalability, recoverability, observability, and controlled cost

# Regions, Zones, and Edge Locations

- A region is a geographic area containing provider infrastructure

- An availability zone is an isolated failure domain inside a region

- Edge locations place caching or compute closer to users

- Multi-zone deployment protects against a single-zone failure. Multi-region deployment adds stronger geographic resilience but increases data consistency and operational complexity

- Choose regions based on user latency, legal/data residency, service availability, cost, and disaster-recovery requirements

- Keep application compute close to its database unless a deliberate distributed data architecture supports otherwise

# Compute Models

- Virtual Machine: Full operating-system instance with maximum control and maintenance responsibility

- Container: Isolated process package sharing the host kernel

- Managed container platform: Runs container images while the provider manages much of scheduling and infrastructure

- Function/serverless runtime: Starts code for events or requests, often with time, memory, filesystem, connection, and startup constraints

- Static hosting/CDN: Serves prebuilt files without application compute

- Choose the simplest model satisfying runtime duration, protocols, state, scaling, networking, and portability requirements

- Stateless application instances are easier to scale. Store durable state in databases, object storage, or other shared systems

# Networking

- An IP address identifies a network interface. A port identifies a listening service on that machine

- DNS maps domain names to records such as IP addresses or other names

- TLS encrypts transport and authenticates the server certificate. HTTPS is HTTP over TLS

- A Virtual Private Cloud/Network is a logically isolated provider network

- Subnets divide address space. Public subnets have direct internet routing; private subnets normally use controlled outbound gateways

- Firewalls/security groups restrict inbound and outbound traffic

- Load balancers distribute connections or HTTP requests across healthy instances

- Reverse proxies receive requests before the application and may terminate TLS, route paths, compress, cache, rate-limit, and enforce size/time limits

- NAT allows private systems to initiate internet traffic without accepting direct inbound connections

- Private endpoints and peering connect services without using the public internet path

- Zero Trust means identity and authorization are checked for each access rather than trusting everything on one network

# Storage

- Object storage holds blobs by key and is ideal for files, media, backups, logs, and static assets

- Block storage acts like a disk attached to compute

- File storage exposes shared hierarchical files through a network filesystem

- Database storage provides structured access, indexes, constraints, and query behavior

- Storage properties to understand:

    Durability: Probability data remains intact

    Availability: Probability the service can answer

    Consistency: What reads observe after writes

    IOPS/throughput/latency: Performance dimensions

    Lifecycle: Archival and deletion rules

- Encrypt data in transit and at rest, limit access, version important objects, and test restoration

# Scaling and Reliability

- Vertical scaling gives one machine more resources

- Horizontal scaling adds more instances

- Auto-scaling adjusts capacity from load or schedules but cannot fix slow queries, hot partitions, or an overloaded database automatically

- Redundancy removes single points of failure

- Health checks:

    Liveness: Is the process alive enough to restart decision-making?

    Readiness: Can this instance safely receive traffic?

- Timeout every network request. A missing timeout can exhaust workers during downstream failure

- Retry only transient and safe/idempotent operations, using exponential backoff and jitter

- Circuit breakers stop repeatedly calling a failing dependency and allow recovery checks

- Bulkheads isolate resource pools so one failing dependency does not consume everything

- Backpressure limits incoming work when capacity is full

- Graceful shutdown stops new traffic, finishes bounded in-flight work, releases resources, and exits before the platform deadline

# Disaster Recovery

- Backup: Recoverable copy separate from active data

- Replication: Live copy for availability; it can also copy bad writes and deletions

- RPO (Recovery Point Objective): Maximum acceptable data loss measured in time

- RTO (Recovery Time Objective): Maximum acceptable restoration time

- Disaster recovery designs include backup/restore, pilot light, warm standby, and active-active

- A backup is not proven until restoration is tested

- Document recovery ownership, credentials, DNS changes, data reconciliation, and rollback

# Infrastructure as Code

- Infrastructure as Code (IaC) declares infrastructure in version-controlled files

- Common tools include Terraform/OpenTofu, AWS CloudFormation/CDK, Pulumi, and provider-specific configuration

- IaC enables reviewable changes, repeatable environments, drift detection, and automated provisioning

- State records the mapping between declarations and real resources. Protect remote state because it may contain sensitive infrastructure data

- Review plans before applying them and separate environment configuration without copying entire systems manually

- Avoid making emergency console changes without importing the result into code

# Observability

- Logs record discrete events

- Metrics aggregate numeric behavior over time

- Traces connect work across services and dependencies

- Profiles show CPU, memory, allocation, and call-stack behavior

- Good telemetry includes service, environment, version, request/trace ID, result, duration, and useful domain identifiers without secrets

- RED for services: Rate, Errors, Duration

- USE for resources: Utilization, Saturation, Errors

- SLI: Measured reliability indicator

- SLO: Target for an SLI

- Error budget: Permitted unreliability before reliability work takes priority

- Alerts should indicate actionable user impact, not every small metric fluctuation

# Security and Secrets

- IAM controls which identities may perform which actions on which resources

- Apply least privilege, short-lived credentials, role-based access, multi-factor authentication, and separate human/service identities

- A secret manager stores and rotates credentials. Environment variables are a delivery mechanism, not automatically secure storage

- Never commit secrets, place them in public client bundles, log them, or bake them permanently into container images

- Segment networks and databases, restrict administrative interfaces, and record privileged actions

- Patch base images and dependencies, scan artifacts, sign important releases, and maintain a software inventory

- Shared-responsibility models divide security duties between provider and customer. Managed does not mean the customer has no security responsibilities

# Supabase

- Supabase is a backend platform centered on a managed PostgreSQL database

- Main products:

    Database: Full PostgreSQL with extensions, backups, SQL, and connection options

    Auth: Users, sessions, OAuth providers, magic links, and token handling

    Storage: Object storage integrated with authorization policies

    Realtime: Database-change and broadcast/presence features

    Edge Functions: Deno-based server-side functions

    Supavisor: Managed database connection pooling

- Supabase is not a replacement for understanding PostgreSQL. Tables, constraints, indexes, transactions, functions, migrations, and query plans remain PostgreSQL behavior

- Client setup:

    ```ts
    import { createClient } from '@supabase/supabase-js'

    export const supabase = createClient(
        process.env.NEXT_PUBLIC_SUPABASE_URL!,
        process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    )
    ```

- The project URL and anonymous/publishable key are designed for client use when Row Level Security protects data

- Service-role or secret keys bypass normal RLS behavior and must remain server-only

- Row Level Security (RLS) applies PostgreSQL policies to every table operation for applicable roles

    ```SQL
    ALTER TABLE notebooks ENABLE ROW LEVEL SECURITY;

    CREATE POLICY "Users read their notebooks"
    ON notebooks
    FOR SELECT
    USING (auth.uid() = user_id);
    ```

- Enabling RLS without correct policies blocks client access. Leaving RLS disabled on exposed tables can expose all rows

- Policies should cover `SELECT`, `INSERT`, `UPDATE`, and `DELETE` behavior as needed, including `WITH CHECK` for new row values

- Use server-side authorization in addition to RLS for sensitive business operations. Defense in depth protects against mistakes at either boundary

- Supabase Auth sessions use signed tokens. Verify identity through supported server helpers rather than trusting client-provided user IDs

- Storage buckets may be public or private. Private object access uses policies and signed URLs

- Realtime does not replace durable querying. Subscribers may disconnect and should resynchronize state

- Edge Functions are useful for secrets, webhooks, and server integrations. Validate authorization and input inside every function

- Connection modes:

    Direct: Long-lived servers and administrative/migration work where supported

    Session pooling: Session semantics through a pooler

    Transaction pooling: Serverless/high-concurrency short transactions with limitations on session features

- Use a direct connection for migrations and a suitable pooler connection for application traffic

- Supabase supports PostgreSQL extensions including pgvector. Vector columns, indexes, distance operators, RLS, and SQL functions remain pgvector/PostgreSQL concepts

# Supabase with Next.js and Prisma

- A common architecture is:

    Next.js UI/Server Actions -> Supabase Auth -> Prisma or Supabase client -> Supabase PostgreSQL -> RLS/constraints

- The Supabase browser client is useful for authenticated browser access, Realtime, and Storage

- Prisma is useful for typed server-side relational queries and migrations

- Prisma server queries commonly use a database role with broader access than the anonymous client, so application authorization must be enforced before each query

- Decide whether migrations are owned by Prisma Migrate, Supabase migrations, or another migration system. Two independent schema histories will conflict

- Reuse a Prisma client in long-lived development/server environments and use the correct connection pool strategy in serverless environments

- Do not expose `DATABASE_URL` or service-role credentials to the browser

# Vercel

- Vercel is a deployment platform with first-class Next.js integration

- A Git-connected project can create preview deployments for branches/pull requests and production deployments for the production branch

- Build-time settings include framework preset, root directory, install/build/output commands, environment variables, and runtime configuration

- Environment variable scopes usually separate development, preview, and production

- Preview deployments should not automatically receive unrestricted production secrets or production database mutation access

- Main runtime shapes include static assets, server functions, edge behavior where supported, streaming, image optimization, and scheduled jobs

- Serverless instances are ephemeral. Do not store durable files or required shared state in local memory/filesystem

- Function cold starts, execution limits, regions, database connection counts, payload limits, and caching affect architecture

- Place functions near the primary database when possible

- Use deployment protection and access controls for private preview environments

- Rollback/redeploy selects a known build, but database migrations and external side effects may require a separate rollback plan

- Domain configuration uses DNS verification, TLS certificates, redirects, and environment assignment

# GitHub Actions

- GitHub Actions runs automated workflows from `.github/workflows/*.yml`

- Core hierarchy:

    Workflow: Automation file triggered by events

    Job: Group of steps running on one runner

    Step: Shell command or reusable action

    Runner: Machine executing a job

    Action: Reusable packaged step

- Example quality workflow:

    ```yaml
    name: CI

    on:
      pull_request:
      push:
        branches: [main]

    permissions:
      contents: read

    jobs:
      test:
        runs-on: ubuntu-latest
        timeout-minutes: 15

        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-node@v4
            with:
              node-version: 22
              cache: npm
          - run: npm ci
          - run: npm run lint
          - run: npm run typecheck
          - run: npm test -- --coverage
          - run: npm run build
    ```

- Events include `push`, `pull_request`, `workflow_dispatch`, `schedule`, releases, and reusable workflow calls

- Use job dependencies with `needs`, conditions with `if`, and matrices for multiple versions/platforms

- Cache downloaded dependencies and build intermediates, not arbitrary untrusted outputs

- Artifacts transfer reports/builds between jobs or preserve them after a run

- Secrets are encrypted workflow inputs but can still be exposed by malicious commands or dependencies. Restrict permissions and environments

- `GITHUB_TOKEN` permissions should be set to the minimum needed

- Pin third-party actions to trusted versions or commit SHAs according to the repository's security policy

- Pull requests from forks require special caution because untrusted code must not receive secrets

- Use protected environments for production approvals, scoped secrets, and deployment records

- OpenID Connect allows a workflow to request short-lived cloud credentials instead of storing permanent cloud keys

- A quality gate should fail the workflow when formatting, lint, type, tests, security, coverage, or build requirements fail

- Coverage thresholds are useful guardrails but do not prove test quality. Protect critical authentication, ownership, validation, conflict, and injection boundaries with behavior tests

# Cost Management

- Main cost drivers include compute duration, memory, requests, database size/IO, egress, logs, storage operations, and managed-service tiers

- Add budgets and alerts before usage grows

- Tag resources by application, environment, owner, and cost center

- Delete abandoned preview environments and unattached storage

- Reduce waste through right-sizing, caching, lifecycle policies, efficient queries, compression, and rate/usage limits

- Cost optimization must preserve reliability and security requirements

# Quick Reference

- Global static content -> CDN/object storage

- Long-running flexible process -> VM or managed container

- Short request/event work -> Serverless function

- Durable file -> Object storage

- Horizontal app scaling -> Stateless instances plus shared data systems

- Supabase browser access -> Publishable key plus RLS

- Supabase administrative access -> Server-only secret/service role

- Serverless PostgreSQL traffic -> Correct pooler mode and bounded connections

- Next.js managed deployment -> Vercel

- Pull-request quality gates -> GitHub Actions

- Cloud credentials in CI -> OIDC short-lived role when supported

- Availability copies -> Replication

- Deletion/disaster recovery -> Tested backup

- Know reliability target -> SLI, SLO, and error budget
