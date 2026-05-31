# Local Agent Lab

Lab for learning and experiement with local AI agents. 
Using a llama-cpp server running on a WSL2 instance on a Windows host with a 5080 GPU (16Gb VRAM) and 32Gb system memory.


## Dependencies
1) Use [wsl-builds](https://github.com/SpoddyCoder/wsl-builds) to make installing the complex software stack really easy.

2) Run a llama-cpp server, [local-llama-lab](https://github.com/SpoddyCoder/local-llama-lab).

3) Install the LangChain eco-system...

```bash
./wsl-builder.sh dev-python python3 conda
./wsl-builder.sh dev-js node
./wsl-builder.sh ai-agents setup-env,langchain,langgraph,langchain-llama-cpp,langsmith,

# optionals
./wsl-builder.sh ai-agents openai-agent,mcp,mcp-inspector
```

## First Steps
TODO!

---

## Key Learnings

### The Re-Act Loop (Reason and Act)
* **Re**ason + **Act**: the agent thinks step by step, then does something (e.g. call a tool), then uses the result to decide the next step.
* It repeats that cycle—think → act → observe → think again, until the task is done or it gives up.
* The agent alternates reasoning and acting instead of planning everything upfront or blindly calling tools.
* Reasoning is usually explicit (e.g. "Thought: …") so the next step follows from logic, not just the latest tool output.
* **Act** means a structured tool call with arguments (search, run code, read a file)—not free-form text pretending to do something.
* Tool results come back as **observations** the model must read and react to; that keeps it tied to real state instead of guessing.
* Complex tasks are broken into small steps—the agent only decides the *next* step after seeing what happened last time.
* The loop stops when the agent gives a final answer, hits a step limit, or fails (bad tool output, repeated mistakes, timeout).
* It combines chain-of-thought (planning) with tool use (doing)—that's what makes agentic behavior work in practice.
* Watch for: infinite loops, ignoring tool output, over-calling tools, and context growing with every thought/action/observation (cost + confusion).
* In frameworks (LangChain, LangGraph, etc.) this shows up as an agent executor or graph: LLM node → tool node → back to LLM until done.

### Tools
* **Tools** are how an agent *does* things outside the model—search the web, read a file, run code, call an API, query a database. The LLM doesn't execute them; the framework runs them and returns the result as an observation.
* Each tool has a **name**, a **description** (what it's for, when to use it), and a **schema** (what arguments it accepts). The model picks a tool and fills in those arguments; bad descriptions or vague schemas lead to wrong or failed calls.
* Descriptions matter more than you'd think—the model chooses tools from text alone, so write them like instructions for a colleague: purpose, inputs, limits, and when *not* to use it.
* **Structured output** is the contract: the agent emits a tool call (name + JSON args), not prose like "I'll search for that now." Frameworks parse that and invoke the real function.
* Common categories:
  * **Retrieval** — RAG, document lookup, vector search (often the agent decides *when* to fetch context).
  * **Action** — send email, create ticket, update a record (side effects; needs guardrails).
  * **Computation** — calculator, code interpreter, SQL (deterministic results the model shouldn't guess).
  * **Environment** — read/write files, shell commands, browser automation (powerful; restrict scope in local labs).
* **MCP (Model Context Protocol)** standardizes how external capabilities are exposed to agents—servers advertise tools/resources; clients (Cursor, inspectors, custom apps) connect and call them. Useful when tools live outside your Python process or you want reuse across hosts.
* Tool results should be **concise and readable**—trim huge API payloads, summarize errors clearly, and cap size so context doesn't balloon every loop turn.
* Design tips:
  * Prefer **small, focused tools** over one giant "do everything" tool—the model picks more reliably.
  * Return **actionable errors** ("file not found: /path/x") so the agent can recover instead of retrying blindly.
  * Separate **read** tools from **write** tools when you need approval gates or audit trails.
* Watch for: calling the wrong tool, invented arguments, retry loops on the same failed call, and giving the agent tools it doesn't need (more choices → more mistakes).
* In LangChain/LangGraph, tools are usually plain functions or `@tool` wrappers bound to the agent; graphs can route tool nodes explicitly for stricter control than a free-form Re-Act loop.

### RAG (Retrieval Augmented Generation)
* RAG means: find relevant documents, put them in the prompt, then let the LLM answer using that context.
* **Index pipeline**: load documents, split into chunks, turn chunks into embeddings, store in a searchable index.
* **Retrieval**: turn the user's question into a search, fetch the best-matching chunks (often hybrid keyword + semantic search, sometimes reranking).
* **Generation**: pack retrieved chunks into the prompt; the model may still ignore them or hallucinate, so grounding and citations matter.
* Challenges:
  * Chunking and embedding choices strongly affect what gets found later.
  * Retrieval quality is a constant challenge — wrong or missing chunks mean bad answers even with a good model.
  * Evaluation is hard: you need to test both "did we find the right stuff?" and "did the answer use it correctly?"
  * Keeping indexes in sync with changing documents — re-embedding, updates, broken pipelines — is ongoing work.
  * In agents, RAG is often just another tool the agent calls when it needs external knowledge (can be one-shot or multi-step).

---

## What about the Cloud? (AWS + GCP)
Yes, this is local tests and experiments - but my work is in the Cloud, so having an awareness of their offering and crossover points is useful.

### AWS Bedrock
* `Bedrock Models` — **110+ models** from **18 providers** (single serverless API; [models at a glance](https://docs.aws.amazon.com/bedrock/latest/userguide/model-cards.html))
  * **Amazon first-party** — Nova (text, vision, speech, video, embeddings) and legacy Titan (embeddings, image gen)
  * **Major LLM partners** — Anthropic Claude (4.x), Meta Llama (3.x + 4), Mistral AI, Cohere (Command, Embed, Rerank)
  * **Open-weight hosts** — DeepSeek, Qwen, OpenAI gpt-oss (open-weight only—not ChatGPT/GPT-4 API), Google Gemma 3, MiniMax, Moonshot Kimi, Z.AI GLM, NVIDIA Nemotron, Writer Palmyra
  * **Beyond chat** — Stability AI (image gen/editing), TwelveLabs (video understanding), Luma (video gen), plus speech and embedding models
  * **Availability** — choice is model ID + region + inference profile (`us.`, `global.`, etc.); GA, Legacy, and Preview vary by region
* `Bedrock Agents`
  * Low-code orchestration for agents (knowledge bases, action groups, Lambda)—not fully no-code
  * Limited, but good for simple use cases or prototyping.
* `Strands Agents SDK`
  * Open-source Python framework for building agents; model-driven loop where the LLM picks tools each step.
  * Contrast with LangGraph-style graphs where the developer defines nodes, edges, and control flow (often clearer for production guardrails).
  * Model agnostic—Bedrock, Anthropic, Ollama, LiteLLM, etc.—with strong Bedrock integration.
  * Local dev via **ADT** (Agent Development Toolkit for Strands)—don't confuse with Google's **ADK**.
* `Amazon Bedrock AgentCore`
  * Agentic platform beyond runtime: Runtime, Memory, Gateway, Identity, Registry (serverless hosting with session isolation).
  * Framework agnostic—Strands, LangGraph, CrewAI, custom code—with minimal porting effort.

### Gemini Enterprise Agent Platform (formerly Vertex AI)
* `The Model Garden` — **200+ models** (first-party, partner MaaS, open MaaS, and self-deploy OSS; [available models](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/available-models))
  * **Google proprietary** — Gemini (2.x/3.x, Flash, Pro, Live), Imagen, Veo, Chirp
  * **Google open-weight** — Gemma (fine-tunable; listed separately from Gemini in docs)
  * **Third-party partner MaaS** — Anthropic Claude, Mistral (Medium, Codestral, OCR), AI21 Jamba, xAI Grok (preview)
  * **Open-model MaaS** — Meta Llama (3.3, 4 Scout/Maverick), DeepSeek, Qwen, MiniMax M2, GLM 4.7/5, Kimi K2, gpt-oss
  * **Self-deploy OSS** — large catalog (Llama, Mistral, Falcon, BERT, ViT, Hugging Face, etc.) tuned and served on your GPUs
  * **Availability** — enablement and licensing (e.g. Llama) differ per model; docs split across first-party, partner, and open MaaS paths
* `Agent Studio` (formerly Agent Builder)
  * Low-code agent building in the platform UI—roughly equivalent to Bedrock Agents
  * Limited, but good for simple use cases or prototyping.
* `Agent Development Kit (ADK)`
  * Google's open-source framework for building agents.
  * Model agnostic—Gemini, Claude, Llama, etc.—with strongest Gemini integration.
  * Runs locally; **ADK** on GCP vs **ADT** on AWS (both "agent dev kit" acronyms—easy to mix up).
* `Agent Engine`
  * Google's runtime / hosting infrastructure for agents.
  * More tied to ADK - but can be used with LangChain/Graph etc. with additional effort.