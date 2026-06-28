intelligent-devops-aws-pipeline
Production-ready GitHub Actions CI/CD pipeline for an AWS ECS microservice, featuring OIDC authentication, immutable artifact caching, and an automated offline AI security &amp; architectural audit.


Enterprise Microservice CI/CD Workflow with Offline AI Audit

An enterprise-grade, multi-stage GitOps pipeline designed for deploying a containerized Reports Microservice to Amazon ECS via AWS CloudFormation. 

Unlike standard, boilerplate CI/CD setups, this pipeline is hardened for production environments, utilizing cryptographic identity federation, strict deterministic artifact management, native build performance caching, and a cutting-edge, 100% air-gapped local AI architectural audit engine.

 Unique Architecture & Engineering Standouts

Here is what makes this pipeline different from 95% of standard GitHub Actions setups:

1. Air-Gapped AI Architecture Review
The Problem: Traditional linters check formatting, but they cannot verify if your Docker container's runtime configurations actually align with your cloud network blueprints. 
The Innovation: Stage 3.5 spins up an isolated, local LLM instance right inside the runner using  "Ollama ". It cross-references your application’s  "Dockerfile " and your infrastructure's  "CloudFormation " template to detect structural mismatches, exposed ports, or open security groups. Because it runs locally, your intellectual property and cloud configurations never leave the secure pipeline infrastructure.

 2. Elimination of the Timestamp Race Condition
  The Problem: Standard pipelines run  "date " commands in separate jobs. Because jobs execute on independent runner VMs and scale out at different times, this causes a mismatched tag mismatch where the backup S3 package filename differs from the deployed ECR image tag.
  The Innovation: This pipeline establishes a strict Single Source of Truth. Stage 2 generates the immutable  "TIMESTAMP " once, packages the assets, and distributes the unified zip using GitHub's native v4 Artifact Storage. Stage 3 completely skips Git checkout, pulling the identical file to maintain absolute build traceability.

 3. Zero Permanent AWS Secrets (OIDC Federation)
  The Problem: Storing  "AWS_ACCESS_KEY_ID " and  "AWS_SECRET_ACCESS_KEY " in GitHub Secrets risks credential exposure if a third-party action is compromised.
  The Innovation: Uses OpenID Connect (OIDC) via  "aws-actions/configure-aws-credentials ". GitHub assumes short-lived IAM Roles dynamically using cryptographically signed JSON Web Tokens (JWT tokens), restricting access to the specific context of this repository branch execution.

 4. Advanced Docker Layer Cache Routing ( "type=gha ")
  The Problem: Containerizing microservices on ephemeral runners rebuilds base images and dependencies from scratch every single run, driving up compute bills and slowing down feedback loops.
  The Innovation: Leverages  "docker/setup-buildx-action " along with  "docker/build-push-action ". It maps external metadata caches natively into the GitHub Actions storage backend ( "cache-from/to: type=gha,mode=max "), dropping containerization wait-times drastically on subsequent runs.

---
Pipeline Blueprint & Job Flow

The workflow is architected into 5 distinct, sequential execution stages to maintain clear boundaries of separation:

1.  Stage 1: Code Quality Scan – Validates source formatting and code sanity utilizing containerized static analyzers.
2.  Stage 2: Source Packaging – Utilizes Git  "sparse-checkout " to cleanly isolate  "AWS-Cloudformation " definitions, compiling a time-stamped asset package.
3.  Stage 3: S3 Backup – Assumes an AWS IAM Role via OIDC and syncs the immutable build package into an enterprise cold-storage S3 bucket.
4.  Stage 3.5: Offline AI Architecture Review – Automatically provisions a local LLM, auditing configurations for misalignments or hidden structural risks.
5.  Stage 4: Docker Build & ECR Push – Logs into Amazon ECR safely, provisions Docker Buildx cache lines, and updates the registry.
6.  Stage 5: Deployment & Stabilization Verification – Executes targeted CloudFormation stack updates and programmatically polls Amazon ECS via  "aws ecs wait " until the cluster stabilizes safely.

---

 📋 Technology Stack Matrix

| Layer | Implementation | Purpose |
|
| CI/CD Platform | GitHub Actions | Execution engine and job orchestration |
| Cloud Provider | Amazon Web Services (AWS) | Infrastructure host (ECS, ECR, S3, CloudFormation) |
| Authentication | OpenID Connect (OIDC) JWT | Short-lived, secure cryptographic session handshakes |
| Artifact State | GitHub Artifacts API v4 | Preserving data integrity between decoupled runner environments |
| Caching Engine | Buildx GHA Backend | Storing Docker image layer deltas to cut execution costs |
| AI Audit Engine| Ollama Engine ( "qwen2.5-coder:7b ") | 100% private, contextual linting and structural analysis |

