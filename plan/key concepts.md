AGENTIC AI
│
├── 1. CORE AGENT ARCHITECTURE
│   ├── 1.1 Agent Loop (Perception → Reasoning → Action → Observation)
│   │   ├── Input Processing
│   │   ├── Decision Making
│   │   ├── Action Execution
│   │   ├── Observation / Feedback Capture
│   │   └── Loop Termination Conditions
│   │
│   ├── 1.2 Agent Types
│   │   ├── Simple Reflex Agents
│   │   ├── Model-Based Agents
│   │   ├── Goal-Based Agents
│   │   ├── Utility-Based Agents
│   │   ├── Learning Agents
│   │   └── LLM-Based Autonomous Agents
│   │
│   ├── 1.3 Cognitive Architectures
│   │   ├── ReAct (Reasoning + Acting)
│   │   ├── RAISE (Reasoning, Acting, Interacting, Self-improving, Evolving)
│   │   ├── Cognitive Architectures for Language Agents (CoALA)
│   │   ├── LATS (Language Agent Tree Search)
│   │   └── Plan-and-Execute Architecture
│   │
│   └── 1.4 Agent Components
│       ├── LLM (Brain / Core Engine)
│       ├── Prompt / System Instructions
│       ├── Tools / Actions
│       ├── Memory Systems
│       ├── Planning Module
│       └── Output Parsers
│
├── 2. REASONING & THINKING
│   ├── 2.1 Prompting Strategies
│   │   ├── Chain-of-Thought (CoT)
│   │   ├── Zero-Shot CoT
│   │   ├── Few-Shot CoT
│   │   ├── Tree-of-Thought (ToT)
│   │   ├── Graph-of-Thought (GoT)
│   │   ├── Self-Consistency
│   │   └── Step-Back Prompting
│   │
│   ├── 2.2 Reflection & Self-Critique
│   │   ├── Self-Reflection
│   │   ├── Reflexion (iterative refinement with memory)
│   │   ├── Self-Evaluation / Scoring
│   │   ├── Critic Agents
│   │   ├── Constitutional AI Principles
│   │   └── Verification Chains
│   │
│   ├── 2.3 Inner Monologue
│   │   ├── Scratchpad Reasoning
│   │   ├── Thinking Tokens / Hidden Reasoning
│   │   └── Verbal Reasoning Traces
│   │
│   └── 2.4 Structured Reasoning
│       ├── Logical Reasoning
│       ├── Analogical Reasoning
│       ├── Causal Reasoning
│       ├── Abductive Reasoning
│       └── Commonsense Reasoning
│
├── 3. PLANNING
│   ├── 3.1 Task Decomposition
│   │   ├── Top-Down Decomposition
│   │   ├── Bottom-Up Composition
│   │   ├── Recursive Decomposition
│   │   ├── Subtask Identification
│   │   └── Dependency Graph Construction
│   │
│   ├── 3.2 Planning Strategies
│   │   ├── Sequential Planning
│   │   ├── Parallel Planning
│   │   ├── Hierarchical Task Networks (HTN)
│   │   ├── Goal-Oriented Action Planning (GOAP)
│   │   ├── Plan-and-Solve
│   │   └── Least-to-Most Prompting
│   │
│   ├── 3.3 Adaptive Planning
│   │   ├── Re-Planning on Failure
│   │   ├── Dynamic Plan Adjustment
│   │   ├── Contingency Planning
│   │   └── Backtracking
│   │
│   ├── 3.4 Search & Exploration
│   │   ├── Breadth-First Search
│   │   ├── Depth-First Search
│   │   ├── Monte Carlo Tree Search (MCTS)
│   │   ├── Beam Search
│   │   └── Best-First Search
│   │
│   └── 3.5 Plan Representation
│       ├── Natural Language Plans
│       ├── Structured Plans (JSON/YAML)
│       ├── DAGs (Directed Acyclic Graphs)
│       └── State Machines
│
├── 4. MEMORY
│   ├── 4.1 Short-Term / Working Memory
│   │   ├── Context Window Management
│   │   ├── Conversation History
│   │   ├── Scratchpad / Notepad
│   │   ├── Sliding Window Strategies
│   │   └── Summarization-Based Compression
│   │
│   ├── 4.2 Long-Term Memory
│   │   ├── Episodic Memory (past experiences / interactions)
│   │   ├── Semantic Memory (facts / knowledge)
│   │   ├── Procedural Memory (how-to / skills)
│   │   └── Declarative Memory (explicit knowledge)
│   │
│   ├── 4.3 Memory Storage & Retrieval
│   │   ├── Vector Databases (Pinecone, Weaviate, Chroma, Qdrant)
│   │   ├── Embedding-Based Retrieval
│   │   ├── Keyword / BM25 Retrieval
│   │   ├── Hybrid Search (vector + keyword)
│   │   ├── Knowledge Graphs
│   │   ├── SQL / Relational Storage
│   │   └── Key-Value Stores
│   │
│   ├── 4.4 Memory Operations
│   │   ├── Storage / Write
│   │   ├── Retrieval / Read
│   │   ├── Update / Modify
│   │   ├── Deletion / Forgetting
│   │   ├── Consolidation / Summarization
│   │   └── Relevance Ranking
│   │
│   └── 4.5 RAG (Retrieval-Augmented Generation)
│       ├── Naive RAG
│       ├── Advanced RAG
│       │   ├── Query Transformation
│       │   ├── Re-Ranking
│       │   ├── Contextual Compression
│       │   └── Multi-Query Retrieval
│       ├── Modular RAG
│       ├── Agentic RAG (agent decides when/what to retrieve)
│       ├── Graph RAG
│       ├── Chunking Strategies
│       │   ├── Fixed-Size Chunking
│       │   ├── Semantic Chunking
│       │   ├── Recursive Chunking
│       │   └── Document-Aware Chunking
│       └── Indexing Strategies
│           ├── Flat Index
│           ├── Hierarchical Index
│           ├── Parent-Child Index
│           └── Multi-Vector Index
│
├── 5. TOOL USE & FUNCTION CALLING
│   ├── 5.1 Tool Types
│   │   ├── APIs (REST, GraphQL)
│   │   ├── Code Execution (Python, JavaScript)
│   │   ├── Database Queries (SQL, NoSQL)
│   │   ├── Web Search / Browsing
│   │   ├── File System Operations
│   │   ├── Calculator / Math Tools
│   │   ├── Image Generation / Analysis
│   │   └── Custom Domain Tools
│   │
│   ├── 5.2 Function Calling Mechanisms
│   │   ├── Native Function Calling (OpenAI, Anthropic)
│   │   ├── Tool Descriptions / Schemas (JSON Schema)
│   │   ├── Tool Selection / Routing
│   │   ├── Parallel Tool Calls
│   │   ├── Sequential Tool Chains
│   │   └── Nested Tool Calls
│   │
│   ├── 5.3 Tool Management
│   │   ├── Tool Registration / Discovery
│   │   ├── Tool Documentation / Descriptions
│   │   ├── Tool Versioning
│   │   ├── Dynamic Tool Loading
│   │   └── Tool Permissions / Access Control
│   │
│   ├── 5.4 Code Generation & Execution
│   │   ├── Code Interpreters
│   │   ├── Sandboxed Execution Environments
│   │   ├── REPL-Based Interaction
│   │   └── Code Validation / Testing
│   │
│   └── 5.5 MCP (Model Context Protocol)
│       ├── MCP Servers
│       ├── MCP Clients
│       ├── Resource Exposure
│       ├── Tool Exposure
│       └── Prompt Templates
│
├── 6. MULTI-AGENT SYSTEMS
│   ├── 6.1 Communication Patterns
│   │   ├── Direct Messaging
│   │   ├── Broadcast / Pub-Sub
│   │   ├── Shared Blackboard
│   │   ├── Structured Debate
│   │   └── Negotiation Protocols
│   │
│   ├── 6.2 Orchestration Patterns
│   │   ├── Supervisor / Manager Pattern
│   │   ├── Hierarchical (Layered Management)
│   │   ├── Sequential Pipeline (Chain)
│   │   ├── Parallel Execution
│   │   ├── Round-Robin
│   │   ├── Voting / Consensus
│   │   ├── Market-Based / Auction
│   │   └── Swarm Intelligence
│   │
│   ├── 6.3 Agent Roles
│   │   ├── Planner Agent
│   │   ├── Executor Agent
│   │   ├── Critic / Reviewer Agent
│   │   ├── Researcher Agent
│   │   ├── Coder Agent
│   │   ├── Summarizer Agent
│   │   ├── Router / Dispatcher Agent
│   │   └── Domain Expert Agents
│   │
│   ├── 6.4 Collaboration Strategies
│   │   ├── Cooperative (shared goal)
│   │   ├── Competitive (adversarial)
│   │   ├── Mixed (cooperative-competitive)
│   │   ├── Delegation
│   │   ├── Task Allocation
│   │   └── Conflict Resolution
│   │
│   ├── 6.5 Multi-Agent Frameworks
│   │   ├── AutoGen (Microsoft)
│   │   ├── CrewAI
│   │   ├── LangGraph (multi-agent)
│   │   ├── MetaGPT
│   │   ├── ChatDev
│   │   └── CAMEL
│   │
│   └── 6.6 Emergent Behavior
│       ├── Collective Intelligence
│       ├── Self-Organization
│       ├── Specialization
│       └── Coordination Without Central Control
│
├── 7. CONTROL FLOW & ORCHESTRATION
│   ├── 7.1 Flow Patterns
│   │   ├── Linear / Sequential
│   │   ├── Branching / Conditional
│   │   ├── Looping / Iterative
│   │   ├── Parallel / Fan-Out
│   │   ├── Fan-In / Aggregation
│   │   ├── State Machines
│   │   └── DAG-Based Workflows
│   │
│   ├── 7.2 Routing
│   │   ├── LLM-Based Routing (semantic)
│   │   ├── Rule-Based Routing
│   │   ├── Classifier-Based Routing
│   │   ├── Intent Detection
│   │   └── Fallback / Default Routes
│   │
│   ├── 7.3 State Management
│   │   ├── Agent State
│   │   ├── Conversation State
│   │   ├── Task State
│   │   ├── Global Shared State
│   │   ├── Checkpointing
│   │   └── State Persistence
│   │
│   ├── 7.4 Error Handling
│   │   ├── Retry Mechanisms
│   │   ├── Fallback Strategies
│   │   ├── Graceful Degradation
│   │   ├── Timeout Management
│   │   └── Dead Letter Queues
│   │
│   └── 7.5 Orchestration Frameworks
│       ├── LangGraph
│       ├── Temporal (workflow engine)
│       ├── Prefect / Airflow (data pipelines)
│       └── Custom State Machines
│
├── 8. GROUNDING & KNOWLEDGE
│   ├── 8.1 Knowledge Sources
│   │   ├── Parametric Knowledge (model weights)
│   │   ├── Non-Parametric Knowledge (external retrieval)
│   │   ├── Structured Data (databases, APIs)
│   │   ├── Unstructured Data (documents, web)
│   │   └── Real-Time Data (live feeds, sensors)
│   │
│   ├── 8.2 Knowledge Representation
│   │   ├── Knowledge Graphs
│   │   ├── Ontologies
│   │   ├── Embeddings / Vector Representations
│   │   ├── Symbolic Representations
│   │   └── Hybrid (neuro-symbolic)
│   │
│   ├── 8.3 Grounding Techniques
│   │   ├── RAG (see section 4.5)
│   │   ├── Web Search Integration
│   │   ├── Database Querying
│   │   ├── API Calls for Real-Time Data
│   │   └── Multi-Modal Grounding (images, audio)
│   │
│   └── 8.4 Knowledge Updates
│       ├── Fine-Tuning
│       ├── Knowledge Base Updates
│       ├── Continual Learning
│       └── Cache Invalidation
│
├── 9. EVALUATION & OBSERVABILITY
│   ├── 9.1 Agent Evaluation
│   │   ├── Task Completion Rate
│   │   ├── Accuracy / Correctness
│   │   ├── Efficiency (steps taken, tokens used)
│   │   ├── Cost per Task
│   │   ├── Latency / Response Time
│   │   ├── Tool Usage Accuracy
│   │   └── Plan Quality
│   │
│   ├── 9.2 Benchmarks
│   │   ├── AgentBench
│   │   ├── WebArena
│   │   ├── SWE-Bench
│   │   ├── GAIA
│   │   ├── ToolBench
│   │   └── HumanEval (code agents)
│   │
│   ├── 9.3 Observability & Tracing
│   │   ├── LLM Call Tracing
│   │   ├── Tool Call Logging
│   │   ├── Decision Path Visualization
│   │   ├── Token Usage Tracking
│   │   ├── Cost Monitoring
│   │   └── Latency Profiling
│   │
│   ├── 9.4 Observability Tools
│   │   ├── LangSmith
│   │   ├── Arize Phoenix
│   │   ├── Weights & Biases (W&B)
│   │   ├── Helicone
│   │   ├── Braintrust
│   │   └── OpenTelemetry (custom)
│   │
│   └── 9.5 Testing Strategies
│       ├── Unit Testing (individual components)
│       ├── Integration Testing (tool + LLM)
│       ├── End-to-End Testing
│       ├── Regression Testing
│       ├── Adversarial Testing
│       └── A/B Testing
│
├── 10. SAFETY, GUARDRAILS & ALIGNMENT
│   ├── 10.1 Input Guardrails
│   │   ├── Prompt Injection Detection
│   │   ├── Jailbreak Prevention
│   │   ├── Input Validation / Sanitization
│   │   ├── Content Filtering
│   │   └── Rate Limiting
│   │
│   ├── 10.2 Output Guardrails
│   │   ├── Output Validation
│   │   ├── Hallucination Detection
│   │   ├── Toxicity Filtering
│   │   ├── PII Detection / Redaction
│   │   ├── Fact-Checking
│   │   └── Format Enforcement (structured output)
│   │
│   ├── 10.3 Action Guardrails
│   │   ├── Tool Permission Boundaries
│   │   ├── Sandboxing / Isolation
│   │   ├── Resource Limits (budget, time, tokens)
│   │   ├── Irreversibility Checks
│   │   └── Scope Constraints
│   │
│   ├── 10.4 Human-in-the-Loop (HITL)
│   │   ├── Approval Gates (before critical actions)
│   │   ├── Escalation Paths
│   │   ├── Human Feedback Integration
│   │   ├── Override Mechanisms
│   │   └── Confidence-Based Routing
│   │
│   ├── 10.5 Alignment & Ethics
│   │   ├── Goal Alignment
│   │   ├── Value Alignment
│   │   ├── Reward Hacking Prevention
│   │   ├── Instrumental Convergence Awareness
│   │   ├── Transparency / Explainability
│   │   └── Bias Detection & Mitigation
│   │
│   └── 10.6 Guardrail Frameworks
│       ├── NeMo Guardrails (NVIDIA)
│       ├── Guardrails AI
│       ├── LLM Guard
│       └── Custom Rule Engines
│
├── 11. LLM FOUNDATIONS (Underlying Technology)
│   ├── 11.1 Model Architecture
│   │   ├── Transformer Architecture
│   │   ├── Attention Mechanisms
│   │   ├── Context Windows
│   │   └── Tokenization
│   │
│   ├── 11.2 Model Capabilities
│   │   ├── Instruction Following
│   │   ├── In-Context Learning
│   │   ├── Few-Shot Learning
│   │   ├── Multi-Modal Understanding
│   │   └── Structured Output Generation
│   │
│   ├── 11.3 Model Selection for Agents
│   │   ├── Reasoning Capability
│   │   ├── Tool Calling Support
│   │   ├── Context Window Size
│   │   ├── Latency / Speed
│   │   ├── Cost
│   │   └── Fine-Tuning Availability
│   │
│   ├── 11.4 Prompt Engineering for Agents
│   │   ├── System Prompts / Personas
│   ││   │   ├── Role Definition
│   │   ├── Instruction Formatting
│   │   ├── Output Format Specification
│   │   ├── Few-Shot Examples
│   │   ├── Constraint Setting
│   │   └── Dynamic Prompt Construction
│   │
│   └── 11.5 Model Optimization
│       ├── Fine-Tuning for Agent Behavior
│       ├── RLHF (Reinforcement Learning from Human Feedback)
│       ├── DPO (Direct Preference Optimization)
│       ├── Distillation (smaller agent models)
│       ├── Quantization
│       └── Caching / KV Cache Optimization
│
├── 12. AGENT DESIGN PATTERNS
│   ├── 12.1 Single Agent Patterns
│   │   ├── ReAct (Reason + Act)
│   │   ├── Plan-and-Execute
│   │   ├── Reflection Pattern
│   │   ├── Self-Ask with Search
│   │   ├── Chain-of-Agents
│   │   ├── Tool-Augmented Generation
│   │   └── Iterative Refinement
│   │
│   ├── 12.2 Multi-Agent Patterns
│   │   ├── Debate / Adversarial Collaboration
│   │   ├── Generator-Critic
│   │   ├── Supervisor-Worker
│   │   ├── Assembly Line / Pipeline
│   │   ├── Mixture of Agents
│   │   ├── Society of Mind
│   │   └── Agent-as-a-Tool (nested agents)
│   │
│   ├── 12.3 Workflow Patterns
│   │   ├── Prompt Chaining
│   │   ├── Routing / Dispatching
│   │   ├── Parallelization
│   │   ├── Orchestrator-Worker
│   │   ├── Evaluator-Optimizer Loop
│   │   └── Human-in-the-Loop Workflow
│   │
│   └── 12.4 Anti-Patterns (What to Avoid)
│       ├── Over-Autonomy (too little human oversight)
│       ├── Infinite Loops
│       ├── Tool Overload (too many tools)
│       ├── Context Window Overflow
│       ├── Premature Multi-Agent (unnecessary complexity)
│       └── Vague Goal Specification
│
├── 13. REAL-WORLD AGENT APPLICATIONS
│   ├── 13.1 Software Engineering Agents
│   │   ├── Code Generation
│   │   ├── Code Review
│   │   ├── Bug Fixing / Debugging
│   │   ├── Test Generation
│   │   ├── Documentation Generation
│   │   ├── Repository-Level Understanding
│   │   └── CI/CD Integration
│   │
│   ├── 13.2 Research & Analysis Agents
│   │   ├── Literature Review
│   │   ├── Data Analysis
│   │   ├── Report Generation
│   │   ├── Competitive Intelligence
│   │   └── Fact-Checking Agents
│   │
│   ├── 13.3 Customer-Facing Agents
│   │   ├── Customer Support Bots
│   │   ├── Sales Assistants
│   │   ├── Onboarding Agents
│   │   ├── FAQ / Knowledge Base Agents
│   │   └── Voice Agents
│   │
│   ├── 13.4 Data & Analytics Agents
│   │   ├── Text-to-SQL Agents
│   │   ├── Data Pipeline Agents
│   │   ├── Dashboard Generation
│   │   ├── Anomaly Detection
│   │   └── ETL Automation
│   │
│   ├── 13.5 Personal / Productivity Agents
│   │   ├── Email Management
│   │   ├── Calendar / Scheduling
│   │   ├── Task Management
│   │   ├── Note-Taking / Summarization
│   │   └── Personal Knowledge Management
│   │
│   ├── 13.6 Browser / Computer Use Agents
│   │   ├── Web Navigation
│   │   ├── Form Filling
│   │   ├── Screen Understanding
│   │   ├── GUI Interaction
│   │   └── RPA (Robotic Process Automation)
│   │
│   └── 13.7 Domain-Specific Agents
│       ├── Legal Agents
│       ├── Medical / Healthcare Agents
│       ├── Financial Agents
│       ├── Education / Tutoring Agents
│       └── Scientific Discovery Agents
│
├── 14. INFRASTRUCTURE & DEPLOYMENT
│   ├── 14.1 Execution Environment
│   │   ├── Cloud Deployment (AWS, GCP, Azure)
│   │   ├── Serverless Functions
│   │   ├── Containerization (Docker, K8s)
│   │   ├── Edge Deployment
│   │   └── Local / On-Premise
│   │
│   ├── 14.2 Scalability
│   │   ├── Horizontal Scaling
│   │   ├── Load Balancing
│   │   ├── Queue-Based Architecture
│   │   ├── Async / Event-Driven
│   │   └── Rate Limit Management
│   │
│   ├── 14.3 Cost Management
│   │   ├── Token Optimization
│   │   ├── Model Selection (cost vs. capability)
│   │   ├── Caching (semantic caching)
│   │   ├── Budget Limits per Agent/Task
│   │   └── Tiered Model Usage (cheap → expensive)
│   │
│   ├── 14.4 Reliability & Resilience
│   │   ├── Retry Logic
│   │   ├── Circuit Breakers
│   │   ├── Fallback Models
│   │   ├── Idempotency
│   │   ├── Checkpointing & Recovery
│   │   └── Multi-Provider Failover
│   │
│   └── 14.5 Security
│       ├── API Key Management
│       ├── Secret Management
│       ├── Authentication / Authorization
│       ├── Data Encryption
│       ├── Audit Logging
│       ├── Network Isolation
│       └── Principle of Least Privilege
│
├── 15. ADVANCED & EMERGING TOPICS
│   ├── 15.1 Self-Improving Agents
│   │   ├── Learning from Feedback
│   │   ├── Experience Replay
│   │   ├── Skill Acquisition
│   │   ├── Meta-Learning
│   │   └── Curriculum Learning
│   │
│   ├── 15.2 World Models
│   │   ├── Internal Environment Simulation
│   │   ├── Predictive Modeling
│   │   ├── Counterfactual Reasoning
│   │   └── Mental Simulation
│   │
│   ├── 15.3 Embodied Agents
│   │   ├── Robotics Integration
│   │   ├── Physical World Interaction
│   │   ├── Sensor Fusion
│   │   └── Sim-to-Real Transfer
│   │
│   ├── 15.4 Multi-Modal Agents
│   │   ├── Vision + Language
│   │   ├── Audio / Speech Integration
│   │   ├── Video Understanding
│   │   ├── Document Understanding (OCR + LLM)
│   │   └── Cross-Modal Reasoning
│   │
│   ├── 15.5 Agent-to-Agent Ecosystems
│   │   ├── Agent Marketplaces
│   │   ├── Inter-Agent Protocols
│   │   ├── Agent Identity & Trust
│   │   ├── Agent Registries
│   │   └── Federated Agent Systems
│   │
│   ├── 15.6 Reinforcement Learning for Agents
│   │   ├── Reward Design
│   │   ├── Policy Learning
│   │   ├── Exploration vs. Exploitation
│   │   ├── Environment Interaction
│   │   └── Online Learning
│   │
│   └── 15.7 Theoretical Foundations
│       ├── Bounded Rationality
│       ├── Decision Theory
│       ├── Game Theory (multi-agent)
│       ├── Information Theory
│       ├── Control Theory
│       └── Computational Complexity of Agent Tasks
│
└── 16. ECOSYSTEM & TOOLING
    ├── 16.1 Agent Frameworks
    │   ├── LangChain / LangGraph
    │   ├── LlamaIndex
    │   ├── CrewAI
    │   ├── AutoGen (Microsoft)
    │   ├── Semantic Kernel (Microsoft)
    │   ├── OpenAI Agents SDK
    │   ├── Haystack (deepset)
    │   ├── Pydantic AI
    │   └── Smolagents (Hugging Face)
    │
    ├── 16.2 LLM Providers
    │   ├── OpenAI (GPT-4, GPT-4o)
    │   ├── Anthropic (Claude)
    │   ├── Google (Gemini)
    │   ├── Meta (Llama)
    │   ├── Mistral
    │   ├── Cohere
    │   └── Open-Source Models (via Ollama, vLLM, etc.)
    │
    ├── 16.3 Vector Databases
    │   ├── Pinecone
    │   ├── Weaviate
    │   ├── Chroma
    │   ├── Qdrant
    │   ├── Milvus
    │   ├── pgvector (PostgreSQL)
    │   └── FAISS
    │
    ├── 16.4 Observability & Evaluation
    │   ├── LangSmith
    │   ├── Arize Phoenix
    │   ├── Braintrust
    │   ├── Helicone
    │   ├── Weights & Biases
    │   └── Ragas (RAG evaluation)
    │
    ├── 16.5 Guardrail Tools
    │   ├── NeMo Guardrails
    │   ├── Guardrails AI
    │   ├── LLM Guard
    │   └── Rebuff (prompt injection)
    │
    └── 16.6 Development Tools
        ├── Prompt Playgrounds (OpenAI, Anthropic)
        ├── Agent IDEs (Flowise, Dify, n8n)
        ├── No-Code / Low-Code Agent Builders
        ├── Version Control for Prompts
        └── CI/CD for AI Agents
