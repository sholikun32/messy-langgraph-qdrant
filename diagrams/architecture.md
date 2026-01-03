# System Architecture

This document describes the current state of the application architecture, the identified design issues, and the proposed target architecture for improving maintainability, testability, and extensibility.

---

## Current Architecture Overview

The current implementation combines multiple responsibilities into a single module:

- FastAPI routing and request handling
- Business logic and workflow orchestration
- Embedding generation
- Vector database access
- Global state management

Most components rely on **global variables** and **implicit dependencies**, which introduces tight coupling and makes the system harder to test and evolve safely.

---

## Key Architectural Issues

The main issues identified in the current architecture are:

- **Global state usage**  
  Shared global variables are used for clients, workflows, and counters, creating risks in concurrent environments.

- **Tight coupling**  
  Business logic depends directly on infrastructure details such as Qdrant and embedding implementations.

- **Low testability**  
  Components cannot be tested in isolation without bootstrapping the full application.

- **Hardcoded configuration**  
  Environment-specific values are embedded directly in code, limiting portability across environments.

---

## Proposed Target Architecture

The following diagram illustrates a cleaner separation of responsibilities and a more maintainable dependency direction.

```mermaid
graph TD
    API[FastAPI Routers]
    UC[Application Use Cases]
    WF[LangGraph Workflow]

    ES[Embedding Service Interface]
    VR[Vector Repository Interface]

    EMB[Embedding Model Implementation]
    QD[Qdrant Implementation]

    API --> UC
    UC --> WF
    WF --> ES
    WF --> VR

    ES --> EMB
    VR --> QD

