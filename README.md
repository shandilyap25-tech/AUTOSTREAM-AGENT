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

