# Orchestra Framework: Project Kickoff Plan & Specification

**Version:** 0.1.0 (Genesis)
**Date:** 2025-03-28
**Project Lead:** [Your Name]
**Team Size:** 2-3 Developers (Graduate Students + Lead)

## 1. Introduction & Vision

**What is Orchestra?**

Orchestra is envisioned as an open-source C#/.NET framework enabling developers to build scalable, real-time applications orchestrated by collaborating AI agents. It integrates powerful technologies:

*   **Apache Kafka:** For resilient, high-throughput event-driven communication.
*   **Kafka Schema Registry:** For robust message schema management and evolution (using Avro/Protobuf).
*   **Microsoft Semantic Kernel:** For AI agent logic, blending Large Language Models (LLMs) with native C# code.
*   **Microsoft Orleans:** For managing the state and lifecycle of distributed entities (workflows, user sessions) using the virtual actor model.

**Why build Orchestra?**

The AI agent landscape is rapidly evolving. While Python has many tools, there's a need for a cohesive, C#-native framework that leverages the strengths of the .NET ecosystem for building enterprise-grade, real-time AI orchestration systems. Orchestra aims to fill this gap.

**Vision:** To become the go-to C# framework for developers building complex, event-driven applications featuring collaborating AI agents and human-in-the-loop interaction.

## 2. Project Goals (MVP Focus)

The primary goal is to build a **Minimum Viable Product (MVP)** that demonstrates the core architectural patterns and end-to-end flow.

**MVP Goals:**

1.  **Establish Core Infrastructure:** Set up local development environment using Docker Compose (Kafka, Schema Registry, Orleans Silo).
2.  **Implement Basic Event Flow:** Demonstrate a message flowing from a trigger -> Kafka -> Orleans Grain -> Kafka -> Agent Service -> Kafka -> back to Orleans/Log.
3.  **Integrate Schema Registry:** Use Avro (recommended) or Protobuf for schema definition and serialization/deserialization across Kafka producers/consumers and Orleans Streams.
4.  **Integrate Orleans:** Implement a simple `WorkflowGrain` that holds minimal state, subscribes to a Kafka topic via Orleans Streams, and publishes a task back to Kafka via Streams.
5.  **Integrate Semantic Kernel:** Implement a basic Agent Service that consumes a task from Kafka, uses SK `Kernel` to invoke one simple Semantic Function (LLM call) and one simple Native Function (C# code), and publishes a result to Kafka.
6.  **Implement Basic Gateway:** Create a simple trigger mechanism (e.g., a minimal Web API endpoint) that initiates the workflow by publishing an event to Kafka.
7.  **Provide Core Documentation:** README, basic architecture diagram, setup instructions.

## 3. Core Architecture Recap

**(Refer to detailed specification previously generated. A diagram is essential for the team.)**

*   **Central Bus:** Kafka + Schema Registry
*   **Stateful Components:** Orleans Cluster hosting Grains (Coordinator, Notification, Session state)
*   **AI Logic:** Agent Services (C#) using Semantic Kernel
*   **External Actions:** Tool Executor Services (C#)
*   **User Interaction:** Gateway Services (Discord, WebSocket, API)
*   **Communication:** Primarily asynchronous via Kafka topics using registered Avro/Protobuf schemas. Orleans Streams used for Kafka integration within Grains.

**[TODO: Insert Architecture Diagram Here]**

## 4. Technology Stack (MVP)

*   **.NET 8+** (LTS)
*   **C#**
*   **Microsoft.Orleans** (Core, Server, Streaming, Streaming.Kafka provider)
*   **Microsoft.SemanticKernel** (Core)
*   **Confluent.Kafka** (.NET Client for Kafka)
*   **Confluent.SchemaRegistry** (Client)
*   **Confluent.SchemaRegistry.Serdes.Avro** (Or specific serializer for chosen format)
*   **ASP.NET Core** (For Gateway/API)
*   **Worker Services** (For Agents, Tools)
*   **Docker / Docker Compose** (Local Development Environment)
*   **Avro / Protobuf** (Schema Definition)
*   **Git / GitHub (or similar)** (Version Control)

## 5. MVP Scope Definition

To make this achievable for a small team, the MVP scope is tightly focused:

**What's IN for MVP:**

*   **Infrastructure:** Docker Compose setup for Kafka, Schema Registry, 1 Orleans Silo (in-memory persistence ok for now).
*   **Schemas:** 2-3 basic Avro schemas defined and registered (e.g., `WorkflowStartEvent`, `AgentTask`, `AgentResult`).
*   **Coordinator:** 1 simple `WorkflowGrain` that activates on `WorkflowStartEvent`, holds a basic state (e.g., "started"), and publishes one `AgentTask`.
*   **Agent:** 1 Agent Service type. Consumes `AgentTask`, calls *one* simple Semantic Function (e.g., "Summarize this text: {{$input}}") and *one* dummy Native Function (e.g., returns a fixed string), publishes `AgentResult`.
*   **Gateway:** 1 minimal ASP.NET Core Web API endpoint that takes simple input and publishes the `WorkflowStartEvent` to Kafka.
*   **End-to-End Flow:** Trigger API -> Kafka (`WorkflowStartEvent`) -> Orleans Stream -> `WorkflowGrain` -> Orleans Stream -> Kafka (`AgentTask`) -> Agent Service (uses SK) -> Kafka (`AgentResult`). We will verify this flow via logging initially.
*   **Configuration:** Basic setup via `appsettings.json` / environment variables.
*   **Documentation:** Basic README, setup guide, simple architecture diagram.

**What's OUT for MVP:**

*   Complex coordination logic or multi-agent collaboration.
*   Discord or WebSocket Gateways (API endpoint is simpler).
*   Notification Service or User Subscriptions.
*   Tool Executors.
*   Robust error handling, retries, Dead Letter Queues (DLQs).
*   Orleans persistence beyond in-memory/development storage.
*   Scalable deployment configurations (Kubernetes, etc.).
*   Monitoring and distributed tracing setup.
*   UI.
*   Advanced Semantic Kernel features (Planner, Memory connectors).
*   Security considerations (AuthN/Z).

## 6. Development Phases & Milestones (Suggested)

*(These phases can overlap, especially with parallel work)*

**Phase 1: Foundation & Setup (Est. 1 Week)**

*   **Goal:** Get core infrastructure running locally and establish basic project structure.
*   **Tasks:**
    *   Set up Git repository.
    *   Create solution structure (.NET solution with initial projects).
    *   Develop `docker-compose.yml` for Kafka, Schema Registry, basic Orleans Silo host.
    *   Verify basic connectivity to Kafka and Schema Registry from a simple C# console app.
    *   Basic README with setup instructions.
*   **Milestone:** Local infrastructure runs via `docker-compose up`.

**Phase 2: Core Communication Plumbing (Est. 1-2 Weeks)**

*   **Goal:** Implement basic Kafka producer/consumer with Schema Registry integration.
*   **Tasks:**
    *   Define initial Avro schemas (`WorkflowStartEvent`, `AgentTask`, `AgentResult`).
    *   Register schemas manually or via script with local Schema Registry.
    *   Implement a simple Kafka Producer (e.g., in the API Gateway stub) using `Confluent.SchemaRegistry.Serdes.Avro`.
    *   Implement a simple Kafka Consumer (e.g., in a basic Worker Service stub) using Avro deserializer.
    *   Verify end-to-end message production/consumption with schema validation.
*   **Milestone:** Messages with registered Avro schemas can be sent and received between basic C# apps via Kafka.

**Phase 3: Orleans Core & Stream Integration (Est. 1-2 Weeks)**

*   **Goal:** Set up Orleans Silo, simple Grain, and connect it to Kafka via Streams.
*   **Tasks:**
    *   Configure Orleans Silo Host project (clustering, development storage).
    *   Implement basic `IWorkflowGrain` and `WorkflowGrain` (minimal state).
    *   Configure `Orleans.Streaming.Kafka` provider, connecting to Kafka & Schema Registry (using Avro SerDes). **(Complex Part - See Code Example)**
    *   Implement `WorkflowGrain` logic to subscribe to the input stream (`WorkflowStartEvent`) and publish to the output stream (`AgentTask`).
    *   Test Grain activation and stream processing (may require publishing test events directly to Kafka).
*   **Milestone:** `WorkflowGrain` activates on Kafka event via Streams and publishes another Kafka event via Streams.

**Phase 4: Agent Core (Semantic Kernel + Kafka) (Est. 1-2 Weeks)**

*   **Goal:** Implement the basic Agent Service using Semantic Kernel and Kafka.
*   **Tasks:**
    *   Create Agent Worker Service project.
    *   Integrate Semantic Kernel SDK, configure LLM connection.
    *   Define simple Semantic Function (`.txt` prompt) and Native Function (C# method).
    *   Implement Kafka Consumer for `AgentTask` (using Avro deserializer).
    *   Implement Agent logic: on message receipt, invoke SK Kernel (`Kernel.InvokeAsync`) with Semantic/Native functions.
    *   Implement Kafka Producer for `AgentResult` (using Avro serializer).
*   **Milestone:** Agent Service consumes task from Kafka, executes SK functions, publishes result to Kafka.

**Phase 5: Gateway & End-to-End Integration (Est. 1 Week)**

*   **Goal:** Connect all pieces and achieve the MVP end-to-end flow.
*   **Tasks:**
    *   Implement minimal ASP.NET Core API endpoint to trigger `WorkflowStartEvent`.
    *   Ensure all Kafka topics, schemas, and stream configurations align across services.
    *   Run all services together (`docker-compose`).
    *   Trigger the API endpoint and trace the flow through logs across Kafka -> Orleans -> Agent -> Kafka.
    *   Debug and resolve integration issues.
*   **Milestone:** MVP end-to-end flow is functional and demonstrable via API trigger and logs.

**Phase 6: Documentation & Cleanup (Ongoing, Focus ~0.5 Week)**

*   **Goal:** Polish initial documentation and code.
*   **Tasks:**
    *   Update README with usage examples.
    *   Add basic architecture diagram.
    *   Code cleanup, add comments.
    *   Ensure setup instructions are accurate.
*   **Milestone:** MVP code and documentation ready for initial review/sharing.

**Total Estimated MVP Time:** Approx **6 - 9 Weeks** (adjust based on team velocity and complexity encountered).

## 7. Team Roles & Collaboration (Suggested for 2-3 People)

*   **Team Lead ([Your Name]):** Overall architecture guidance, unblocking technical challenges, final code reviews, potentially takes on complex parts (e.g., initial Orleans/Stream setup or SK integration).
*   **Developer 1 (Backend/Infra Focus):**
    *   Focus: Docker Compose, Kafka/SR setup, Orleans Silo configuration, Orleans Stream Provider setup, Gateway API implementation, Kafka producer/consumer plumbing in non-agent services, CI/CD setup later.
*   **Developer 2 (Application/AI Focus):**
    *   Focus: Avro Schema definition & registration, Agent Service implementation (Semantic Kernel functions, prompts, Kafka client integration), potentially Tool Executor later, writing unit/integration tests for agent logic.
*   **(If 3 People) Developer 3 (Cross-Cutting/Support):**
    *   Focus: Documentation (README, diagrams), testing (integration, manual E2E), potentially assists Developer 2 with Agent/Tool logic or Developer 1 with Gateway/Orleans configuration, could build a simple test client for the API/WebSocket later.

**Collaboration:**

*   **Daily Standups:** Quick sync on progress, blockers.
*   **Pair Programming:** Especially crucial for complex parts like Orleans Streams + Kafka + Schema Registry integration.
*   **Code Reviews:** Maintain code quality and knowledge sharing. Use Pull Requests.
*   **Shared Task Board:** (e.g., GitHub Projects, Trello, Jira) to track tasks.
*   **Centralized Documentation:** Keep design decisions and setup guides updated.

## 8. Key Technical Deep Dives & Code Examples

**(These are illustrative and may need refinement based on exact library versions)**

### 8.1. Kafka + Schema Registry + Avro Client (Agent/Gateway/Tool)

\`\`\`csharp
// --- Add Nuget Packages ---
// Confluent.Kafka
// Confluent.SchemaRegistry
// Confluent.SchemaRegistry.Serdes.Avro
// Your generated Avro classes (or use GenericRecord)

// --- Producer Example (e.g., in Gateway) ---
using Confluent.Kafka;
using Confluent.SchemaRegistry;
using Confluent.SchemaRegistry.Serdes;

// Configuration (from appsettings/env vars)
var kafkaBootstrapServers = "localhost:9092";
var schemaRegistryUrl = "http://localhost:8081";
var topic = "workflow-start-events"; // Example topic

var producerConfig = new ProducerConfig { BootstrapServers = kafkaBootstrapServers };
var schemaRegistryConfig = new SchemaRegistryConfig { Url = schemaRegistryUrl };

using var schemaRegistry = new CachedSchemaRegistryClient(schemaRegistryConfig);
using var producer = new ProducerBuilder<string, YourNamespace.WorkflowStartEvent>(producerConfig)
    .SetKeySerializer(Serializers.Utf8)
    .SetValueSerializer(new AvroSerializer<YourNamespace.WorkflowStartEvent>(schemaRegistry)) // Use Avro serializer
    .Build();

var startEvent = new YourNamespace.WorkflowStartEvent { /* ... populate properties ... */ };
var message = new Message<string, YourNamespace.WorkflowStartEvent> { Key = Guid.NewGuid().ToString(), Value = startEvent };

try {
    var deliveryResult = await producer.ProduceAsync(topic, message);
    Console.WriteLine(\$"Delivered '{deliveryResult.Value}' to '{deliveryResult.TopicPartitionOffset}'");
} catch (ProduceException<string, YourNamespace.WorkflowStartEvent> e) {
    Console.WriteLine(\$"Delivery failed: {e.Error.Reason}");
    // Handle schema registry errors specifically if needed (e.g., invalid schema)
}

// --- Consumer Example (e.g., in Agent Service / Orleans Ingestor if not using Streams directly) ---
using Confluent.Kafka;
using Confluent.SchemaRegistry;
using Confluent.SchemaRegistry.Serdes;

// Configuration
var kafkaBootstrapServers = "localhost:9092";
var schemaRegistryUrl = "http://localhost:8081";
var topic = "agent-task.some-agent"; // Example topic
var consumerGroup = "agent-service-group";

var consumerConfig = new ConsumerConfig {
    BootstrapServers = kafkaBootstrapServers,
    GroupId = consumerGroup,
    AutoOffsetReset = AutoOffsetReset.Earliest
};
var schemaRegistryConfig = new SchemaRegistryConfig { Url = schemaRegistryUrl };

using var schemaRegistry = new CachedSchemaRegistryClient(schemaRegistryConfig);
using var consumer = new ConsumerBuilder<string, YourNamespace.AgentTask>(consumerConfig)
    .SetKeyDeserializer(Deserializers.Utf8)
    .SetValueDeserializer(new AvroDeserializer<YourNamespace.AgentTask>(schemaRegistry).AsSyncOverAsync()) // Use Avro deserializer
    .SetErrorHandler((_, e) => Console.WriteLine(\$"Error: {e.Reason}")) // Handle Kafka errors
    .Build();

consumer.Subscribe(topic);

CancellationTokenSource cts = new CancellationTokenSource();
try {
    while (true) {
        var consumeResult = consumer.Consume(cts.Token);
        var agentTask = consumeResult.Message.Value;
        Console.WriteLine(\$"Consumed Key: {consumeResult.Message.Key}, Task: {agentTask.Description}"); // Example property
        // ---> Process the agentTask using Semantic Kernel <---
    }
} catch (ConsumeException e) {
    Console.WriteLine(\$"Consume error: {e.Error.Reason}");
} catch (OperationCanceledException) {
    // Ensure clean shutdown
} finally {
    consumer.Close();
}
\`\`\`

### 8.2. Orleans Streams + Kafka + Schema Registry (Orleans Silo Host)

\`\`\`csharp
// --- Add Nuget Packages ---
// Microsoft.Orleans.Server
// Microsoft.Orleans.Streaming.Kafka // Ensure this package is available and compatible
// Confluent.SchemaRegistry.Serdes.Avro // Or your chosen format serializer

// --- Silo Host Configuration (Program.cs or Startup.cs) ---
// Assume \`hostBuilder\` is an IHostBuilder

hostBuilder.UseOrleans(siloBuilder => {
    siloBuilder
        // ... other Orleans config (clustering, persistence) ...
        .AddMemoryGrainStorage("PubSubStore") // Needed by stream internals
        .AddKafka(OrchestraConstants.StreamProviderName) // Use a constant for the provider name
            .WithOptions(options => { // Configure KafkaStreamProviderOptions
                options.BrokerList = "localhost:9092"; // From config
                options.ConsumerGroupId = "orleans-coordinator-group"; // Unique group ID
                options.TopicCreationTimeout = TimeSpan.FromSeconds(10); // Example

                // --- Schema Registry Integration ---
                options.SchemaRegistryUrl = "http://localhost:8081"; // From config
                // Tell the provider to use the Avro Kafka Serializer Factory
                // Ensure correct namespace and class name for your AvroSerializerFactory implementation
                options.KafkaSerdeFactoryType = typeof(Orchestra.Infrastructure.Serialization.AvroSerializerFactory).AssemblyQualifiedName;
            })
            .AddTopic(OrchestraConstants.WorkflowStartStreamNamespace, "workflow-start-events") // Map incoming topic
            .AddTopic(OrchestraConstants.AgentResultStreamNamespace, "agent-results") // Example incoming
            .AddTopic(OrchestraConstants.AgentTaskStreamNamespace, "agent-tasks", isWriteOnly: true) // Map outgoing topic
            .Build(); // Finalize Kafka provider config
});

// --- AvroSerializerFactory Implementation (Example) ---
// Create a separate infrastructure project for this
// Needs to implement IKafkaSerdeFactory or similar interface provided by Orleans.Streaming.Kafka
// The exact interface/method signatures might vary slightly between Orleans Kafka provider versions - CHECK DOCUMENTATION
using Confluent.SchemaRegistry;
using Confluent.SchemaRegistry.Serdes;
using Orleans.Serialization.Kafka; // Check correct namespace

namespace Orchestra.Infrastructure.Serialization
{
    public class AvroSerializerFactory : IKafkaSerdeFactory // Or similar interface name
    {
        private readonly ISchemaRegistryClient _schemaRegistryClient;

        public AvroSerializerFactory(IServiceProvider serviceProvider)
        {
            // Resolve Schema Registry client (assuming it's configured elsewhere or build it here)
            // This example assumes a basic CachedSchemaRegistryClient setup
            var schemaRegistryConfig = new SchemaRegistryConfig { Url = "http://localhost:8081" /* Get from config */ };
            _schemaRegistryClient = new CachedSchemaRegistryClient(schemaRegistryConfig);

            // Alternative: Resolve via IServiceProvider if registered
            // _schemaRegistryClient = serviceProvider.GetRequiredService<ISchemaRegistryClient>();
        }

        // Check exact method signature from Orleans.Streaming.Kafka documentation
        public Confluent.Kafka.ISerializer<T> BuildSerializer<T>()
        {
            // Return Schema Registry aware Avro serializer for type T
            return new AvroSerializer<T>(_schemaRegistryClient);
        }

        // Check exact method signature
        public Confluent.Kafka.IDeserializer<T> BuildDeserializer<T>()
        {
            // Return Schema Registry aware Avro deserializer for type T
            return new AvroDeserializer<T>(_schemaRegistryClient);
        }

        // Potentially other methods required by the interface...
    }
}


// --- WorkflowGrain Example ---
using Microsoft.Extensions.Logging;
using Orleans;
using Orleans.Streams;
using YourNamespace; // For Avro generated classes

namespace Orchestra.Grains
{
    // Assuming Avro classes: WorkflowStartEvent, AgentTask
    // Use constants for Stream provider name and namespaces

    public interface IWorkflowGrain : IGrainWithStringKey {} // Simple example key

    public class WorkflowGrain : Grain, IWorkflowGrain, IAsyncObserver<WorkflowStartEvent>
    {
        private readonly ILogger<WorkflowGrain> _logger;
        private IAsyncStream<AgentTask> _agentTaskStream = null!;
        private StreamSubscriptionHandle<WorkflowStartEvent>? _subscriptionHandle;
        private string _workflowState = "Initial"; // Example simple state

        public WorkflowGrain(ILogger<WorkflowGrain> logger) { _logger = logger; }

        public override async Task OnActivateAsync(CancellationToken cancellationToken)
        {
            var streamProvider = this.GetStreamProvider(OrchestraConstants.StreamProviderName);

            // Get stream for publishing agent tasks
            _agentTaskStream = streamProvider.GetStream<AgentTask>(
                StreamId.Create(OrchestraConstants.AgentTaskStreamNamespace, this.GetPrimaryKeyString()) // Example stream ID strategy
            );

            // Subscribe to incoming workflow start events for this specific workflow ID
            var inputStream = streamProvider.GetStream<WorkflowStartEvent>(
                 StreamId.Create(OrchestraConstants.WorkflowStartStreamNamespace, this.GetPrimaryKeyString()) // Match based on workflow ID
            );

             _subscriptionHandle = await inputStream.SubscribeAsync(this); // \`this\` implements IAsyncObserver<WorkflowStartEvent>

            _logger.LogInformation("WorkflowGrain {WorkflowId} activated and subscribed.", this.GetPrimaryKeyString());
            await base.OnActivateAsync(cancellationToken);
        }

        // --- IAsyncObserver<WorkflowStartEvent> Implementation ---
        public Task OnNextAsync(WorkflowStartEvent item, StreamSequenceToken? token = null)
        {
            _logger.LogInformation("WorkflowGrain {WorkflowId} received WorkflowStartEvent: {Description}",
                this.GetPrimaryKeyString(), item.Description); // Example property

            _workflowState = "Started";
            // --> Add logic here to decide which agent task to dispatch <--

            var agentTask = new AgentTask {
                 TaskId = Guid.NewGuid().ToString(),
                 WorkflowId = this.GetPrimaryKeyString(),
                 InputData = item.InitialInput, // Example data passing
                 Description = "Please summarize the input data." // Example task
            };

            _logger.LogInformation("WorkflowGrain {WorkflowId} publishing AgentTask {TaskId}",
                this.GetPrimaryKeyString(), agentTask.TaskId);

            // Publish task via the output stream
            return _agentTaskStream.OnNextAsync(agentTask);
        }

        public Task OnCompletedAsync() => Task.CompletedTask; // Handle stream completion if needed
        public Task OnErrorAsync(Exception ex) {
            _logger.LogError(ex, "Error processing input stream for Workflow {WorkflowId}", this.GetPrimaryKeyString());
            return Task.CompletedTask; // Handle errors
        }
         // Optional: Add method to handle agent results coming back via another stream subscription
    }
}
\`\`\`

### 8.3. Agent Service (Worker + SK + Kafka Client)

\`\`\`csharp
// --- Add Nuget Packages ---
// Microsoft.Extensions.Hosting
// Microsoft.SemanticKernel
// Confluent.Kafka
// Confluent.SchemaRegistry.Serdes.Avro

// --- Program.cs ---
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.SemanticKernel;
using Orchestra.AgentService; // Your namespace

HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);

// --- Configure Semantic Kernel ---
// Assuming config is in appsettings.json or environment variables
// Example for OpenAI:
builder.Services.AddOpenAIChatCompletion(
        modelId: builder.Configuration["SemanticKernel:AIService:OpenAI:ModelId"]!,
        apiKey: builder.Configuration["SemanticKernel:AIService:OpenAI:ApiKey"]!
    );
// Add other AI services as needed (Azure OpenAI, etc.)

builder.Services.AddTransient<Kernel>(); // Add Kernel itself

// --- Add Agent's Native Functions ---
// If using dependency injection within native functions
builder.Services.AddTransient<MyAgentNativeFunctions>(); // Example native function class

// --- Configure and Add Kafka Consumer Background Service ---
builder.Services.AddHostedService<AgentKafkaConsumerService>();
builder.Services.Configure<KafkaConsumerConfig>(builder.Configuration.GetSection("KafkaConsumer"));
builder.Services.Configure<SchemaRegistryConfig>(builder.Configuration.GetSection("SchemaRegistry"));

// Add Kafka Producer Factory (example pattern)
builder.Services.AddSingleton<KafkaProducerFactory>(); // Factory to create producers

using IHost host = builder.Build();
await host.RunAsync();


// --- AgentKafkaConsumerService.cs (Background Service) ---
using Confluent.Kafka;
using Confluent.SchemaRegistry;
using Confluent.SchemaRegistry.Serdes;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using Microsoft.SemanticKernel;
using YourNamespace; // Avro classes: AgentTask, AgentResult

namespace Orchestra.AgentService
{
    public class KafkaConsumerConfig {
        public string BootstrapServers { get; set; } = null!;
        public string GroupId { get; set; } = "agent-service-group";
        public string InputTopic { get; set; } = "agent-tasks"; // Make specific if needed
    }
    // SchemaRegistryConfig from Confluent.SchemaRegistry namespace

    public class AgentKafkaConsumerService : BackgroundService
    {
        private readonly ILogger<AgentKafkaConsumerService> _logger;
        private readonly KafkaConsumerConfig _consumerConfig;
        private readonly SchemaRegistryConfig _schemaRegistryConfig;
        private readonly IServiceProvider _serviceProvider; // To get Kernel instance per request
        private readonly KafkaProducerFactory _producerFactory; // To get producer for results

        public AgentKafkaConsumerService(
            ILogger<AgentKafkaConsumerService> logger,
            IOptions<KafkaConsumerConfig> consumerConfig,
            IOptions<SchemaRegistryConfig> schemaRegistryConfig,
            IServiceProvider serviceProvider,
            KafkaProducerFactory producerFactory)
        {
            _logger = logger;
            _consumerConfig = consumerConfig.Value;
            _schemaRegistryConfig = schemaRegistryConfig.Value;
            _serviceProvider = serviceProvider;
            _producerFactory = producerFactory;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            var consumerConfig = new ConsumerConfig {
                BootstrapServers = _consumerConfig.BootstrapServers,
                GroupId = _consumerConfig.GroupId,
                AutoOffsetReset = AutoOffsetReset.Earliest,
                EnableAutoCommit = false // Recommended for processing guarantees
            };

            using var schemaRegistry = new CachedSchemaRegistryClient(_schemaRegistryConfig);
            using var consumer = new ConsumerBuilder<string, AgentTask>(consumerConfig)
                .SetValueDeserializer(new AvroDeserializer<AgentTask>(schemaRegistry).AsSyncOverAsync())
                .SetErrorHandler((_, e) => _logger.LogError("Kafka Error: {Reason}", e.Reason))
                .Build();

            consumer.Subscribe(_consumerConfig.InputTopic);
            _logger.LogInformation("Agent consumer started for topic {Topic}", _consumerConfig.InputTopic);

            try {
                while (!stoppingToken.IsCancellationRequested)
                {
                    var consumeResult = consumer.Consume(TimeSpan.FromSeconds(1)); // Timeout allows cancellation check
                    if (consumeResult == null) continue;

                    var agentTask = consumeResult.Message.Value;
                    _logger.LogInformation("Received Task {TaskId} for Workflow {WorkflowId}", agentTask.TaskId, agentTask.WorkflowId);

                    try {
                        // ---> Process using Semantic Kernel <---
                        using var scope = _serviceProvider.CreateScope(); // Create scope for transient Kernel
                        var kernel = scope.ServiceProvider.GetRequiredService<Kernel>();

                        // --- Load Native Functions ---
                        // Option 1: Load specific instance if using DI
                         kernel.ImportPluginFromObject(scope.ServiceProvider.GetRequiredService<MyAgentNativeFunctions>(), "NativeTools");
                        // Option 2: Load static methods if not using DI
                        // kernel.ImportPluginFromType<MyStaticNativeFunctions>("NativeTools");


                        // --- Load Semantic Functions ---
                        // Assumes prompts are in a "Prompts" directory relative to execution
                        var promptsDir = Path.Combine(AppContext.BaseDirectory, "Prompts", "AgentFunctions");
                        var agentPlugin = kernel.ImportPluginFromPromptDirectory(promptsDir);

                        // --- Prepare Arguments ---
                        var kernelArgs = new KernelArguments {
                            ["input"] = agentTask.InputData, // Map task data to kernel args
                            ["taskId"] = agentTask.TaskId
                            // Add other args needed by functions
                        };

                        // --- Invoke Function(s) ---
                        // Example: Chaining - result of one feeds into the next might need separate calls
                        var summaryResult = await kernel.InvokeAsync<string>(agentPlugin["Summarize"], kernelArgs, stoppingToken);

                        // Use the result or run another function
                        var nativeResult = await kernel.InvokeAsync<string>("NativeTools", "MyDummyNativeFunction", kernelArgs, stoppingToken); // Assuming MyDummyNativeFunction exists

                        _logger.LogInformation("Task {TaskId} processed. Summary: {Summary}, Native: {Native}",
                            agentTask.TaskId, summaryResult, nativeResult);

                        // ---> Produce Result <---
                        var agentResult = new AgentResult {
                            TaskId = agentTask.TaskId,
                            WorkflowId = agentTask.WorkflowId,
                            ResultData = \$"Summary: {summaryResult} | Native: {nativeResult}", // Combine results
                            Status = "Success"
                        };
                        await ProduceResultAsync(agentResult, stoppingToken);

                        consumer.Commit(consumeResult); // Commit offset after successful processing
                    } catch (Exception ex) {
                        _logger.LogError(ex, "Error processing Task {TaskId}", agentTask.TaskId);
                        // --> Implement error handling / DLQ logic here <--
                        // Do not commit if processing failed and retry is desired (depends on strategy)
                    }
                }
            } catch (OperationCanceledException) {
                _logger.LogInformation("Agent consumer stopping.");
            } catch (Exception ex) {
                _logger.LogError(ex, "Unhandled exception in agent consumer loop.");
            } finally {
                consumer.Close();
            }
        }

        private async Task ProduceResultAsync(AgentResult result, CancellationToken cancellationToken)
        {
             // Use the factory to get a producer instance
            using var producer = _producerFactory.GetProducer<string, AgentResult>();
            var message = new Message<string, AgentResult> { Key = result.WorkflowId, Value = result };
            var resultTopic = "agent-results"; // Get from config

            try {
                await producer.ProduceAsync(resultTopic, message, cancellationToken);
                 _logger.LogInformation("Published result for Task {TaskId}", result.TaskId);
            } catch (ProduceException<string, AgentResult> e) {
                _logger.LogError(e, "Failed to produce result for Task {TaskId}", result.TaskId);
                 // Handle production failure
            }
        }
    }

     // --- KafkaProducerFactory.cs (Example) ---
    public class KafkaProducerFactory : IDisposable {
        private readonly IOptions<ProducerConfig> _producerConfig;
        private readonly IOptions<SchemaRegistryConfig> _schemaRegistryConfig;
        private readonly ISchemaRegistryClient _schemaRegistryClient; // Share client

        // Store producers if needed, or create new ones
        // Careful with lifecycle management if storing

        public KafkaProducerFactory(IOptions<ProducerConfig> producerConfig, IOptions<SchemaRegistryConfig> schemaRegistryConfig)
        {
            _producerConfig = producerConfig; // Inject basic Kafka config
            _schemaRegistryConfig = schemaRegistryConfig; // Inject SR config
            _schemaRegistryClient = new CachedSchemaRegistryClient(_schemaRegistryConfig.Value);
        }

        public IProducer<TKey, TValue> GetProducer<TKey, TValue>()
        {
            // Assumes Avro for this example
             return new ProducerBuilder<TKey, TValue>(_producerConfig.Value)
                .SetKeySerializer(GetKeySerializer<TKey>()) // Helper to get appropriate key serializer
                .SetValueSerializer(new AvroSerializer<TValue>(_schemaRegistryClient))
                .Build();
        }

         private ISerializer<TKey> GetKeySerializer<TKey>() {
            if (typeof(TKey) == typeof(string)) return (ISerializer<TKey>)Serializers.Utf8;
            if (typeof(TKey) == typeof(Null)) return (ISerializer<TKey>)Serializers.Null;
            // Add other key types if needed
             throw new NotSupportedException(\$"Key type {typeof(TKey)} not supported.");
         }

        public void Dispose() {
            _schemaRegistryClient?.Dispose();
            // Dispose stored producers if any
        }
    }

     // --- MyAgentNativeFunctions.cs (Example) ---
    public class MyAgentNativeFunctions {
        private readonly ILogger<MyAgentNativeFunctions> _logger;
        // Inject other services if needed

        public MyAgentNativeFunctions(ILogger<MyAgentNativeFunctions> logger) { _logger = logger; }

        [KernelFunction]
        [Description("A dummy native function that returns a fixed string.")]
        public Task<string> MyDummyNativeFunction(
            [Description("The ID of the current task")] string taskId
        ) {
            _logger.LogInformation("Executing MyDummyNativeFunction for task {TaskId}", taskId);
            return Task.FromResult(\$"Native result for {taskId} at {DateTime.UtcNow}");
        }
    }
}
\`\`\`

## 9. Setup & Tooling

*   **.NET SDK:** Ensure version 8+ is installed.
*   **IDE:** Visual Studio 2022, VS Code with C# Dev Kit, or JetBrains Rider.
*   **Docker Desktop:** Required for running local infrastructure via Docker Compose.
*   **Git:** For version control.
*   **(Optional) Avro/Protobuf Tools:** Tools like `avrogen` (for Avro C# class generation) or `protoc` (for Protobuf) can be helpful.
*   **(Optional) Kafka UI Tool:** A tool like Conduktor (paid) or Offset Explorer / kcat (free) to inspect Kafka topics.

## 10. Testing Strategy (MVP Focus)

*   **Unit Tests:** Test individual components in isolation (e.g., Grain logic without streams, SK function logic, utility functions). Use mocking frameworks (e.g., Moq).
*   **Integration Tests:**
    *   Test Kafka produce/consume with Schema Registry integration (can use `Confluent.Kafka` mock clusters or Testcontainers).
    *   Test basic Orleans Grain activation and method calls (using `TestCluster`).
    *   *Testing Orleans Streams + Kafka integration is complex - may rely more on manual E2E for MVP.*
*   **Manual End-to-End:** Trigger the API and verify logs across all services to confirm the flow works as expected.

## 11. Documentation Strategy

*   **README.md:** The primary entry point (as drafted above). Keep it updated.
*   **Architecture Diagram:** Maintain a clear visual representation of the components and flow.
*   **Setup Guide:** Detailed steps for local development setup.
*   **Code Comments:** Explain non-obvious logic within the code.
*   **(Future) API Documentation:** Use Swagger/OpenAPI for any Web APIs.
*   **(Future) Usage Examples:** More detailed examples beyond the basic MVP flow.

## 12. Risks & Mitigation

*   **Complexity:** Integrating Kafka + Orleans Streams + Schema Registry + SK is inherently complex.
    *   *Mitigation:* Start with the simplest possible MVP flow. Tackle integration points systematically. Use pair programming. Rely on the Lead's experience.
*   **Learning Curve:** Team members may be new to Orleans, SK, or advanced Kafka concepts.
    *   *Mitigation:* Allocate time for learning. Use official documentation. Start with simple use cases. Share knowledge within the team.
*   **LLM Dependency:** Cost, rate limits, and reliability of the chosen LLM provider.
    *   *Mitigation:* Use LLMs sparingly in the MVP. Implement basic error handling around SK calls. Be mindful of costs.
*   **Schema Management:** Incorrect schema evolution can break consumers.
    *   *Mitigation:* Use Avro/Protobuf with clear compatibility rules. Test schema changes carefully.
*   **Scope Creep:** Temptation to add more features beyond the MVP.
    *   *Mitigation:* Strictly adhere to the defined MVP scope. Defer non-essential features to later versions.

## 13. Next Steps

1.  **Team Kickoff Meeting:** Review this plan, assign initial roles, clarify questions.
2.  **Setup Repository:** Create Git repository, establish branching strategy (e.g., Gitflow).
3.  **Setup Task Board:** Create tasks based on Phase 1 & 2.
4.  **Begin Phase 1:** Start working on infrastructure setup (Docker Compose) and basic project structure.
5.  **Schedule Regular Syncs:** Daily standups, weekly planning/review.

---
