---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "A Framework to Evaluate LLM Agents for Network Configuration"
abbrev: "Eval4LLM"
category: info

docname: draft-cui-nmrg-llm-benchmark
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: IRTF
workgroup: Network Management Research Group
keyword:
 - Large Language Model
 - Network Configuration
 - Benchmark
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
- role:  # remove if not true
  ins: Y. Cui
  name: Yong Cui
  org: Tsinghua University
  street:
  city:
  region: Beijing # not always available
  code: 100084
  country: China # use TLD (except UK) or country name
  phone: 
  email: cuiyong@tsinghua.edu.cn
  uri: http://www.cuiyong.net/
- role: # remove if not true
  ins: C. Liu
  name: Chang Liu
  org: Tsinghua University
  street:
  city: 
  region: Beijing # not always available
  code: 100084
  country: China # use TLD (except UK) or country name
  phone:
  email: liuchang@tsinghua.edu.cn
- role: # remove if not true
  ins: C. Du
  name: Chenguang Du
  org: Zhongguancun Laboratory
  street:
  city: 
  region: Beijing # not always available
  code: 100094
  country: China # use TLD (except UK) or country name
  phone: 
  email: ducg@zgclab.edu.cn

normative:
  RFC8341:
  RFC6241:
informative:
  TM-IG1230:
    title: Autonomous Networks Technical Architecture
    author:
    - name: Kevin McDonnell
    - name: Azahar Machwe
    - name: Dave Milham
    - name: James O’Sullivan
    - name: Jörg Niemöller
    - name: Luca Franco Varvello
    - name: Vinay Devadatta
    - name: Wang Lei
    - name: Wang Xu
    - name: Xie Yuan
    - name: Yuval Stein
    date: 2023-02
  Huang25:
    title: A Survey on Hallucination in Large Language Models Principles, Taxonomy, Challenges, and Open Questions
    author:
    - name: Lei Huang,
    - name: Weijiang Yu
    - name: Weitao Ma
    - name: Weihong Zhong
    - name: Zhangyin Feng
    - name: Haotian Wang
    - name: Qianglong Chen
    - name: Weihua Peng
    - name: Xiaocheng Feng
    - name: Bing Qin
    - name: Ting Liu
  Hu22:
    title: LoRA Low-Rank Adaptation of Large Language Models
    author:
    - name: Edward J Hu
    - name: Yelong Shen
    - name: Phillip Wallis
    - name: Zeyuan Allen-Zhu
    - name: Yuanzhi Li
    - name: Shean Wang
    - name: Lu Wang
    - name: Weizhu Chen
  Lewis20:
    title: Retrieval-augmented generation for knowledge-intensive NLP tasks
    author:
    - name: Patrick Lewis
    - name: Ethan Perez
    - name: Aleksandra Piktus
    - name: Fabio Petroni
    - name: Vladimir Karpukhin
    - name: Naman Goyal
    - name: Heinrich Küttler
    - name: Mike Lewis
    - name: Wen-tau Yih
    - name: Tim Rocktäschel
    - name: Sebastian Riede


...

--- abstract

This document specifies an evaluation framework and related definitions
for intent-driven network configuration using Large Language Model
(LLM)–based agents.  The framework defines representative scenarios
(basic configuration, fault remediation, policy optimization), intent
expression formats (natural language and structured objectives),
multi-dimensional evaluation metrics (configuration accuracy and task
completion), and a minimal interactive interface for LLM agents.  It
is intended to facilitate reproducible benchmarking of autonomous
network‐configuration agents across simple (single-device) and complex
(multi-device, interdependent) tasks.


--- middle

# Introduction

Network configuration tasks range from simple port and VLAN configuration on a single device to complex multi-device coordination for policy enforcement, traffic engineering, and fault remediation. Traditional automation tools rely on manually written scripts or narrow-scope synthesis modules; recent research shows promising results in using LLMs to translate intent into configuration commands. However, there is no standardized methodology to evaluate LLM agents in realistic, intent-driven workflows across diverse scenarios.

This document defines the Intent-Driven Network Configuration Evaluation Framework (IDNCEF), which specifies:

    A taxonomy of representative scenarios, classified by complexity (simple vs. complex) and task category (basic configuration, fault remediation, policy optimization).

    An intent expression format that supports both free-form natural language and a minimal structured objective schema.

    Multi-dimensional evaluation metrics: configuration accuracy, task completion rate, reasoning consistency, and interaction efficiency.

    A minimal, emulator-agnostic interaction interface that enables closed-loop agent workflows (observe, reason, apply, verify).

The framework ensures reproducible, fair evaluation of different LLM-based designs, identifies current limitations, and guides future improvements in autonomous network configuration.


# Terminology

For clarity within this document, the following terms and abbreviations
are defined:

Agent
  A software component powered by an LLM that consumes a task intent, interacts with a network environment, and issues configuration commands autonomously.

Configuration Command
  A device-specific instruction (e.g., a Cisco IOS CLI line or a Juniper Junos set statement) sent by the agent to a network device.

Environment
  An emulated or real network instance that exposes device status, topology information, and feedback on applied commands.

Intent
  A high-level specification of desired network behavior or objective, expressed in natural language or a structured format defined in this document.

Task
  A single evaluation unit defined by (1) a scenario category, (2) an environment topology, (3) initial device configurations, and (4) an intent. The agent is evaluated on its ability to fulfill the intent in the given environment.

Testcase
  A concrete, executable set of verification steps (e.g., ping tests, traffic-flow validation, policy checks) used to assert whether the agent’s final configuration satisfies the intent.

# Framework Overview

+-------------------+      +------------------+                                   
|    Task Datase    |      |    LLM Agent     |       +-------------------------+
|+----------------+ |      |+----------------+|       |        Evaluator        |
||Network Intents | |      || Decision-making||       |+----------+ +----------+|
||+--------+      |---(1)->|+----------------+|<-(4)-->|Reasoning | |Grnd Truth||
|||Routing |      | |      |+------+ +------+||       ||Trajectory| |Reasoning ||
|||Policy  | +---+| |      ||Percep| |Action|||       |+----------+ +----------+|
||+--------+ |QoS|| |      |+------+ +------+||       |     \             /     |
||+--------+ +---+| |      +------------------+       |      Rouge/Cos. Sim.    |
|||Security|      | |               |                 |                         |
||+--------+      | |               |                 |+----------+ +----------+|
|+----------------+ |              (3)                || Final    | |Grnd Truth|| 
|+----------------+ |               |            +---->| Configs  | |Configs   ||
||Network Topology| |      +------------------+  |    |+----------+ +----------+|
||+-----+ +-----+ | |      |   Environment    |  |    |     \             /     |
|||Nodes| |Links| |---(2)->|                  |--+    |    Precision/Recall     |
||+-----+ +-----+ | |      |    R2 --- R1     |       |                         |
|+----------------+ |      |    |      |      |       | +---------------------+ | 
|                   |      |    R3 --- R4     |<-(6)--->|      Testcases      | |
|+----------------+ |      |     (GNS3)       |       | +---------------------+ |
||Initial Configs |---(2)->|                  |       |            |            |
|+----------------+ |      |  Emulator-based  |       |        Pass Rate        |
|                   |      +------------------+       +-------------------------+
+-------------------+                                            

Legend:
(1)Task Assignment             (2)Environment Setup
(3)Interactive Task Execution  (4)Reasoning Trajectory Export      
(5)Final Configuration Export  (6)Testcase Execution

Figure 1: The LLM-Assisted Network Evaluation Framework

The proposed framework is shown in Figure 1. The flow begins with a **Task Dataset** defining network intents and topologies. The **LLM Agent** perceives the environment, reasons about required actions, and applies configuration commands. The **Environment** simulates or controls real devices, providing feedback for each action. Finally, the **Evaluator** compares the agent’s outputs against ground-truth configurations and reasoning, computing scores for accuracy, completion, consistency, and efficiency.

## Scenarios and Task Taxonomy

IDNCEF categorizes tasks by scenario and complexity. Scenario categories include basic configuration, fault remediation, and policy optimization. Each category further distinguishes simple tasks (single-device only) from complex tasks (multiple devices with dependencies).

### Basic Configuration Tasks

Basic configuration tasks involve establishing fundamental network services on one or more devices without dependencies requiring inter-device coordination beyond connectivity. These tasks are further classified as “simple” (single-device) or “complex” (multi-device but still protocol-independent).

#### Simple Basic Configuration (Single Device)

These tasks involve only one network device. The agent receives an intent and must generate device-specific commands to configure the device. Example intents include:

* “Configure VLAN 10 on switch SW1 with name ‘Users’ and assign ports Gi1/0/1–Gi1/0/10.”
* “On router R1, enable BGP peering to AS 65001 using neighbor 192.0.2.2, advertise network 203.0.113.0/24.”

In each case, the device has minimal or no pre-existing topology context. The agent simply transforms intent to configuration commands and verifies locally that the device responds without errors and that basic protocol sessions come up.

#### Complex Basic Configuration (Multi-Device)

These tasks span multiple devices but do not require cross-domain policies. They typically involve establishing connectivity (e.g., VLAN trunking across switches) or protocol peering (e.g., a two-router OSPF area). Example intent:

> “In the topology with switches SW1 and SW2 connected via trunk ports Gi1/0/48 and Gi2/0/48, create VLAN 20 called ‘Servers’ on both devices, and ensure hosts in VLAN 20 on SW1 can ping hosts in VLAN 20 on SW2. On R1 and R2, configure OSPF area 0, advertise 198.51.100.0/24, and verify adjacency.”

The agent must parse the intent, gather topology via the interface, plan per-device commands (e.g., switchport trunk configuration, OSPF settings), apply them in the correct order, and confirm inter-device connectivity. This requires basic multi-device state awareness but no advanced policy logic.

### Fault Remediation Tasks

Fault remediation tasks require the agent to detect or diagnose an existing issue (e.g., connectivity loss, interface down) and apply corrective configuration actions. Again, tasks are split into simple (single-device) and complex (multi-device) categories.

#### Simple Fault Remediation (Single Device)

These tasks involve one device with a localized failure. Example intent:

> “Interface Gi1/0/10 on switch SW1 is down. Ensure that the link is brought up, configured at 1000 Mbps full-duplex, and port security is enabled with a maximum of 2 MAC addresses.”

The agent must query live device state, recognize that interface Gi1/0/10 is administratively or operationally down, and issue correct commands to fix speed/duplex settings and enable port security.

#### Complex Fault Remediation (Multi-Device)

These tasks involve a failure affecting multiple devices or requiring inter-device coordination. Example intent:

> “Hosts in VLAN 30 cannot reach the Internet. On firewall FW1, verify that ACL ‘OUT-TO-IN’ allows 10.0.30.0/24 to egress and NAT is enabled. On router R3, ensure default route to ISP (203.0.113.1) is present. On switch SW3, check if VLAN 30 SVI is up. Restore full Internet connectivity for VLAN 30.”

The agent must collect state from SW3 (SVI), FW1 (ACL and NAT hits), and R3 (routing table). It must identify missing or misaligned entries (e.g., absent default route, incorrect ACL), apply corrections, then validate end-to-end reachability. This requires cross-device diagnosis and coordination.

### Policy Optimization Tasks

Policy optimization tasks involve higher-level goals such as traffic shaping, access-control adjustments, or quality-of-service (QoS) rules, typically across several devices. These are inherently complex.

#### Simple Policy Optimization (Single Device)

Although rare, some policy changes affect only one device. Example intent:

> “On router R2, limit outbound traffic to 500 Mbps on interface Gi0/0. Apply a QoS policy that shapes SSH traffic to 10 Mbps and guarantees 100 Mbps to DNS.”

The agent must convert bandwidth constraints into device-specific policer and class-map/policy-map commands, apply them, and check interface statistics.

#### Complex Policy Optimization (Multi-Device)

These tasks involve coordinated policy enforcement across multiple devices. Example intent:

> “Segment Guest traffic (VLAN 40) on SW4 and SW5 from Corporate traffic (VLAN 10). On firewall FW2, create policy so that VLAN 40 can only access the Internet on TCP/80 and TCP/443, and cannot reach internal servers. On switch SW4, implement ACL 100 to deny VLAN 40 access to VLAN 10. Confirm that VLAN 10 still has full access to internal servers and the Internet.”

The agent must partition VLANs at the switch layer, push down appropriate access-control entries, configure firewall policies, and validate isolation and permitted paths. This entails reasoning about device interdependencies and policy semantics.

## Intent Expression Formats

IDNCEF supports two complementary intent expression formats: free-form natural language and a minimal structured schema. Each intent **MUST** specify certain required fields to ensure that the agent has sufficient context to plan actions.

### Natural-Language Intents

Free-form natural language is the most flexible way to express intent but may introduce ambiguity. The agent **SHOULD** be capable of extracting actionable items from a user-provided sentence or paragraph.

#### Characteristics

* **MAY** contain domain-specific terminology (e.g., “OSPF area 0”, “ACL 101 permit ip any any”).
* **SHOULD** refer to device identifiers unambiguously (e.g., “SW1”, “R3”, “FW1”). If device names are ambiguous, parenthetical clarifications (e.g., “R3 (edge router)”) are **RECOMMENDED**.
* Intended outcomes **SHOULD** be explicit (e.g., “hosts in VLAN 30 must ping 8.8.8.8”), not merely descriptive (e.g., “fix Internet issue”).
* **SHOULD** mention SLA or performance bounds when relevant (e.g., “limit streaming traffic to 10 Mbps”).

#### Ambiguity Handling

If the agent cannot resolve critical details (e.g., missing interface names, unclear IP prefixes), it **MUST** either:

* Prompt the user for clarification via an interactive sub-dialog (if multi-turn interaction is supported), or
* Fall back to using default assumptions clearly documented in its reasoning trace (e.g., “Assumed IPv4 prefix 10.0.1.0/24 for VLAN1”).

For fair benchmarking, the agent’s assumption steps **SHALL** be logged and analyzed as part of the reasoning consistency metric.

### Structured Objective Format

A minimal structured format provides a schema that reduces ambiguity. Structured intents consist of a JSON-like object with predefined fields. The fields are:

```jsonc
{
  "task_name": <string>,
  "devices": [
    {
      "id": <string>,
      "role": <string>
    },
    …
  ],
  "topology": {
    "links": [
      {
        "endpoints": [<device_id>, <device_id>],
        "interfaces": [<iface1>, <iface2>]
      },
      …
    ]
  },
  "intent": {
    "type": <string>,
    "description": <string>,
    "parameters": { … }
  }
}
```

#### Parameters for Basic Configuration

For `type = "basic_config"`, `"parameters"` **MUST** include:

* `"config_type": <string>` (e.g., `"VLAN"`, `"BGP"`, `"OSPF"`)
* `"objects": [<object definition>]`, where each object is a nested JSON object specifying configuration details. Example:

  ```jsonc
  {
    "config_type": "VLAN",
    "objects": [
      {
        "vlan_id": 10,
        "name": "Users",
        "ports": ["SW1:Gi1/0/1","SW1:Gi1/0/2"]
      }
    ]
  }
  ```

#### Parameters for Fault Remediation

For `type = "fault_remediation"`, `"parameters"` **MUST** include:

* `"failure_type": <string>` (e.g., `"interface_down"`, `"routing_blackhole"`)
* `"scope": [<device_id>]` (devices to inspect)
* `"symptoms": [<string>]` (e.g., `"interface Gi1/0/10 down"`, `"no route to host 203.0.113.5"`)
* `"desired_state": { … }` (optional device-specific state to restore)

#### Parameters for Policy Optimization

For `type = "policy_optimization"`, `"parameters"` **MUST** include:

* `"policy_domain": <string>` (e.g., `"access_control"`, `"QoS"`)
* `"criteria": { … }` (e.g., allow/deny prefixes, bandwidth bounds)
* `"devices_involved": [<device_id>]`
* `"performance_metrics": [<string>]` (e.g., `"throughput"`, `"latency"`)

#### Device Referencing

Device IDs in all structured fields **MUST** match entries in the `"devices"` array. Interfaces **MUST** be fully qualified (e.g., `"R1:Gig0/1/0"`).

### Required Intent Fields

Regardless of format, each intent **SHALL** minimally specify:

* Target device(s) or roles (e.g., `"SW1"`, `"edge_router"`).
* Objective summary (human-readable description).
* Desired outcome, expressed as a measurable condition (e.g., “OSPF adjacency with cost 10 between R1 and R2”, “HTTP servers in VLAN 20 reachable from VLAN 10”).
* Any performance constraints or policy rules (if applicable).

If any of these are missing, the agent **MUST** either prompt for clarification or record assumptions and treat reduced context as part of its reasoning consistency evaluation.

## Evaluation Metrics

IDNCEF defines four orthogonal metrics. Each metric is quantified for every scenario and then aggregated or reported per category.

### Configuration Accuracy

Configuration Accuracy measures the semantic overlap between the agent-generated configuration commands and a ground-truth reference. Ground-truth references are curated per task by domain experts.

#### Comparison Method

1. Extract the set of commands actually applied by the agent from device running configurations (e.g., via `show running-config`).
2. Normalize both the agent’s commands and the ground-truth commands by removing vendor-specific ordering differences and canonicalizing whitespace.
3. Use a diff-style tree comparison (similar to `ciscoconfparse`) to identify missing or extraneous commands.

#### Scoring

* Precision = (number of correct commands applied) / (number of total commands applied by agent)
* Recall = (number of correct commands applied) / (number of total ground-truth commands)

Configuration Accuracy Score =

```
2 × (Precision × Recall) / (Precision + Recall)
```

Wildcard matching is applied for parameter values that can vary (e.g., automatically assigned AS numbers, dynamic interface indexes).

### Task Completion Rate

Task Completion Rate measures the proportion of testcases passed after the agent’s final configuration is deployed. Each task includes a suite of device- and network-level testcases.

#### Pass/Fail Criteria

* Each testcase is marked PASS if the observed device or network state matches the expected state (e.g., ping success, BGP adjacency up, ACL enforced).
* FAIL if any discrepancy is detected (e.g., unreachable host, BGP neighbor not established).

#### Score Calculation

Task Completion Rate =

```
(number of passed testcases) / (number of total testcases)
```

A Task Completion Rate of 1.0 indicates that the agent’s configuration fully satisfies the intent’s functional requirements.

### Reasoning Consistency

Reasoning Consistency measures the semantic similarity between the agent’s reasoning trace (the internal “chain-of-thought” or planned steps) and a ground-truth reasoning outline prepared by experts.

#### Reasoning Extraction

* For single-turn agents, the prompt must request an explicit rationale before issuing commands. The agent responds with two fields:

  1. Reasoning (plaintext outline)
  2. Commands
* For multi-turn (ReAct-style) agents, an auxiliary procedure concatenates interleaved “Thought” and “Action” steps into a unified reasoning narrative.

#### Embedding-Based Similarity

1. Encode both the agent’s reasoning and the ground-truth outline using a sentence embedding model.
2. Compute cosine similarity to yield S\_reason in the range \[0,1].
3. If S\_reason < 0.7, mark “Low Reasoning Consistency” and log for further analysis.

Because reasoning traces can vary in wording, embedding similarity is preferred to simple token-overlap.

### Interaction Efficiency

Interaction Efficiency quantifies how many interaction steps (API calls, command attempts, or clarification prompts) the agent needed to complete a task.

#### Metrics

* T\_total = Total wall-clock time from task assignment to final configuration.
* N\_actions = Number of `execute_cmd` or `update_cfg` calls made by the agent.
* N\_queries = Number of state-query calls (e.g., `get_running_cfg`, `get_topology`).
* N\_prompts = Number of clarification questions issued by the agent.

#### Scoring

The raw efficiency score is computed as:

```
E_raw = α · T_total_norm + β · N_actions_norm + γ · N_queries_norm + δ · N_prompts_norm
```

where:

* `T_total_norm = T_total / T_max`
* `N_actions_norm = N_actions / A_max`
* `N_queries_norm = N_queries / Q_max`
* `N_prompts_norm = N_prompts / P_max`
  and α, β, γ, δ are weight factors (default α = 0.4, β = 0.3, γ = 0.2, δ = 0.1). Lower E\_raw is more efficient. Then:

```
Interaction Efficiency Score = 1 − E_raw  (clamped to [0,1])
```

This composite metric balances speed, communication overhead, and human-agent clarifications.

## Agent–Environment Interaction Interface

IDNCEF specifies a minimal set of operations that every LLM agent **MUST** use to interact with the environment. The interface is vendor-agnostic.

### Primitive Actions

The following actions **SHALL** be supported by the environment and documented in an API specification.

#### get\_topology

* **Input Parameters**: List of device IDs (optional; if omitted, the entire topology is returned).
* **Returns**: Structured description of the requested devices and their neighbor links, including interface names, IP addresses, roles (switch/router/firewall), and current statuses (up/down).
* **Usage**:

  ```
  Agent calls get_topology(['SW1','SW2']) to retrieve only switch-related links.
  ```

#### get\_running\_cfg

* **Input Parameters**: Device ID string.
* **Returns**: The full running configuration text (or parsed JSON), including VLAN entries, routing processes, ACLs, QoS policies, etc.
* **Usage**:

  ```
  Agent calls get_running_cfg('R3') to inspect current OSPF settings.
  ```

#### update\_cfg

* **Input Parameters**: Device ID, list of configuration commands.
* **Returns**: For each command, a status code (OK/ERROR) and error message (if any).
* **Behavior**: Applies commands in order. If a command fails (e.g., syntax error), it is marked ERROR and the next command is processed. The agent **MUST** handle partial failures by reissuing corrected commands or aborting.
* **Usage**:

  ```
  Agent issues update_cfg('SW1', ['vlan 10 name Users', 'interface Gi1/0/1 switchport access vlan 10'])
  ```

#### execute\_cmd

* **Input Parameters**: Device ID, single command string for status checks (e.g., "show ip route").
* **Returns**: The command’s text output or parsed data.
* **Usage**:

  ```
  Agent invokes execute_cmd('FW1', 'show access-list OUT-TO-IN counters')
  ```

### State Queries

In addition to topology and configuration, the agent may need to retrieve dynamic state such as interface counters, routing tables, firewall flow logs, or BGP neighbor statuses. These can be obtained via `execute_cmd` or dedicated query actions:

#### get\_interface\_status

* **Input**: Device ID, interface list (optional).
* **Returns**: Link status (up/down), speed, duplex, input/output counters, error counts.

#### get\_policy\_status

* **Input**: Device ID, policy name (e.g., ACL or QoS policy).
* **Returns**: Summary of matches or drops (e.g., "ACL 101: 50 permits, 5 denies").

#### get\_ospf\_summary

* **Input**: Device ID.
* **Returns**: OSPF neighbor adjacency table, route summary, interface OSPF cost.

#### get\_bgp\_summary

* **Input**: Device ID.
* **Returns**: BGP neighbor statuses, prefixes received, route-map hits.

> Note: The environment **MAY** expose other read-only queries. Agents **SHOULD** use the narrowest query consistent with need to reduce overhead.

### Feedback and Error Reporting

The environment **MUST** provide clear, structured feedback for `update_cfg` and `execute_cmd`:

* For `update_cfg`, each command returns:

  ```
  { "command": "...", "status": "OK" }
  ```

  or

  ```
  { "command": "...", "status": "ERROR", "error_msg": "<description>" }
  ```

* For `execute_cmd`, returns:

  ```
  { "output": "<raw text>", "parsed": { ... } }
  ```

The agent **SHOULD** log all feedback to its reasoning trace. When a syntax or semantic error occurs, the agent **MUST** either:

* Attempt a correction (e.g., context switch for correct CLI mode), or
* Halt and record the failure for analysis (considered a FAIL for configuration accuracy).

Agents **MAY** implement retry loops but **SHOULD** not exceed five retries per command.

### Extensibility Considerations

Although IDNCEF specifies a minimal core interface, implementations **MAY** extend it. Any additional actions or queries **MUST** be documented in the environment’s API. Agents **SHOULD** ignore unrecognized optional actions or fail if a required action is missing.

Example extensions include:

* Automated knowledge retrieval: `search_command_manual(term)` to retrieve vendor CLI snippets.
* Real-time telemetry streaming: `subscribe_telemetry(stream_id)` for asynchronous interface counter updates.

To maintain fairness, extended features **SHOULD** be disabled when comparing agents unless all agents under test share the same extensions.

## Example Workflow

This section outlines a sample end-to-end run for a complex policy optimization task. The agent uses single-turn reasoning with explicit chain-of-thought prompts.

**Task Definition (Structured):**

```jsonc
{
  "task_name": "Guest Traffic Isolation",
  "devices": [
    { "id": "SW4", "role": "access_switch" },
    { "id": "SW5", "role": "access_switch" },
    { "id": "FW2", "role": "firewall" }
  ],
  "topology": {
    "links": [
      { "endpoints": ["SW4","FW2"], "interfaces": ["SW4:Gi1/0/24","FW2:Gi0/1"] },
      { "endpoints": ["SW5","FW2"], "interfaces": ["SW5:Gi1/0/24","FW2:Gi0/2"] }
    ]
  },
  "intent": {
    "type": "policy_optimization",
    "description": "Enforce that VLAN 40 (Guest) on SW4 and SW5 can only access the Internet on TCP/80 and TCP/443, and cannot reach internal servers in VLAN 10.",
    "parameters": {
      "policy_domain": "access_control",
      "criteria": {
        "allow": ["any->any:TCP/80","any->any:TCP/443"],
        "deny": ["10.0.10.0/24->10.0.10.0/24:any"]
      },
      "devices_involved": ["SW4","SW5","FW2"]
    }
  }
}
```

1. Agent reads structured task and calls `get_topology(None)` to fetch the entire topology. Receives JSON with two switch-to-firewall links.
2. Agent extracts that VLAN 40 SVI must exist on SW4 and SW5. Calls `get_running_cfg("SW4")` and `get_running_cfg("SW5")`, finds no VLAN 40 configured.
3. Constructs reasoning:

   ```
   Create VLAN 40 on SW4 & SW5, assign access ports for Guest.
   On FW2, create ACL "GUEST_OUT" permitting TCP/80 and TCP/443 for source 10.0.40.0/24, then deny other internal traffic (10.0.10.0/24). Apply NAT rule for Internet egress.
   Verify pings from VLAN 40 host to 8.8.8.8 and failure to 10.0.10.5.
   ```

   Logs the reasoning outline.
4. Issues `update_cfg("SW4", ["vlan 40 name Guest", "interface Gi1/0/1 switchport access vlan 40", ...])`. Receives OK for each.
5. Issues `update_cfg("SW5", ...)`. Receives OK.
6. Calls `get_running_cfg("FW2")` to inspect existing ACLs. Finds none matching "Guest". Plans:

   ```
   [
     "ip access-list extended GUEST_OUT",
     " permit tcp 10.0.40.0 0.0.0.255 any eq 80",
     " permit tcp 10.0.40.0 0.0.0.255 any eq 443",
     " deny ip 10.0.40.0 0.0.0.255 10.0.10.0 0.0.0.255",
     " permit ip any any"
   ]
   ```

   Issues `update_cfg("FW2", ...)`. Receives OK.
7. Calls `execute_cmd("FW2", "show access-list GUEST_OUT")` to verify counters are zero initially.
8. Simulates test traffic: Agent issues `execute_cmd("SW4", "ping 8.8.8.8 source 10.0.40.10 repeat 5")`. Observes 5/5 success.
9. Agent issues `execute_cmd("SW4", "ping 10.0.10.5 source 10.0.40.10 repeat 5")`, observes 0/5 success.
10. Reasoning trace completed. Agent returns final configuration and a summary:

    * Task Completion Rate = 1.0
    * S\_reason = 0.92 (cosine similarity with ground-truth outline)
    * S\_command = 0.95
    * E\_raw = 0.18 -> Efficiency = 0.82

This workflow demonstrates how IDNCEF’s components work together for a complex multi-device policy task.

## Security Considerations

This framework does not introduce new protocols that carry security implications. However, when agents fetch sensitive configuration (e.g., firewall policies, BGP passwords) via `get_running_cfg`, care **SHOULD** be taken to secure the control channel. All agent–environment interactions **SHOULD** be performed over TLS or an equivalent secure channel to avoid credential leakage. Agents **MUST** sanitize any user-provided intents to guard against injection attacks (e.g., `"ip access-list permit ip any any; no logging"`).

Additionally, agent logic that retries failing commands could be abused to launch denial-of-service against emulated devices if not properly rate-limited. Emulated environments **SHOULD** implement control message rate limiting during benchmarking to avoid infinite loops. Ground-truth reasoning outlines contain proprietary design information and **SHOULD** be stored securely and not published outside test teams.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
