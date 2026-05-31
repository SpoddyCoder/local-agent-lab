# Local Agent Lab

Lab for learning and experiement with local AI agents. 
Using a llama-cpp server running on a WSL2 instance on a Windows host with a 5080 GPU (16Gb VRAM) and 32Gb system memory.


## Dependencies
1) Use [wsl-builds](https://github.com/SpoddyCoder/wsl-builds) to make installing the complex software stack really easy.

2) Run a llama-cpp server, [local-llama-lab](https://github.com/SpoddyCoder/local-llama-lab) project.

3) Install the LangChain eco-system...

```bash
./wsl-builder.sh dev-python python3 conda
./wsl-builder.sh dev-js node
./wsl-builder.sh ai-agents setup-env,langchain,langgraph,langchain-llama-cpp,langsmith,

# optionals
./wsl-builder.sh ai-agents openai-agent,mcp,mcp-inspector
```

## First Steps
