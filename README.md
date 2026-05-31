# local-agent-lab

Lab for learning and experiement with local AI agents. Using a llama-cpp server running LLM's running on a 5080 GPU.


## Dependencies
1) Run a llama-cpp server, [local-lllama-lab](https://github.com/SpoddyCoder/local-llama-lab) project.

2) Install the LangChain eco-system...

```bash
./wsl-builder.sh dev-python python3
./wsl-builder.sh dev-js node
./wsl-builder.sh ai-agents setup-env,langchain,langgraph,langchain-llama-cpp,langsmith,

# optionals
./wsl-builder.sh ai-agents openai-agent,mcp,mcp-inspector
```

## First Steps
