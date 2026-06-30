---
title: "A Gateway for Network Knowledge Graph Management"
abbrev: "NKG Gateway"
docname: draft-nmop-cui-nkg-gateway-latest
category: info
submissiontype: IETF
ipr: trust200902
date: 2026-07-01
v: 3
# area: "OPS"
# workgroup: "NMOP"
author:
  - ins: "Y. Cui"
    name: "Yong Cui"
    org: "Tsinghua University"
    city: "Beijing"
    code: "100084"
    country: "China"
    email: "cuiyong@tsinghua.edu.cn"
    uri: "http://www.cuiyong.net/"
  - ins: "J. Niu"
    name: "Jialin Niu"
    org: "Beijing University of Posts and Telecommunications"
    city: "Beijing"
    code: "100876"
    country: "China"
    email: "njl@bupt.edu.cn"
  - ins: "M. Xing"
    name: "Mingzhe Xing"
    org: "Zhongguancun Laboratory"
    city: "Beijing"
    code: "100094"
    country: "China"
    email: "xingmz@zgclab.edu.cn"
  - ins: "L. Zhang"
    name: "Lei Zhang"
    org: "Zhongguancun Laboratory"
    city: "Beijing"
    code: "100094"
    country: "China"
    email: "zhanglei@zgclab.edu.cn"
informative:
  I-D.mackey-nmop-kg-for-netops:
  I-D.marcas-nmop-knowledge-graph-yang:
  I-D.pang-nmop-kg-for-traffic-monitoring-analysis:
  I-D.tailhardat-nmop-incident-management-noria:

--- abstract

This document specifies an interaction gateway for Network Knowledge Graphs (NKGs) to simplify graph-based network operations. The gateway architecture defines a Unified Intent Gateway (UIG) that supports Natural Language (NL) requests from human operators and Domain-Specific Language (DSL) or API directives from applications and orchestration systems. The UIG interprets user expectations and application directives as structured intents, aligns them with the NKG structure and exposed capabilities, and maps them to graph queries, graph inferences, API calls, or operational workflows through an Intermediate Representation (IR). The gateway architecture incorporates intent discovery, capability mapping, policy enforcement, audit logging, and response synthesis to make NKG-based operations more accessible, secure, and controllable.

--- middle

# Introduction

## Motivation

Network Knowledge Graphs (NKGs) provide a unified, associative, and semantic representation of network states and structures, supporting observability, automated operations, and intelligent management. Existing NKG systems can expose structured query, graph traversal, reasoning, validation, and data integration capabilities. In current operational scenarios, however, these capabilities are usually accessed through graph schemas, query languages (e.g., Cypher or SPARQL), or tool-specific APIs, which introduces the following challenges:

- **High Usage Barrier:** Operators and applications often need to understand the underlying graph schema, entity types, relationship definitions, and query language syntax before they can effectively use NKG capabilities.
- **Lack of Intent Abstraction:** Operators usually express desired operational outcomes in natural language, such as investigating a service degradation, identifying affected resources, or finding mitigation candidates. Such user expectations need to be interpreted, refined, and aligned with the NKG structure before they can become executable graph-based operations.

Therefore, a unified and secure intent-aware gateway tailored for NKG operations is required. The gateway should reduce the usage barrier for operators, abstract user expectations and application directives into structured intents, and map these intents to supported NKG capabilities in a controllable and auditable manner.

## Problem Statement

The central challenge is the absence of an intelligent gateway capable of translating high-level user expectations or structured application directives into validated NKG operations, while preserving security, policy control, and operational traceability. Such a gateway needs to bridge the gap between human- or application-level requests and the graph-level capabilities exposed by NKG systems.

An intent-aware gateway for NKG management needs to satisfy the following requirements:

- **Unified Heterogeneous Input:** The gateway should consolidate natural-language requests from human operators and structured directives from applications, controllers, or orchestration systems.
- **Intent Discovery and Interpretation:** The gateway should expose supported intent types according to the NKG structure and capability metadata, and should infer candidate intents from incomplete or ambiguous user expectations.
- **Intent Representation:** The gateway should normalize interpreted intents into an IR that captures the intent type, target scope, constraints, context, required capabilities, authorization context, and output preference.
- **Schema-aware Capability Mapping:** The gateway should map intents to graph queries, graph inferences, API calls, or workflows according to the current NKG schema, relationship definitions, and exposed capabilities.
- **Multi-step Execution Planning:** The gateway should support intents that require multiple graph queries, reasoning steps, validation checks, or external API calls.
- **Intent and Operation Validation:** The gateway should validate the IR and generated operations before execution, including schema constraints, policy constraints, authorization scope, and semantic consistency.
- **Security and Compliance:** The gateway should ensure rigorous authentication, authorization, policy enforcement, output filtering, and audit logging. The underlying NKG and graph engine should also enforce access control for defense in depth.
- **Response Synthesis:** The gateway should synthesize execution results into machine-consumable structured outputs and, when requested, operator-facing natural-language summaries or explanations.

# Terminology

- **NKG (Network Knowledge Graph):** A graph-based representation of network entities, their attributes, operational states, and semantic relationships.
- **User Expectation:** An operator-level expression of a desired operational outcome. It is typically conveyed via Natural Language (NL) and may be incomplete, ambiguous, or independent of the underlying NKG schema.
- **Application Directive:** A structured request issued by an application, controller, orchestration system, or automation tool. It may be encoded using a Domain-Specific Language (DSL), an API request, or another machine-readable format.
- **Intent:** A structured representation of a user expectation or application directive after interpretation, clarification, and alignment with the NKG structure and available capabilities.
- **Intent Catalog:** A registry of supported intent types and related metadata exposed by the UIG. It describes the capabilities exposed through the UIG, including supported operational intents, required parameters, mapped NKG capabilities, and supported output formats.
- **IR (Intermediate Representation):** A normalized and instantiated representation of a specific intent before execution. While the Intent Catalog describes what the UIG can support, the IR captures what the current request intends to do, including the intent type, targets, constraints, context, required capabilities, authorization context, and execution preferences.
- **NKG Capability:** A query, inference, validation, update, or external operation that can be invoked over or through the NKG and its associated graph engine or management APIs.
- **Agent:** An optional LLM-based or rule-assisted component within the UIG that helps parse user expectations, infer intents, complete missing slots, or generate operator-facing explanations.

# Gateway Architecture Overview

This gateway architecture adopts a three-layer architecture, with the Unified Intent Gateway (UIG) serving as the central mediator between upstream requests and downstream NKG capabilities. In this architecture, the gateway is an intent-aware interaction component that connects operators and applications with graph-based knowledge operations.

~~~~
+-----------------------------------------------------------+
|         Upstream: Input & Interaction Layer               |
|         (Users [NL], Applications [DSL/API])              |
+----------------------------^------------------------------+
                             |
          (1) User Expectation or Directive
                             |
                             | (5) Final Response
                             |     (Structured Result / Summary)
+----------------------------v------------------------------+
|             Midstream: Unified Intent Gateway             |
| [ Authorization | Intent Catalog | Resolution (IR) |      |
|   Query/Plan | Policy | Synthesis ]                       |
+----------------------------^------------------------------+
                             |
          (2) Capability Request / Query / Workflow
                             |
                             | (4) Execution Result
+----------------------------v------------------------------+
|             Downstream: Data & Knowledge Layer            |
| [ NKG Schema | Graph Engine | Reasoning | APIs | Policies]|
+-----------------------------------------------------------+
~~~~

- **Input and Interaction Layer (User/Application)**
- **Unified Intent Gateway (UIG)**
- **Data and Knowledge Layer (NKG, Graph Engine, and Related Capabilities)**

**Overall processing flow:**

Upstream input (NL/DSL) -> UIG (authentication/authorization, intent catalog lookup, intent resolution, Intermediate Representation (IR) construction and validation, capability mapping, query/execution planning) -> NKG execution -> UIG response synthesis -> upstream response delivery.

## Input and Interaction Layer

The upstream layer accepts inputs from human operators and automated systems:

- **User input (NL):** e.g., "Check whether service_X is affected by an HTTP Flood."
- **Application input (DSL/API):** e.g., `INVESTIGATE_INCIDENT(target="service_X", window="5m")`.
- **Contextual metadata:** e.g., requester identity, tenant, target domain, time window, ticket identifier, or operational scope.

Primary responsibilities of this layer include:

- Providing unified interfaces (e.g., REST, gRPC, WebSocket, command-line interfaces, or conversational interfaces);
- Transmitting requests with identity information, authorization context, and operational metadata;
- Receiving structured results (e.g., JSON), graph paths or subgraphs, audit references, and optional natural-language explanations.

## Unified Intent Gateway

The UIG receives requests, performs authentication/authorization, consults the Intent Catalog to identify supported intent types, resolves the request into an intent, normalizes the resolved intent into an IR, and maps the IR to graph queries, graph inferences, API calls, or execution plans. After receiving results from the NKG or related management systems, the UIG aggregates the returned data and synthesizes responses for upstream consumers.

~~~~
+-----------------------------------------------------------------------+
|                       Unified Intent Gateway (UIG)                    |
|                                                                       |
|  +---------------+     +-------------------+      +----------------+  |
|  |   Ingress     |     | Intent Catalog &  |      | IR Generation  |  |
|  |      &        |---->| Intent Resolution |----->| & Validation   |  |
|  | Authorization |     +---------+---------+      +-------+--------+  |
|  +---------------+               |                        |           |
|                                  |                        v           |
|                         +--------v---------+      +----------------+  |
|                         | Capability       |----->| Query/Plan     |  |
|                         | Mapping          |      | Executor       |  |
|                         +--------+---------+      +-------+--------+  |
|                                  |                        |           |
|                                  v                        v           |
|                         +----------------+      +------------------+  |
|                         | Policy         |<---->| Response         |  |
|                         | Enforcement    |      | Synthesis & Log  |  |
|                         +----------------+      +------------------+  |
+-----------------------------------------------------------------------+
~~~~

**Defense-in-depth requirement:** the underlying NKG/graph engine itself MUST provide built-in security capabilities. It MUST authenticate the UIG's Query Executor and authorize operations using fine-grained policies for node, edge, subgraph, capability, and tenant access, so that the architecture does not rely on the UIG as a single point of failure.

The UIG's main functions include:

1. **Input parsing and intent recognition**
   - For NL: an Agent MAY perform intent extraction, semantic slot completion, clarification, and task generation. This process can be guided by domain-specific ontology, intent catalog entries, and NKG capability metadata;
   - For DSL/API directives: a rule-based parser or structured validator parses machine-readable requests;
   - Both forms are normalized into a unified IR.

2. **Intent discovery and catalog support**
   - Expose available intent types according to NKG schema, capability metadata, requester role, tenant, and operational scope;
   - Provide required parameters, optional parameters, and supported output formats for each intent type;
   - Support user interaction when an operator needs to discover what NKG-based operations are available.

3. **Validation and policy enforcement**
   - Validate syntactic correctness, required fields, and executability of the IR;
   - Authenticate requesters and authorize operations (e.g., by role/tenant/scope);
   - Check consistency with the NKG schema, capability metadata, and policy constraints;
   - Reject ambiguous, unsupported, or unauthorized requests before execution.

4. **Statement generation and execution orchestration**
   - Translate IR into standardized Cypher, SPARQL, or equivalent graph query statements when graph queries are required;
   - Construct execution plans for graph inference, external API calls, or multi-step workflows;
   - Record audit logs covering the request, authorization decisions, generated operations, execution plan, and result summaries.

5. **Response synthesis**
   - Normalize query and execution results into structured outputs (e.g., JSON);
   - Return graph paths, affected entities, evidence chains, confidence values, or audit references when applicable;
   - Optionally generate natural-language summaries or explanations for operators.

## Data and Knowledge Layer

This layer consists of the NKG, graph database/engine, reasoning functions, policy metadata, and optional external management APIs. It is responsible for executing graph queries, graph inferences, graph updates, and related operations, and returning execution results to the UIG.

It supports:

- Efficient retrieval and updates over topology, device status, telemetry-derived facts, service dependencies, and related knowledge;
- Graph traversal, causal path extraction, and reasoning over network entities and relationships;
- Constraint checking and semantic validation based on schema or policy metadata;
- Structured result formats (e.g., Node-Relation-Attribute, graph paths, or JSON objects);
- Invocation of external management APIs or workflow systems associated with NKG entities, when explicitly authorized.

**Security requirement:** the NKG MUST authenticate querying entities and authorize access to nodes/edges/subgraphs, capabilities, and tenants using fine-grained access control. This provides a foundational security safeguard for the overall gateway architecture.

# Use Cases

This section provides illustrative use cases demonstrating how the gateway architecture supports automated and controlled NKG interactions.

## Use Case 1: HTTP Flood Investigation (NL)

**Scenario:** A service reports abnormal response latency, and an operator suspects an HTTP Flood.

**User expectation (NL):** "Check whether there is an HTTP Flood attack affecting service_X."

**Expected UIG behavior:**

- Authenticate and authorize the requester;
- Infer an `incident_investigation` intent with `suspected_attack` set to `HTTP_Flood`;
- Clarify or assign a time window if the request does not provide one;
- Resolve the intent into IR, including target service, suspected attack, time window, required capabilities, and output preference;
- Map the IR to NKG capabilities for traffic fact retrieval, request-rate anomaly checking, and affected service lookup;
- Generate graph queries or execution plans over traffic/telemetry facts;
- Synthesize results into structured output and, when requested, an operator-facing explanation.

**Example IR:**

~~~~
{
  "intent": "incident_investigation",
  "target": "service_X",
  "suspected_attack": "HTTP_Flood",
  "time_window": "5m",
  "required_capabilities": [
    "traffic_fact_retrieval",
    "request_rate_anomaly_check",
    "affected_service_lookup"
  ],
  "output_preference": [
    "structured_json",
    "operator_summary"
  ]
}
~~~~

**Illustrative query (Cypher):**

~~~~
MATCH (src:IP)-[r:SENDS_HTTP]->(dst:Server)
WHERE r.request_rate > $threshold AND dst.name = "service_X" AND r.window = "5m"
RETURN src, r.request_rate, dst
ORDER BY r.request_rate DESC
~~~~

**Output:**

- Structured JSON containing suspected sources, request rates, affected services, and evidence paths;
- Optional explanation: "High-rate HTTP requests from suspicious source nodes to service_X are observed within the last five minutes; the evidence path is consistent with an HTTP Flood affecting the target service."

## Use Case 2: Admin-Controlled NKG Node Maintenance (DSL)

**Scenario:** A network administrator or authorized inventory system dynamically maintains the NKG by creating, updating, or deleting graph nodes (e.g., device nodes, service nodes, or asset entries). All write operations MUST be strictly controlled by role-based authorization and fully audited.

**Input (DSL examples):**

- `UPSERT_NODE(type="Device", key="name", value="FW1", props={interface:"eth0", status:"active"})`
- `DELETE_NODE(type="Device", key="name", value="FW1")`

**Expected UIG behavior:**

- Enforce elevated privileges for write operations;
- Parse the DSL directive into IR (e.g., `{op: upsert_node, type: Device, match_key: name, match_value: FW1, props: {...}}`);
- Validate schema constraints, node labels, required properties, property types, tenant boundaries, and write scope;
- Map the IR to an authorized graph update capability;
- Generate graph write statements or invoke an authorized update API;
- Record an audit trail covering the request, authorization decision, generated operation, affected node identifiers, and result summary.

**Example operations (Cypher):**

~~~~
MERGE (n:Device {name:"FW1"})
SET n.interface="eth0",
    n.status="active";
~~~~

~~~~
MATCH (n:Device {name:"FW1"})
DELETE n;
~~~~

**Output:**

- Structured JSON acknowledgment containing operation status (success/failure), affected node identifiers, and an audit reference.
- Example: `{"status": "success", "operation": "upsert_node", "affected_nodes": ["FW1"], "audit_reference": {"trace_id": "req-9a8b7c6d"}}`

## Use Case 3: Intent Discovery for Network Troubleshooting (NL)

**Scenario:** An operator observes that a service has degraded latency, but does not know which troubleshooting intents are supported by the NKG.

**User expectation (NL):** "The latency of service_X increased suddenly. What can I check?"

**Expected UIG behavior:**

* Authenticate the requester and collect available context metadata;
* Consult the Intent Catalog and NKG capability metadata;
* Identify troubleshooting intents that are applicable to the target service and requester role;
* Suggest candidate investigation intents based on the user's natural-language request;
* Indicate which parameters are required before a selected intent can be executed;
* Return available investigation options in structured form and provide an operator-facing summary.

**Example response:**

~~~~
{
  "target": "service_X",
  "available_intents": [
    "traffic_anomaly_investigation",
    "affected_dependency_discovery",
    "path_congestion_analysis",
    "related_incident_correlation"
  ],
  "parameters_required_for_execution": [
    "selected_intent",
    "time_window"
  ],
  "available_output_formats": [
    "structured_json",
    "operator_summary"
  ]
}
~~~~

**Output:**

- Structured result containing supported troubleshooting intents, required parameters for execution, and available output formats;
- Operator-facing summary: "The NKG supports several troubleshooting intents for service_X, including traffic anomaly investigation, affected dependency discovery, and path congestion analysis. A time window is required before execution. Please provide a time window or select an investigation intent."

# Security Considerations

This document describes an interaction gateway for a NKG mediated by a UIG. Implementations MUST consider the following:

- The graph engine/NKG service MUST enforce authentication and fine-grained authorization for all operations initiated by the UIG. This defense-in-depth requirement prevents the UIG from becoming a single point of failure.
- NL, DSL, and API inputs MUST be validated and constrained to prevent malformed, ambiguous, or unauthorized graph operations.
- Intent inference mechanisms, including LLM-based components when used, MUST be constrained by policy, schema, and capability metadata before execution.
- IR validation MUST be performed before any graph query, graph update, API call, or workflow execution.
- Write operations (create/update/delete) MUST be restricted to explicitly authorized roles and scopes, and SHOULD be fully audited.
- Responses MUST be filtered/redacted according to the requester's authorization scope to avoid data leakage.
- Audit logs SHOULD record the original request, inferred intent, IR, authorization decision, selected capabilities, execution plan, and result summary.

# IANA Considerations

This document has no IANA actions.

--- back
