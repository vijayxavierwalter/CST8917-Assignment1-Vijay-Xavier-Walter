# Serverless Computing - Critical Analysis

---

# Part 1: Paper Summary

In *Serverless Computing: One Step Forward, Two Steps Back*, Hellerstein et al. argue that first-generation serverless computing, especially Function-as-a-Service (FaaS) platforms such as AWS Lambda, is useful but too limited to represent the full future of cloud programming. The central thesis is that serverless computing moves cloud development forward by offering automatic scaling, managed execution, and pay-as-you-go pricing, but it also moves backward because it restricts efficient data processing and distributed computing. By “one step forward, two steps back,” the authors mean that FaaS improves operational simplicity and elasticity, but sacrifices important capabilities needed for modern data-intensive and distributed applications.

## Key Limitations of First-Generation FaaS

### Execution Time Constraints

AWS Lambda functions have maximum runtime limits, meaning long-running tasks must be broken into smaller pieces or restarted. Since functions are temporary and may not run on the same virtual machine again, developers cannot depend on local state being preserved between executions.

### Communication and Network Limitations

Serverless functions usually communicate with storage services rather than directly with each other. They are not directly addressable over the network, so one function cannot efficiently contact another running function. This creates I/O bottlenecks because data must pass through slower services such as object storage or databases instead of using fast point-to-point networking.

### Data Shipping Anti-Pattern

A major concern is the “data shipping” anti-pattern. Instead of moving computation close to where data already exists, FaaS often moves data to the function. This “shipping data to code” model increases latency, bandwidth usage, and cost. The authors argue that cloud systems should often do the opposite: move code closer to the data, especially for large-scale analytics and machine learning workloads.

### Limited Hardware Access

Current serverless functions generally expose only CPU and memory, without direct support for GPUs or specialized accelerators. This makes them a poor fit for workloads such as deep learning, scientific computing, and hardware-accelerated database processing.

### Distributed and Stateful Workloads

Because functions are short-lived, stateless, and unable to communicate directly, building systems that require coordination, leader election, transactions, or consistent shared state becomes inefficient and expensive. This limits innovation in open-source distributed systems and pushes developers toward proprietary cloud services.

## Proposed Future Directions

The authors propose a more flexible model of cloud programming, including:

- Fluid code and data placement
- Support for heterogeneous hardware
- Long-running, addressable virtual agents
- Asynchronous programming models
- Common intermediate representations for cloud execution
- Better service-level objectives
- Stronger security mechanisms

Overall, the paper does not reject serverless computing. Instead, it argues that today’s FaaS platforms are only an early version of what cloud programming could become. To unlock the true potential of the cloud, serverless systems must support data-intensive, distributed, stateful, and hardware-aware applications more effectively.

---

# Part 2: Azure Durable Functions Deep Dive

## Orchestration Model

Azure Durable Functions extends the traditional Function-as-a-Service (FaaS) model by introducing three components:

- **Client Functions**
- **Orchestrator Functions**
- **Activity Functions**

A client function starts a workflow instance and communicates with the Durable Functions runtime. The orchestrator function defines the workflow logic and coordinates the execution of activity functions, which perform the actual work such as data processing or API calls. Unlike basic FaaS, where each function executes independently and remains stateless, Durable Functions maintain workflow progress and coordinate multiple function executions over time through a durable orchestration engine.

This architecture directly addresses one of the criticisms raised by Hellerstein et al. (2019), who argued that traditional serverless platforms make complex, distributed workflows difficult to build because functions are isolated and short-lived. Durable Functions provide built-in workflow orchestration and coordination, reducing the need for developers to manually manage function interactions. However, they do not completely eliminate the underlying storage-based coordination mechanisms used to maintain workflow state.

## State Management

Azure Durable Functions address the stateless nature of traditional serverless functions by using:

- Event sourcing
- Checkpointing
- Replay mechanisms

The Durable Functions runtime stores the execution history and state of an orchestration in durable storage. Each completed activity is recorded as an event, allowing the orchestrator to rebuild its state by replaying these events when needed. Checkpointing ensures that workflow progress is saved automatically, enabling long-running processes to recover from failures without restarting from the beginning.

This capability addresses the criticism by Hellerstein et al. (2019) that serverless functions are inherently stateless and cannot easily maintain information across executions. Durable Functions provide reliable state persistence and workflow continuity while preserving the scalability benefits of serverless computing.

## Execution Timeouts

Azure Durable Functions can support long-running workflows because the orchestrator function does not execute continuously. Instead, the Durable Functions runtime saves the orchestration state and execution history to durable storage. When an activity completes or an external event occurs, the orchestrator is replayed from its saved state, allowing the workflow to continue without being constrained by the execution timeout of a single function instance.

As a result, orchestrations can run for days, weeks, or even months.

However, activity functions are still regular Azure Functions and remain subject to the timeout limits of the hosting plan. For example, on the Consumption plan, activity functions have execution time limits, while Premium and Dedicated plans allow much longer or unlimited execution times. This partially addresses Hellerstein et al.’s concern about short-lived serverless functions by enabling long-running workflows through orchestration.

## Communication Between Functions

In Azure Durable Functions, orchestrator functions communicate with activity functions through the Durable Task Framework. The orchestrator schedules activity functions and waits for their results, while the runtime manages message passing and state persistence behind the scenes. Developers do not need to manually coordinate communication using queues, databases, or storage services because the Durable Functions runtime handles these interactions automatically.

This partially addresses the criticism that serverless functions rely on slow storage intermediaries for communication. Durable Functions simplify coordination and workflow management, making distributed processes easier to build. However, the runtime still stores orchestration history and messages in durable storage, meaning communication remains storage-backed rather than direct.

## Parallel Execution (Fan-Out/Fan-In)

The fan-out/fan-in pattern in Azure Durable Functions enables an orchestrator function to:

1. Start multiple activity functions in parallel (**fan-out**)
2. Wait for all of them to complete (**fan-in**)

This pattern is useful for:

- Processing files
- Analyzing data
- Performing parallel API calls

The orchestrator tracks the status of each activity and automatically aggregates the results once all tasks have finished.

This capability helps address the concern that traditional serverless platforms make distributed computing difficult because functions are isolated and coordination is complex. Durable Functions provide built-in coordination for parallel tasks, allowing developers to create distributed workflows more easily. However, coordination is still managed through the Durable Functions runtime and backing storage rather than direct communication between function instances.

---

# Part 3: Critical Evaluation

## Position

My position is that Azure Durable Functions represent meaningful progress for serverless computing, but they do not fully achieve the future envisioned by Hellerstein et al. Instead, they mainly work around the limitations of first-generation Function-as-a-Service platforms by adding orchestration, durable state, and workflow coordination on top of the existing serverless model.

## Remaining Limitations

### Communication Between Functions

Hellerstein et al. criticize FaaS platforms because functions cannot directly communicate with each other and often rely on slow storage intermediaries. Durable Functions improve this from a developer perspective because orchestrator functions can call activity functions and coordinate their results through the Durable Task Framework. However, the default Azure Storage provider still uses queues, tables, and blobs to persist orchestration state and messages. Therefore, Durable Functions hide the complexity of storage-based coordination, but they do not remove the underlying dependency.

### Data Processing and Data Locality

The paper argues that serverless platforms often follow a “data shipping” model, where data is moved to code rather than moving code closer to data. Durable Functions help coordinate workflows, but they do not automatically colocate compute with large datasets or provide a true code-to-data execution model. They are useful for controlling a workflow, but the actual data movement problem still depends on how the developer designs the application and which Azure services are used.

## Overall Assessment

Durable Functions do solve some important practical problems. The runtime uses event sourcing and replay to restore workflow progress from saved history, which addresses the paper’s criticism that basic functions are stateless and short-lived. Durable Functions also support autoscaling in Consumption and Premium plans, where the Azure Functions scale controller adds or removes workers based on task latency.

Overall, my verdict is that Azure Durable Functions are an evolutionary improvement, not a complete redesign of serverless computing. They make serverless more practical for long-running, stateful, and coordinated workflows, which is a major step forward compared with basic FaaS.

However, because they still depend on storage-backed coordination and do not solve data locality or specialized hardware access, they only partially address the deeper architectural concerns raised by the paper. Durable Functions move serverless computing in the right direction, but they do not fully deliver the radical cloud programming model the authors envisioned for truly data-centric cloud computing environments that require high-performance distributed processing and efficient data locality.

---
