# PlatformAI Domain Architecture

## Version

**Domain Architecture v0.2**

This revision evolves the initial PlatformAI domain model by introducing:

- The Environment Domain
- Environment Types, Profiles, Flavours, and Instances
- Product Instances
- Product Add-on Definitions
- Customer Entitlement as an architectural concern
- Atomic and Composite Services
- A clearer separation between business-facing workflows and reusable engineering services

## Purpose

This document describes the evolving domain architecture of PlatformAI.

The goal is to define the major platform domains, their responsibilities, and the relationships between them before implementation-specific decisions are introduced.

This architecture remains technology-agnostic at the domain level. Technologies such as Kubernetes, Terraform, FluxCD, Fleet, Helm, MCP, RAG, and LLM frameworks are implementation mechanisms that support the domains described here.

## Architectural Context

PlatformAI is an AI-enhanced Internal Developer Platform designed to provide a unified platform experience across existing engineering tools and workflows.

The platform separates user experience, identity and authorization, product definitions, environment definitions, service definitions, orchestration, reusable platform engines, and execution systems.

> **PlatformAI augments and orchestrates existing engineering capabilities rather than replacing them.**

Users express desired outcomes through platform services. PlatformAI translates those requests into governed, observable workflows executed through trusted engineering systems.

## Domain Model

PlatformAI v0.2 is organized around the following major domains:

1. Identity
2. Authorization
3. Service Marketplace
4. Service Domain
5. Product Domain
6. Environment Domain
7. Customer Entitlement
8. Service Orchestration
9. Platform Engines
10. Automation and Execution

These domains have separate responsibilities and communicate through well-defined boundaries.

# Identity Domain

## Responsibility

The Identity Domain determines who is interacting with PlatformAI.

PlatformAI delegates authentication to an enterprise Identity Provider rather than implementing its own identity system.

Identity information may include user identity, groups, claims, and organizational attributes.

## Boundary

Identity answers:

> **Who are you?**

It does not determine what the user is allowed to do. Authorization is handled separately.

# Authorization Domain

## Responsibility

The Authorization Domain determines which actions a user is permitted to perform against platform resources.

Authorization may consider identity groups, roles, permissions, resource scope, product, environment, service, and organizational policies.

## Personas Are Not Roles

PlatformAI personas describe user archetypes and their goals. They are not authorization roles.

A Software Developer persona may map to several roles with different permissions.

## Conceptual Authorization Request

```text
Subject + Action + Resource + Context
```

## Boundary

Authorization determines what actions are permitted.

The user interface may hide unavailable actions for usability, but backend services must enforce authorization independently.

# Service Marketplace

## Responsibility

The Service Marketplace is the self-service experience through which users discover and consume PlatformAI services.

The Marketplace is permission-aware.

Business-facing personas may see outcome-oriented services such as:

```text
Onboard Customer
Enable Contracted Add-on
Track Environment Status
```

Engineering personas may also see lower-level reusable services such as:

```text
Provision Environment
Install Product
Configure Network Access
Enable Product Add-on
Troubleshoot Kubernetes
```

## Boundary

The Marketplace does not own authorization policy, product definitions, environment definitions, or service execution.

# Service Domain

## Responsibility

The Service Domain defines reusable platform operations that users can request.

A Service Definition may include:

- Service name
- Description
- Required inputs
- Supported resource types
- Owning team
- Required approvals
- Policy references
- Workflow definition
- Composition rules

## Atomic Services

Atomic services represent reusable platform operations with a focused responsibility.

Examples:

```text
Provision Environment
Install Product
Enable Product Add-on
Disable Product Add-on
Configure Network Access
Request Database
Troubleshoot Kubernetes
```

Atomic services remain independently reusable.

## Composite Services

Composite services represent higher-level business outcomes.

They orchestrate multiple reusable services so users do not need to understand the underlying engineering workflow.

Example:

```text
Onboard Customer
        ↓
Validate Customer
        ↓
Validate Contract / Entitlements
        ↓
Provision Environment
        ↓
Install Product(s)
        ↓
Enable Entitled Add-ons
        ↓
Configure Network Access
        ↓
Validate Environment
        ↓
Customer Ready
```

> **Expose business outcomes to business-facing personas; expose reusable engineering primitives to engineering personas.**

# Product Domain

## Responsibility

The Product Domain describes what can be deployed and how product instances evolve over time.

The Product Domain contains:

- Product Definitions
- Product Bundles
- Product Add-on Definitions
- Product Instances

## Product Definitions

A Product Definition describes a deployable business application or SaaS product.

It may include:

- Product metadata
- Supported versions
- Components
- Helm charts
- Dependencies
- Required configuration
- Infrastructure requirements
- Ownership
- Deployment constraints

## Product Bundles

A Product Bundle groups multiple products into a reusable solution definition.

A service may operate on one product, multiple products, or a product bundle.

## Product Add-on Definitions

An Add-on is an optional capability tied to a product.

An Add-on Definition may describe:

- Compatible product versions
- Required feature flags
- Git configuration changes
- Additional infrastructure
- Secrets or configuration
- Restart or rollout requirements
- Validation steps
- Dependencies
- Required entitlement

The user requests the capability. PlatformAI owns the implementation steps required to enable it.

## Product Instances

A Product Definition describes what a product is.

A Product Instance represents an installed instance of that product in a specific environment.

Example:

```text
Product Instance

Customer: ABC
Environment: Production
Product: Product A
Version: 4.2

Enabled Add-ons:
✓ Advanced Reporting
✓ External Integration
✗ Analytics Export
```

Add-on lifecycle operations act on Product Instances.

# Environment Domain

## Responsibility

The Environment Domain defines reusable standards for where products run and how environments are operated.

The Environment Domain contains:

- Environment Types
- Environment Profiles
- Environment Flavours
- Environment Instances

These concepts are intentionally separated so they can be composed and reused across services and products.

## Environment Types

Environment Type answers:

> **What is this environment for?**

Example:

```text
Customer
├── Test
└── Production

Internal
├── Development
├── Test
├── Staging
└── Sandbox
```

Environment Types may define lifecycle expectations, governance level, approvals, access rules, availability, backup, data handling, allowed products, and observability requirements.

## Environment Profiles

Environment Profile answers:

> **How much infrastructure does this environment require?**

Examples:

```text
Small
Medium
Large
Custom
```

Profiles are defined and governed by Platform Engineering.

Consumers select approved profiles through the Service Marketplace.

The same profile may resolve differently depending on Environment Flavour.

Custom profiles represent an exception path and may require policy validation and approval.

## Environment Flavours

Environment Flavour answers:

> **Where and how is this environment hosted?**

Examples:

```text
Cloud
└── GCP

On-Prem
└── Customer-provided infrastructure
```

Environment Flavour may influence provisioning workflow, networking, supported infrastructure, cluster bootstrap, security controls, and prerequisites.

## Environment Instances

An Environment Instance represents a provisioned realization of:

```text
Environment Type
+
Environment Profile
+
Environment Flavour
+
Owner / Customer
+
Configuration
```

Environment Instances can host one or many Product Instances.

# Service, Product, and Environment Relationships

The three domains are intentionally independent.

Service Domain answers:

> **What can the platform do?**

Product Domain answers:

> **What can be deployed or changed?**

Environment Domain answers:

> **Where and under what operating characteristics does it run?**

> **Standardize the workflow; parameterize what it operates on.**

# Customer Entitlement

## Responsibility

Customer entitlement answers:

> **Is this customer contractually allowed to use this product or add-on?**

This is distinct from user authorization.

Authorization asks:

> Can this user perform this action?

Entitlement asks:

> Is this customer allowed to receive this capability?

Example:

```text
Request: Enable Add-on X
        ↓
Authorization
Can this user request add-ons?
        ↓
Entitlement
Has this customer purchased Add-on X?
        ↓
Compatibility
Does this product version support Add-on X?
        ↓
Lifecycle
What changes are required?
```

## Current Architectural Status

Customer Entitlement is recognized as an architectural concern in v0.2.

It is not yet promoted to a full independent bounded context.

# Service Orchestration Domain

## Responsibility

The Service Orchestrator coordinates the execution of requested platform services.

It may coordinate:

- Atomic services
- Composite services
- Authorization checks
- Policy evaluation
- Entitlement validation
- Workflow execution
- Product lifecycle operations
- Environment lifecycle operations
- Infrastructure automation
- GitOps changes
- Validation
- Observability
- Audit recording

## Boundary

The Service Orchestrator coordinates.

It does not reimplement Terraform, Kubernetes, GitOps, identity, policy, or observability systems.

# Platform Engines

Platform Engines are reusable internal capabilities used by one or more services.

Users do not interact with engines directly.

## Workflow Engine

Coordinates multi-step workflows, state transitions, retries, approvals, and compensation.

## Product Lifecycle Engine

Manages:

- Install Product
- Upgrade Product
- Rollback Product
- Remove Product
- Enable Add-on
- Disable Add-on
- Upgrade Add-on
- Validate Product Instance

## Environment Lifecycle Capability

Coordinates:

- Provision
- Validate
- Update
- Resize
- Decommission

The exact implementation boundary between this capability and the general Workflow Engine remains open for future design.

## AI Reasoning Engine

Provides investigation, reasoning, recommendations, and agent behavior.

## Knowledge Engine

Provides retrieval of platform knowledge such as documentation, runbooks, ADRs, product documentation, and previous incidents.

RAG may be used as an implementation technique.

## Policy Engine

Evaluates platform policies, guardrails, approval requirements, environment constraints, and security requirements.

## Observability Engine

Provides logs, metrics, traces, audit information, and workflow evidence.

# Automation and Execution Layer

PlatformAI does not replace existing engineering tools.

Platform services and engines use existing systems to execute changes.

Examples include:

- Terraform
- FluxCD or Fleet
- Helm
- Kubernetes APIs
- GitHub
- Cloud APIs
- Automation scripts
- Secret management systems
- OpenTelemetry-compatible systems
- MCP servers or adapters

# High-Level Lifecycle Model

PlatformAI v0.2 introduces three important lifecycle levels:

```text
Environment Definition
        │
        │ Provision Environment
        ▼
Environment Instance
        │
        │ Install Product
        ▼
Product Instance
        │
        │ Enable Add-on
        ▼
Product Instance
with Add-on Enabled
```

These operations remain independently reusable even when composed into higher-level services.

# Example: Customer Onboarding

A Customer Success user requests:

```text
Onboard Customer
```

PlatformAI:

1. Authenticates the user.
2. Evaluates user authorization.
3. Validates customer information.
4. Validates product and add-on entitlements.
5. Resolves Environment Type, Profile, and Flavour.
6. Provisions the Environment Instance.
7. Installs requested products.
8. Creates Product Instances.
9. Enables entitled add-ons where requested.
10. Configures network access.
11. Produces desired state for GitOps.
12. Allows existing systems to reconcile the environment.
13. Validates environment and product health.
14. Emits telemetry and audit events.
15. Reports customer readiness.

Customer Success consumes a business outcome while PlatformAI orchestrates reusable engineering services.

# Architectural Boundaries

```text
Persona ≠ Role
Identity ≠ Authorization
Authorization ≠ Entitlement
Product Definition ≠ Product Instance
Environment Definition ≠ Environment Instance
Environment Type ≠ Environment Profile
Environment Profile ≠ Environment Flavour
Product ≠ Service
Atomic Service ≠ Composite Service
Catalog ≠ Marketplace
Service ≠ Engine
Orchestration ≠ Execution
AI Reasoning ≠ Infrastructure Access
```

# Evolution from v0.1

Domain Architecture v0.1 established:

- Identity
- Authorization
- Service Marketplace
- Service Catalog
- Product Catalog
- Service Orchestration
- Platform Engines
- Execution Layer

Domain Architecture v0.2 adds and refines:

- Environment Domain
- Environment Types
- Environment Profiles
- Environment Flavours
- Environment Instances
- Product Instances
- Product Add-on Definitions
- Customer Entitlement concern
- Atomic Services
- Composite Services
- Generalized `Provision Environment`
- Business-outcome-oriented service composition

This evolution emerged from testing the architecture against real operational workflows.

# Open Architecture Questions

- Should Customer Entitlement become an independent domain?
- Should Environment lifecycle have a dedicated engine?
- Where should Environment Profile schemas and versions be stored?
- How should Product and Add-on compatibility be modeled?
- How should composite service definitions reference atomic services?
- How should service state and long-running workflow state be persisted?
- How should multi-tenancy boundaries be modeled?
- How should approval policies vary by Environment Type and resource risk?
- How should service definitions be versioned?

# Guiding Question

When evolving the domain model, ask:

> **Does this abstraction represent an independently owned, reusable responsibility, or are we creating a new domain where composition would be simpler?**
