# Serverless Computing - Critical Analysis - CST8917

Student Name: Olivie Bergeron
Student #: 41068227
Date Due: 13/02/2025

## Part 1: Paper Summary (400-600 words)

### Main Argument: What is the central thesis of the paper? What do the authors mean by "one step forward, two steps back"?

The central thesis of this paper is to articulate how the first generation of serverless computing, particularly the focus on FaaS, introduces autoscaling and pay-as-you-go compute, which simplifies cloud operations. However, this advancement also takes "two steps back" because it fails to support data-intensive and distributed operations (Hellerstein et al., 2019). In essence, serverless advances cloud operations but hinders cloud programming and systems innovation.

### Key Limitations Identified: Summarize the specific limitations the authors identify with first-generation serverless/FaaS platforms. Your summary should cover:

#### Execution time constraints

- FaaS functions are short-lived

- No mechanism ensures "to ensure that subsequent invocations
  are run on the same VM" (Hellerstein et al., 2019).

#### Communication and network limitations (I/O bottleneck, lack of direct addressability)

- Network bandwidth is far lower than disk or SSD bandwidth.
  - Empirical measurement: average 538 Mbps per Lambda function, "an order of magnitude slower than a single modern SSD" (Hellerstein et al., 2019).

  - As more functions run on a single VM, bandwidth per function drops further (e.g., 28.7 Mbps across 20 functions (Hellerstein et al., 2019)).

- FaaS functions are not network-addressable meaning they cannot communicate directly with one another. (Hellerstein et al., 2019)

- All inter-function communication must go through slow, expensive cloud storage services. (Hellerstein et al., 2019)

#### The "data shipping" anti-pattern (moving data to code vs. code to data)(Hellerstein et al., 2019)

- FaaS enforces a data-shipping model: each function invocation pulls data from storage across the network.

- The outcome is high latency, wasted bandwidth, and increased cost for data-driven workloads like machine learning, analytics, and transactions.

#### Limited hardware access (no GPUs or specialized hardware)(Hellerstein et al., 2019)

- FaaS users have no access to specialized or high-performance hardware

- This prevents both deep learning and high-performance data processing.

#### Challenges for distributed computing and stateful workloads (Hellerstein et al., 2019)

- Because functions are stateless and isolated, implementing coordination or distributed protocols is infeasible (very slow and long; goes against FaaS core princible)

- Stateful applications cannot be efficiently built on current FaaS.

### Proposed Future Directions: What characteristics do the authors propose for future cloud programming? Mention at least three of their proposals. (Hellerstein et al., 2019)

1. Fluid Code and Data Placement

- The platform should physically colocate code with data where beneficial, dynamically moving code toward data to reduce network I/O.

- Elasticity should still allow logical separation of compute and storage, but intelligent placement should minimize data movement.

2. Support for "Heterogeneous Hardware" (Hellerstein et al., 2019)

- Cloud platforms should offer transparent access to specialized hardware.

- Scheduling systems should dynamically allocate workloads to the most cost-effective and performance-appropriate hardware.

- Programmers can optionally specify hardware affinities or let compilers optimize toward hardware targets.

3. Long-Running, Addressable Virtual Agents

- Introduce persistent, addressable functions or actors that can retain state and communicate directly.

- These should be virtualized to allow for elasticity while still supporting direct addressing and low-latency communication between agents.

## Part 2: Azure Durable Functions Deep Dive (5 topic explanations)

### Orchestration model: How do orchestrator, activity, and client functions work together? How does this differ from basic FaaS?

In a Durable Function, the orchectration model acts as a stateful parent component that coordinates stateless children functions. This is a shift from FaaS model because each function is an isolated stateless event. (Microsoft, 2025)

### State management: How does Durable Functions manage state (event sourcing, checkpointing, replay)? How does this address the paper's criticism that functions are stateless?

Durable Functions addresses the criticisms in the paper by using a stateful workflow layer on top of the stateless FaaS. Durable Functions on Azure uses the Event Sourcing pattern to provide context/state to its stateless function. (Microsoft, n.d.; Microsoft, 2025)

### Communication between functions: How do orchestrator and activity functions communicate? Does this address the paper's criticism about functions needing slow storage intermediaries?

In Durable Functions, communication is managed by a "Task Hub" which uses storage queues to schedule work and return results. While individual functions remain short-lived and stateless, the Task Hub allows the orchestrator to stitch them into long-running, parallel workflows. This manages the complexity of stateful coordination that the paper identifies as a limitation, but it still relies on the "slow storage" intermediaries the authors criticize, as all inter-function communication must be written to disk to ensure durability (Microsoft, 2023b).

### Parallel execution (fan-out/fan-in): How does the fan-out/fan-in pattern work? How does it address the paper's concern about distributed computing?

In a Durable Functions, fan-in/out works by having the orchestrator schedule multiple activity functions to run in parellel (fan-out), and once all of the function return a result using a programmatic "wait" to aggregate the result (fan-in). This does address the distributed computing limitations that function on their own could not do.
(Microsoft, 2023a)

## Part 3: Critical Evaluation (400-600 words)

### Limitations that remain unresolved: Identify two criticisms from the paper that Azure Durable Functions has not resolved or only partially addresses. Explain why these limitations persist.

#### Data Shipping Anti-Pattern

The Criticism: FaaS enforces a model where data must be moved to the code across the network, which is significantly slower than local disk access.

Why: Durable Functions relies on Azure Storage (Task Hub) for state management and communication, leading to high latency and wasted bandwidth due to data serialization and intermediary storage.

#### Specialized Hardware Access

Criticism: FaaS platforms generally lack access to specialized hardware, hindering high-performance data processing and machine learning workloads.

Why: Durable Function are basically an abtrac layer built on top of Functions. Now fixing the coordination with the Orchestration and Task Hub is benificial, it still does not fix the lack of accessibility of highend hardware. My understanding leads me to think that the reason of this limitation is because of the serverless nature of functions.

### Your Verdict: Does Azure Durable Functions represent the kind of progress the authors envisioned for serverless computing, or does it merely work around the fundamental limitations without truly solving them? Take a clear position and support it with evidence from your research.

Azure Durable Functions is an essential evolution in Serverless Orchestration, making it possible to build complex business workflows that were previously considered infeasible. However, it is not the Serverless 2.0 the authors envisioned. The paper calls for a fundamental redesign of the cloud fabric to eliminate the I/O bottleneck. Durable Functions merely provides a reliable way to live with that latency bottleneck and hardware access limitation.

## Reference

Hellerstein, J., Faleiro, J., Gonzalez, J., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). Serverless Computing: One Step Forward, Two Steps Back. https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf

Microsoft. (n.d.). Event Sourcing pattern - Azure Architecture Center. Learn.microsoft.com. https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

Microsoft. (2023a, March 23). Fan-out/fan-in scenarios in Durable Functions - Azure. Microsoft.com. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-cloud-backup?tabs=csharp

Microsoft. (2023b, May 4). Task hubs in Durable Functions - Azure. Microsoft.com. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-task-hubs?tabs=csharp

Microsoft. (2023c, August 4). Durable Functions Overview - Azure. Learn.microsoft.com. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=in-process%2Cnodejs-v3%2Cv1-model&pivots=csharp

Microsoft. (2025, December 11). Durable Orchestrations - Azure Functions. Microsoft.com. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations?tabs=csharp-inproc

## AI Disclaimer

Used AI to help me summaries the paper. Double check the results by using searching feature in the PDF to confirm that the summary was accurate. Also used the AI Proofread tool on my Mac for grammar and syntax correction.
