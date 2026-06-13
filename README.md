# Serverless Computing and Azure Durable Functions

---

# Part 1: Paper Summary

In *Serverless Computing: One Step Forward, Two Steps Back*, Hellerstein et al. argue that first-generation serverless computing, especially Function-as-a-Service (FaaS) platforms such as AWS Lambda, is useful but too limited to represent the full future of cloud programming.

The central thesis is that serverless computing moves cloud development forward by offering automatic scaling, managed execution, and pay-as-you-go pricing, but it also moves backward because it restricts efficient data processing and distributed computing. By **"one step forward, two steps back,"** the authors mean that FaaS improves operational simplicity and elasticity while sacrificing important capabilities needed for modern data-intensive and distributed applications.

## Key Limitations of First-Generation FaaS

### Execution Time Constraints
AWS Lambda functions have maximum runtime limits, meaning long-running tasks must be divided into smaller operations. Because functions are temporary and may not execute on the same virtual machine, local state cannot be relied upon between invocations.

### Communication and Network Limitations
Functions are not directly addressable and typically communicate through storage services. This creates I/O bottlenecks because data must pass through storage systems instead of fast point-to-point communication.

### Data Shipping Anti-Pattern
Instead of moving computation closer to data, FaaS platforms often move data to functions. This increases latency, bandwidth consumption, and cost.

### Limited Hardware Access
Most FaaS platforms only provide CPU and memory resources, with no direct support for GPUs or specialized accelerators.

### Distributed and Stateful Workloads
Because functions are short-lived, stateless, and unable to communicate directly, building systems that require coordination, transactions, or shared state becomes inefficient and costly.

## Proposed Future Directions

The authors propose future cloud platforms that support:

- Fluid code and data placement
- Heterogeneous hardware support
- Long-running addressable agents
- Asynchronous programming models
- Common execution frameworks
- Service-level objectives (SLOs)
- Stronger security mechanisms

Overall, the paper argues that current FaaS platforms are only an early stage of cloud programming. To fully realize the cloud's potential, future serverless systems must better support data-intensive, distributed, stateful, and hardware-aware applications.

---

# Part 2: Azure Durable Functions Deep Dive

## Orchestration Model

Azure Durable Functions extends the traditional Function-as-a-Service (FaaS) model by introducing three components:

- **Client Functions**
- **Orchestrator Functions**
- **Activity Functions**

A client function starts a workflow instance and communicates with the Durable Functions runtime. The orchestrator function defines workflow logic and coordinates activity functions, which perform the actual work.

Unlike basic FaaS, where each function executes independently and remains stateless, Durable Functions maintain workflow progress and coordinate multiple function executions through a durable orchestration engine.

This architecture addresses one of the criticisms raised by Hellerstein et al. (2019), who argued that traditional serverless platforms make complex distributed workflows difficult to build because functions are isolated and short-lived.

## State Management

Azure Durable Functions address the stateless nature of traditional serverless functions by using:

- Event sourcing
- Checkpointing
- Replay mechanisms

The Durable Functions runtime stores orchestration history and state in durable storage. Each completed activity is recorded as an event, allowing the orchestrator to rebuild its state when needed.

This capability addresses the criticism that serverless functions cannot easily maintain information across executions.

## Execution Timeouts

Azure Durable Functions can support long-running workflows because the orchestrator function does not execute continuously.

Instead, the runtime saves orchestration state and execution history to durable storage. When an activity completes or an external event occurs, the orchestrator is replayed from its saved state.

As a result, orchestrations can run for days, weeks, or even months.

However, activity functions are still regular Azure Functions and remain subject to the timeout limits of the hosting plan.

## Communication Between Functions

In Azure Durable Functions, orchestrator functions communicate with activity functions through the Durable Task Framework.

The runtime manages message passing and state persistence behind the scenes. Developers do not need to manually coordinate communication using queues, databases, or storage services.

However, communication still relies on storage-backed mechanisms rather than direct network communication between function instances.

## Parallel Execution (Fan-Out/Fan-In)

The fan-out/fan-in pattern enables an orchestrator function to:

1. Start multiple activity functions in parallel (**fan-out**)
2. Wait for all activities to complete (**fan-in**)

This pattern is useful for:

- File processing
- Data analysis
- Parallel API calls

Durable Functions provide built-in coordination for parallel tasks, making distributed workflows easier to build.

---

# Part 3: Critical Evaluation

## Position

My position is that Azure Durable Functions represent meaningful progress for serverless computing, but they do not fully achieve the future envisioned by Hellerstein et al.

Instead, they mainly work around the limitations of first-generation Function-as-a-Service platforms by adding orchestration, durable state, and workflow coordination on top of the existing serverless model.

## Remaining Limitations

### Communication Between Functions

Hellerstein et al. criticize FaaS platforms because functions cannot directly communicate with each other and often rely on slow storage intermediaries.

Durable Functions improve this from a developer perspective because orchestrator functions can coordinate activity functions through the Durable Task Framework. However, the underlying storage dependency still exists.

### Data Processing and Data Locality

The paper argues that serverless platforms often follow a **data shipping** model, where data is moved to code rather than moving code closer to data.

Durable Functions help coordinate workflows but do not automatically colocate compute with large datasets or provide a true code-to-data execution model.

## Overall Assessment

Durable Functions solve several practical problems:

- Workflow orchestration
- State persistence
- Long-running execution
- Autoscaling support

These capabilities make serverless applications more practical for enterprise workloads.

## Verdict

Azure Durable Functions are an **evolutionary improvement**, not a complete redesign of serverless computing.

They make serverless more practical for long-running, stateful, and coordinated workflows. However, because they still depend on storage-backed coordination and do not solve data locality or specialized hardware access, they only partially address the deeper architectural concerns raised by the paper.

Durable Functions move serverless computing in the right direction, but they do not fully deliver the radical cloud programming model envisioned for truly data-centric cloud computing environments that require high-performance distributed processing and efficient data locality.

---

# References

1. Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). *Serverless Computing: One Step Forward, Two Steps Back*.

2. Microsoft – Durable Functions Overview  
   https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview

3. Microsoft – Durable Task Orchestrations  
   https://learn.microsoft.com/en-us/azure/durable-task/common/durable-task-orchestrations

4. Microsoft – Durable Functions Code Constraints  
   https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-code-constraints

5. Microsoft – Function App Timeout Duration  
   https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale#function-app-time-out-duration

6. Microsoft – Durable Functions Performance and Scale  
   https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-perf-and-scale
