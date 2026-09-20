# RACCAROO Core

## Purpose

This document defines the responsibilities, behavior, boundaries, and internal structure of the **RACCAROO Core**.

The Core is the central runtime of RACCAROO Systems.

It coordinates requests, intelligence, capabilities, memory, permissions, execution, and system state while remaining independent from specific AI models, interfaces, integrations, and hardware platforms.

The Core is responsible for **orchestration and control flow**, not for implementing every capability itself.

---

## Core Definition

RACCAROO Core is the central execution and orchestration layer of RACCAROO.

Its primary responsibility is to transform an incoming request into a controlled sequence of operations and return a meaningful result.

Conceptually:

```text
Request
   ↓
Core
   ↓
Understand
   ↓
Load Context
   ↓
Plan / Decide
   ↓
Select Capability
   ↓
Check Permissions
   ↓
Execute
   ↓
Process Result
   ↓
Respond
```

The Core must be able to handle both simple requests and multi-step tasks.

---

# Core Responsibilities

The Core is responsible for:

- receiving normalized requests;
- managing request lifecycle;
- managing task execution;
- coordinating Intelligence;
- loading and updating relevant context;
- discovering available capabilities;
- invoking capabilities;
- enforcing permission checks;
- handling capability results;
- coordinating multi-step operations;
- managing execution state;
- handling errors and failures;
- producing normalized responses;
- emitting system events;
- maintaining observability of important operations.

The Core should provide the stable foundation around which the rest of RACCAROO is built.

---

# Core Boundaries

The Core must remain independent from:

- specific AI models;
- specific AI providers;
- Telegram;
- Web interfaces;
- Home Assistant;
- specific databases;
- specific hardware;
- individual external services;
- individual capability implementations.

For example, the Core should know that a capability called `search_web` exists and how to invoke it.

It should not contain the implementation of the web search itself.

Similarly, the Core may request intelligence from an AI provider, but it should not contain logic specific to Ollama, OpenAI, Gemini, or another model provider.

---

# Core Internal Components

The Core is logically composed of several internal components.

These components may initially exist within a single application, but their responsibilities should remain separated.

## Request Manager

The Request Manager accepts requests from interfaces and creates the internal representation used by the Core.

Responsibilities:

- create request identifiers;
- validate request structure;
- establish request metadata;
- associate requests with users and sessions;
- initiate request execution;
- track request state.

A request should have a stable identity throughout its lifecycle.

---

## Task Orchestrator

The Task Orchestrator manages the execution flow of a request.

It determines which stage of execution should happen next and coordinates interaction between other Core components.

Responsibilities:

- start execution;
- call Intelligence;
- obtain context;
- request capability discovery;
- invoke capabilities;
- process results;
- continue multi-step execution;
- finish or terminate the task.

The Orchestrator controls the flow but should not contain provider-specific or capability-specific implementation logic.

---

## Intelligence Coordinator

The Intelligence Coordinator provides a controlled interface between the Core and the Intelligence module.

Responsibilities:

- send relevant context to Intelligence;
- provide available capability information;
- request reasoning or planning;
- receive structured decisions;
- validate the structure of intelligence results;
- return the result to the Orchestrator.

The Intelligence module may suggest an action, but the Core remains responsible for executing that action safely.

---

## Context Manager

The Context Manager obtains the information required for the current task.

Context may include:

- current conversation;
- user identity;
- session information;
- relevant memory;
- current task state;
- previous capability results;
- relevant system state.

The Core should request only the context relevant to the current operation.

The Context Manager should hide the implementation of memory storage from the rest of the Core.

---

## Capability Registry Interface

The Core communicates with the Capability Registry to discover what RACCAROO can currently do.

The Registry provides metadata such as:

- capability identifier;
- name;
- description;
- input requirements;
- output structure;
- current status;
- version;
- required permissions.

The Core should not need to know how the Registry stores this information.

---

## Capability Executor

The Capability Executor invokes capabilities selected during task execution.

Responsibilities:

- validate capability requests;
- prepare execution context;
- pass approved inputs;
- execute the capability;
- collect results;
- report execution status;
- return structured output.

The Executor should provide a consistent execution interface even when capabilities use completely different technologies internally.

---

## Permission Gate

The Permission Gate determines whether a requested operation is allowed.

This is an explicit architectural boundary between **intelligence** and **authority**.

The Intelligence layer may recommend:

> Execute capability X with parameters Y.

The Permission Gate determines:

> Is RACCAROO actually allowed to do that?

Permission decisions must not depend solely on natural-language instructions or AI-generated output.

The Permission system may evaluate:

- user permissions;
- capability permissions;
- resource permissions;
- environment;
- requested operation;
- risk level;
- approval requirements.

Operations requiring additional approval must be paused until the required approval is received.

---

## Execution State Manager

The Core must maintain the state of active operations.

A task may move through states such as:

```text
created
  ↓
understanding
  ↓
planning
  ↓
waiting_for_permission
  ↓
executing
  ↓
processing_result
  ↓
completed
```

Possible failure states include:

```text
failed
cancelled
blocked
timed_out
```

The exact state model may evolve, but execution state must be explicit rather than being inferred from logs.

---

## Response Manager

The Response Manager converts the final result of a task into a normalized response for the interface layer.

The Core should return structured results rather than directly formatting responses for Telegram, Web, CLI, or other interfaces.

Interfaces are responsible for converting the normalized response into their own presentation format.

---

## Event System

The Core should emit structured events for important system operations.

Examples:

- request received;
- task started;
- task state changed;
- capability discovered;
- permission requested;
- capability execution started;
- capability execution completed;
- capability execution failed;
- task completed;
- task cancelled.

Events provide a foundation for:

- logging;
- debugging;
- monitoring;
- future automation;
- audit history;
- user-visible activity information.

---

# Request Lifecycle

A standard request should follow a predictable lifecycle.

## 1. Request Reception

An interface sends a normalized request to the Core.

Example:

```text
User → Telegram → Core
```

The interface should not decide how the request will be solved.

---

## 2. Request Validation

The Core validates the request and establishes its execution context.

The Core identifies information such as:

- request ID;
- user;
- session;
- interface;
- request content;
- request metadata.

Invalid requests should terminate before task execution begins.

---

## 3. Context Collection

The Core obtains relevant context.

This may include:

- recent conversation;
- persistent memory;
- current task state;
- previously produced results;
- available capabilities;
- relevant system information.

The Core should avoid sending irrelevant context to Intelligence.

---

## 4. Intelligence Processing

The Core sends the required information to the Intelligence layer.

Intelligence may determine that the request:

1. can be answered directly;
2. requires one capability;
3. requires multiple capabilities;
4. requires additional information;
5. requires a new capability.

The Intelligence layer should return a structured result that the Core can process.

---

## 5. Capability Discovery

When an action is required, the Core obtains the capabilities relevant to the task.

The Core should prefer existing capabilities whenever possible.

For example:

```text
User:
"Tell me whether the price of this product falls below €500."

Core:
→ Search available capabilities
→ Find price-monitoring capability
→ Execute it
```

---

## 6. Permission Evaluation

Before executing an operation, the Core passes the operation through the Permission Gate.

Depending on the operation, the result may be:

```text
allowed
```

or:

```text
requires_approval
```

or:

```text
denied
```

The Core must not bypass this stage because an AI model requested the action.

---

## 7. Capability Execution

The Capability Executor runs the approved operation.

Execution may be:

- immediate;
- asynchronous;
- multi-step;
- long-running.

The Core must track the execution state.

---

## 8. Result Processing

The capability returns a structured result.

The Core passes the relevant result back to Intelligence when interpretation or further reasoning is required.

This allows multi-step tasks such as:

```text
Search
   ↓
Analyze results
   ↓
Search again
   ↓
Compare
   ↓
Create result
```

The Core controls the lifecycle while Intelligence helps determine what should happen next.

---

## 9. Completion

When no additional actions are required, the Core marks the task as completed.

The final result is passed to the Response Manager.

---

## 10. Response

The Core returns a normalized response to the originating interface.

For example:

```text
Core
 ↓
Normalized Response
 ↓
Telegram
```

or:

```text
Core
 ↓
Normalized Response
 ↓
CLI
```

The Core itself should not contain Telegram-specific or Web-specific presentation logic.

---

# Simple Requests

Not every request requires capability execution.

For a request that can be answered directly:

```text
User
 ↓
Core
 ↓
Context
 ↓
Intelligence
 ↓
Response
```

The Core should avoid unnecessary tool execution.

---

# Multi-Step Tasks

The Core must support tasks consisting of multiple operations.

Example:

```text
User:
"Find three laptops under €1000, compare them, and save the result."

Core
 ↓
Intelligence
 ↓
Search capability
 ↓
Results
 ↓
Intelligence
 ↓
Comparison
 ↓
File capability
 ↓
Result
 ↓
Response
```

The Core must preserve task state between steps.

---

# Missing Capabilities

One of the most important responsibilities of the Core is handling requests for functionality that does not currently exist.

The Core should allow Intelligence to identify:

```text
required capability
        ↓
capability exists?
     ↙       ↘
   yes        no
    ↓          ↓
 execute   Capability Builder
```

When a required capability does not exist, the Core should not silently execute arbitrary code.

Instead, the request should enter the Capability Builder workflow.

Conceptually:

```text
Requirement
     ↓
Capability Analysis
     ↓
Design
     ↓
Development
     ↓
Sandbox
     ↓
Testing
     ↓
Validation
     ↓
Approval
     ↓
Deployment
     ↓
Capability Registry
```

The exact implementation belongs to the Capability Builder and security architecture, not to the Core itself.

The Core's responsibility is to coordinate this lifecycle.

---

# Errors and Failures

The Core must treat failures as explicit execution states.

Potential failures include:

- invalid request;
- unavailable capability;
- permission denied;
- capability execution failure;
- AI provider failure;
- timeout;
- external service failure;
- invalid capability result;
- system resource failure.

The Core should distinguish between:

- temporary failures;
- permanent failures;
- user-action-required failures;
- security-related failures.

The system should avoid silently continuing after an important failure.

---

# Cancellation

Long-running tasks should be cancellable.

A cancellation request should propagate through the execution lifecycle where possible.

Example:

```text
User
 ↓
Cancel task
 ↓
Core
 ↓
Task Orchestrator
 ↓
Capability Executor
```

Capabilities should define whether and how their operations can be cancelled.

---

# Timeouts

Tasks and capability executions should have explicit timeout behavior.

The Core should not assume that external services or capabilities will always respond.

Timeouts should result in an explicit task state.

---

# Asynchronous Execution

The Core should support operations that continue after the original interface request has returned.

Examples:

- price monitoring;
- long-running research;
- scheduled tasks;
- background processing;
- file indexing;
- capability development.

This means the Core must conceptually separate:

**request lifecycle** from **task execution lifecycle**.

A user request may create a task that continues independently.

---

# Sessions

The Core should support sessions to group related interactions.

A session may contain:

- conversation context;
- active tasks;
- user information;
- temporary state;
- relevant memory.

The exact persistence mechanism is an implementation detail.

---

# Users

The Core should treat the user as an explicit security and context boundary.

A request should be associated with an authenticated identity or local user context.

This becomes important when RACCAROO eventually supports:

- multiple users;
- different permissions;
- shared environments;
- household members;
- external integrations.

---

# Core and Intelligence Separation

The following distinction is fundamental:

### Intelligence answers:

> "What should we do?"

### Core answers:

> "How does this operation move through the system?"

### Permission System answers:

> "Are we allowed to do it?"

### Capability answers:

> "How is this operation actually performed?"

This separation must remain intact.

---

# Core and Capabilities

The Core should never need to know the implementation details of a capability.

For example, the Core may see:

```text
Capability:
search_web
```

with structured input:

```text
{
  "query": "...",
  "limit": 5
}
```

The Core does not need to know whether the capability internally uses:

- an HTTP API;
- a browser;
- a search engine;
- another service;
- a local implementation.

The capability owns its implementation.

---

# Core and Memory

Memory is not the same as context.

Memory contains information that may persist beyond one task or session.

Context is the information assembled for a particular execution.

The Core should coordinate both:

```text
Memory
   ↓
Context Manager
   ↓
Task Context
   ↓
Intelligence
```

The Core should not require the Intelligence provider to own persistent user memory.

---

# Core and Security

Security must be an independent layer.

The Core may coordinate security decisions, but the AI model must never be treated as the security authority.

For example:

```text
AI:
"Delete this directory."

Core:
→ Permission Gate
→ Is deletion allowed?
→ Is approval required?
→ Execute or reject
```

This principle becomes especially important when RACCAROO eventually gains the ability to create and modify its own capabilities.

---

# Core and Self-Extension

RACCAROO is designed to become increasingly capable over time.

The Core therefore needs a stable mechanism for handling capabilities that do not yet exist.

However, self-extension must not mean unrestricted self-modification.

The initial architecture should maintain a distinction between:

```text
Production System
```

and:

```text
Development / Sandbox Environment
```

The Core coordinates the transition between them through controlled workflows.

---

# Observability

Core execution should be observable.

The system should be able to answer questions such as:

- What request was received?
- What task was created?
- What did Intelligence request?
- Which capabilities were considered?
- Which capability was executed?
- What permissions were evaluated?
- What happened during execution?
- Why did the task succeed or fail?

Observability is required both for development and for future self-extension.

---

# Core Invariants

The following principles should remain true regardless of implementation technology.

### AI is not the authority

AI may recommend actions, but the system enforces permissions.

### Interfaces do not own business logic

Interfaces communicate with the Core.

### Capabilities own their implementations

The Core orchestrates capabilities but does not implement them.

### Integrations remain replaceable

The Core should not depend on specific external platforms.

### Production and development remain separated

Self-extension must use controlled environments.

### Execution state is explicit

Important operations must have observable states.

### The Core remains stable

Capabilities and integrations should evolve without requiring constant changes to the Core.

---

# Initial Core Scope

The first implementation should intentionally remain small.

The minimum useful Core should contain:

1. Request Manager
2. Task Orchestrator
3. Intelligence interface
4. Model Provider interface
5. Context Manager
6. Capability Registry
7. Capability Executor
8. Basic Permission Gate
9. Execution State
10. Response Manager
11. Basic event logging
12. CLI interface

The following should initially remain outside the first implementation:

- Telegram;
- Web Dashboard;
- Voice;
- Home Assistant;
- complex automation;
- advanced persistent memory;
- multi-user infrastructure;
- autonomous capability development.

These can be added after the Core's basic execution loop is stable.

---

# First Working Execution Loop

The first successful version of RACCAROO should demonstrate this complete loop:

```text
CLI
 ↓
Request
 ↓
RACCAROO Core
 ↓
Context
 ↓
Intelligence
 ↓
Capability Selection
 ↓
Permission Check
 ↓
Capability Execution
 ↓
Result
 ↓
Intelligence
 ↓
Response
 ↓
CLI
```

The first implementation is successful when RACCAROO can execute this loop reliably using a small number of real capabilities.

At that point, the system has moved from an architectural concept to a functioning RACCAROO runtime.
