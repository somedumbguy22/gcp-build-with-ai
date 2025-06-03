# Google's Agent Stack in Action:ADK, A2A, MCP on Google Cloud

**agent in 4 parts:**

- llm/model - as the brain
- input - use queries
- state - stores both short term and long term knowledge. Short term is like recent interactions and long term is user profiles, grounding knowledge
- tools - provides the agent to ability to interact with the world and external systems

**Levels of abstraction:**

- DIY APIs
- low-level LLM framrworks like LangChain
  - this abstracts the model choice from the prompt from other capabilities like RAG
- Low-level orchestration framework like LangGraph - this now supports tools like local functions, etc.
- Agent framework like the ADK - encompasses all of the previous layers

**ADK Framework**:
Models -> Base Agent (model, instructions, tools) -> out of the box tools + custom functions -> Agent

- out of the box tools are google search and RAG with Vertex AI Search. More coming in the future
- Input: video, databases, files, signals, audio, cache, vector embeddings
- outputs: API, gRPC, HTTP, AMPQ

**ADK - Runner**

- provides state, session config, etc.

```python
runner = Runner(
  app_name='planner_app',
  agent=root_agent,
  session_service=session_service
)

runner.run_async(
  session_id = session.id,
  ...
)
```

**ADK Memories**

- These are held in sessions, consist of state and events and artifacts
- There's also long term memory for context in future sessions (vertex AI storage)
- Events are "like conversation history that you can store"
- Events consist of history of the conversation, so messages and tool calls and state changes
- State is a dictionary (key.value pair)
- Artifacts store binary data for non-text based input (like video, audio)

**Architecture Patterns**

- Single - one llm for all tasks
- Sequential - chains multiple specialized LLMs in a pipeline
- Parallel - multiple LLMs concurrently and aggregate outputs
- Hierarchical - router model to direct inputs to specialized llms based on context or intent
- Loop Agent - goes around until a condition is met in a multiagent system (workflow agent)
- Cooperative Multi-agent system
- Competitive Multi-agent system

```python
loop_agent = LoopAgent(
  name="asdad",
  sub_agents=[
    agent_a,
    agent_b,
    CheckCondition(name="check")
  ],
)
```

**Callbacks**

- manage state read or dynamically update the agents session state
- implement guardrails enforce saftey rules
- control modify data flowing through the agent or bypass certain steps
- caching temprory data store
- observe and debug log detailed information at ciritcal steps for monitoring
- notifications tracing human in the loop

```python
root_agent = Agfent(
  name="",
  model="",
  description="",
  instruction="",
  callback=""
)
```

**Evaluations**
Design -> Develop -> Evaluate -> Deploy

- There will be a "black box test" for with a list of questions and expected answers.
- These control answers will be evaluated against model output and a judge will emit a score
- You will want to test for apporpriate tool calls based on context

**Runtime**

- Agent Engine on Vertex AI
- Comes with tracing and metrics by default
- Basically just a container running on cloud run, with out of the box telemetry included

**MCP**

- MCP is a way to secure tools
- Before MCP, we used to write the functions or tools in the same module as the agent or LLM.
- Now we can add tools via an MCP server, either locally or remotely, giving access to existing APIs, vendor tools, etc.
- exposes "list tools" and "call tools" methods for the agent (client) to call

**A2A**

- A protocol to allow Agents to easily communicate with each other
- Agents could be using different models, deployed in different DCs or servers, etc.
- This allows you to build an "Agent mesh"
- Agents come with an "Agent card" that includes a name, a provider (URL/address) capabilities(streaming, notifications, etc.), skills, and an authentication scheme
- Skills are a description with an input and output scheme (say tools)
- A2A servers exist to help with Agent discovery
  - the server exposes a "Get agent cards" to let a caller know what agents are available and what they can do, the number of agents, etc.
  - it also comes with a task registry that agents can pick up
  - The server is called from a workflow agent or orchestration agent
  - The A2A protocol is mostly about the communication schema to ensure standardization. It's not an implementation! Just a schema
  - Agent engine doesn't support acting as an A2A server as of right now. So far, only can deploy a server as a container

Definitions from the Lab:
Google Cloud Platform (GCP):

- Vertex AI:
  - Gemini Models: Provides access to Google's state-of-the-art Large Language Models (LLMs) like Gemini, which power the reasoning and decision-making capabilities of our agents.
  - Vertex AI Agent Engine: A managed service used to deploy, host, and scale our orchestrator agent, simplifying productionization and abstracting infrastructure complexities.
- Cloud Run: A serverless platform for deploying containerized applications. We use it to:
  - Host the main InstaVibe web application.
  - Deploy individual A2A-enabled agents (Planner, Social Profiling, Platform Interaction) as independent microservices.
  - Run the MCP Tool Server, making InstaVibe's internal APIs available to agents.
- Spanner: A fully managed, globally distributed, and strongly consistent relational database. In this workshop, we leverage its capabilities as a Graph Database using its GRAPH DDL and query features to:
  - Model and store complex social relationships (users, friendships, event attendance, posts).
  - Enable efficient querying of these relationships for the Social Profiling agents.
- Artifact Registry: A fully managed service for storing, managing, and securing container images.
- Cloud Build: A service that executes your builds on Google Cloud. We use it to automatically build Docker container images from our agent and application source code.
- Cloud Storage: Used by services like Cloud Build for storing build artifacts and by Agent Engine for its operational needs.
- Core Agent Frameworks & Protocols:
  - Google's Agent Development Kit (ADK): The primary framework for:
    - Defining the core logic, behavior, and instruction sets for individual intelligent agents.
    - Managing agent lifecycles, state, and memory (short-term session state and potentially long-term knowledge).
    - Integrating tools (like Google Search or custom-built tools) that agents can use to interact with the world.
    - Orchestrating multi-agent workflows, including sequential, loop, and parallel execution of sub-agents.
  - Agent-to-Agent (A2A) Communication Protocol: An open standard enabling:
    - Direct, standardized communication and collaboration between different AI agents, even if they are running as separate services or on different machines.
    - Agents to discover each other's capabilities (via Agent Cards) and delegate tasks. This is crucial for our Orchestrator agent to interact with the specialized Planner, Social, and Platform agents.
  - Model Context Protocol (MCP): An open standard that allows agents to:
    - Connect with and utilize external tools, data sources, and systems in a standardized way.
    - Our Platform Interaction Agent uses an MCP client to communicate with an MCP server, which in turn exposes tools to interact with the InstaVibe platform's existing APIs.
- Language Models (LLMs): The "Brains" of the System:
  - Google's Gemini Models: Specifically, we utilize versions like gemini-2.0-flash. These models are chosen for:
    - Advanced Reasoning & Instruction Following: Their ability to understand complex prompts, follow detailed instructions, and reason about tasks makes them suitable for powering agent decision-making.
    - Tool Use (Function Calling): Gemini models excel at determining when and how to use the tools provided via ADK, enabling agents to gather information or perform actions.
    - Efficiency (Flash Models): The "flash" variants offer a good balance of performance and cost-effectiveness, suitable for many interactive agent tasks that require quick responses.

What we'll build in the lab

- **Foundations with Google's ADK**: Master the fundamentals of building your first intelligent agent using Google's Agent Development Kit (ADK). Understand the essential components, the agent lifecycle, and how to leverage the framework's built-in tools effectively.
- **Extending Agent Capabilities with Model Context Protocol (MCP)**: Learn to equip your agents with custom tools and context, enabling them to perform specialized tasks and access specific information. Introduce the Model Context Protocol (MCP) concept. You'll learn how to set up an MCP server to provide this context.
- **Designing Agent Interactions & Orchestration**: Move beyond single agents to understand agent orchestration. Design interaction patterns ranging from simple sequential workflows to complex scenarios involving loops, conditional logic, and parallel processing. Introduce the concept of sub-agents within the ADK framework to manage modular tasks.
- **Building Collaborative Multi-Agent Systems**: Discover how to architect systems where multiple agents collaborate to achieve complex goals. Learn and implement the Agent-to-Agent (A2A) communication protocol, establishing a standardized way for distributed agents (potentially running on different machines or services) to interact reliably.
- **Productionizing Agents on Google Cloud**: Transition your agent applications from development environments to the cloud. Learn best practices for architecting and deploying scalable, robust multi-agent systems on Google Cloud Platform (GCP). Gain insights into leveraging GCP services like Cloud Run and explore the capabilities of the latest Google Agent Engine for hosting and managing your agents.

Agents to build:

- Social profiling Agent: This agent employs social listening techniques to analyze user connections, interactions and potentially broader public trends related to the user's preferences. Its purpose is to identify shared interests and suitable activity characteristics (e.g., preferences for quieter gatherings, specific hobbies).
- Event Planning Agent: Using the insights from the Social profiling Agent, this agent searches online resources for specific events, venues, or ideas that align with the identified criteria (such as location, interests).
- Platform Interaction Agent (using MCP): This agent takes the finalized plan from the Activity Planning Agent. Its key function is to interact directly with the InstaVibe platform by utilizing a pre-defined MCP (Model Context Protocol) tool. This tool provides the agent with the specific capability to draft an event suggestion and create a post outlining the plan.
- Orchestrator Agent: This agent acts as the central coordinator. It receives the initial user request from the InstaVibe platform, understands the overall goal (e.g., "plan a night out for me and my friends"), and then delegates specific tasks to the appropriate specialized agents in a logical sequence. It manages the flow of information between agents and ensures the final result is delivered back to the user.

Some startup scripts:

Enable APIs on GCP

```shell
gcloud services enable  run.googleapis.com \
                        cloudfunctions.googleapis.com \
                        cloudbuild.googleapis.com \
                        artifactregistry.googleapis.com \
                        spanner.googleapis.com \
                        apikeys.googleapis.com \
                        iam.googleapis.com \
                        compute.googleapis.com \
                        aiplatform.googleapis.com \
                        cloudresourcemanager.googleapis.com \
                        maps-backend.googleapis.com
```

Set env variables

```shell
export PROJECT_ID=$(gcloud config get project)
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format="value(projectNumber)")
export SERVICE_ACCOUNT_NAME=$(gcloud compute project-info describe --format="value(defaultServiceAccount)")
export SPANNER_INSTANCE_ID="instavibe-graph-instance"
export SPANNER_DATABASE_ID="graphdb"
export GOOGLE_CLOUD_PROJECT=$(gcloud config get project)
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
export GOOGLE_CLOUD_LOCATION="us-central1"
```

Binding IAM permissions to a service account

```shell
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/spanner.admin"

# Spanner Database User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/spanner.databaseUser"

# Artifact Registry Admin
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/artifactregistry.admin"

# Cloud Build Editor
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/cloudbuild.builds.editor"

# Cloud Run Admin
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/run.admin"

# IAM Service Account User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/iam.serviceAccountUser"

# Vertex AI User
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/aiplatform.user"

# Logging Writer (to allow writing logs)
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/logging.logWriter"


gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT_NAME" \
  --role="roles/logging.viewer"
```

Setting up an Artifact Registry

```shell
export REPO_NAME="introveally-repo"
gcloud artifacts repositories create $REPO_NAME \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker repository for InstaVibe workshop"
```

<setup an MAPS API key>

Create Spanner (Graph DB) Instance

```shell
. ./set_env.sh

gcloud spanner instances create $SPANNER_INSTANCE_ID \
  --config=regional-us-central1 \
  --description="GraphDB Instance InstaVibe" \
  --processing-units=100 \
  --edition=ENTERPRISE

gcloud spanner databases create $SPANNER_DATABASE_ID \
  --instance=$SPANNER_INSTANCE_ID \
  --database-dialect=GOOGLE_STANDARD_SQL
```

IAM bindings for Spanner

```shell
echo "Granting Spanner read/write access to ${SERVICE_ACCOUNT_NAME} for database ${SPANNER_DATABASE_ID}..."

gcloud spanner databases add-iam-policy-binding ${SPANNER_DATABASE_ID} \
  --instance=${SPANNER_INSTANCE_ID} \
  --member="serviceAccount:${SERVICE_ACCOUNT_NAME}" \
  --role="roles/spanner.databaseUser" \
  --project=${PROJECT_ID}
```

Setup Python Virtual Env:

```shell
python -m venv env
source env/bin/activate
pip install -r requirements.txt
cd instavibe
python setup.py
```

Setting up GMaps API

```shell
./instavibe-bootstrap/set_env.sh
export KEY_DISPLAY_NAME="Maps Platform API Key"

GOOGLE_MAPS_KEY_ID=$(gcloud services api-keys list \
  --project="${PROJECT_ID}" \
  --filter="displayName='${KEY_DISPLAY_NAME}'" \
  --format="value(uid)" \
  --limit=1)

GOOGLE_MAPS_API_KEY=$(gcloud services api-keys get-key-string "${GOOGLE_MAPS_KEY_ID}" \
    --project="${PROJECT_ID}" \
    --format="value(keyString)")

echo "${GOOGLE_MAPS_API_KEY}" > ~/mapkey.txt

echo "Retrieved GOOGLE_MAPS_API_KEY: ${GOOGLE_MAPS_API_KEY}"
```

push to artifact registry

```shell
. ~/instavibe-bootstrap/set_env.sh

cd ~/instavibe-bootstrap/instavibe/
export IMAGE_TAG="latest"
export APP_FOLDER_NAME="instavibe"
export IMAGE_NAME="instavibe-webapp"
export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
export SERVICE_NAME="instavibe"

gcloud builds submit . \
  --tag=${IMAGE_PATH} \
  --project=${PROJECT_ID}
```

Deploy to Cloud Run

```shell
. ~/instavibe-bootstrap/set_env.sh

cd ~/instavibe-bootstrap/instavibe/
export IMAGE_TAG="latest"
export APP_FOLDER_NAME="instavibe"
export IMAGE_NAME="instavibe-webapp"
export IMAGE_PATH="${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
export SERVICE_NAME="instavibe"

gcloud run deploy ${SERVICE_NAME} \
  --image=${IMAGE_PATH} \
  --platform=managed \
  --region=${REGION} \
  --allow-unauthenticated \
  --set-env-vars="SPANNER_INSTANCE_ID=${SPANNER_INSTANCE_ID}" \
  --set-env-vars="SPANNER_DATABASE_ID=${SPANNER_DATABASE_ID}" \
  --set-env-vars="APP_HOST=0.0.0.0" \
  --set-env-vars="APP_PORT=8080" \
  --set-env-vars="GOOGLE_CLOUD_LOCATION=${REGION}" \
  --set-env-vars="GOOGLE_CLOUD_PROJECT=${PROJECT_ID}" \
  --set-env-vars="GOOGLE_MAPS_API_KEY=${GOOGLE_MAPS_API_KEY}" \
  --project=${PROJECT_ID} \
  --min-instances=1
```

Run the agent through the adk

```shell
. ~/instavibe-bootstrap/set_env.sh
source ~/instavibe-bootstrap/env/bin/activate
cd  ~/instavibe-bootstrap/agents
sed -i "s|^\(O\?GOOGLE_CLOUD_PROJECT\)=.*|GOOGLE_CLOUD_PROJECT=${PROJECT_ID}|" ~/instavibe-bootstrap/agents/planner/.env
adk web
```

ADK Components
While the ADK Dev UI is great for interactive testing, we often need to run our agents programmatically, perhaps as part of a larger application or backend service. To understand how this works, let's look at some core ADK concepts related to runtime and context management.

Meaningful, multi-turn conversations require agents to understand context – recalling what's been said and done to maintain continuity. ADK provides structured ways to manage this context through Session, State, and Memory:

Session: When a user starts interacting with an agent, a Session is created. Think of it as the container for a single, specific chat thread. It holds a unique ID, the history of interactions (Events), the current working data (State), and metadata like the last update time.
State: This is the agent's short-term, working memory within a single Session. It's a mutable dictionary where the agent can store temporary information needed to complete the current task (e.g., user preferences collected so far, intermediate results from tool calls).
Memory: This represents the agent's potential for long-term recall across different sessions or access to external knowledge bases. While Session and State handle the immediate conversation, Memory (often managed by a MemoryService) allows an agent to retrieve information from past interactions or structured data sources, giving it a broader knowledge context. (Note: Our simple client uses in memory services for simplicity, meaning memory/state only persists while the script runs).
Event: Every interaction within a Session (user message, agent response, tool use request, tool result, state change, error) is recorded as an immutable Event. This creates a chronological log, essentially the transcript and action history of the conversation.
So, how are these managed when an agent runs? That's the job of the Runner.

Runner: The Runner is the core execution engine provided by ADK. You define your agent and the tools it uses, and the Runner orchestrates the process of fulfilling a user's request. It manages the Session, handles the flow of Events, updates the State, invokes the underlying language model, coordinates tool calls, and potentially interacts with MemoryService. Think of it as the conductor making sure all the different parts work together correctly.
We can use the Runner to run our agent as a standalone Python application, completely independent of the Dev UI.

Model Context Protocol (MCP)
The Model Context Protocol (MCP) is an open standard designed to standardize how AI applications like agents, connect with external data sources, tools, and systems. It aims to solve the problem of needing custom integrations for every AI application and data source combination by providing a universal interface. MCP utilizes a client-server architecture where MCP clients, residing within AI applications (hosts), manage connections to MCP servers. These servers are external programs that expose specific functionalities like accessing local data, interacting with remote services via APIs, or providing predefined prompts, allowing AI models to access current information and perform tasks beyond their initial training. This structure enables AI models to discover and interact with external capabilities in a standardized way, making integrations simpler and more scalable.

MCP Server Implementation

Now that we have the Python functions that perform the actions (calling the InstaVibe APIs), we need to build the MCP Server component. This server will expose these functions as "tools" according to the MCP standard, allowing MCP clients (like our agents) to discover and invoke them.

An MCP Server typically implements two key functionalities:

list_tools: responsible for allowing the client to discover the available tools on the server, providing metadata like their names, descriptions, and required parameters, often defined using JSON Schema
call_tool: handles the execution of a specific tool requested by the client, receiving the tool's name and arguments and performing the corresponding action, such as in our case interacting with an API
MCP servers are used to provide AI models with access to real-world data and actions, enabling tasks like sending emails, creating tasks in project management systems, searching databases, or interacting with various software and web services. While initial implementations often focused on local servers communicating via standard input/output (stdio) for simplicity, particularly in development or "studio" environments, the move towards remote servers utilizing protocols like HTTP with Server-Sent Events (SSE) makes more sense for broader adoption and enterprise use cases.

The remote architecture, despite the added network communication layer, offers significant advantages: it allows multiple AI clients to share access to a single server, centralizes management and updates of tools, enhances security by keeping sensitive data and API keys on the server side rather than distributed across potentially many client machines, and decouples the AI model from the specifics of the external system integration, making the entire ecosystem more scalable, secure, and maintainable than requiring every AI instance to manage its own direct integrations

MCP Client The MCP Client is a component that resides within an AI application or agent, acting as the interface between the AI model and one or more MCP Servers; in our implementation, this client will be integrated directly within our agent. This client's primary function is to communicate with MCP Servers to discover available tools via the list_tools function and subsequently request the execution of specific tools using the call_tool function, passing necessary arguments provided by the AI model or the agent orchestrating the call.

In Google's ADK, a Workflow Agent doesn't perform tasks itself but orchestrates other agents, called sub-agents. This allows for modular design, breaking down complex problems into specialized components. ADK provides built-in workflow types like

Sequential (step-by-step)
Parallel (concurrent execution)
and Loop (repeated execution)

State and Callbacks for Loop Results

In Google's ADK, State is a crucial concept representing the memory or working data of an agent during its execution. It's essentially a persistent context that holds information an agent needs to maintain across different steps, tool calls, or interactions. This state can store intermediate results, user information, parameters for subsequent actions, or any other data the agent needs to remember as it progresses through a task.

Callbacks in ADK allow us to inject custom logic to be executed at specific points during an agent's lifecycle or in response to certain events, such as the completion of a tool call or before the agent finishes its execution. They provide a way to customize the agent's behavior and process results dynamically.

The A2A protocol is an open standard specifically designed for interoperable communication between AI agents. While MCP focuses on agent-to-tool interaction, A2A focuses on agent-to-agent interaction. It allows agents to:

Discover: Find other agents and learn their capabilities via standardized Agent Cards.
Communicate: Exchange messages and data securely.
Collaborate: Delegate tasks and coordinate actions to achieve complex goals.
The A2A protocol facilitates this communication through mechanisms like "Agent Cards," which agents can use to advertise their capabilities and connection information.
A2A utilizes familiar web standards (HTTP, SSE, JSON-RPC) and often employs a client-server model where one agent (client) sends tasks to another (remote agent/server). This standardization is key to building modular, scalable systems where agents developed independently can work together.
