# Google's Agent Stack in Action:ADK, A2A, MCP on Google Cloud

**Foundations with Google's ADK**: Master the fundamentals of building your first intelligent agent using Google's Agent Development Kit (ADK). Understand the essential components, the agent lifecycle, and how to leverage the framework's built-in tools effectively.

**Extending Agent Capabilities with Model Context Protocol (MCP)**: Learn to equip your agents with custom tools and context, enabling them to perform specialized tasks and access specific information. Introduce the Model Context Protocol (MCP) concept. You'll learn how to set up an MCP server to provide this context.

**Designing Agent Interactions & Orchestration**: Move beyond single agents to understand agent orchestration. Design interaction patterns ranging from simple sequential workflows to complex scenarios involving loops, conditional logic, and parallel processing. Introduce the concept of sub-agents within the ADK framework to manage modular tasks.

**Building Collaborative Multi-Agent Systems**: Discover how to architect systems where multiple agents collaborate to achieve complex goals. Learn and implement the Agent-to-Agent (A2A) communication protocol, establishing a standardized way for distributed agents (potentially running on different machines or services) to interact reliably.

**Productionizing Agents on Google Cloud**: Transition your agent applications from development environments to the cloud. Learn best practices for architecting and deploying scalable, robust multi-agent systems on Google Cloud Platform (GCP). Gain insights into leveraging GCP services like Cloud Run and explore the capabilities of the latest Google Agent Engine for hosting and managing your agents.

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
