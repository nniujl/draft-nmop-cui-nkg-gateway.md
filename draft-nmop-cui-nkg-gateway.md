---
title: "A Gateway for Network Knowledge Graph Management"
abbrev: "NKG Gateway"
docname: draft-nmop-cui-nkg-gateway-latest
category: info
submissiontype: IETF
ipr: trust200902
date: 2026-02-27
v: 3
area: "OPS"
workgroup: "NMOP"
author:
  - ins: "Y. Cui"
    name: "Yong Cui"
    org: "Tsinghua University"
    email: "cuiyong@tsinghua.edu.cn"
informative:
  I-D.mackey-nmop-kg-for-netops:
  I-D.marcas-nmop-knowledge-graph-yang:
  I-D.pang-nmop-kg-for-traffic-monitoring-analysis:
  I-D.tailhardat-nmop-incident-management-noria:

--- abstract

This document specifies an interaction gateway for Network Knowledge Graphs (NKG) to simplify graph-based operations. The gateway architecture defines a Unified Intent Gateway (UIG) that supports Natural Language (NL) and Domain-Specific Language (DSL) inputs. By utilizing LLM-based agents and rule engines, the UIG translates high-level intents into precise graph queries via an Intermediate Representation (IR). The gateway architecture incorporates multi-layer authentication and fine-grained access control to ensure secure and compliant network management.

--- middle

# Introduction

## Motivation

Network Knowledge Graphs (NKG) provide a unified, associative, and semantic representation of network states and structures, supporting observability, automated operations, and intelligent management. In current operational scenarios, engineers primarily rely on structured query languages (e.g., Cypher or SPARQL) for interaction, which introduces the following challenges:

- **High Usage Barrier:** Users must possess deep understanding of the underlying graph schema and relationship definitions.
- **Schema Sensitivity:** Evolution or changes in the graph schema often lead to query failures or biased results.
- **High Cost of Multi-step Queries:** Manual parameter tuning and logical debugging significantly reduce operational efficiency.
- **Lack of Intent Abstraction and Semantic Validation:** It is difficult to support high-level automation and security governance.

Therefore, a unified and secure interaction gateway tailored for network operations is urgently required.

## Problem Statement
The central challenge is the absence of an intelligent interface capable of automatically translating high-level human intents (NL) or declarative application directives (DSL) into deep queries, inferences, or operations against the NKG, while meeting the following requirements:

- **Unified Heterogeneous Input:** Consolidating inputs from both human operators and automated machines.
- **Security and Compliance:** Ensuring rigorous authentication, authorization, and operational compliance.
- **Intent-to-Query Translation:** Accurately transforming ambiguous intents into precise graph queries.
- **Result Synthesis:** Synthesizing structured graph data into comprehensible and consumable outputs (both structured and natural language).

# Terminology
- **NKG (Network Knowledge Graph):** A graph-based representation of network entities, their attributes, and their relationships.
- **Agent:** An LLM-based system that acts as a conversational interface for network operators.
- **NL (Natural Language):** Natural-language requests issued by network operators.
- **DSL (Domain-Specific Language):** A structured directive language used by applications or orchestration systems.

# Gateway Architecture Overview
This gateway architecture adopts a three-layer architecture, with the Unified Intent Gateway (UIG) serving as the central mediator between upstream requests and downstream knowledge execution:
```text
+-----------------------------------------------------------+
|         Upstream: Input & Interaction Layer               |
|         (Users [NL], Applications [DSL])                  |
+----------------------------^------------------------------+
                             |
          (1) Intent Request | (4) Final Response
                             |
+----------------------------v------------------------------+
|             Midstream: Unified Intent Gateway             |
| [ Authorization | Resolution (IR) | Query  | Synthesis ]  |
+----------------------------^------------------------------+
                             |
          (2) Cypher Query   | (3) Structured Result
                             |
+----------------------------v------------------------------+
|             Downstream: Data & Knowledge Layer            |
|  [ Authorization Check | Graph Engine | NKG Storage ]     |
+-----------------------------------------------------------+
```

- **Input and Interaction Layer (User/Application)**
- **Unified Intent Gateway (UIG)**
- **Data and Knowledge Layer (NKG and Graph Engine)**

**Overall processing flow:**

Upstream input (NL/DSL) -> UIG (authentication/authorization, parsing, Intermediate Representation (IR) construction, query/execution planning) -> NKG execution -> UIG response synthesis -> upstream response delivery.

## Input and Interaction Layer
The upstream layer accepts two categories of inputs:

- **User input (NL):** e.g., "List the nodes with abnormal traffic in the last five minutes."
- **Application input (DSL):** e.g., `SELECT_TRAFFIC(srcIP=10.0.0.1, window=5m)`.

Primary responsibilities of this layer include:

- Providing unified interfaces (e.g., REST, gRPC, WebSocket);
- Transmitting requests with identity information and contextual metadata;
- Receiving structured results (e.g., JSON) and optional natural-language explanations.

## Unified Intent Gateway
The UIG receives requests, performs authentication/authorization and intent resolution, normalizes requests into an IR, and generates graph query/operation statements (e.g., Cypher). After receiving results from the NKG, the UIG aggregates and formats the structured data and returns it to upstream consumers.
```text
+-----------------------------------------------------------------------+
|                       Unified Intent Gateway (UIG)                    |
|                                                                       |
|  +---------------+     +-------------------+      +----------------+  |
|  |   Ingress     |     | Intent Resolution |      | Query Executor |  |
|  |      &        |---->| (Agent / Parser)  |----->|       &        |  |
|  | Authorization |     +---------+---------+      |  Synthesizer   |  |
|  +---------------+               |                +-------+--------+  |
|          |                (IR Generation)                 |           |
|          |                       |                        |           |
|          +-----------------------+------------------------+           |
|                                  |                                    |
|                       +----------v----------+                         |
|                       |      Audit Log      |                         |
|                       |  (Policy & Action)  |                         |
|                       +---------------------+                         |
+-----------------------------------------------------------------------+
```

**Defense-in-depth requirement:** the underlying NKG/graph engine itself MUST provide built-in security capabilities. It MUST authenticate the UIG’s Query Executor and authorize operations using fine-grained policies for node/subgraph access, so that the framework does not rely on the UIG as a single point of failure.

The UIG’s main functions include:

1. Input parsing and intent recognition
   - For NL: an LLM Agent performs intent extraction, semantic slot completion, and task generation. Crucially, this process is guided by a domain-specific ontology to map unstructured text to precise semantic anchors and ensure accurate intent resolution;
   - For DSL: a rule-based engine parses structured directives;
   - Both forms are normalized into a unified IR.

2. **Validation and policy enforcement**
   - Validate syntactic correctness and executability of commands;
   - Authenticate requesters and authorize operations (e.g., by role/tenant/scope);
   - Check consistency with the graph schema and reject illegal or unauthorized requests.

3. **Statement generation and execution orchestration**
   - Translate IR into standardized Cypher (or an equivalent graph query language);
   - Construct execution plans (read/write/multi-step);
   - Record audit logs (request, authorization decisions, generated statements, and result summaries).

4. **Response synthesis**
   - Normalize query results into structured outputs (e.g., JSON);
   - Optionally generate natural-language explanations for operators.

## Data and Knowledge Layer
This layer consists of the NKG and the graph database/engine, and is responsible for executing graph queries/operations and returning structured results. It supports:

- Efficient retrieval and updates over topology, device status, telemetry-derived facts, and related knowledge;
- Structured result formats (e.g., Node–Relation–Attribute).

**Security requirement:** the NKG MUST authenticate querying entities and authorize access to nodes/edges/subgraphs using fine-grained access control (e.g., restricting sensitive subgraphs based on role). This provides a foundational security safeguard for the overall gateway architecture.

# Use Cases
This section provides illustrative use cases demonstrating how the gateway architecture supports automated and controlled NKG interactions.

## Use Case 1: HTTP Flood Investigation (NL)
**Scenario:** A service reports abnormal response latency, and an operator suspects an HTTP Flood.

**Input (NL):** "Check whether there is an HTTP Flood attack affecting the target service."

**Expected UIG behavior:**
- Authenticate and authorize the requester.
- Resolve intent into IR (e.g., `{intent: http_flood_investigation, window: 5m, target: service_X}`).
- Translate IR into one or more graph queries over traffic/telemetry facts.
- Synthesize results into a structured response and an optional operator-facing summary.

**Example query (Cypher):**
```cypher
MATCH (src:IP)-[r:SENDS_HTTP]->(dst:Server)
WHERE r.request_rate > $threshold AND dst.port = 80 AND r.window = "5m"
RETURN src, r.request_rate, dst
ORDER BY r.request_rate DESC
```

**Output:**
- Structured JSON containing suspected sources, request rates, and affected servers.
- Optional explanation: “High-rate HTTP requests from 1.2.3.4 to server S are observed within the last 5 minutes; behavior is consistent with an HTTP Flood.”

## Use Case 2: Admin-Controlled NKG Node Maintenance (DSL)

**Scenario:** A network administrator dynamically maintains the NKG by creating, updating, or deleting graph nodes (e.g., device nodes, service nodes, or asset entries). All write operations MUST be strictly controlled by role-based authorization and fully audited.

**Input (DSL examples):**
- `UPSERT_NODE(type="Device", key="name", value="FW1", props={interface:"eth0", status:"active"})`
- `DELETE_NODE(type="Device", key="name", value="FW1")`

**Expected UIG behavior:**
- Enforce elevated privileges for write operations (administrator-only).
- Parse DSL into IR (e.g., `{op: upsert_node, type: Device, match_key: name, match_value: FW1, props: {...}}`).
- Validate schema constraints (node label, required properties, property types) and policy constraints (tenant boundary, write scope).
- Generate graph write statements and record an audit trail (request, auth decision, generated statements, and result summary).

**Example operations (Cypher):**
```cypher
MERGE (n:Device {name:"FW1"})
SET n.interface="eth0",
    n.status="active";
```

```cypher
MATCH (n:Device {name:"FW1"})
DELETE n;
```

**Output:**
- Structured JSON acknowledgment containing operation status (success/failure), affected node identifiers, and an audit reference.
- Example: `{"status": "success", "operation": "upsert_node", "affected_nodes": ["FW1"], "audit_reference": {"trace_id": "req-9a8b7c6d"}}`

# Security Considerations
This document describes an interaction gateway for a NKG mediated by a UIG. Implementations MUST consider the following:

- The graph engine/NKG service MUST enforce authentication and fine-grained authorization for all operations initiated by the UIG (defense-in-depth), so the UIG is not a single point of failure.
- NL/DSL inputs MUST be validated and constrained to prevent malformed or unauthorized graph operations.
- Write operations (create/update/delete) MUST be restricted to explicitly authorized roles and scopes, and SHOULD be fully audited.
- Responses MUST be filtered/redacted according to the requester’s authorization scope to avoid data leakage.

# IANA Considerations

This document has no IANA actions.

--- back
