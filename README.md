# AutoStream Conversational Lead Agent

This repository implements the ServiceHive assignment as a full Python project for the fictional SaaS product AutoStream. The agent can classify intent, answer pricing and policy questions from a local knowledge base, remember conversation state across turns, and capture qualified leads only after collecting name, email, and creator platform.

## Features

- LangGraph-based conversation flow with persistent session memory
- Local RAG over a Markdown knowledge base
- Intent detection for greeting, product/pricing inquiry, and high-intent lead
- Guarded `mock_lead_capture()` execution only after all required fields are present
- CLI chat app and FastAPI API for easy demos
- Unit tests covering pricing, policy retrieval, and the lead-capture workflow

## Project Structure

- `autostream_agent/graph.py` - LangGraph workflow and routing
- `autostream_agent/rag.py` - local Markdown retriever
- `autostream_agent/intents.py` - intent classification and entity extraction
- `autostream_agent/tools.py` - mock lead capture tool
- `autostream_agent/service.py` - application-facing agent wrapper
- `data/knowledge_base.md` - AutoStream pricing, features, and policies
- `app.py` - CLI entrypoint
- `api.py` - FastAPI server

## How To Run Locally

1. Create and activate a virtual environment.
2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Run the CLI:

```bash
python app.py
```

4. Or run the API:

```bash
uvicorn api:app --reload
```

5. Run tests:

```bash
python -m unittest discover -s tests -v
```

Optional: set `OPENAI_API_KEY` to enable GPT-4o-mini for response generation. If no key is present, the project falls back to a deterministic response composer so the assignment still runs locally and can be demoed offline.

## Architecture Explanation

I used LangGraph because the assignment is fundamentally a stateful workflow rather than a single prompt-response exchange. The conversation needs to branch differently when the user asks a pricing question, signals buying intent, or sends missing lead details over multiple turns. LangGraph makes that control flow explicit through nodes for intent analysis, knowledge retrieval, lead collection, tool execution, and response generation. That keeps the agent easier to debug and more production-friendly than hiding everything inside one LLM call.

State is managed inside the graph and checkpointed in memory by `thread_id`, which acts as the session identifier. Each turn appends the latest human and AI messages to the conversation history and preserves structured fields such as detected intent, retrieved knowledge snippets, lead details, missing fields, and whether the lead has already been captured. This lets the agent remember partial information like a creator platform from one message and an email from a later message. The RAG layer uses a local Markdown knowledge base so pricing and policy answers stay grounded in explicit data. Tool execution is gated by state: the capture function is only called when `name`, `email`, and `platform` are all present.

## WhatsApp Deployment Using Webhooks

To deploy this on WhatsApp, I would place the FastAPI app behind a public HTTPS endpoint and connect it to the WhatsApp Business Platform or Twilio WhatsApp sandbox. Incoming WhatsApp messages would hit a webhook route that maps the sender phone number to `session_id`. The webhook handler would forward the text into the same `AutoStreamAgent.chat()` method used locally, then send the generated reply back through the WhatsApp messaging API. Media, delivery receipts, and retry logic would be handled in the webhook layer, while the agent logic would stay unchanged. For production, I would replace the in-memory checkpoint with Redis or a database-backed store so memory survives restarts and scales across multiple workers.

## Suggested Demo Flow

Use this sequence for the 2-3 minute screen recording:

# AutoStream Agent — Social-to-Lead Agentic Workflow

This project is a complete submission-ready implementation for the **ServiceHive / Inflx Machine Learning Intern assignment**. It builds a conversational AI agent for **AutoStream**, a fictional SaaS platform offering automated video editing tools for content creators.

The agent supports:
- **Intent identification**: greeting, product/pricing inquiry, and high-intent lead detection
- **Local RAG** over a JSON knowledge base
- **Stateful multi-turn memory** using **LangGraph**
- **Tool execution** through a mock lead-capture API that runs **only after** all required details are collected

## Project Structure

```bash
autostream_agent_project/
├── app.py
├── kb.json
├── requirements.txt
├── .env.example
└── README.md
```

## Features

### 1) Intent Detection
The agent classifies the user into:
- Casual greeting
- Product or pricing inquiry
- High-intent lead

### 2) Local RAG
The knowledge base is stored in `kb.json` and contains:
- Basic Plan — $29/month, 10 videos/month, 720p
- Pro Plan — $79/month, unlimited videos, 4K, AI captions
- No refunds after 7 days
- 24/7 support only on Pro

### 3) Lead Capture Tool
When the user shows buying intent, the agent collects:
- Name
- Email
- Creator platform

Only after all three fields are available does it call:

```python
def mock_lead_capture(name, email, platform):
    print(f"Lead captured successfully: {name}, {email}, {platform}")
```

## How to Run Locally

### Step 1: Create a virtual environment

**Windows**
```bash
python -m venv venv
venv\Scripts\activate
```

**Mac/Linux**
```bash
python3 -m venv venv
source venv/bin/activate
```

### Step 2: Install dependencies
```bash
pip install -r requirements.txt
```

### Step 3: Add environment variables
Copy `.env.example` to `.env` and add your key:

```bash
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=gpt-4o-mini
```

### Step 4: Run the agent
```bash
python app.py
```

## Example Demo Conversation

```text
You: Hi, tell me about your pricing.
Agent: AutoStream offers a Basic plan for $29/month with 10 videos and 720p, and a Pro plan for $79/month with unlimited videos, 4K resolution, AI captions, and 24/7 support.

You: That sounds good, I want to try the Pro plan for my YouTube channel.
Agent: Based on our plans, the Pro plan is $79/month with unlimited videos, 4K resolution, AI captions, and 24/7 support.
To get you signed up, please share your name, email.

You: My name is Prahalad.
Agent: Please share your email.

You: prahalad@example.com
Agent: Perfect — I have captured your details successfully.
```

## Architecture Explanation (≈200 words)

I chose **LangGraph** because the assignment requires a workflow-oriented conversational agent with reliable state management across multiple turns. LangGraph is a strong fit for this because it makes the control flow explicit: user input is ingested, intent is detected, the local knowledge base is queried when needed, a response is generated, and the lead-capture tool is triggered only when all required fields are available. This graph-based design is easier to reason about than a single monolithic chain and is closer to how production agents are built.

State is stored in a typed `AgentState` object, which tracks the conversation messages, detected intent, retrieved knowledge snippets, collected lead details, and whether lead capture has already been completed. For memory across 5–6 turns, the project uses LangGraph’s `MemorySaver` checkpointing with a fixed `thread_id`, allowing the agent to remember previously collected fields such as the user’s name or platform. The RAG layer is intentionally local and lightweight: the agent loads `kb.json`, scores documents using keyword overlap, and passes the retrieved context into the response generation step. This keeps the implementation simple, transparent, and easy to evaluate while still satisfying the RAG requirement.

## WhatsApp Deployment via Webhooks

To deploy this on WhatsApp, I would place the LangGraph agent behind a small web server using **FastAPI** or **Flask**. WhatsApp messages would be received through a webhook endpoint provided by the WhatsApp Business API provider, such as Meta Cloud API or Twilio WhatsApp. When a new message arrives, the webhook handler would extract the sender ID, use it as the session or thread identifier, and pass the incoming text into the LangGraph agent.

The agent response would then be sent back to the user through the provider’s outbound message API. The sender’s phone number would act as the conversation key, allowing the state to persist across turns. For production use, I would replace `MemorySaver` with Redis or a database-backed state store, add validation for lead fields, log tool execution events, and route successful lead captures into a CRM like HubSpot or Salesforce. This design would make the same local agent workflow usable in a real customer acquisition channel.

## Demo Video Checklist (2–3 min)
Record your screen and show:
1. Pricing question
2. Accurate RAG answer
3. High-intent detection
4. Name/email/platform collection
5. Successful `mock_lead_capture()` output in terminal

## Notes
- The tool is **not triggered prematurely**.
- The knowledge base is fully local.
- The flow is intentionally simple and interview-friendly.
- You can extend this project with FastAPI, Redis, vector DB retrieval, or CRM integrations.

1. Ask: `Hi, tell me about your pricing.`
2. Ask: `That sounds good, I want to try the Pro plan for my YouTube channel.`
3. Reply with your name.
4. Reply with your email.
5. Show the successful lead capture confirmation.
