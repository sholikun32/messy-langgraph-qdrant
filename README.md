# Code Quality & Design Principles – Onboarding Exercise

## Overview

This repository contains a functional FastAPI application that integrates **LangGraph** and **Qdrant** for document ingestion and semantic search.

The system works end-to-end but is intentionally implemented in an unstructured way. Important engineering practices such as separation of concerns, explicit dependency management, and testability are deliberately missing.

The purpose of this exercise is **not to refactor the code immediately**, but to analyze the existing design, identify architectural issues, and propose a clear and incremental path toward a more maintainable and production-ready architecture.

---

## What Works Well

Despite its limitations, the current implementation demonstrates several positive aspects:

- The application runs successfully and exposes clear API endpoints.
- LangGraph is used effectively to model a retrieval → generation workflow.
- Qdrant is integrated for vector storage and similarity search.
- The system behavior is observable and easy to follow at runtime.
- The code favors simplicity and avoids premature abstraction.

These strengths provide a solid foundation for iterative improvement.

---

## Design Smells & Issues Observed

### 1. Global State and Implicit Dependencies

Several critical components are stored as global variables, including the Qdrant client, workflow instance, in-memory state, and counters.

**Impact:**
- Risk of race conditions in concurrent environments
- Hidden dependencies that are difficult to reason about
- Reduced testability and limited isolation
- Side effects that span multiple requests

---

### 2. Lack of Separation of Concerns

The `main.py` file contains multiple responsibilities at once:
- HTTP request handling
- Business logic
- Workflow orchestration
- Database access
- Embedding generation
- Configuration management

This makes the code harder to understand, modify, and test in isolation.

---

### 3. Tight Coupling to Infrastructure

Business logic and workflows depend directly on:
- The Qdrant SDK
- A concrete embedding implementation

As a result:
- Changing the vector database would require changes across the codebase
- Introducing a new embedding model would affect multiple layers

---

### 4. Limited Testability

- No dependency injection
- No clear boundaries between logic and side effects
- External services cannot be easily mocked
- Unit testing requires bootstrapping the full application

---

### 5. Hardcoded Configuration

Environment-specific values such as host, port, collection name, and embedding dimensions are hardcoded, making the application fragile across different environments (local, staging, production).

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

    QD[Qdrant Implementation]
    EMB[Embedding Model Implementation]

    API --> UC
    UC --> WF
    WF --> ES
    WF --> VR

    ES --> EMB
    VR --> QD
```

### Architecture Explanation

- **FastAPI Routers** are responsible only for HTTP concerns such as request parsing and response formatting.
- **Application Use Cases** represent business actions and orchestrate workflows without knowing infrastructure details.
- **LangGraph Workflow** models domain logic and coordinates steps like retrieval and answer generation.
- **Interfaces** define contracts for embedding generation and vector storage without binding to specific technologies.
- **Implementations** encapsulate external systems such as Qdrant and embedding models.

This design follows the Dependency Inversion Principle and improves clarity, extensibility, and testability.

## Refactoring Priorities

If refactoring were to begin, the highest-impact priorities would be:

1. Remove global state and introduce explicit dependency injection
2. Decouple business logic from FastAPI routing
3. Introduce interfaces for embedding and vector storage
4. Centralize configuration management
5. Isolate side effects to improve testability
