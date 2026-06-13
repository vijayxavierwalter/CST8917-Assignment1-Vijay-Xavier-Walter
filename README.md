Serverless Computing and Azure Durable Functions
Part 1: Paper Summary

In Serverless Computing: One Step Forward, Two Steps Back, Hellerstein et al. argue that first-generation serverless computing, especially Function-as-a-Service (FaaS) platforms such as AWS Lambda, is useful but too limited to represent the full future of cloud programming. The central thesis is that serverless computing moves cloud development forward by offering automatic scaling, managed execution, and pay-as-you-go pricing, but it also moves backward because it restricts efficient data processing and distributed computing. By "one step forward, two steps back," the authors mean that FaaS improves operational simplicity and elasticity, but sacrifices important capabilities needed for modern data-intensive and distributed applications.

The paper identifies several major limitations in current FaaS platforms. First, execution time is restricted. AWS Lambda functions have maximum runtime limits, meaning long-running tasks must be divided into smaller operations. Because functions are temporary and may not execute on the same virtual machine, local state cannot be relied upon between invocations.

Second, communication and networking are limited. Functions are not directly addressable and typically communicate through storage services. This creates I/O bottlenecks because data must pass through storage systems instead of fast point-to-point communication.

Another major concern is the "data shipping" anti-pattern. Instead of moving computation closer to data, FaaS platforms often move data to functions. This increases latency, bandwidth consumption, and cost. The authors argue that cloud systems should instead move code closer to where data resides whenever possible.

The paper also highlights limited hardware access. Most FaaS platforms only provide CPU and memory resources, with no direct support for GPUs or specialized accelerators. This makes them unsuitable for workloads such as machine learning, scientific computing, and hardware-accelerated analytics.

Finally, the authors argue that FaaS makes distributed and stateful workloads difficult. Because functions are short-lived, stateless, and unable to communicate directly, building systems that require coordination, transactions, or shared state becomes inefficient and costly.

To address these challenges, the authors propose future cloud platforms that support fluid code and data placement, heterogeneous hardware, long-running addressable agents, asynchronous programming models, common execution frameworks, service-level objectives, and stronger security mechanisms.

Overall, the paper argues that current FaaS platforms are only an early stage of cloud programming. To fully realize the cloud's potential, future serverless systems must better support data-intensive, distributed, stateful, and hardware-aware applications.

Part 2: Azure Durable Functions Deep Dive
Orchestration Model

Azure Durable Functions extends the traditional Function-as-a-Service (FaaS) model by introducing three components: client functions, orchestrator functions, and activity functions. A client function starts a workflow instance and communicates with the Durable Functions runtime. The orchestrator function defines the workflow logic and coordinates the execution of activity functions, which perform the actual work such as data processing or API calls. Unlike basic FaaS, where each function executes independently and remains stateless, Durable Functions maintain workflow progress and coordinate multiple function executions over time through a durable orchestration engine.

This architecture directly addresses one of the criticisms raised by Hellerstein et al. (2019), who argued that traditional serverless platforms make complex, distributed workflows difficult to build because functions are isolated and short-lived. Durable Functions provide built-in workflow orchestration and coordination, reducing the need for developers to manually manage function interactions. However, they do not completely eliminate the underlying storage-based coordination mechanisms used to maintain workflow state.

State Management

Azure Durable Functions address the stateless nature of traditional serverless functions by using event sourcing, checkpointing, and replay mechanisms. The Durable Functions runtime stores the execution history and state of an orchestration in durable storage. Each completed activity is recorded as an event, allowing the orchestrator to rebuild its state by replaying these events when needed. Checkpointing ensures that workflow progress is saved automatically, enabling long-running processes to recover from failures without restarting from the beginning.

This capability addresses the criticism by Hellerstein et al. (2019) that serverless functions are inherently stateless and cannot easily maintain information across executions. Durable Functions provide reliable state persistence and workflow continuity while preserving the scalability benefits of serverless computing.

Execution Timeouts

Azure Durable Functions can support long-running workflows because the orchestrator function does not execute continuously. Instead, the Durable Functions runtime saves the orchestration state and execution history to durable storage. When an activity completes or an external event occurs, the orchestrator is replayed from its saved state, allowing the workflow to continue without being constrained by the execution timeout of a single function instance. As a result, orchestrations can run for days, weeks, or even months.

However, activity functions are still regular Azure Functions and remain subject to the timeout limits of the hosting plan. On the Consumption plan, activity functions have execution time limits, while Premium and Dedicated plans allow much longer or unlimited execution times. This partially addresses the concern about short-lived serverless functions by enabling long-running workflows through orchestration.

Communication Between Functions

In Azure Durable Functions, orchestrator functions communicate with activity functions through the Durable Task Framework. The orchestrator schedules activity functions and waits for their results, while the runtime manages message passing and state persistence behind the scenes. Developers do not need to manually coordinate communication using queues, databases, or storage services because the Durable Functions runtime handles these interactions automatically.

This partially addresses the criticism that serverless functions rely on slow storage intermediaries for communication. Durable Functions simplify coordination and workflow management, making distributed processes easier to build. However, the runtime still stores orchestration history and messages in durable storage, meaning communication remains storage-backed rather than direct.

Parallel Execution (Fan-Out/Fan-In)

The fan-out/fan-in pattern in Azure Durable Functions enables an orchestrator function to start multiple activity functions in parallel (fan-out) and then wait for all of them to complete before continuing (fan-in). This pattern is useful for workloads such as processing files, analyzing data, or performing parallel API calls. The orchestrator tracks the status of each activity and automatically aggregates the results once all tasks have finished.

This capability helps address the concern that traditional serverless platforms make distributed computing difficult because functions are isolated and coordination is complex. Durable Functions provide built-in coordination for parallel tasks, allowing developers to create distributed workflows more easily. However, coordination is still managed through the Durable Functions runtime and backing storage rather than direct communication between function instances.

Part 3: Critical Evaluation

My position is that Azure Durable Functions represent meaningful progress for serverless computing, but they do not fully achieve the future envisioned by Hellerstein et al. Instead, they mainly work around the limitations of first-generation Function-as-a-Service platforms by adding orchestration, durable state, and workflow coordination on top of the existing serverless model.

One unresolved limitation is communication between functions. Hellerstein et al. criticize FaaS platforms because functions cannot directly communicate with each other and often rely on slow storage intermediaries. Durable Functions improve this from a developer perspective because orchestrator functions can call activity functions and coordinate their results through the Durable Task Framework. However, the default Azure Storage provider still uses queues, tables, and blobs to persist orchestration state and messages. Therefore, Durable Functions hide the complexity of storage-based coordination, but they do not remove the underlying dependency.

A second unresolved limitation is efficient data processing and data locality. The paper argues that serverless platforms often follow a "data shipping" model, where data is moved to code rather than moving code closer to data. Durable Functions help coordinate workflows, but they do not automatically colocate compute with large datasets or provide a true code-to-data execution model.

Durable Functions do solve some important practical problems. The runtime uses event sourcing and replay to restore workflow progress from saved history, which addresses the paper's criticism that basic functions are stateless and short-lived. Durable Functions also support autoscaling and long-running orchestrations, making serverless applications more practical for enterprise workloads.

Overall, my verdict is that Azure Durable Functions are an evolutionary improvement, not a complete redesign of serverless computing. They make serverless more practical for long-running, stateful, and coordinated workflows. However, because they still depend on storage-backed coordination and do not solve data locality or specialized hardware access, they only partially address the deeper architectural concerns raised by the paper. Durable Functions move serverless computing in the right direction, but they do not fully deliver the radical cloud programming model the authors envisioned for truly data-centric cloud computing environments that require high-performance distributed processing and efficient data locality.
