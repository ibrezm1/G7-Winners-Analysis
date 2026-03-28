Based on the visual interface and the founders' transcript, here is a detailed technical design of the **Innervation Multiagent Framework**. 

This design breaks down how the visual nodes translate into actual code architecture, covering the frontend, backend, agent orchestration, and infrastructure.

### 1. High-Level Architecture
The system uses a decoupled, microservices-oriented architecture designed for scalability, security (government compliance), and modularity.
* **Frontend (Client/UI):** React.js
* **Backend (API & Logic):** Django (Python) via REST API
* **Agent Orchestration:** Custom Python framework (likely wrapping LangChain or LlamaIndex)
* **Infrastructure:** Docker & Kubernetes

---

### 2. Frontend Design (React)
The UI is a visual node-based editor, allowing users to build and monitor agent pipelines.

* **Node-Graph Interface:** The core UI relies on a library like **React Flow** to render the draggable nodes, edges (connecting lines), and the canvas.
* **Component Structure:**
    * `NodeComponent`: Renders the individual boxes (Input, Agent, Aggregator).
    * `PropertiesPanel`: The right-hand sidebar that populates dynamically based on the selected node.
* **Accessibility (a11y):** As highlighted in the transcript, the React frontend includes built-in toggles for high-contrast/colorblind modes and dynamic text scaling to meet WCAG compliance.
* **State Management:** Redux or React Context manages the state of the graph (which nodes are connected to what) and the real-time execution status (e.g., highlighting a node when it is "running").

---

### 3. Backend Design (Django & REST API)
Django serves as the control plane, managing users, saving workflows, and initiating the AI agent runs.

* **REST API Layer:** Built with Django REST Framework (DRF). The frontend is entirely optional; clients can trigger workflows directly via API endpoints (e.g., `POST /api/v1/workflows/{id}/execute`).
* **Database Schema (Relational):**
    * `Workflow`: Stores the overarching graph (e.g., "Ableism Debiasing MAS Updated").
    * `Node`: Stores configurations for specific agents, inputs, and guides (e.g., System Prompts, Selected Tools, Max Tokens).
    * `Edge`: Defines the data routing (e.g., User Input -> Framing Agent).
    * `ExecutionRun`: Tracks the auditable history of every request, storing the inputs, agent notes, and final outputs.

---

### 4. Agent Orchestration Logic (The Pipeline)
This is where the actual AI work happens. The graph is parsed into an execution graph (likely a Directed Acyclic Graph - DAG) within Python.

* **Parallel Execution:** The user input is routed to the four initial agents (Framing, Advocate, Psychology, Literary) simultaneously. This is handled using asynchronous Python (`asyncio`) or a task queue like **Celery** to ensure the agents run in parallel, reducing overall latency.
* **Node Configurations:** Looking at the "Agent Function Node" settings in the UI, the backend code for an agent class includes:
    * `sub_llm_model`: Dynamically loads the chosen LLM (OpenAI, Anthropic, or local open-source models).
    * `tools`: A registry of accessible functions. For example, the `Arxiv Search` tool maps to a Python function that pings the Arxiv API to retrieve academic papers.
    * `context_threshold` & `in_context_history`: Manages memory and token limits before summarizing or truncating the conversation history.
* **The Aggregation & Validation Node (The Critical Step):** This is a distinct Python class that waits for the parallel agents to finish (`await asyncio.gather(*agent_tasks)`). 
    * *Data Assembly:* It concatenates the outputs ("notes") from the four agents alongside the original text.
    * *Structured Output Extraction:* It prompts an LLM to generate the final response in a strict JSON schema using a library like **Pydantic** (e.g., requiring fields for `original_text`, `analysis_log`, and `debiased_text`).
    * *Self-Correction Loop (While Loop):*
        ```python
        max_retries = 3
        attempts = 0
        while attempts < max_retries:
            response = llm.generate(prompt)
            if validate_schema(response) and validate_attachments(response):
                return response # Success
            attempts += 1
            prompt += "Your previous output failed validation. Correct the structure."
        raise ValidationError("Agent failed to output correct format.")
        ```

---

### 5. Infrastructure & Deployment
Built for high-security, enterprise, and government environments.

* **Containerization (Docker):** Every component (Frontend, Django API, Celery Workers) is packaged into isolated Docker containers.
* **Orchestration (Kubernetes):** Kubernetes (K8s) manages the deployment. This is crucial for MAS architectures:
    * **Auto-scaling:** If 100 government employees submit documents simultaneously, K8s can spin up additional worker pods to handle the massive spike in parallel agent executions.
    * **Data Sovereignty:** By using Kubernetes and being model-agnostic, the entire system can be deployed "on-premise" or in a secure government cloud (like AWS GovCloud), ensuring sensitive constituent data never leaves the protected network.


Based on the image provided, you are looking at a visual pipeline for a **Multi-Agent System (MAS)** designed to take a piece of text and rewrite it to remove ableist language or biases. 

Here is a detailed breakdown of the workflow happening on the screen:

### 1. The Goal
The project is titled **"Ableism Debiasing MAS Updated."** The system's purpose is to process a user's text through multiple specialized AI "agents" to identify and correct ableism from various distinct perspectives before combining those insights into a final, polished output.

### 2. The Input Stage (Top Row)
The process begins with the **"User Input"** node (green). This represents the raw text or query that needs to be debiased. 
Alongside the user input are three specialized "Guide" nodes (purple). These likely contain specific instructions, prompt templates, or foundational documents that dictate *how* the text should be analyzed:
* **Advocate Backed Debiasing Guide**
* **Psychology Based Debiasing Guide**
* **Journalistic and Literary Debiasing Guide**

### 3. The Specialized Agents (Middle Row)
The user's query is split and sent simultaneously to four distinct AI agents (light blue nodes). Three of these agents are paired with the specific guides mentioned above:
* **Framing Debiaser Agent:** Looks at the overall structure and framing of the text.
* **Advocate Guided Debiaser:** Analyzes the text using the principles provided by disability advocates.
* **Psychology Guided Debiaser:** Analyzes the text through a psychological lens.
* **Literary Guided Debiaser:** Analyzes the text based on journalistic and literary standards.

**Tool Usage:** Looking at the right-hand panel, we can see the settings for one of these agents (likely the Advocate Guided Debiaser, as the cursor is near it). It has been given access to a tool called **"Arxiv Search."** This means this specific agent can actively search the Arxiv database for academic papers to verify claims or find the most up-to-date, scientifically accurate terminology while it evaluates the text.

### 4. Aggregation and Verification (Bottom Green Node)
Instead of just giving the user four different rewritten versions, the system uses an **"Aggregation and Verification Agent."** * It collects the "notes" (the critiques, suggestions, or rewritten drafts) from all four specialized agents.
* It also receives a direct feed of the original "query" from the User Input node (indicated by the dotted line running straight down the middle).
* This agent's job is to review all the different perspectives, resolve any conflicting advice between the specialized agents, and synthesize everything into one cohesive response.

### 5. Final Output (Bottom Red Node)
Finally, the aggregator passes its finalized answer to the **"Debiased Output"** node, which presents the corrected, bias-free text back to the user. 

In short, it is a highly sophisticated, parallel-processing AI workflow designed to ensure text is evaluated from multiple expert angles before a final draft is produced.
