# Overview

- CICD stands for Continuous Integration and Continuous Deployment

- CICD is related to handling the deployment of initial code as well as updated code in already deployed applications

- CICD is a pipeline which involves automated testing of code features individually before integrating them in the main system, preventing system level errors which would be harder to decipher and more costly to fix

- An analogy is a car part assembly line. When building cars, manufacturers do not assemble the entire car and then test the entire car for issues. This would be very expensive to fix as they would have to disassemble and reassemble the entire car at least once. Instead, an assembly line tests every part individually for quality assurance. Then, when the car is assembled, there is a far lower chance of any mishappenings. This is how code is pushed with a CICD pipeline. Features are tested one at a time before they are assembled together and launched to users

- CICD pipelines are manually created and apply to all future updates and deployments. Pipelines should be able to automatically compile, test, and finally intgrate code to be deployed

# CICD Pipeline Stages

Stage 0: Source/Commit

- This is simply pushing your local changes to a remote repository to begin the next stages

- Ex. (In terminal)

    git add "committed files"
    git commit (add message in text editor)
    git push origin main

Stage 1: Building

- This stage converting your source code (what you coded) into runnable artifacts.

- Here you should be installing relevant dependencies (npm install, pip install -r requirements.txt)

- You should also be compiling code and creating deployment packages (Docker images, ZIPs)

- Ex. (In terminal)

    npm install
    npm run build
    docker build -t my-app:latest .

Stage 2: Testing

- This is where code is tested for quality and functionality

- Common test Types:

    Unit Tests: Test individual functions/components

    Integration Tests: Test how components work together

    Linting: Testing for code standards and quality (not functionality)

    Security Scanning: Testing for vulnerabilities

Note: Unit Testing and Linting may come before Stage 1, pre-Build. Stage 2 is mainly for integration testing

- Ex. (In terminal)

    npm run test
    npm run lint
    sonar-scanner

Stage 3: Deployment

- This is where tested code is shipped to target environments

- Common environments: Staging (duplicate production environment for final testing), Production (Live user-facing environment)

- Common deployment types:

    Blue-Green: A duplicate production environment ("Green") is created where new updates are pushed. When updates are ready for production the traffic is switched from the original user environment ("Blue") to the new production environment

    Canary: An update is released to a sample of users and expanded gradually

# Continuous Integration, Delivery, and Deployment

- Continuous Integration means developers merge small changes frequently and an automated system verifies each change

- Continuous Delivery means every accepted change is kept deployable, while production release may require a manual approval

- Continuous Deployment means accepted changes automatically reach production without a manual release step

- A pipeline should provide fast feedback first and expensive broad confidence later

- A typical order is:

    Checkout -> Dependency restore -> Formatting/lint -> Type check -> Unit tests -> Security checks -> Build -> Integration tests -> Package/sign -> Deploy staging -> Smoke/E2E -> Approval -> Production -> Verify/monitor

- Independent jobs may run in parallel when they do not depend on one another

- Every failure should be clear, reproducible locally, and linked to useful logs or artifacts

# Pipeline Triggers

- Common triggers:

    Pull request: Validate a proposed merge

    Push to protected branch: Build or deploy accepted code

    Tag/release: Create versioned artifacts and production releases

    Schedule: Repeated security, dependency, cleanup, or synthetic checks

    Manual dispatch: Controlled operation or recovery

    External event: Registry, deployment platform, or upstream service event

- Pull-request pipelines should not receive high-privilege production secrets when they execute untrusted code

- Branch protection can require successful checks, reviews, current base branch, and signed/verified rules before merging

# Artifacts and Reproducibility

- An artifact is the immutable output deployed or distributed, such as a container image, JAR, wheel, static bundle, or ZIP

- Build once and promote the same artifact through environments. Rebuilding for production can create an untested result

- A reproducible build uses pinned tool versions, a lockfile, controlled inputs, and deterministic packaging where practical

- Give artifacts a unique version tied to the commit, build, or release

- A container tag is a convenient label; a digest identifies exact image content

- Store test reports, coverage, logs, provenance, software bills of materials, and deployment metadata as pipeline artifacts

- Define retention so useful evidence is preserved without unlimited storage cost

# Dependency and Build Caching

- Caching speeds the pipeline by reusing downloaded dependencies or safe build intermediates

- A cache key should include relevant lockfiles, toolchain versions, platform, and build configuration

- A cache miss should slow the build but not make it fail

- Do not cache generated state which can hide required work or cross a security boundary

- Artifacts carry an intentional output between jobs. Caches are optional acceleration and may disappear

- Use clean installs such as `npm ci` when lockfile fidelity matters

# Testing Layers

- Formatting check: Consistent source layout

- Linting: Static rules for likely errors and code standards

- Type checking: Static contracts across code

- Unit tests: Isolated behavior

- Integration tests: Real cooperating modules such as app and database

- Contract tests: Producer/consumer API expectations

- End-to-End tests: Complete user behavior

- Smoke tests: Small critical checks after deployment

- Performance tests: Latency, throughput, resource, and regression behavior

- Security tests: Dependency, source, secret, image, infrastructure, and dynamic analysis

- Place fast deterministic checks early and run slower environment-heavy checks after the build is credible

- Quarantine is not a permanent solution for flaky tests. Find timing, isolation, shared state, order dependence, and external-service causes

# Coverage and Quality Gates

- Code coverage reports which lines, branches, or functions ran during tests

- A threshold prevents large untested regressions but does not prove assertions are meaningful

- High-risk authentication, ownership isolation, validation, revision conflicts, caching, and prompt-injection boundaries deserve explicit behavior tests

- Mutation testing can reveal tests which execute code without detecting incorrect behavior

- Quality gates should be visible, version-controlled, and difficult to bypass silently

- A failed optional analytics upload should not necessarily fail correct code, while a failed security or test gate should

# Environments

- Local: Developer machine and local dependencies

- CI/Test: Automated isolated verification

- Preview: Per-branch or pull-request application

- Staging: Production-like shared environment

- Production: Live user environment

- Keep configuration environment-specific while keeping the built artifact the same

- Production-like staging helps but cannot reproduce all production traffic, scale, data, permissions, and third-party behavior

- Preview environments need lifecycle cleanup and limited data/secrets

# Deployment Strategies

- Rolling deployment replaces instances gradually. It uses fewer duplicate resources but temporarily runs multiple versions

- Blue-green maintains old and new environments and switches traffic. Rollback is fast until database/external state changes become incompatible

- Canary sends a small traffic percentage to the new version and expands based on health

- Feature flags deploy code separately from releasing behavior to users

- Shadow traffic copies requests to a new system without using its result. Protect private data and avoid duplicate side effects

- Recreate stops the old version before starting the new one and causes downtime

- Choose strategy based on compatibility, cost, traffic control, observability, and rollback needs

# Database Migrations

- Database changes often determine whether application rollback is safe

- Expand-and-contract pattern:

    1. Add backward-compatible schema

    2. Deploy code which can use old and new forms

    3. Backfill data in controlled batches

    4. Switch reads/writes

    5. Verify

    6. Remove old schema in a later deployment

- Avoid combining a destructive migration with code which requires the destruction immediately

- Large indexes and backfills may lock tables, increase replication lag, or overload production

- Test migrations against realistic size, monitor progress, and define recovery before execution

- Apply each migration once through a controlled job, not independently from every auto-scaled application instance

# Secrets and Pipeline Security

- Use least-privilege short-lived credentials for pipelines

- OpenID Connect can exchange a workflow identity for temporary cloud credentials without storing permanent access keys

- Protect production deployment through environments, approvals, restricted branches, and scoped secrets

- Masking reduces accidental log exposure but does not make it safe to print secrets

- Pin trusted third-party actions/plugins and review their permissions

- Scan commits for secrets and rotate a secret immediately after exposure

- Generate an SBOM and scan dependencies/container images while maintaining a process for triage and patching

- Sign important artifacts and record provenance so deployments can verify where they came from

- Self-hosted runners require patching, isolation, cleanup, and protection from untrusted jobs

# GitHub Actions Reference

    ```yaml
    name: CI

    on:
      pull_request:
      push:
        branches: [main]

    concurrency:
      group: ci-${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true

    permissions:
      contents: read

    jobs:
      quality:
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

- `on`: Workflow events

- `jobs`: Independently scheduled units

- `runs-on`: Runner environment

- `steps`: Commands/actions within one job workspace

- `uses`: Reusable action or workflow

- `run`: Shell command

- `needs`: Job dependency

- `if`: Conditional execution

- `strategy.matrix`: Run combinations of versions or platforms

- `permissions`: `GITHUB_TOKEN` scope

- `environment`: Protected deployment environment

- `concurrency`: Prevent overlapping obsolete or unsafe runs

- `timeout-minutes`: Bound stuck work

- Reusable workflows centralize organization standards. Composite actions package repeated steps

- Service containers can provide PostgreSQL, Redis, and similar dependencies for integration tests

- Upload coverage/test reports even on failure when diagnostics are required, using a suitable `if` condition

# Release and Versioning

- Semantic Versioning uses `MAJOR.MINOR.PATCH` for breaking, compatible feature, and compatible fix releases where the project follows that contract

- A changelog explains user-visible changes, migration needs, deprecations, and known issues

- Automated release tools can create tags, notes, packages, and version updates from structured commits or release metadata

- Tag creation and package publication should happen from protected verified code

- Never overwrite a published immutable version with different content

# Monitoring and Rollback

- Deployment success is not the same as process start success

- Verify health, error rate, latency, saturation, critical business transactions, logs, and user impact after release

- Automatically stop or roll back a canary when defined metrics exceed thresholds, while avoiding unstable metric noise

- Rollback procedure must include application, configuration, database compatibility, queues/events, caches, and external side effects

- Roll forward may be safer than rollback after irreversible data changes

- Record deployment version, actor/workflow, time, environment, artifact digest, and change set

# Pipeline Design Practices

- Keep pipeline definitions in source control

- Make jobs idempotent and safe to retry

- Add timeouts to commands and network operations

- Cancel obsolete pull-request runs to save time and avoid misleading results

- Avoid one enormous opaque script; use clear stages while sharing implementation through normal tested scripts

- Make the local developer command and CI command as similar as possible

- Measure queue time, execution time, failure rate, flaky reruns, deployment frequency, lead time, change failure rate, and recovery time

- Improve the slowest high-value feedback path instead of optimizing an infrequent job first

# Quick Reference

- Every merge tested -> Continuous Integration

- Always ready for manual release -> Continuous Delivery

- Every accepted change released -> Continuous Deployment

- Move exact build between environments -> Immutable artifact promotion

- Fast feedback -> Parallel early static/unit checks

- Production access -> Protected environment and short-lived identity

- Server/schema compatibility -> Expand and contract

- Partial traffic release -> Canary

- Instant traffic switch -> Blue-green

- Separate deploy from user release -> Feature flag

- Repeatable GitHub workflow -> `.github/workflows/*.yml`

- Coverage target -> Guardrail plus critical behavior assertions

- Failed deployment -> Verify rollback covers data and side effects, not only code
