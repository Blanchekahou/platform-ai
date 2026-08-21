# PlatformAI Service Catalog

## Version

**Service Catalog v0.1**

## Purpose

The PlatformAI Service Catalog defines the reusable platform capabilities that can be exposed through the Service Marketplace and orchestrated by PlatformAI.

The catalog translates the PlatformAI Domain Architecture into concrete user-facing and platform-facing capabilities while preserving clear boundaries between:

- User intent
- Service definitions
- Domain models
- Authorization
- Entitlement
- Policy
- Workflow orchestration
- Platform engines
- Provider-specific adapters
- Existing engineering systems

The Service Catalog describes **what the platform can do**.

It does not define the implementation details of how every capability is executed.

> **PlatformAI exposes meaningful user intent while hiding unnecessary implementation complexity.**

---

# Design Principles

The Service Catalog follows the PlatformAI engineering and architectural principles.

## Services Represent User Intent

Services should represent meaningful outcomes rather than direct wrappers around infrastructure tools.

For example:

```text
Provision Environment
```

is preferred over:

```text
Run Terraform Module
```

Similarly:

```text
Release Product Version
```

is preferred over exposing a sequence of Git, Helm, or Kubernetes operations directly to the consumer.

---

## Services Are Independent from Execution Technologies

Service definitions should remain independent from specific implementation technologies.

A service may eventually execute through:

- Terraform
- Kubernetes APIs
- Helm
- FluxCD
- Fleet
- GitHub
- Cloud APIs
- Automation scripts
- MCP servers
- Other engineering systems

These technologies are execution mechanisms.

They do not define the service itself.

---

## Service Contracts Are Provider-Neutral

PlatformAI service contracts should not depend directly on a specific cloud provider.

For example:

```text
Provision Environment
```

remains the service regardless of whether the environment is implemented using:

```text
GCP
AWS
Azure
On-Prem
```

Provider-specific behavior is isolated behind provider adapters and execution strategies.

The initial cloud implementation may use a GCP adapter while preserving the ability to introduce additional provider adapters later.

---

## Persona Does Not Equal Role

Personas describe categories of PlatformAI users and their goals.

They do not directly determine permissions.

A persona may map to multiple roles, and users belonging to the same persona may have different capabilities.

Authorization is evaluated independently.

---

## Authorization Does Not Equal Entitlement

Authorization answers:

> **Is this user permitted to perform this action?**

Entitlement answers:

> **Is this customer eligible to receive this product or capability?**

Both may be required before a service can execute.

---

## Definitions Are Different from Instances

PlatformAI distinguishes reusable definitions from their runtime realizations.

Examples:

```text
Product Definition
≠
Product Instance
```

and:

```text
Environment Definition
≠
Environment Instance
```

Lifecycle operations normally act on runtime instances.

---

# Service Types

PlatformAI defines two primary service types:

1. Atomic Services
2. Composite Services

---

# Atomic Services

An Atomic Service represents a focused, reusable platform operation.

Examples include:

```text
Provision Environment
Install Product
Upgrade Product Instance
Enable Product Add-on
Suspend Environment
Configure Network Access
```

Atomic does not necessarily mean technically simple.

An Atomic Service may internally require multiple execution steps.

Atomic means that the service represents one reusable platform intent from the perspective of the Service Catalog.

For example:

```text
Provision Environment
```

may internally require:

```text
Validate request
→ Resolve Environment Profile
→ Resolve provider strategy
→ Generate infrastructure desired state
→ Execute provisioning
→ Bootstrap platform components
→ Validate environment
```

It remains one reusable platform capability.

---

# Composite Services

A Composite Service represents a higher-level outcome produced by coordinating multiple reusable services.

Examples include:

```text
Onboard Customer
Deactivate Customer
Reactivate Customer
Decommission Customer
Release Product Version
```

Composite Services allow PlatformAI to expose business-oriented outcomes without forcing consumers to understand the engineering operations underneath them.

For example:

```text
Onboard Customer
        ↓
Provision Environment
        ↓
Install Product
        ↓
Enable requested Add-ons
        ↓
Configure requested Network Access
        ↓
Validate readiness
```

The consumer requests the outcome.

PlatformAI owns the orchestration.

---

# Service Definition Model

Every PlatformAI service should follow a common conceptual definition.

This model is intentionally technology-agnostic.

It is not yet an API schema or persistence schema.

## Service Identity

A Service Definition should describe:

```text
Name
Description
Service Type
Owner
Version
```

---

## Consumer

A Service Definition identifies the intended consumers.

This may include:

```text
Personas
Required permissions
Applicable roles
```

The Service Marketplace may use this information to determine which capabilities should be visible or selectable.

Backend authorization remains authoritative.

---

## Intent

Each service defines its desired outcome.

Example:

```text
Service:
Provision Environment

Intent:
Create a valid Environment Instance from approved
Environment definitions and requested configuration.
```

---

## Inputs

Service inputs may be:

```text
Required
Optional
Conditional
```

### Required

The service cannot proceed without the value.

### Optional

The consumer may explicitly choose not to request the capability.

### Conditional

The input becomes required depending on another selection or context.

Example:

```text
Environment Flavour = Cloud
→ Cloud region may be required

Environment Flavour = On-Prem
→ On-Prem infrastructure information may be required
```

---

## Targets

Services may operate on domain resources such as:

```text
Customer
Environment Instance
Product Definition
Product Version
Product Instance
Product Add-on
Release
```

---

## Constraints

A service may be subject to:

```text
Authorization
Entitlement
Compatibility
Policy
Approval
```

These concerns remain distinct.

---

## Workflow

The Service Definition describes the conceptual workflow required to reach the desired outcome.

The workflow may be synchronous or asynchronous.

---

## Execution

Execution describes the platform capabilities or external systems required to realize the service.

Examples include:

```text
Workflow Engine
Product Lifecycle Engine
Environment Lifecycle capability
Policy Engine
AI Reasoning Engine
Provider Adapter
GitOps
Terraform
Kubernetes
```

Service consumers should not need to understand these implementation details.

---

## Validation

Every mutating service should define how PlatformAI determines whether the desired outcome was achieved.

Validation may include:

```text
Infrastructure state
Application health
Product health
Environment health
Configuration state
Migration status
Policy state
External system confirmation
```

---

## Observability

Services should produce sufficient evidence to understand:

```text
Who requested the operation?
What was requested?
What decisions were made?
What actions were executed?
What changed?
What succeeded?
What failed?
What is currently pending?
```

This information supports:

- Auditability
- Troubleshooting
- AI reasoning
- Incident investigation
- Release history
- Operational learning

---

# Standard Service Execution Model

PlatformAI services follow a common conceptual execution path.

```text
Request
   ↓
Authorization
   ↓
Entitlement
   ↓
Compatibility
   ↓
Policy Evaluation
   ↓
Approval
   ↓
Orchestration
   ↓
Execution
   ↓
Validation
   ↓
Observability / Audit
   ↓
Result
```

Not every gate applies to every service.

For example, Customer Entitlement may not apply to an internal sandbox environment.

---

# Policy-Driven Approval

Approval requirements should not be permanently hard-coded into Service Definitions.

The same service may require different approval behavior depending on context.

For example:

```text
Provision Environment
```

for:

```text
Internal / Sandbox / Small
```

may execute automatically.

The same service for:

```text
Customer / Production / Custom
```

may require approval.

Policy evaluation may consider:

- Environment Type
- Environment Profile
- Environment Flavour
- Cloud provider
- Resource cost
- Production impact
- Requested overrides
- User permissions
- Customer context
- Operational risk

Destructive operations may require stronger approval policies.

---

# Customer Management

PlatformAI requires a representation of customers that can be referenced by platform workflows.

Customer Management provides basic lifecycle and data management capabilities.

Initial native capabilities include:

```text
Create Customer
Read Customer
Update Customer
Delete Customer when safe
```

These CRUD operations do not automatically need to appear as Service Marketplace services.

---

# Customer Provider Boundary

PlatformAI should remain open to consuming customers from an external authoritative business system.

Conceptually:

```text
                  Customer Domain
                        │
                Customer Provider
                  ┌─────┴─────┐
                  │           │
                  ▼           ▼
             Native Store   External CRM
```

PlatformAI services should consume customer information through the Customer Domain rather than depending directly on a particular CRM.

Two operating modes may eventually exist.

## Native Mode

```text
PlatformAI
→ authoritative customer source
```

## External Mode

```text
CRM / Business System
→ authoritative customer source
→ PlatformAI consumes customer records
```

Bidirectional synchronization is not required for Service Catalog v0.1.

---

# Customer Lifecycle State

Customer lifecycle state is distinct from the workflows used to reach that state.

Possible states include:

```text
Created
Active
Deactivating
Inactive
Reactivating
Decommissioning
Decommissioned
Archived
```

The Customer Domain owns customer state.

Composite services coordinate the operational work required to safely transition between states.

---

# Deactivate Customer

**Type:** Composite Service

## Intent

Temporarily stop customer consumption of platform-managed products and eligible infrastructure while preserving recoverability.

Deactivation does not mean deletion.

## Conceptual Workflow

```text
Active
  ↓
Deactivating
  ↓
Disable customer/product access
  ↓
Disable or suspend Product Instances
  ↓
Suspend Environment Instances where appropriate
  ↓
Reduce eligible compute resources
  ↓
Preserve required data
  ↓
Preserve configuration
  ↓
Preserve audit/history
  ↓
Validate
  ↓
Inactive
```

The exact realization depends on the Environment Flavour.

---

# Cloud Deactivation

Cloud environments may support aggressive resource optimization.

Possible operations include:

```text
Disable product access
Suspend workloads
Scale workloads down
Scale eligible compute resources down
Preserve persistent storage
Preserve databases
Preserve configuration
Preserve backups
Preserve audit history
```

Provider-specific execution is delegated to the appropriate provider adapter.

The initial cloud provider adapter may target GCP.

---

# On-Prem Deactivation

On-Prem environments require a different realization.

Possible operations include:

```text
Disable product access
Suspend product workloads
Preserve application data
Preserve configuration
Preserve platform metadata
Preserve backup state
```

PlatformAI may not own the physical infrastructure and therefore may not be responsible for powering down or disposing of customer hardware.

---

# Reactivate Customer

**Type:** Composite Service

## Intent

Restore a previously deactivated customer to an operational state.

The workflow may coordinate:

```text
Resume Environment
Restore required compute capacity
Resume Product Instances
Restore customer access
Validate products
Validate environment
Mark Customer Active
```

The realization remains Environment-Flavour-specific.

---

# Decommission Customer

**Type:** Composite Service

## Intent

Permanently remove operational customer resources according to platform and retention policies while preserving required historical information.

## Conceptual Workflow

```text
Validate request
        ↓
Validate approvals
        ↓
Validate retention requirements
        ↓
Customer → Decommissioning
        ↓
Remove Product Instances
        ↓
Decommission Environment Instances
        ↓
Remove platform-managed network resources
        ↓
Apply backup/data retention policy
        ↓
Preserve required audit/history
        ↓
Customer → Decommissioned / Archived
```

Decommissioning does not require physical deletion of the Customer record.

---

# Delete Customer

Deletion is different from deactivation and decommissioning.

Physical deletion should be restricted.

For example, deletion may only be allowed when:

```text
No Environment Instances exist
AND
No Product Instances exist
AND
No required audit/history depends on the Customer
```

Deletion may primarily serve correction of erroneously created customer records.

---

# Environment Lifecycle Services

Initial Environment services include:

```text
Provision Environment
Suspend Environment
Resume Environment
Decommission Environment
```

A generic `Update Environment` service is intentionally excluded from v0.1.

More specific operations should be introduced when meaningful user intents are identified.

Future examples may include:

```text
Resize Environment
Change Environment Profile
Modify Network Configuration
Migrate Environment
```

---

# Provision Environment

**Type:** Atomic Service

## Intent

Create an Environment Instance from approved Environment definitions and requested configuration.

## Core Inputs

```text
Environment Type
Environment Profile
Environment Flavour
Owner / Customer
Configuration
```

Additional inputs may be conditional.

For example:

```text
Cloud
→ provider
→ region
→ cloud-specific allowed configuration

On-Prem
→ infrastructure information
→ connectivity information
→ customer-provided prerequisites
```

---

# Provider-Neutral Provisioning

The service remains:

```text
Provision Environment
```

regardless of provider.

Conceptually:

```text
Provision Environment
        ↓
Environment Domain
        ↓
Environment Flavour
        ↓
Cloud Provider Interface
        ↓
Provider Adapter
        ↓
Execution
```

Initial implementation:

```text
GCP Adapter
```

Future implementations may include:

```text
AWS Adapter
Azure Adapter
```

Multi-cloud awareness exists at the architecture level while full multi-cloud implementation is deferred.

---

# Suspend Environment

**Type:** Atomic Service

## Intent

Move an Environment Instance into a suspended operational state while preserving the resources and data required for later recovery.

Execution depends on Environment Flavour and applicable policy.

---

# Resume Environment

**Type:** Atomic Service

## Intent

Restore a suspended Environment Instance to its expected operational state.

---

# Decommission Environment

**Type:** Atomic Service

## Intent

Remove an Environment Instance and its platform-managed operational resources according to retention and governance policies.

Cloud and On-Prem realizations may differ significantly.

---

# Product Lifecycle

PlatformAI distinguishes:

```text
Product Definition Lifecycle
```

from:

```text
Product Instance Lifecycle
```

---

# Product Definition Lifecycle

Product Definitions describe reusable deployable products.

Their lifecycle may eventually include:

```text
Create Product Definition
Update Product Definition
Publish Product Version
Deprecate Product Version
Retire Product Definition
```

These capabilities are primarily owned by appropriate engineering/product roles and do not necessarily appear to all Service Marketplace consumers.

---

# Product Instance Lifecycle

A Product Instance represents an installed realization of a Product Definition in an Environment Instance.

Initial Product Instance services include:

```text
Install Product
Upgrade Product Instance
Rollback Product Instance
Remove Product Instance
Enable Product Add-on
Disable Product Add-on
```

---

# Install Product

**Type:** Atomic Service

## Intent

Install a selected Product Definition and version into an existing Environment Instance.

Successful installation creates a Product Instance.

```text
Product Definition
        +
Environment Instance
        ↓
Install Product
        ↓
Product Instance
```

---

# Upgrade Product Instance

**Type:** Atomic Service

## Intent

Move an existing Product Instance from its current Product Version to a compatible target Product Version.

Example:

```text
Product A Instance

v4.2
  ↓
Upgrade Product Instance
  ↓
v4.3
```

The Product Instance identity remains while its deployed version changes.

---

# Upgrade Compatibility

Before execution, PlatformAI may evaluate:

```text
Supported upgrade path
Environment compatibility
Environment Profile requirements
Add-on compatibility
Configuration requirements
Migration awareness
Startup/init changes
Rollback constraints
Required approvals
```

An unsupported upgrade should not proceed automatically.

---

# Migration Awareness

PlatformAI should be aware of migration requirements without assuming ownership of product-specific migration logic.

Possible strategies include:

```text
NONE

PRODUCT_MANAGED

PLATFORM_ORCHESTRATED

EXTERNAL
```

## Product-Managed Migration

The product performs its own migration.

Examples may include:

```text
Application startup migration
Application migration framework
Helm hook
Product-owned Kubernetes Job
```

PlatformAI coordinates the upgrade and observes the resulting state.

It does not duplicate the product's migration implementation.

## Platform-Orchestrated Migration

The Product Release Contract may require PlatformAI to coordinate a defined migration workflow.

The product still owns the product-specific migration logic.

PlatformAI owns orchestration around that logic.

## External Migration

A migration may depend on another system or operational process.

PlatformAI may wait for external completion or confirmation before continuing.

---

# Migration Ownership Principle

> **Product teams own product-specific migration logic. PlatformAI owns awareness, orchestration, governance, validation, evidence, and historical traceability around the upgrade.**

This principle also applies to product-specific startup tasks and initialization behavior.

---

# Rollback Product Instance

**Type:** Atomic Service

## Intent

Restore a Product Instance to a previously supported Product Version when rollback is permitted.

Rollback capability may depend on:

```text
Product compatibility
Database migration reversibility
Configuration compatibility
Backup state
Release Contract
Policy
```

Not every release is necessarily automatically reversible.

---

# Remove Product Instance

**Type:** Atomic Service

## Intent

Remove an installed Product Instance from an Environment Instance while respecting applicable data and retention policies.

Removing a Product Instance does not remove its Product Definition from the Product Catalog.

---

# Product Add-on Lifecycle

An Add-on represents an optional capability associated with a Product.

Add-on lifecycle services include:

```text
Enable Product Add-on
Disable Product Add-on
```

These operations act on Product Instances.

---

# Enable Product Add-on

**Type:** Atomic Service

## Intent

Enable a selected Product Add-on on an eligible Product Instance.

Before execution:

```text
Authorization
        ↓
Customer Entitlement
        ↓
Product / Version Compatibility
        ↓
Policy
        ↓
Approval if required
        ↓
Execution
```

Implementation may require:

```text
Feature flags
Git configuration
Infrastructure changes
Secrets
Rollout/restart
Product validation
```

The user requests the capability.

PlatformAI coordinates the required implementation.

---

# Disable Product Add-on

**Type:** Atomic Service

## Intent

Disable an enabled Product Add-on while preserving required data and respecting Product-specific lifecycle requirements.

---

# Release Management

Product Release Management coordinates rollout of Product Versions across existing Product Instances.

It is distinct from upgrading a single Product Instance.

---

# Publish Product Version

Publishing a Product Version makes a new version available for controlled deployment.

Example:

```text
Product A

4.1
4.2
4.3 ← newly published
```

Publishing does not necessarily modify any running Product Instance.

---

# Release Product Version

**Type:** Composite Service

## Intent

Roll out a selected Product Version across a defined Release Scope of eligible Product Instances.

Conceptually:

```text
Product Version
      ↓
Release Product Version
      ↓
Resolve Release Scope
      ↓
Resolve eligible Product Instances
      ↓
Evaluate Release Contract
      ↓
Apply Release Strategy
      ↓
Upgrade Product Instance
      ↓
Observe / Validate
      ↓
Continue / Pause / Rollback
```

---

# Release Scope

Release Scope determines which Product Instances are targeted by a release.

Initial scope capabilities may include:

```text
Environment Type
Explicit Product Instance selection
```

Example:

```text
Product: Product A
Version: 4.3

Scope:
Customer / Test
```

PlatformAI resolves the Product Instances matching that scope.

---

# Future Release Scope

Release Scope may later support:

```text
Customer cohort
Region
Environment Flavour
Cloud provider
Tags / labels
Percentage rollout
Explicit customer selection
```

These capabilities should extend Release Management without creating separate release services.

---

# Release Strategy

A Release Strategy determines how the selected Product Instances are upgraded.

Initial conceptual strategies include:

```text
All at once
Progressive rollout
Manual promotion
```

Example progressive release:

```text
Product A v4.3

Customer / Production
        ↓
Pilot
        ↓
Observe
        ↓
20%
        ↓
Observe
        ↓
50%
        ↓
Observe
        ↓
100%
```

At every stage, PlatformAI may validate health before continuing.

---

# Environment-Aware Release Targeting

Release Management reuses Environment Types rather than defining separate release services.

For example:

```text
Internal / Development
Internal / Test
Internal / Staging
Customer / Test
Customer / Production
```

This enables workflows such as:

```text
Release Product A v4.3

Target:
Customer / Test

Strategy:
All at once
```

or:

```text
Release Product A v4.3

Target:
Customer / Production

Strategy:
Progressive
```

There should not be separate services such as:

```text
Release Product to Test
Release Product to Production
```

when Release Scope can express the difference.

---

# Release Contract / Release Metadata

A Product Version may carry structured release knowledge describing how it should be safely released.

This information forms the Product Version's Release Contract.

The Release Contract may include:

```text
Product Version
Supported upgrade paths
Migration awareness
Migration strategy
Startup/init task changes
Required pre-actions
Required post-actions
Configuration changes
Secret changes
Add-on compatibility
Environment requirements
Rollback constraints
Validation expectations
Operational notes
```

---

# Why Release Metadata Matters

Important release knowledge often exists only in human-readable release tickets, documentation, or tribal knowledge.

For example:

```text
Startup task introduced
Migration changed
Manual pre-action required
Configuration must be changed before rollout
Rollback requires database restoration
```

PlatformAI should be able to preserve this information as structured release knowledge.

This supports:

- Safer releases
- Release validation
- Historical traceability
- Incident investigation
- AI-assisted troubleshooting
- Progressive rollout decisions

---

# Release Contract Ownership

Product teams own the product-specific information contained in the Release Contract.

PlatformAI consumes this information to safely orchestrate releases.

> **PlatformAI orchestrates product lifecycle contracts rather than absorbing implementation responsibilities already owned by the product.**

---

# Release Validation

Before and during a fleet release, PlatformAI may evaluate:

```text
Is the source version supported?
Is the target version compatible?
Are enabled Add-ons compatible?
Does the Environment satisfy requirements?
Is a migration involved?
Who owns the migration?
Are pre-actions required?
Are rollback constraints understood?
Are required approvals satisfied?
```

During rollout:

```text
Deploy
  ↓
Observe
  ↓
Validate
  ↓
Healthy?
 ├── Yes → Continue
 └── No  → Pause / Rollback / Human Decision
```

---

# Composite Business Service: Onboard Customer

`Onboard Customer` is the first major business-oriented Composite Service in Service Catalog v0.1.

Its goal is to provide Customer Success with a simple customer onboarding experience while PlatformAI coordinates the underlying engineering capabilities.

---

# Onboard Customer

**Type:** Composite Service

## Consumer Intent

> Provision the required environment and products for an existing customer using approved platform definitions and selected optional capabilities.

---

# Customer Selection

Customer Success selects an existing Customer from PlatformAI.

The Customer may originate from:

```text
Native PlatformAI Customer Management
```

or eventually:

```text
External CRM / Business System
```

Customer creation is not implicitly performed as part of onboarding.

---

# Environment Selection

The user selects the required environment characteristics.

Example:

```text
Environment Type
→ Customer / Production

Environment Profile
→ Medium

Environment Flavour
→ Cloud
```

PlatformAI owns the technical definition behind those choices.

Customer Success selects approved options rather than defining infrastructure specifications directly.

---

# Conditional Environment Inputs

The form may adapt based on Environment Flavour.

## Cloud

Possible conditional inputs include:

```text
Cloud provider
Region
Network requirements
Allowed provider-specific configuration
```

## On-Prem

Possible conditional inputs include:

```text
Customer infrastructure information
Connectivity information
Server prerequisites
Network information
Other On-Prem requirements
```

Cloud and On-Prem may therefore have different workflows while remaining part of the same `Provision Environment` service.

---

# Product Selection

Customer Success selects the Product or Products required for the customer.

The UI should present valid selections based on available Product Definitions and applicable policies.

---

# Add-on Selection

Add-ons are optional.

Customer Success may select entitled Add-ons for the selected Product.

Example:

```text
Product A

Add-ons:
☑ Advanced Reporting
☐ External Integration
```

Entitlement constrains the available choices but does not necessarily make the selection on behalf of the user.

A valid selection may also be:

```text
Add-ons:
None
```

---

# Network Access

Network Access is optional unless policy or Product requirements make it mandatory for a specific request.

A valid onboarding request may specify:

```text
Network Access:
None
```

This must not be interpreted as pending work.

---

# Optional Capability States

Optional onboarding capabilities should distinguish at least:

```text
NOT_REQUESTED
REQUESTED
PENDING
COMPLETED
FAILED
```

`NOT_REQUESTED` is a valid resolved state.

It does not block customer readiness.

---

# Onboarding Inputs

Conceptually:

```text
Customer                 Required
Environment Type         Required
Environment Profile      Required
Environment Flavour      Required
Products                 Required

Cloud Region             Conditional
On-Prem Details          Conditional

Add-ons                  Optional
Network Access           Optional
```

Additional inputs may be introduced as concrete workflows require them.

---

# Onboarding Workflow

Conceptually:

```text
Select Customer
        ↓
Validate Customer
        ↓
Select Environment configuration
        ↓
Validate Environment selections
        ↓
Select Product(s)
        ↓
Select optional Add-ons
        ↓
Select optional Network Access
        ↓
Review
        ↓
Authorization / Entitlement / Policy
        ↓
Submit
        ↓
Provision Environment
        ↓
Install Product(s)
        ↓
Core Ready
        ↓
Execute requested follow-up operations
        ├── Enable Add-ons
        └── Configure Network Access
        ↓
Validate
        ↓
Ready
```

Not all follow-up operations need to execute immediately.

---

# Long-Running Onboarding

Customer onboarding should not be modeled as one synchronous transaction.

Some operations may require:

```text
External information
Approval
Customer action
Long-running infrastructure operations
Delayed configuration
Manual intervention
```

Therefore `Onboard Customer` may remain active for an extended period.

---

# Onboarding States

Possible conceptual states include:

```text
REQUESTED
ENVIRONMENT_PROVISIONING
PRODUCTS_INSTALLING
CORE_READY
PENDING_CONFIGURATION
PARTIALLY_READY
READY
FAILED
```

These states are conceptual and may evolve during implementation design.

---

# Core Ready

`CORE_READY` means the core requested environment and required products are operational.

Optional requested capabilities may still be processing.

---

# Partially Ready

A customer may be partially ready when the core platform is operational but one or more requested follow-up operations remain incomplete.

Example:

```text
Customer ABC

Environment ............. Ready
Product A ............... Ready
Network Access .......... Pending customer IP
Advanced Reporting ...... Pending approval

Overall ................. Partially Ready
```

---

# Ready

A customer reaches `READY` when all readiness requirements associated with the actual onboarding request are satisfied.

Example:

```text
Customer XYZ

Environment ............. Ready
Product A ............... Ready
Network Access .......... Not Requested
Add-ons ................. Not Requested

Overall ................. Ready
```

Optional capabilities explicitly set to `None` do not block readiness.

---

# Delayed Operations

Requested Add-ons and Network Access may execute after core onboarding.

Because they are reusable Atomic Services, they may also be requested independently later.

For example:

```text
Customer initially onboarded

Add-ons:
None
```

Three months later:

```text
Enable Product Add-on
→ Advanced Reporting
```

No customer re-onboarding is required.

---

# Workflow Persistence

Long-running Composite Services require durable workflow state.

The future Workflow Engine should be capable of supporting:

```text
Persisted workflow state
Retries
Waiting conditions
Approval gates
Resumability
Timeout handling
Failure handling
Compensation where appropriate
Human intervention
```

The implementation mechanism is intentionally not selected in Service Catalog v0.1.

---

# Initial Service Catalog Summary

## Customer

```text
Customer Management
├── Create
├── Read
├── Update
└── Safe Delete

Composite Lifecycle Services
├── Deactivate Customer
├── Reactivate Customer
└── Decommission Customer
```

---

## Environment

```text
Provision Environment
Suspend Environment
Resume Environment
Decommission Environment
```

---

## Product Instance

```text
Install Product
Upgrade Product Instance
Rollback Product Instance
Remove Product Instance
Enable Product Add-on
Disable Product Add-on
```

---

## Release Management

```text
Publish Product Version

Release Product Version
└── Resolve Release Scope
    └── Upgrade Product Instance(s)
```

---

## Business Outcomes

```text
Onboard Customer
```

Additional Composite Services will be introduced only when justified by concrete user workflows.

---

# Architectural Boundaries Reinforced by the Catalog

```text
Persona ≠ Role

Identity ≠ Authorization

Authorization ≠ Entitlement

Product Definition ≠ Product Instance

Product Version ≠ Product Instance Version State

Environment Definition ≠ Environment Instance

Environment Type ≠ Environment Profile

Environment Profile ≠ Environment Flavour

Service Intent ≠ Execution Technology

Atomic Service ≠ Composite Service

Customer State ≠ Customer Lifecycle Workflow

Deactivate ≠ Decommission ≠ Delete

Publish Product Version ≠ Release Product Version

Release Product Version ≠ Upgrade Product Instance

Product Migration Logic ≠ Platform Orchestration

Cloud Provider ≠ Environment Domain

Optional / Not Requested ≠ Pending
```

---

# Architecture Findings Emerging from Service Catalog v0.1

Service Catalog design has exposed several concepts that may justify future Domain Architecture evolution.

These include:

- Customer Domain
- Customer Provider boundary
- Customer lifecycle
- Provider abstraction
- GCP provider adapter
- Release Management
- Release Scope
- Release Contract / Release Metadata
- Long-running Composite Service state
- Environment-Flavour-specific lifecycle strategies

These findings should continue to be tested before being promoted into Domain Architecture v0.3.

---

# Out of Scope

Service Catalog v0.1 intentionally does not define:

- REST or GraphQL APIs
- Database schemas
- Event schemas
- Workflow engine technology
- UI implementation
- GCP adapter implementation
- Terraform module implementation
- CRM integration implementation
- AWS adapter
- Azure adapter
- Full multi-cloud support
- Release automation implementation
- Persistence implementation
- Domain Architecture v0.3

These concerns will be addressed in later design and implementation phases.

---

# Next Steps

After Service Catalog v0.1:

1. Review the catalog against PlatformAI personas.
2. Validate service ownership and authorization boundaries.
3. Validate the catalog against additional real operational workflows.
4. Identify concepts that have sufficient evidence for Domain Architecture v0.3.
5. Define Service Marketplace interaction flows.
6. Begin detailed design of selected services.
7. Select the first implementation slice.

---

# Guiding Questions

When adding a new service to PlatformAI, ask:

> **Does this represent a meaningful user or platform intent?**

> **Can an existing service express this behavior through parameters or composition?**

> **Is this a reusable operation or a higher-level business outcome?**

> **Are we exposing implementation details that should remain behind the platform boundary?**

> **Does this service remain valid regardless of the underlying execution technology or cloud provider?**

> **What evidence proves that the requested outcome was actually achieved?**
