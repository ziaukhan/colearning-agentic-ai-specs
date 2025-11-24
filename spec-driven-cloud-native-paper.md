# Spec-Driven Cloud-Native Architecture: Governing AI Agents for Managed Services with Claude Code and Spec Kit

## Abstract

The rapid adoption of AI in software development has given rise to "Vibe Coding"—a practice where developers rely on loose instructions to generate configuration. While efficient for simple applications, this approach is dangerous for cloud-native architectures, where a single misconfiguration in Helm charts, Dapr components, or Kafka topics can lead to catastrophic security breaches, data loss, or compliance violations. This paper proposes a **Spec-Driven Development (SDD)** framework specifically designed for managed service architectures. By combining **GitHub Spec Kit** (to define a "Constitution") with **Claude Code** (to enforce it via Agent Skills), we create a "Super-Spec" architecture that transforms AI from a risky configuration generator into a governed, safety-conscious engineering partner. This approach addresses the unique challenges of cloud-native deployments: Helm chart misconfigurations, Dapr component security, Kafka event governance, and database connection management—all while leveraging managed services that eliminate traditional infrastructure provisioning concerns.

---

## 1. The Foundation: Understanding Cloud-Native's Unique Nature

### 1.1 Why Managed Services Demand a Different Approach

In a managed services architecture, you don't provision infrastructure—you *configure* pre-existing platforms. This fundamentally changes what "infrastructure as code" means.

**Traditional Infrastructure Approach:**
```
You provision: VPCs, servers, databases, load balancers
You manage: OS patches, scaling, backups, networking
Risk: Misconfiguration at infrastructure level
```

**Managed Services Approach:**
```
Provider provisions: Kubernetes clusters, Kafka brokers, database servers
You configure: Helm charts, Dapr components, connection strings, event schemas
Risk: Misconfiguration at application/integration level
```

The shift from infrastructure provisioning to service configuration doesn't reduce risk—it changes where risk lives.

#### The Application Code Model: Creativity Valued

When you ask ten developers to build a shopping cart feature, you'll receive ten different implementations. Developer A might optimize for speed using aggressive caching. Developer B might prioritize user experience with real-time updates. Developer C might focus on mobile optimization. **All ten solutions can be equally valid.** This creative diversity represents a strength of software development.

#### The Cloud-Native Configuration Model: Reproducibility Required

Now consider deploying microservices across multiple Kubernetes namespaces (development, staging, production). You need **exactly identical** configurations for each microservice. Not "similar." Not "creatively interpreted." **Exactly. The. Same.**

**Why Reproducibility is Non-Negotiable:**

1. **Predictable Behavior:** If development environment works, production should work identically
2. **Simplified Debugging:** Identical configurations mean issues are reproducible across environments
3. **Security Consistency:** Same RBAC rules, network policies, and secrets management everywhere
4. **Compliance:** Regulatory requirements demand identical controls across all namespaces
5. **Event-Driven Reliability:** Kafka topics, Dapr pub/sub, and event schemas must be consistent

**Bad Example - "Creative" Helm Configuration (Disaster Waiting to Happen):**

```yaml
# Development namespace deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: development
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: user-service
        image: myregistry.io/user-service:latest  # Using 'latest' tag!
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        env:
        - name: DATABASE_URL
          value: "postgres://admin:password123@db:5432/users"  # Hardcoded credentials!
        - name: LOG_LEVEL
          value: "DEBUG"

---
# Production namespace deployment (someone got "creative")
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: production
spec:
  replicas: 10  # Different scaling!
  template:
    spec:
      containers:
      - name: user-service
        image: myregistry.io/user-service:v1.2.3  # Specific version
        resources:
          limits:
            memory: "256Mi"  # Different resources!
            cpu: "500m"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:  # Different credential management!
              name: db-secret
              key: connection-string
        - name: LOG_LEVEL
          value: "INFO"  # Different logging!
```

This "creative" variation causes:
- **Unpredictable deployments:** Development uses `latest` tag (moving target), production uses versioned images
- **Security vulnerability:** Development has hardcoded credentials in plain text
- **Resource inconsistency:** Different memory/CPU limits mean different performance characteristics
- **Debugging nightmares:** Different log levels hide issues until production
- **Configuration drift:** No single source of truth

### 1.2 The Declarative Nature of Cloud-Native Configuration

Modern cloud-native tools (Helm, Kubernetes, Dapr) are inherently **declarative**. You tell the system *what* the final result should look like, not how to achieve it.

**The Restaurant Analogy:**

**Imperative Approach (Step-by-Step):**
```
1. Walk to the kitchen
2. Get a pan
3. Turn on the stove to medium heat
4. Crack two eggs
5. Add salt and pepper
6. Stir for 2 minutes
7. Plate the eggs
```

**Declarative Approach (Desired State):**
```
"I want scrambled eggs."
(The chef figures out the steps)
```

**Helm Chart Example - Declarative Configuration:**

```yaml
# values.yaml - Declarative description of desired state
replicaCount: 3

image:
  repository: myregistry.io/user-service
  tag: "1.2.3"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

dapr:
  enabled: true
  appId: user-service
  appPort: 8080
```

This says "I want 3 replicas of this service with these resource limits" without specifying *how* Kubernetes should create them. Kubernetes figures out:
- Which nodes to schedule pods on
- How to distribute replicas for high availability
- When to trigger autoscaling
- How to handle rolling updates

**The Cloud-Native Advantage:** Helm charts, Dapr components, and Kubernetes manifests are already specifications. They describe the desired state declaratively.

### 1.3 The Missing Link: Governance for Managed Services

Here's the critical insight: **Since cloud-native configuration is already declarative, why do we need a new "Spec-Driven" methodology?**

The answer lies in **governance**. While Helm charts describe *what* to deploy, they do not inherently know *what is allowed*.

**The Real-World Problem:**

A perfectly skilled AI agent can generate flawless Helm charts that:
- Pull container images from untrusted registries (supply chain attack)
- Deploy services with excessive RBAC permissions (privilege escalation)
- Connect to Kafka topics without proper authentication (data leak)
- Store secrets in plain text ConfigMaps (credential exposure)
- Disable Dapr mTLS for "easier debugging" (man-in-the-middle attacks)
- Create database connections without connection pooling (DoS from connection exhaustion)

The configuration is syntactically perfect. It will deploy successfully. But it violates organizational policies, security requirements, or compliance mandates.

**What's Missing:**
We need a specification not just for the **configuration**, but for the **constraints**. We need to define:
- Which container registries are allowed
- What RBAC policies are mandatory
- Which Kafka topics can be accessed
- How secrets must be managed
- What Dapr security features are required
- How database connections must be configured

This is where the Super-Spec architecture comes in.

---

## 2. The Current State: Three Configuration Barriers to AI Orchestration

Before introducing the solution, we must understand why current configuration organization prevents AI agents from safely orchestrating cloud-native deployments.

### 2.1 Problem One: Helm Chart Chaos (Configuration Sprawl)

Many organizations have created sprawling Helm chart repositories where values, templates, and configurations tangle together chaotically.

**Example - Monolithic Values File (Simplified Excerpt):**

```yaml
# values.yaml (1500 lines of chaos)

# Some microservice configs
userService:
  replicas: 3
  image: registry.internal/user-service:v1.2.3
  database:
    host: postgres-primary.databases.svc.cluster.local
    port: 5432
    name: users

# Suddenly Kafka topics
kafka:
  bootstrap: kafka-brokers.kafka.svc.cluster.local:9092
  topics:
    - user-events
    - order-events
  security:
    enabled: false  # DANGEROUS!

# Random Dapr config
dapr:
  components:
    - statestore-redis
    - pubsub-kafka
  security:
    mtls: false  # DANGEROUS!

# Back to another service
orderService:
  replicas: 5
  image: registry.external/order-service:latest  # Using external registry!
  
# Database connection (different pattern!)
database:
  connectionString: "Server=sqlserver;Database=orders;User Id=admin;Password=Pass123!"  # Hardcoded!

# More microservices...
paymentService:
  # References userService database for some reason
  userDb: postgres-primary.databases.svc.cluster.local

# 1400 more lines like this...
```

**Why This is Catastrophic:**

1. **Unknowable Blast Radius:** Change Kafka security settings, accidentally break three microservices
2. **Hidden Dependencies:** Payment service somehow depends on user service database
3. **Inconsistent Patterns:** Each service configures databases differently
4. **Security Inconsistency:** Some services have mTLS, some don't
5. **AI Cannot Reason About It:** No way for an agent to determine "what will this change affect?"

### 2.2 Problem Two: Heterogeneous Configuration Management

A typical cloud-native environment uses multiple configuration approaches:

**Example - Multi-Pattern Configuration:**

```
Cloud-Native Stack:
├── Helm Charts (Kubernetes deployments)
│   ├── Input: values.yaml
│   ├── Templates: Go templates
│   └── Output: Kubernetes YAML
│
├── Dapr Components (Service mesh)
│   ├── Input: component.yaml
│   ├── Configuration: Various component types
│   └── Output: Runtime configuration
│
├── Kafka Configurations (Event streaming)
│   ├── Input: topic-config.json
│   ├── ACLs: kafka-acls.yaml
│   └── Schema Registry: avro schemas
│
├── Database Migrations (Schema management)
│   ├── Input: migration SQL files
│   ├── State: Migration history
│   └── Output: Database schema
│
└── ConfigMaps/Secrets (Runtime config)
    ├── Input: key-value pairs
    ├── State: Kubernetes etcd
    └── Output: Environment variables
```

**The Orchestration Problem:**

Each configuration type has:
- **Different formats:** YAML vs JSON vs SQL vs Avro
- **Different validation:** Helm lint vs Dapr validation vs Kafka topic validation
- **Different deployment:** `helm install` vs `kubectl apply` vs `kafka-topics.sh`
- **Different versioning:** Helm releases vs Dapr component versions vs Kafka topic schemas
- **Hidden dependencies:** Helm chart depends on Dapr component depends on Kafka topic

**The Integration Nightmare:**

An AI agent trying to deploy a new microservice must:
1. Create Helm chart with proper values
2. Configure Dapr components for state/pub-sub
3. Create Kafka topics with correct schemas
4. Set up database connection strings
5. Ensure all pieces reference each other correctly
6. Deploy in correct order (database → Kafka → Dapr → application)

One missing piece or wrong reference breaks everything.

### 2.3 Problem Three: Configuration Drift in Managed Services

Configuration drift in managed services is different but equally dangerous. It occurs when runtime configurations diverge from declared configurations.

**Example - Drift in Kubernetes:**

**What Your Helm Chart Says:**
```yaml
# values.yaml
replicaCount: 3

resources:
  limits:
    memory: "512Mi"
    cpu: "500m"

env:
  - name: LOG_LEVEL
    value: "INFO"
```

**What Actually Runs in Kubernetes:**
```yaml
# kubectl get deployment user-service -o yaml
spec:
  replicas: 8  # ← DIFFERENT! Someone scaled manually
  template:
    spec:
      containers:
      - name: user-service
        resources:
          limits:
            memory: "1Gi"  # ← DIFFERENT! Changed during incident
            cpu: "1000m"   # ← DIFFERENT!
        env:
        - name: LOG_LEVEL
          value: "DEBUG"   # ← DIFFERENT! Debugging never turned off
        - name: FEATURE_FLAG_X
          value: "true"    # ← NEW! Added manually
```

**How Drift Happens - Real Cloud-Native Scenarios:**

**Scenario 1: The Midnight Scaling Event**
```
02:30 AM - Traffic spike, services struggling
02:35 AM - On-call engineer: kubectl scale deployment user-service --replicas=8
02:40 AM - Crisis averted
02:41 AM - Engineer goes back to sleep
         - (Forgets to update Helm chart)

Result: Helm says 3 replicas, Kubernetes runs 8
```

**Scenario 2: The Debug Flag That Never Got Removed**
```
Week 1: Debugging production issue
        kubectl set env deployment/user-service LOG_LEVEL=DEBUG

Week 2: Issue resolved, team moves on
        (LOG_LEVEL=DEBUG still running)

Week 3: Performance degrading from excessive logging
        (Nobody realizes DEBUG logging still on)

Result: Helm says INFO, Kubernetes runs DEBUG
```

**Scenario 3: The Manual Kafka Topic Creation**
```
Sprint 1: Dev team needs new Kafka topic "user-profile-updates"
Sprint 1: Creates topic manually via Kafka UI (urgent feature)
Sprint 2: Topic works fine, team ships feature
Sprint 3: Nobody documented the topic configuration
Sprint 4: Need to recreate environment, topic config lost

Result: Production has topic, configuration not in source control
```

**Why This Breaks AI Orchestration:**

```python
# AI Agent's Perspective
def deploy_microservice():
    # Step 1: Read Helm chart
    desired_config = read_helm_values()
    # desired_config says: 3 replicas, 512Mi memory
    
    # Step 2: ???
    # The AI has NO IDEA what actually runs in Kubernetes
    # Is it safe to apply this?
    # Will this scale down 8 replicas to 3 during peak traffic?
    # Will reducing memory kill pods that were working?
    
    # Step 3: Apply blindly and hope?
    helm.upgrade(desired_config)  # DANGEROUS!
```

---

## 3. The Solution: The "Super-Spec" Architecture for Cloud-Native

To solve AI configuration chaos in managed services, we introduce the **Super-Spec**—a governance layer that sits above configuration files and enforces organizational constraints.

### 3.1 The Legislative and Executive Model

We implement this using a "Constitutional" governance model adapted for cloud-native architectures:

**The Architecture:**

```
┌──────────────────────────────────────────────┐
│         THE CONSTITUTION                     │
│      (GitHub Spec Kit)                       │
│   .specify/memory/constitution.md            │
│                                              │
│  • Container Security Rules                  │
│  • RBAC Policies                             │
│  • Dapr Configuration Standards              │
│  • Kafka Event Governance                    │
│  • Database Connection Security              │
│  • Resource Limits                           │
└──────────────────────────────────────────────┘
                    ↓
          ┌─────────────────────┐
          │   AGENT SKILLS      │
          │  (Claude Code)      │
          │ .claude/skills/     │
          │                     │
          │ • constitution-     │
          │   enforcer/         │
          │ • helm-chart-       │
          │   generator/        │
          │ • dapr-validator/   │
          │                     │
          │ Skills autonomously │
          │ invoked by Claude   │
          └─────────────────────┘
                    ↓
          ┌─────────────────┐
          │ CLOUD-NATIVE    │
          │ CONFIGURATION   │
          │ (Helm/Dapr/Kafka)│
          └─────────────────┘
```

**The Components:**

1. **The Constitution (The Law):** `.specify/memory/constitution.md` - Rules defining what's allowed
2. **Agent Skills (The Enforcers):** `.claude/skills/*` - Specialized capabilities that validate and enforce
3. **The Configuration:** Helm charts, Dapr components, Kafka topics that get deployed

**How It Works:**

- Constitution stored in `.specify/memory/constitution.md`
- Skills stored as directories in `.claude/skills/`
- Each skill has a `SKILL.md` file with YAML frontmatter
- Claude **autonomously invokes** skills based on their descriptions
- Skills load Constitution and enforce rules
- User never manually selects skills - Claude does it automatically

**Why This Works:**

Traditional Approach:
```
User Request → AI → Generate Config → Deploy
                    ↑
              (No guardrails)
```

Super-Spec Approach with Agent Skills:
```
User Request 
    ↓
Claude analyzes request
    ↓
Claude decides which skill is relevant (autonomous invocation)
    ↓
Skill loads → Reads Constitution → Validates request
    ↓
If compliant: Generate code → Self-validate → Present
If violates: Refuse → Cite rule → Propose alternative
    ↓
Deploy (after human approval)
```

**Key Innovation: Model-Invoked Skills**

Unlike traditional tools where you explicitly call them, Agent Skills are **model-invoked**:
- You don't type "@constitution-enforcer" or select from a menu
- Claude reads skill descriptions and **autonomously decides** which to use
- When you say "create a Helm chart", Claude automatically invokes `helm-chart-generator`
- That skill automatically invokes `constitution-enforcer` to validate
- All happens transparently based on skill descriptions

**Progressive Disclosure:**

Skills use a three-tier loading strategy:
1. **Startup:** Only metadata (name + description) loads (~100 tokens per skill)
2. **Triggered:** Full SKILL.md instructions load (~5k tokens)
3. **As needed:** Supporting files/scripts load only when referenced

This means you can have dozens of skills with minimal context window impact.

---

## 4. Detailed Implementation: Building the Cloud-Native Super-Spec

### 4.0 Understanding Agent Skills Architecture

Before implementing the Super-Spec, it's crucial to understand how Claude Code Agent Skills actually work, as they're fundamentally different from traditional configuration or prompt engineering.

#### What Are Agent Skills?

Agent Skills are **directories containing a SKILL.md file** that Claude loads dynamically when relevant. Think of them as specialized training manuals that give Claude domain-specific expertise.

**Key Characteristics:**

1. **Model-Invoked, Not User-Invoked:**
   - You DON'T manually select or call skills
   - Claude autonomously decides which skills to use
   - Decision based on skill descriptions matching your request
   - Similar to how Claude chooses which built-in tools to use

2. **Progressive Disclosure:**
   - **Level 1 (Startup):** Only metadata loads (name + description, ~100 tokens each)
   - **Level 2 (Triggered):** Full SKILL.md instructions load (~5k tokens)
   - **Level 3 (As needed):** Supporting files/scripts load only when referenced
   - This enables dozens of skills without overwhelming context window

3. **Filesystem-Based:**
   - Skills are **folders on disk**, not prompt snippets
   - Each skill is a directory with a `SKILL.md` file
   - Can include supporting files: scripts, templates, references

#### Two Types of Skills

**Personal Skills (`~/.claude/skills/`):**
- Available across ALL your projects
- Useful for your individual workflows
- Not shared with team

**Project Skills (`.claude/skills/`):**
- Live in project repository
- Shared via git with entire team
- Everyone gets the same capabilities
- **This is what we'll use for Constitution enforcement**

#### SKILL.md File Structure

Every skill requires a `SKILL.md` file with this structure:

```markdown
---
name: skill-name
description: What this skill does and when Claude should use it. This description is HOW Claude decides to invoke the skill, so be specific and include trigger keywords.
---

# Skill Name

## Instructions

Step-by-step guidance for Claude on how to use this skill...

## Examples

Concrete examples of skill usage...

## When to Use This Skill

Explicit guidance on when this skill is/isn't appropriate...
```

**The description field is critical** - it's the primary signal Claude uses to determine when to invoke a skill.

#### How Skills Differ from CLAUDE.md

This is a common point of confusion:

**CLAUDE.md (Optional Project Context):**
- Provides general project information
- Describes what your project is about
- NOT for defining agent capabilities
- Just helpful context

**Skills (Agent Capabilities):**
- Define specific tasks Claude can perform
- Contain executable instructions and workflows
- Loaded dynamically when relevant
- The actual "special powers" Claude gets

**Example:**
```
CLAUDE.md: "This is our Kubernetes config repo for microservices"
Skill: "Here's how to generate Constitution-compliant Helm charts"
```

CLAUDE.md provides context; skills provide capabilities.

#### Skills Can Invoke Other Skills

One powerful feature: skills can reference and trigger other skills.

**Example:**
```markdown
# In helm-chart-generator/SKILL.md

## Step 2: Invoke Constitution Enforcer

Before generating any configuration, invoke the constitution-enforcer 
skill to load all Constitutional rules.
```

When `helm-chart-generator` is triggered, it can programmatically invoke `constitution-enforcer`, creating a chain of specialized capabilities.

#### Discovering Available Skills

To see what skills are available:

```bash
# Ask Claude Code directly
"What skills are available?"

# Or check filesystem
ls ~/.claude/skills/      # Personal skills
ls .claude/skills/        # Project skills
```

Claude will list all discovered skills with their descriptions.

Now that we understand the architecture, let's build the Constitution enforcement system.

---

### 4.1 Step 1: Initialize GitHub Spec Kit

First, we establish the legislative branch—the Constitution for cloud-native deployments.

**Initialize Spec Kit in your Kubernetes configuration repository:**

```bash
# In your kubernetes-configs project directory
cd kubernetes-platform-configs

# Initialize Spec Kit
npx @github/spec-kit init

# This creates:
# .specify/
# ├── memory/
# │   └── constitution.md
# └── config.yaml
```

### 4.2 Step 2: Write the Cloud-Native Constitution

The Constitution defines immutable rules for managed services deployments.

**Example - Complete Cloud-Native Constitution:**

```markdown
# Cloud-Native Architecture Constitution
**Version:** 2.0
**Last Updated:** November 2025
**Authority:** CTO, Security Team, Compliance Officer, Platform Team

## SECTION 1: CONTAINER SECURITY MANDATES (Non-Negotiable)

### Rule 1.1: Approved Container Registries Only
**Law:** All container images must be pulled from approved registries ONLY.

**Approved Registries:**
- `registry.company.io/*` (Internal registry)
- `ghcr.io/company-org/*` (GitHub Container Registry for company)
- `mcr.microsoft.com/dotnet/*` (Microsoft official .NET images)
- `gcr.io/distroless/*` (Google distroless images)

**Prohibited:**
- Docker Hub (`docker.io/*`) - Too many supply chain attacks
- Public registries without verification
- `latest` tags - Non-deterministic deployments

**Rationale:** Supply chain security. 62% of breaches in 2024 came from compromised container images.

**Enforcement:** Any Helm chart using non-approved registry must be REFUSED.

**Example Violation:**
```yaml
image:
  repository: someuser/random-image  # ❌ Not approved registry
  tag: latest                        # ❌ Using 'latest' tag
```

**Example Compliance:**
```yaml
image:
  repository: registry.company.io/user-service
  tag: "v1.2.3-sha256-abc123"  # ✓ Internal registry with SHA
```

---

### Rule 1.2: Container Image Immutability
**Law:** All container images must use immutable tags (semantic version + SHA256 digest).

**Format:** `{registry}/{image}:{semver}-{sha256-first8}`

**Examples:**
- ✓ `registry.company.io/api:v1.2.3-sha256-a1b2c3d4`
- ❌ `registry.company.io/api:latest`
- ❌ `registry.company.io/api:v1.2.3` (missing SHA)
- ❌ `registry.company.io/api:dev`

**Rationale:** Reproducible deployments. Must be able to redeploy exact same image.

---

### Rule 1.3: No Root Containers
**Law:** ALL containers must run as non-root users.

**Required Configuration:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000  # Or higher
  fsGroup: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

**Exceptions:** NONE. No container should ever need root.

**Rationale:** Container escape vulnerabilities. Root containers can compromise entire node.

---

### Rule 1.4: Container Image Scanning
**Law:** All container images must pass security scanning before deployment.

**Required Scans:**
- Vulnerability scanning (Critical/High CVEs = block)
- License compliance check
- Secrets detection (no API keys, passwords in image)

**Enforcement:** CI/CD pipeline must include scanning step. Failed scans block deployment.

---

## SECTION 2: KUBERNETES RBAC MANDATES

### Rule 2.1: Service Account Isolation
**Law:** Every microservice must have its own dedicated ServiceAccount. NEVER use `default`.

**Required Pattern:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {service-name}-sa
  namespace: {namespace}
```

**Rationale:** Blast radius containment. Compromised service can't access other services' resources.

---

### Rule 2.2: Principle of Least Privilege
**Law:** Service accounts must have ONLY the minimum permissions required.

**Prohibited:**
- `cluster-admin` role (except for platform operators)
- `*` verbs (use specific verbs: get, list, create, etc.)
- `*` resources (specify exact resources)
- Cross-namespace access (unless explicitly documented)

**Example Violation:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: user-service-role
rules:
- apiGroups: ["*"]        # ❌ Too broad
  resources: ["*"]        # ❌ Too broad
  verbs: ["*"]            # ❌ Too broad
```

**Example Compliance:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: user-service-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]  # ✓ Specific resource
  verbs: ["get", "list"]     # ✓ Specific verbs
  resourceNames: ["user-service-config"]  # ✓ Even more specific
```

---

### Rule 2.3: Network Policy Enforcement
**Law:** Every namespace must have NetworkPolicies configured (deny-all default).

**Required Pattern:**
```yaml
# Default deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Then explicitly allow only required traffic.

**Rationale:** Zero-trust networking. Services should only communicate with authorized peers.

---

## SECTION 3: DAPR CONFIGURATION MANDATES

### Rule 3.1: mTLS Always Enabled
**Law:** Dapr mutual TLS (mTLS) must ALWAYS be enabled. No exceptions.

**Required Annotation:**
```yaml
annotations:
  dapr.io/enabled: "true"
  dapr.io/app-id: "{service-name}"
  dapr.io/app-port: "{port}"
  dapr.io/enable-mtls: "true"  # ✓ MUST be true
```

**Prohibited Configuration:**
```yaml
annotations:
  dapr.io/enable-mtls: "false"  # ❌ NEVER ALLOWED
```

**Rationale:** Service-to-service encryption. Prevents man-in-the-middle attacks.

---

### Rule 3.2: Dapr Component Security
**Law:** All Dapr components must use authentication and encryption.

**State Stores - Required Configuration:**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: redis-master.redis.svc.cluster.local:6379
  - name: redisPassword
    secretKeyRef:          # ✓ Using secret reference
      name: redis-secret
      key: password
  - name: enableTLS
    value: "true"          # ✓ TLS enabled
auth:
  secretStore: kubernetes  # ✓ Using secret store
```

**Pub/Sub - Required Configuration:**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.kafka
  metadata:
  - name: brokers
    value: kafka-brokers.kafka.svc.cluster.local:9093  # ✓ Port 9093 = TLS
  - name: authType
    value: "certificate"   # ✓ Certificate authentication
  - name: caCert
    secretKeyRef:
      name: kafka-certs
      key: ca.crt
```

**Prohibited:**
```yaml
metadata:
- name: enableTLS
  value: "false"  # ❌ NEVER
- name: authType
  value: "none"   # ❌ NEVER
```

---

### Rule 3.3: Dapr App-ID Naming Convention
**Law:** Dapr app-id must follow naming convention: `{namespace}-{service-name}`

**Examples:**
- ✓ `production-user-service`
- ✓ `staging-order-service`
- ❌ `userservice` (missing namespace)
- ❌ `prod-user-svc` (inconsistent abbreviation)

**Rationale:** Namespace isolation and traceability in logs.

---

## SECTION 4: KAFKA EVENT GOVERNANCE MANDATES

### Rule 4.1: Topic Naming Convention
**Law:** Kafka topics must follow strict naming: `{domain}.{entity}.{event-type}`

**Examples:**
- ✓ `users.profile.updated`
- ✓ `orders.payment.completed`
- ✓ `inventory.stock.depleted`
- ❌ `user_events` (not following convention)
- ❌ `OrderUpdated` (wrong case)

**Rationale:** Discoverability and event catalog organization.

---

### Rule 4.2: Schema Registry Requirement
**Law:** ALL Kafka topics must have schemas registered in Schema Registry (Avro or JSON Schema).

**Required:** Every event must have:
1. Schema definition in Schema Registry
2. Version number
3. Compatibility mode set (BACKWARD, FORWARD, or FULL)

**Prohibited:**
- Schema-less events
- JSON without JSON Schema validation
- Breaking schema changes without version bump

**Example Schema Registration:**
```json
{
  "type": "record",
  "name": "UserProfileUpdated",
  "namespace": "com.company.users.events",
  "fields": [
    {"name": "userId", "type": "string"},
    {"name": "email", "type": ["null", "string"], "default": null},
    {"name": "updatedAt", "type": "long"},
    {"name": "version", "type": "int", "default": 1}
  ]
}
```

---

### Rule 4.3: Kafka Authentication Required
**Law:** All Kafka connections must use SASL authentication with TLS encryption.

**Required Connection Configuration:**
```yaml
kafka:
  brokers: "kafka-brokers.kafka.svc.cluster.local:9093"  # Port 9093 = TLS
  security:
    protocol: "SASL_SSL"           # ✓ SASL with TLS
    sasl:
      mechanism: "SCRAM-SHA-512"   # ✓ Strong mechanism
      username: 
        secretKeyRef:
          name: kafka-credentials
          key: username
      password:
        secretKeyRef:
          name: kafka-credentials
          key: password
```

**Prohibited:**
```yaml
security:
  protocol: "PLAINTEXT"  # ❌ NEVER - No encryption
```

---

### Rule 4.4: Topic Access Control Lists (ACLs)
**Law:** Every service must have explicit ACLs defining which topics it can produce to or consume from.

**Pattern:** Service can only access topics in its domain unless explicitly approved.

**Example:**
```
user-service:
  - PRODUCE: users.profile.*
  - CONSUME: users.auth.*

order-service:
  - PRODUCE: orders.*.* 
  - CONSUME: users.profile.updated  # Explicitly approved cross-domain
  - CONSUME: orders.*.*
```

**Rationale:** Prevent services from accidentally (or maliciously) accessing unauthorized events.

---

## SECTION 5: DATABASE CONNECTION MANDATES

### Rule 5.1: Connection String Security
**Law:** Database connection strings must NEVER be in plain text. Must use Kubernetes Secrets.

**Prohibited:**
```yaml
env:
- name: DATABASE_URL
  value: "Server=db.example.com;Database=users;User=admin;Password=Pass123!"  # ❌
```

**Required:**
```yaml
env:
- name: DATABASE_URL
  valueFrom:
    secretKeyRef:
      name: database-credentials
      key: connection-string  # ✓ From secret
```

**Alternative (Dapr Secret Store):**
```yaml
# Dapr handles secret retrieval
# Application gets connection string from Dapr API
```

---

### Rule 5.2: Connection Pooling Required
**Law:** All database connections must use connection pooling with limits.

**Required Configuration:**
```yaml
database:
  pool:
    minConnections: 5
    maxConnections: 20      # ✓ Explicit limit prevents connection exhaustion
    connectionTimeout: 30s
    idleTimeout: 600s
```

**Rationale:** Prevent connection exhaustion attacks and improve performance.

---

### Rule 5.3: Read Replicas for Read Operations
**Law:** Read-heavy operations must use read replicas, not primary database.

**Pattern:**
```yaml
database:
  primary:
    host: "postgres-primary.databases.svc.cluster.local"
    purpose: "writes"
  
  replica:
    host: "postgres-replica.databases.svc.cluster.local"
    purpose: "reads"
```

**Application must route:**
- Writes → Primary
- Reads → Replica

**Rationale:** Prevent read queries from impacting write performance.

---

## SECTION 6: RESOURCE MANAGEMENT MANDATES

### Rule 6.1: Resource Limits Required
**Law:** Every container must have both requests and limits defined.

**Required:**
```yaml
resources:
  requests:
    memory: "256Mi"  # ✓ Guaranteed minimum
    cpu: "250m"
  limits:
    memory: "512Mi"  # ✓ Maximum allowed
    cpu: "500m"
```

**Prohibited:**
```yaml
resources: {}  # ❌ No limits = can consume entire node
```

**Rationale:** Prevent single service from monopolizing cluster resources.

---

### Rule 6.2: Environment-Specific Resource Tiers
**Law:** Resource allocations must follow environment-specific guidelines.

**Tiers:**

**Development:**
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

**Staging:**
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**Production:**
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

**Exceptions:** High-traffic services require capacity planning document.

---

### Rule 6.3: Horizontal Pod Autoscaling
**Law:** Production services must have HPA configured.

**Required:**
```yaml
autoscaling:
  enabled: true
  minReplicas: 3      # ✓ Minimum for high availability
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

**Development/Staging:**
```yaml
autoscaling:
  enabled: false  # ✓ Fixed replicas acceptable for non-prod
replicaCount: 1
```

---

## SECTION 7: SECRETS MANAGEMENT MANDATES

### Rule 7.1: External Secrets Operator Required
**Law:** All secrets must be managed via External Secrets Operator syncing from Azure Key Vault or equivalent.

**Prohibited:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-creds
data:
  password: UGFzczEyMyE=  # ❌ Hardcoded secret in Git
```

**Required:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-creds
spec:
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: database-creds
  data:
  - secretKey: connection-string
    remoteRef:
      key: prod-db-connection-string  # ✓ From Key Vault
```

**Rationale:** Secrets should never be committed to Git. Central management enables rotation.

---

### Rule 7.2: Secret Rotation
**Law:** Database credentials and API keys must rotate every 90 days.

**Implementation:** Use managed identity where possible to eliminate long-lived credentials.

---

## SECTION 8: DEPLOYMENT MANDATES

### Rule 8.1: Helm Chart Version Pinning
**Law:** All Helm chart dependencies must be pinned to specific versions.

**Prohibited:**
```yaml
dependencies:
- name: redis
  repository: https://charts.bitnami.com/bitnami
  version: "*"  # ❌ Unpredictable
```

**Required:**
```yaml
dependencies:
- name: redis
  repository: https://charts.bitnami.com/bitnami
  version: "17.8.0"  # ✓ Specific version
```

---

### Rule 8.2: Progressive Rollout Strategy
**Law:** Production deployments must use rolling updates with proper readiness checks.

**Required:**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1    # ✓ Only 1 pod down at a time
    maxSurge: 1          # ✓ Only 1 extra pod during update

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

---

### Rule 8.3: Namespace Isolation
**Law:** Environments must be isolated in separate namespaces.

**Required Structure:**
```
Cluster:
├── development (namespace)
├── staging (namespace)
└── production (namespace)
```

**Cross-namespace access:** Prohibited unless explicitly documented and approved.

---

## SECTION 9: MONITORING AND OBSERVABILITY MANDATES

### Rule 9.1: Structured Logging Required
**Law:** All services must emit structured JSON logs.

**Required Format:**
```json
{
  "timestamp": "2025-11-24T10:30:00Z",
  "level": "INFO",
  "service": "user-service",
  "traceId": "abc123...",
  "spanId": "def456...",
  "message": "User profile updated",
  "userId": "user-123"
}
```

**Prohibited:** Plain text logs without structure.

---

### Rule 9.2: Distributed Tracing Required
**Law:** All services must participate in distributed tracing (Dapr provides this automatically).

**Required:** Trace context must propagate across:
- HTTP calls
- Kafka events
- Database operations

---

### Rule 9.3: Metrics Exposure
**Law:** All services must expose Prometheus metrics at `/metrics` endpoint.

**Required Metrics:**
- Request count
- Request duration
- Error rate
- Active connections (for databases)
- Queue depth (for Kafka consumers)

---

## SECTION 10: AGENT BEHAVIORAL RULES

### Rule 10.1: Refusal Protocol
When a user request violates ANY rule in this Constitution:

1. The agent must REFUSE the request
2. The agent must cite the specific rule number
3. The agent must propose a compliant alternative
4. The agent must explain the security/operational rationale

**Example Refusal:**
> "I cannot fulfill this request as stated. Your Helm chart specifies `image: someuser/api:latest` which violates:
>
> **Constitution Rule 1.1** - Approved Container Registries
> **Constitution Rule 1.2** - Container Image Immutability
>
> **Why these rules exist:** 
> - Unapproved registries expose us to supply chain attacks
> - The `latest` tag is non-deterministic and prevents reproducible deployments
>
> **Compliant Alternative:** 
> Use our internal registry with immutable tags:
> `image: registry.company.io/api:v1.2.3-sha256-a1b2c3d4`
>
> Would you like me to update the configuration with the approved registry?"

---

### Rule 10.2: Validation Protocol
After generating ANY configuration:

1. Agent must re-read Security Mandates
2. Cross-reference generated config against each rule
3. Explicitly confirm compliance before presenting
4. If any violation detected, self-correct or refuse

---

### Rule 10.3: Drift Detection Protocol
Before modifying deployed resources:

1. Agent must compare desired state to actual state in cluster
2. Agent must identify and document differences
3. If significant drift detected, warn about unintended changes
4. Require explicit confirmation before applying changes

**Example:**
```
⚠️ DRIFT DETECTED

Your Helm chart specifies:
  replicas: 3

Current state in Kubernetes:
  replicas: 8 (manually scaled)

Applying this chart will SCALE DOWN from 8 to 3 replicas.

Is this intentional? [y/N]
```

---

## APPENDIX A: QUICK REFERENCE

**Most Common Violations:**
1. Using unapproved container registries (Rule 1.1)
2. Missing resource limits (Rule 6.1)
3. Dapr mTLS disabled (Rule 3.1)
4. Plain text secrets (Rule 5.1, 7.1)
5. No NetworkPolicies (Rule 2.3)

**Emergency Override Contact:**
- Platform Team Lead: platform-lead@company.com
- Security Lead: security@company.com
- On-Call: oncall@company.com

---

## AMENDMENT HISTORY

| Date | Version | Change | Approved By |
|------|---------|--------|-------------|
| Nov 2025 | 2.0 | Added Kafka governance rules | Platform Team |
| Oct 2025 | 1.9 | Updated Dapr mTLS requirements | Security |
| Sep 2025 | 1.8 | Added External Secrets Operator | Security |
```

### 4.3 Step 3: Create Agent Skills for Constitution Enforcement

Now we create specialized Agent Skills that enforce the cloud-native Constitution. Unlike traditional configuration, skills are **model-invoked**—Claude autonomously decides when to use them based on their descriptions.

**Understanding Agent Skills:**

Agent Skills are directories containing a `SKILL.md` file that Claude loads dynamically when relevant. They use **progressive disclosure**:
1. Metadata (name + description) loads at startup (~100 tokens)
2. Full instructions load only when skill is triggered (<5k tokens)
3. Supporting files load only as needed

**Project Skills vs Personal Skills:**

- **Project Skills** (`.claude/skills/`): Shared with team via git, used by everyone
- **Personal Skills** (`~/.claude/skills/`): Available across all your projects

For our Constitution enforcement, we'll create **Project Skills** so the entire team benefits.

**Create the project skills directory:**

```bash
# In your kubernetes-configs project root
mkdir -p .claude/skills
```

---

#### Skill 1: Constitution Enforcer

**Create `.claude/skills/constitution-enforcer/SKILL.md`:**

```markdown
---
name: constitution-enforcer
description: Enforces cloud-native architecture Constitution rules for all Helm charts, Dapr components, Kafka configurations, and database connections. Use this skill when generating or reviewing ANY Kubernetes configuration, Helm values, Dapr components, or infrastructure config. This skill ensures all configurations comply with security mandates, RBAC policies, resource limits, and organizational standards defined in the Constitution.
---

# Constitution Enforcer

## Purpose

This skill enforces the Cloud-Native Architecture Constitution stored in `.specify/memory/constitution.md`. Every configuration must be validated against Constitutional rules before deployment.

## Instructions

### Step 1: Load the Constitution

**ALWAYS start by reading the Constitution:**

```bash
cat .specify/memory/constitution.md
```

This file contains all non-negotiable rules organized in sections:
- Section 1: Container Security Mandates
- Section 2: Kubernetes RBAC Mandates
- Section 3: Dapr Configuration Mandates
- Section 4: Kafka Event Governance Mandates
- Section 5: Database Connection Mandates
- Section 6: Resource Management Mandates
- Section 7: Secrets Management Mandates
- Section 8: Deployment Mandates
- Section 9: Monitoring and Observability Mandates
- Section 10: Agent Behavioral Rules

### Step 2: Identify Configuration Type

Determine what the user is requesting:
- **Helm Chart:** Kubernetes deployment configuration
- **Dapr Component:** Service mesh/state/pub-sub configuration
- **Kafka Topic/Consumer:** Event streaming configuration
- **Database Connection:** Data persistence configuration

### Step 3: Apply Relevant Constitutional Rules

**For Helm Charts, check:**

Container Security (Rules 1.1-1.4):
- ✓ Image from approved registry? (registry.company.io, ghcr.io/company-org, mcr.microsoft.com/dotnet, gcr.io/distroless)
- ✓ Image tag immutable? (format: v{semver}-sha256-{hash})
- ✓ Container runs as non-root? (runAsNonRoot: true, runAsUser: 1000+)
- ✓ Security scanning configured?

RBAC (Rules 2.1-2.3):
- ✓ Dedicated ServiceAccount? (NOT default)
- ✓ Principle of least privilege? (specific resources/verbs only)
- ✓ NetworkPolicy configured? (default deny-all with explicit allows)

Dapr (Rules 3.1-3.3):
- ✓ mTLS enabled? (dapr.io/enable-mtls: "true")
- ✓ App-ID naming convention? ({namespace}-{service-name})
- ✓ Component authentication configured?

Resource Management (Rules 6.1-6.3):
- ✓ Resource requests defined?
- ✓ Resource limits defined?
- ✓ Correct tier for environment? (dev/staging/prod)
- ✓ HPA configured for production?

**For Dapr Components, check:**

Security (Rule 3.2):
- ✓ TLS enabled? (enableTLS: "true")
- ✓ Authentication configured? (NOT authType: "none")
- ✓ Credentials from secrets? (secretKeyRef, not plain text)

**For Kafka Topics, check:**

Governance (Rules 4.1-4.4):
- ✓ Topic naming convention? ({domain}.{entity}.{event-type})
- ✓ Schema registered in Schema Registry?
- ✓ SASL_SSL authentication? (NOT PLAINTEXT)
- ✓ ACLs defined? (explicit producer/consumer permissions)

**For Database Connections, check:**

Security (Rules 5.1-5.3):
- ✓ Connection string from secret? (NOT plain text)
- ✓ Connection pooling configured? (min/max connections)
- ✓ Read replica used for reads?

### Step 4: Execute Refusal Protocol (If Violations Found)

If ANY Constitutional rule is violated, you MUST refuse and follow this protocol:

1. **State refusal clearly:**
   ```
   ⛔ REQUEST DENIED
   ```

2. **Identify specific violations:**
   ```
   Violates: Constitution Rule X.Y - [Rule Name]
   ```

3. **Explain the rationale:**
   ```
   Why This Rule Exists:
   [Security/operational/compliance reason from Constitution]
   ```

4. **Propose compliant alternative:**
   ```
   Compliant Alternative:
   [Specific, actionable solution that satisfies the rule]
   ```

5. **Ask for confirmation:**
   ```
   Would you like me to proceed with the compliant approach?
   ```

### Step 5: Generate Compliance Report

For EVERY configuration you generate, include a compliance report:

```
🛡️ CONSTITUTION COMPLIANCE REPORT

✅ Container Security: COMPLIANT
  • Rule 1.1: Using approved registry (registry.company.io)
  • Rule 1.2: Immutable tag (v1.2.3-sha256-abc123)
  • Rule 1.3: Running as non-root (uid 1000)

✅ Kubernetes RBAC: COMPLIANT
  • Rule 2.1: Dedicated ServiceAccount (service-name-sa)
  • Rule 2.2: Least privilege RBAC
  • Rule 2.3: NetworkPolicy configured

✅ Dapr Configuration: COMPLIANT
  • Rule 3.1: mTLS enabled
  • Rule 3.2: Components use authentication
  • Rule 3.3: App-ID naming convention followed

[Continue for all relevant sections...]

📊 Deployment Impact:
  • Resources to create: [count]
  • Estimated cost: [amount]/month
  • Deployment time: ~[minutes]

🚀 Next Steps:
[Deployment commands]
```

## Examples

### Example 1: Refusing Unapproved Registry

**User Request:** "Deploy nginx from Docker Hub"

**Response:**
```
⛔ REQUEST DENIED

Requested Configuration: nginx image from Docker Hub (docker.io)
Violates: Constitution Rule 1.1 - Approved Container Registries

Why This Rule Exists:
62% of container supply chain attacks in 2024 originated from compromised 
public registry images. We restrict to approved registries where images are 
scanned for vulnerabilities and provenance is verified.

Compliant Alternative:
I can mirror the official nginx image to our internal registry:

1. Pull official nginx: `docker pull nginx:1.25.3`
2. Tag for internal registry: `docker tag nginx:1.25.3 registry.company.io/nginx:1.25.3-sha256-abc123`
3. Push to registry: `docker push registry.company.io/nginx:1.25.3-sha256-abc123`
4. Use in Helm chart: `image: registry.company.io/nginx:1.25.3-sha256-abc123`

Would you like me to generate the compliant Helm configuration?
```

### Example 2: Enforcing mTLS

**User Request:** "Create Dapr component with mTLS disabled for easier debugging"

**Response:**
```
⛔ REQUEST DENIED

Requested Configuration: Dapr component with mTLS disabled
Violates: Constitution Rule 3.1 - mTLS Always Enabled

Why This Rule Exists:
mTLS provides service-to-service encryption and authentication. Disabling 
it enables man-in-the-middle attacks and service impersonation. This is a 
SOC2 compliance requirement with ZERO exceptions.

Compliant Alternative:
Dapr debugging works perfectly with mTLS enabled:

1. Use Dapr dashboard: `dapr dashboard -k`
2. Check sidecar logs: `kubectl logs <pod> -c daprd`
3. Use Zipkin tracing: Integrated with mTLS
4. Use dapr invoke CLI: Handles mTLS automatically

What specific debugging issue are you experiencing? I can help solve it 
without compromising security.
```

## When NOT to Use This Skill

Do NOT use this skill for:
- General Kubernetes questions unrelated to configuration
- Explaining concepts (Constitution doesn't cover education)
- Troubleshooting runtime issues (unless related to configuration violations)
- Non-infrastructure topics

## Success Criteria

You're successfully enforcing the Constitution when:
✅ Zero violations reach production
✅ Users understand WHY rules exist, not just that they do
✅ Compliant alternatives are always provided
✅ Team velocity increases (rules prevent rework)
✅ Audit trails are clear and comprehensive
```

---

#### Skill 2: Helm Chart Generator

**Create `.claude/skills/helm-chart-generator/SKILL.md`:**

```markdown
---
name: helm-chart-generator
description: Generates Constitution-compliant Helm charts for Kubernetes deployments. Use when creating new Helm charts, values.yaml files, or Kubernetes manifests. Automatically enforces container security, RBAC, resource limits, and all Constitutional requirements for cloud-native deployments.
---

# Helm Chart Generator

## Purpose

Generate production-ready, Constitution-compliant Helm charts that follow all organizational security, operational, and compliance requirements.

## Instructions

### Step 1: Gather Requirements

Ask the user for essential information:
1. Service name (e.g., "user-service", "order-api")
2. Environment (development, staging, production)
3. Service purpose (API, background worker, web app)
4. External dependencies (database, Kafka, Redis)

### Step 2: Invoke Constitution Enforcer

**ALWAYS invoke the constitution-enforcer skill first:**

```bash
# This ensures all generated configurations comply with Constitutional rules
```

The constitution-enforcer skill will load all rules and provide compliance guidance.

### Step 3: Generate Base Values.yaml

Create `values.yaml` with all Constitutional requirements pre-configured:

```yaml
# values.yaml - Constitution-compliant template

# Rule 1.1, 1.2 - Approved registry with immutable tag
image:
  repository: registry.company.io/{service-name}
  tag: "{version}-sha256-{hash}"  # MUST be immutable
  pullPolicy: IfNotPresent

# Rule 2.1 - Dedicated ServiceAccount (NEVER use default)
serviceAccount:
  create: true
  name: {service-name}-sa
  annotations: {}

# Rule 6.1, 6.2 - Resource limits (environment-specific)
resources:
  requests:
    memory: "{env-appropriate}"  # dev: 128Mi, staging: 256Mi, prod: 512Mi
    cpu: "{env-appropriate}"     # dev: 100m, staging: 250m, prod: 500m
  limits:
    memory: "{env-appropriate}"  # dev: 256Mi, staging: 512Mi, prod: 1Gi
    cpu: "{env-appropriate}"     # dev: 200m, staging: 500m, prod: 1000m

# Rule 6.3 - Autoscaling (required for production)
replicaCount: {env-appropriate}  # dev: 1, staging: 2, prod: 3
autoscaling:
  enabled: {true-for-prod}
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

# Rule 3.1, 3.3 - Dapr with mTLS (if service uses Dapr)
dapr:
  enabled: {true-if-needed}
  appId: "{namespace}-{service-name}"  # Follow naming convention
  appPort: 8080
  annotations:
    dapr.io/enable-mtls: "true"  # MANDATORY, no exceptions

# Rule 1.3 - Security context (non-root, dropped capabilities)
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true

# Rule 8.2 - Rolling update strategy
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1

# Health checks (required)
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

# Service configuration
service:
  type: ClusterIP  # Internal only unless explicitly approved
  port: 80
  targetPort: 8080

# Rule 2.3 - NetworkPolicy (required)
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      # Define allowed sources
      ports:
      - protocol: TCP
        port: 8080
  egress:
    # Define allowed destinations

# Rule 9.1, 9.2, 9.3 - Observability
monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    path: /metrics
    interval: 30s
  logging:
    format: json
    level: info
```

### Step 4: Generate Supporting Resources

Create additional required resources:

**NetworkPolicy (Rule 2.3):**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "{chart-name}.fullname" . }}
spec:
  podSelector:
    matchLabels:
      {{- include "{chart-name}.selectorLabels" . | nindent 6 }}
  policyTypes:
    - Ingress
    - Egress
  # Default deny, then explicit allows
```

**ServiceAccount with RBAC (Rules 2.1, 2.2):**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "{chart-name}.serviceAccountName" . }}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: {{ include "{chart-name}.fullname" . }}
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
  resourceNames: ["{service-name}-config"]  # Specific, not "*"
```

### Step 5: Provide Compliance Report

Include detailed compliance report showing which Constitutional rules are satisfied.

### Step 6: Provide Deployment Instructions

```bash
# 1. Validate the chart
helm lint ./chart-name

# 2. Dry-run to preview
helm install {release-name} ./chart-name --dry-run --debug

# 3. Deploy
helm install {release-name} ./chart-name -n {namespace}

# 4. Verify
kubectl get pods -n {namespace} -l app={service-name}
kubectl logs -n {namespace} -l app={service-name}

# 5. Rollback if needed
helm rollback {release-name} -n {namespace}
```

## Important Notes

- **NEVER** generate charts with `latest` tags (Rule 1.2)
- **NEVER** use `default` ServiceAccount (Rule 2.1)
- **NEVER** disable mTLS if Dapr is enabled (Rule 3.1)
- **ALWAYS** include resource limits (Rule 6.1)
- **ALWAYS** configure NetworkPolicy (Rule 2.3)
- **ALWAYS** use approved registries only (Rule 1.1)

## When to Use This Skill

Use this skill when:
- Creating new microservice Helm charts
- Standardizing existing charts
- Updating charts to meet Constitutional requirements
- Generating values.yaml templates
```

---

#### Skill 3: Dapr Component Validator

**Create `.claude/skills/dapr-validator/SKILL.md`:**

```markdown
---
name: dapr-validator
description: Validates and generates Constitution-compliant Dapr components for state stores, pub/sub, bindings, and service invocation. Use when creating or reviewing Dapr component YAML files, ensuring mTLS, authentication, TLS, and proper secret management.
---

# Dapr Component Validator

## Purpose

Ensure all Dapr components comply with Constitutional security and operational requirements (Rules 3.1-3.3).

## Instructions

### Required Validations

**Every Dapr component MUST:**

1. **Enable mTLS (Rule 3.1):**
   - Annotation: `dapr.io/enable-mtls: "true"`
   - NO exceptions

2. **Use authentication (Rule 3.2):**
   - State stores: TLS + password from secret
   - Pub/sub: Certificate or SASL authentication
   - NO plain text credentials
   - NO `authType: "none"`

3. **Follow naming convention (Rule 3.3):**
   - App-ID format: `{namespace}-{service-name}`
   - Component name: descriptive and unique

### Component Templates

**State Store (Redis) - Compliant:**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
  namespace: production
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: redis-master.redis.svc.cluster.local:6380  # TLS port
  - name: redisPassword
    secretKeyRef:  # From secret, NOT plain text
      name: redis-secret
      key: password
  - name: enableTLS
    value: "true"  # REQUIRED
auth:
  secretStore: kubernetes
scopes:
  - service-name  # Limit access
```

**Pub/Sub (Kafka) - Compliant:**
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
  namespace: production
spec:
  type: pubsub.kafka
  version: v1
  metadata:
  - name: brokers
    value: kafka-brokers.kafka.svc.cluster.local:9093  # TLS port
  - name: authType
    value: "certificate"  # NOT "none"
  - name: caCert
    secretKeyRef:
      name: kafka-certs
      key: ca.crt
  - name: clientCert
    secretKeyRef:
      name: kafka-certs
      key: client.crt
  - name: clientKey
    secretKeyRef:
      name: kafka-certs
      key: client.key
auth:
  secretStore: kubernetes
```

### Validation Checklist

Run through this checklist for every Dapr component:

```
□ Component type appropriate? (state, pubsub, bindings, etc.)
□ Namespace specified?
□ TLS enabled? (enableTLS: "true" or TLS port)
□ Authentication configured? (NOT authType: "none")
□ Secrets from secretKeyRef? (NOT plain values)
□ Scopes defined? (limit which services can use)
□ Metadata complete? (all required fields)
```

### Common Violations to Catch

**❌ VIOLATION - Plain text password:**
```yaml
metadata:
- name: redisPassword
  value: "mypassword123"  # NO! Use secretKeyRef
```

**❌ VIOLATION - TLS disabled:**
```yaml
metadata:
- name: enableTLS
  value: "false"  # NEVER allowed
```

**❌ VIOLATION - No authentication:**
```yaml
metadata:
- name: authType
  value: "none"  # NEVER allowed
```

**❌ VIOLATION - Non-TLS port:**
```yaml
metadata:
- name: brokers
  value: "kafka:9092"  # Port 9092 is plaintext, use 9093
```

## When to Use This Skill

Use when:
- Creating new Dapr components
- Reviewing existing Dapr configurations
- Troubleshooting Dapr connection issues
- Migrating to Dapr from other service mesh
```

---

### 4.4 Step 4: Enable Skills in Claude Code

**Initialize Claude Code in your project:**

```bash
# In your project root
claude-code init
```

**Claude Code will automatically discover skills in:**
- `.claude/skills/` (project skills - shared with team)
- `~/.claude/skills/` (personal skills - just for you)

**Verify skills are loaded:**

Ask Claude Code:
```
What skills are available?
```

Claude should list:
- constitution-enforcer
- helm-chart-generator
- dapr-validator

**Skills are now active!** Claude will automatically use them when relevant based on their descriptions.

---

### 4.5 Optional: Create Project Context File

While skills handle specific tasks, you can create an optional `CLAUDE.md` for general project context:

**Create `CLAUDE.md` in project root:**

```markdown
# Project Context - Cloud-Native Platform

## Project Overview

This is our Kubernetes configuration repository containing:
- Helm charts for all microservices
- Dapr component definitions
- Kafka topic configurations
- Database connection manifests

## Important Files

- `.specify/memory/constitution.md` - Our infrastructure Constitution (THE LAW)
- `.claude/skills/` - Agent skills for enforcing Constitution
- `helm-charts/` - Helm chart templates
- `dapr-components/` - Dapr component definitions
- `kafka-configs/` - Kafka topic and schema definitions

## Working with This Repository

1. **ALL configurations must comply with the Constitution**
   - Skills will enforce this automatically
   - constitution-enforcer skill validates everything

2. **Skills are your tools:**
   - Use helm-chart-generator for new charts
   - Use dapr-validator for Dapr components
   - Skills invoke each other automatically

3. **When in doubt, ask:**
   - "Does this comply with the Constitution?"
   - "Generate a compliant Helm chart for {service}"
   - "Validate this Dapr component"

## Project Structure

```
kubernetes-configs/
├── .specify/
│   └── memory/
│       └── constitution.md          # The law
├── .claude/
│   └── skills/                     # Enforcement skills
│       ├── constitution-enforcer/
│       ├── helm-chart-generator/
│       └── dapr-validator/
├── CLAUDE.md                        # This file (project context)
├── helm-charts/                     # Service deployments
├── dapr-components/                 # Service mesh configs
└── kafka-configs/                   # Event streaming configs
```

## Key Contacts

- Platform Team Lead: platform-lead@company.com
- Security Lead: security@company.com
- On-Call: oncall@company.com
```

**Note:** This `CLAUDE.md` provides project context but does NOT define skills. Skills live in `.claude/skills/` directories.

---

## 5. Real-World Scenarios: The Super-Spec in Cloud-Native Action

### Scenario 1: "The Quick Microservice Deploy"

**User Request:**
```
I need to deploy a new user authentication service to production. 
Use the auth-service image from Docker Hub. 
Make it accessible to all services in the cluster.
```

#### Outcome A: Without Super-Spec (Traditional AI)

AI generates this without question:

```yaml
# deployment.yaml - DANGEROUS!
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
  namespace: production
spec:
  replicas: 1  # Single point of failure
  template:
    spec:
      containers:
      - name: auth-service
        image: someuser/auth-service:latest  # ❌ Public Docker Hub + latest tag!
        
        # ❌ No resource limits - can consume entire node
        
        env:
        - name: JWT_SECRET
          value: "super-secret-key-123"  # ❌ Hardcoded secret!
        - name: DATABASE_URL
          value: "postgres://admin:pass@db:5432/auth"  # ❌ Plain text credentials!
        
        # ❌ No security context - running as root
        
      # ❌ No ServiceAccount specified - using default

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  type: LoadBalancer  # ❌ Exposing to internet!
  ports:
  - port: 80
    targetPort: 8080
```

**Result:**
✅ Deploys successfully
❌ **Violates 7+ Constitution rules**
❌ **Using untrusted image source**
❌ **Hardcoded secrets**
❌ **No resource limits**
❌ **Running as root**
❌ **Exposed to internet**
❌ **Single replica = no HA**
❌ **Massive security vulnerability**

#### Outcome B: With Super-Spec (Constitutional AI)

The AI acts as a responsible guardian:

```
🤖 Claude Code: Analyzing your request against Cloud-Native Constitution...

⛔ REQUEST CONTAINS MULTIPLE VIOLATIONS

Requested Configuration: Docker Hub image for production auth service
Violates Multiple Rules:

1. Constitution Rule 1.1 - Approved Container Registries
   ❌ "someuser/auth-service" from Docker Hub (docker.io) is not approved
   
2. Constitution Rule 1.2 - Container Image Immutability
   ❌ ":latest" tag is non-deterministic
   
3. Constitution Rule 6.1 - Resource Limits Required
   ⚠️  No resource limits specified
   
4. Constitution Rule 6.3 - HPA for Production
   ⚠️  Production services require autoscaling
   
5. Constitution Rule 5.1 - Connection String Security
   ⚠️  Database credentials must be in secrets

Why These Rules Exist:

**Registry Security (Rule 1.1):**
62% of container supply chain attacks in 2024 came from compromised public 
registry images. Authentication services are prime targets.

**Image Immutability (Rule 1.2):**
:latest tags are non-deterministic. If something breaks in production, you 
can't roll back to a known-good version.

**Resource Protection (Rule 6.1):**
Without limits, a compromised container can consume entire node resources,
taking down ALL services on that node.

**Secrets Management (Rule 5.1):**
Plain text credentials in environment variables are visible to anyone with
kubectl access and appear in logs/crash dumps.

Compliant Alternative:

I can deploy your authentication service with enterprise-grade security:

1. ✓ Mirror image to internal registry (registry.company.io)
2. ✓ Tag with immutable version + SHA256
3. ✓ Store all secrets in Azure Key Vault via External Secrets
4. ✓ Configure appropriate resource limits
5. ✓ Enable Dapr sidecar with mTLS
6. ✓ Set up 3-replica minimum with autoscaling
7. ✓ Deploy with proper RBAC and NetworkPolicies
8. ✓ Add health checks and monitoring

Setup time: ~20 minutes (including image mirroring)

Would you like me to generate the compliant configuration?
```

**User Response:**
```
Yes, please generate the secure configuration.
```

**AI Generates Constitution-Compliant Configuration:**

```yaml
# values.yaml - Constitution-compliant auth service

# Rule 1.1, 1.2 - Internal registry with immutable tag
image:
  repository: registry.company.io/auth-service
  tag: "v2.1.0-sha256-a1b2c3d4"
  pullPolicy: IfNotPresent

# Rule 2.1 - Dedicated ServiceAccount
serviceAccount:
  create: true
  name: auth-service-sa
  annotations:
    # Azure AD Workload Identity
    azure.workload.identity/client-id: "auth-service-identity"

# Rule 6.1, 6.2 - Production resource tier
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"

# Rule 6.3 - HPA for production
replicaCount: 3  # Minimum for HA
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

# Rule 3.1, 3.3 - Dapr with mTLS
dapr:
  enabled: true
  appId: production-auth-service  # Rule 3.3 naming convention
  appPort: 8080
  annotations:
    dapr.io/enable-mtls: "true"  # Rule 3.1 mandatory

# Rule 1.3 - Non-root security context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true

# Rule 8.2 - Progressive rollout
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3

livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

# Rule 5.1 - Secrets from External Secrets Operator
env:
  - name: JWT_SECRET
    valueFrom:
      secretKeyRef:
        name: auth-service-secrets  # From Azure Key Vault
        key: jwt-secret
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: auth-service-secrets
        key: database-connection-string

# Service configuration - ClusterIP (internal only)
service:
  type: ClusterIP  # Not LoadBalancer - internal only
  port: 80
  targetPort: 8080

# Rule 2.3 - NetworkPolicy for zero-trust
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      # Only allow traffic from ingress controller
      - namespaceSelector:
          matchLabels:
            name: ingress-nginx
      ports:
      - protocol: TCP
        port: 8080
  egress:
    # Allow database connection
    - to:
      - namespaceSelector:
          matchLabels:
            name: databases
      ports:
      - protocol: TCP
        port: 5432
    # Allow DNS
    - to:
      - namespaceSelector:
          matchLabels:
            name: kube-system
      ports:
      - protocol: UDP
        port: 53

# Rule 9.1, 9.2, 9.3 - Observability
monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    path: /metrics
    interval: 30s
  logging:
    format: json
    level: info
```

**Additional Required Resources:**

```yaml
# external-secret.yaml - Rule 7.1
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: auth-service-secrets
  namespace: production
spec:
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: auth-service-secrets
  data:
  - secretKey: jwt-secret
    remoteRef:
      key: prod-auth-jwt-secret
  - secretKey: database-connection-string
    remoteRef:
      key: prod-auth-db-connection

---
# dapr-statestore.yaml - Rule 3.2
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
  namespace: production
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: redis-master.redis.svc.cluster.local:6380
  - name: redisPassword
    secretKeyRef:
      name: redis-secret
      key: password
  - name: enableTLS
    value: "true"
auth:
  secretStore: kubernetes
scopes:
  - auth-service
```

**AI Includes Comprehensive Compliance Report:**

```
🛡️ CLOUD-NATIVE CONSTITUTION COMPLIANCE REPORT

✅ Container Security: FULLY COMPLIANT
  • Rule 1.1: Using approved registry (registry.company.io)
  • Rule 1.2: Immutable tag (v2.1.0-sha256-a1b2c3d4)
  • Rule 1.3: Running as non-root (uid 1000)

✅ Kubernetes RBAC: FULLY COMPLIANT
  • Rule 2.1: Dedicated ServiceAccount (auth-service-sa)
  • Rule 2.2: Least privilege (specific permissions only)
  • Rule 2.3: NetworkPolicy configured (zero-trust)

✅ Dapr Configuration: FULLY COMPLIANT
  • Rule 3.1: mTLS enabled (dapr.io/enable-mtls: "true")
  • Rule 3.2: State store uses TLS and authentication
  • Rule 3.3: App-ID follows naming convention

✅ Database Security: FULLY COMPLIANT
  • Rule 5.1: Connection string from Azure Key Vault
  • Rule 5.2: Connection pooling configured
  
✅ Resource Management: FULLY COMPLIANT
  • Rule 6.1: Requests and limits defined
  • Rule 6.2: Production tier resource allocation
  • Rule 6.3: HPA configured (3-10 replicas)

✅ Secrets Management: FULLY COMPLIANT
  • Rule 7.1: External Secrets Operator used
  • All secrets from Azure Key Vault

✅ Deployment Strategy: FULLY COMPLIANT
  • Rule 8.2: Rolling update with health checks
  • Rule 8.3: Deployed to production namespace

✅ Observability: FULLY COMPLIANT
  • Rule 9.1: JSON structured logging
  • Rule 9.2: Dapr provides distributed tracing
  • Rule 9.3: Prometheus metrics at /metrics

📊 Deployment Impact:
  • Resources to create: 1 Deployment, 1 Service, 1 ServiceAccount, 
    1 NetworkPolicy, 1 ExternalSecret, 1 Dapr Component
  • Initial replicas: 3 (will autoscale to 10 if needed)
  • Estimated cost: $150/month (based on current cluster costs)
  • Deployment time: ~3 minutes

🚀 Deployment Commands:

1. Ensure External Secrets Operator is installed:
   kubectl get crd externalsecrets.external-secrets.io

2. Apply external secrets first:
   kubectl apply -f external-secret.yaml

3. Apply Dapr components:
   kubectl apply -f dapr-statestore.yaml

4. Deploy the Helm chart:
   helm upgrade --install auth-service ./auth-service-chart \
     -f values.yaml \
     --namespace production \
     --create-namespace

5. Verify deployment:
   kubectl get pods -n production -l app=auth-service
   kubectl logs -n production -l app=auth-service -c auth-service
   kubectl logs -n production -l app=auth-service -c daprd

6. Test health endpoints:
   kubectl port-forward -n production svc/auth-service 8080:80
   curl http://localhost:8080/health/ready
   curl http://localhost:8080/health/live

🔄 Rollback Plan:
If issues arise:
  helm rollback auth-service -n production
  (Previous version will be restored in ~2 minutes)

⚠️  Important Notes:
1. Image mirroring: You'll need to push your image to registry.company.io first
   docker tag someuser/auth-service:latest registry.company.io/auth-service:v2.1.0-sha256-a1b2c3d4
   docker push registry.company.io/auth-service:v2.1.0-sha256-a1b2c3d4

2. Azure Key Vault: Ensure these secrets exist in Key Vault:
   - prod-auth-jwt-secret
   - prod-auth-db-connection

3. Network Policy: This restricts traffic. Ensure your ingress controller 
   has the correct namespace label.

Ready to proceed with deployment?
```

### Scenario 2: "The Kafka Event Stream"

**User Request:**
```
I need to set up a Kafka event stream for user profile updates.
Create the topic and configure Dapr pub/sub.
```

#### With Super-Spec:

```
🤖 Claude Code: Analyzing Kafka configuration requirements...

📋 KAFKA EVENT GOVERNANCE CHECK

I'll help you set up a constitution-compliant Kafka event stream.

First, let me ensure we follow our event governance rules:

Rule 4.1 - Topic Naming:
Your topic should be: "users.profile.updated"
  ✓ Domain: users
  ✓ Entity: profile  
  ✓ Event type: updated

Rule 4.2 - Schema Registry:
We need to define an Avro schema for this event.

Rule 4.3 - Authentication:
Kafka connection must use SASL_SSL.

Rule 4.4 - ACLs:
We need to define which services can produce/consume.

Let me generate the complete, compliant configuration:
```

**Generated Configuration:**

```yaml
# kafka-topic-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kafka-topic-users-profile-updated
  namespace: production
data:
  topic-name: "users.profile.updated"  # Rule 4.1
  partitions: "3"
  replication-factor: "3"
  retention-ms: "604800000"  # 7 days
  cleanup-policy: "delete"

---
# avro-schema.json - Rule 4.2
{
  "type": "record",
  "name": "UserProfileUpdated",
  "namespace": "com.company.users.events",
  "doc": "Event emitted when a user profile is updated",
  "fields": [
    {
      "name": "eventId",
      "type": "string",
      "doc": "Unique event identifier"
    },
    {
      "name": "userId",
      "type": "string",
      "doc": "User identifier"
    },
    {
      "name": "profileFields",
      "type": {
        "type": "map",
        "values": "string"
      },
      "doc": "Map of updated fields and their new values"
    },
    {
      "name": "updatedBy",
      "type": "string",
      "doc": "User or system that made the update"
    },
    {
      "name": "timestamp",
      "type": "long",
      "logicalType": "timestamp-millis",
      "doc": "Event timestamp in milliseconds since epoch"
    },
    {
      "name": "version",
      "type": "int",
      "default": 1,
      "doc": "Schema version"
    }
  ]
}

---
# dapr-pubsub-kafka.yaml - Rules 3.2, 4.3
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub-users
  namespace: production
spec:
  type: pubsub.kafka
  version: v1
  metadata:
  # Rule 4.3 - SASL_SSL authentication
  - name: brokers
    value: "kafka-brokers.kafka.svc.cluster.local:9093"
  - name: authType
    value: "certificate"
  - name: caCert
    secretKeyRef:
      name: kafka-certs
      key: ca.crt
  - name: clientCert
    secretKeyRef:
      name: kafka-certs
      key: client.crt
  - name: clientKey
    secretKeyRef:
      name: kafka-certs
      key: client.key
  - name: consumerGroup
    value: "production-user-service-consumer"
  - name: clientId
    value: "production-user-service"
auth:
  secretStore: kubernetes
scopes:
  - user-service  # Rule 4.4 - Only user-service can produce

---
# kafka-acls.yaml - Rule 4.4
# ACL configuration (applied via Kafka Admin API)
acls:
  - principal: "User:user-service"
    operations: ["Write", "Create", "Describe"]
    resourceType: "Topic"
    resourceName: "users.profile.updated"
    
  - principal: "User:notification-service"
    operations: ["Read", "Describe"]
    resourceType: "Topic"
    resourceName: "users.profile.updated"
    
  - principal: "User:analytics-service"
    operations: ["Read", "Describe"]
    resourceType: "Topic"
    resourceName: "users.profile.updated"
```

**Compliance Report:**

```
🛡️ KAFKA GOVERNANCE COMPLIANCE

✅ Topic Naming: COMPLIANT (Rule 4.1)
  • Format: users.profile.updated ✓
  • Domain.Entity.EventType pattern followed

✅ Schema Registry: COMPLIANT (Rule 4.2)
  • Avro schema defined
  • Version number included
  • All fields documented
  • Compatibility: BACKWARD (safe evolution)

✅ Authentication: COMPLIANT (Rule 4.3)
  • Protocol: SASL_SSL ✓
  • Certificates from Kubernetes secrets ✓
  • TLS port 9093 ✓

✅ Access Control: COMPLIANT (Rule 4.4)
  • Producer: user-service only
  • Consumers: notification-service, analytics-service
  • Principle of least privilege applied

📊 Deployment Steps:

1. Register schema in Schema Registry:
   curl -X POST http://schema-registry:8081/subjects/users.profile.updated-value/versions \
     -H "Content-Type: application/json" \
     -d @avro-schema.json

2. Create Kafka topic:
   kubectl apply -f kafka-topic-config.yaml

3. Apply ACLs:
   # (Your Kafka operator will handle this based on kafka-acls.yaml)
   kubectl apply -f kafka-acls.yaml

4. Deploy Dapr component:
   kubectl apply -f dapr-pubsub-kafka.yaml

5. Verify:
   kubectl get component pubsub-users -n production

Ready to proceed?
```

---

## 6. Conclusion: Self-Governing Cloud-Native Infrastructure

Cloud-native architecture with managed services requires a different approach to governance than traditional infrastructure. The **Super-Spec architecture** transforms AI from a configuration generator into a guardian that enforces organizational policies across:

- **Helm charts:** Ensuring container security and resource management
- **Dapr components:** Enforcing mTLS and authentication
- **Kafka topics:** Governing event schemas and access control
- **Database connections:** Protecting credentials and ensuring pooling

By combining **GitHub Spec Kit** (for the Constitution) with **Claude Code** (for enforcement), and leveraging the declarative nature of cloud-native tools, we achieve:

✅ **Zero supply chain attacks** from unapproved registries
✅ **No credential leaks** through proper secrets management
✅ **Consistent security posture** across all microservices
✅ **Event-driven governance** with schema validation
✅ **Reproducible deployments** with immutable configurations

**The paradigm shift:**

```
Old: "AI, generate a Helm chart for me"
     (Hope it's secure, hope it follows standards)

New: "AI, deploy this service according to our Constitution"
     (Guaranteed compliant, guaranteed secure, guaranteed consistent)
```

This is cloud-native architecture that you can trust at scale.

---

## Appendix A: Quick Start Checklist for Managed Services

```
□ Install GitHub Spec Kit
□ Create cloud-native Constitution (.specify/memory/constitution.md)
□ Define container registry rules
□ Define Dapr security requirements
□ Define Kafka governance policies
□ Define database connection standards
□ Get stakeholder approval
□ Create .claude/skills/ directory
□ Create constitution-enforcer skill
□ Create helm-chart-generator skill  
□ Create dapr-validator skill
□ Initialize Claude Code in project
□ Verify skills are discovered
□ Test refusal protocols
□ Create first compliant Helm chart
□ Deploy to development namespace
□ Validate against Constitution
□ Deploy to production
□ Monitor and iterate
□ (Optional) Create CLAUDE.md for project context
```

---

**Version:** 1.0 (Cloud-Native Edition)
**Last Updated:** November 2025
**Target Stack:** Managed Kubernetes, Helm, Dapr, Managed Kafka, Managed Databases
**License:** MIT (for examples and templates)
