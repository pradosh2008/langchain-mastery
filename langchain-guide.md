# 🦜 LangChain + LangGraph Guide
> Everything you need to know to avoid mistakes when building agents with LangChain and Azure AI Foundry.

---

## 1. Package Landscape — Know What to Install

This is the #1 source of confusion. LangChain is split into multiple packages:

| Package | What it does | Install |
|---|---|---|
| `langchain-core` | Base abstractions — messages, tools, prompts | Always needed |
| `langchain` | High-level agents, chains — `create_agent` lives here | Always needed |
| `langchain-openai` | OpenAI + Azure OpenAI models | For OpenAI models only |
| `langchain-azure-ai` | All Azure AI Foundry models (Kimi, Mistral, Llama etc.) | For Foundry models |
| `langgraph` | Graph-based agent orchestration | Always needed |
| `langgraph-prebuilt` | Bundled with langgraph — do NOT install separately | Do NOT install |

**Minimal install for Azure AI Foundry:**
```bash
pip install langchain langchain-core langchain-azure-ai langgraph azure-identity python-dotenv
```

---

## 2. Which LLM Class to Use — Critical Decision

This was the biggest source of errors. Choose based on your model:

```
Is your model GPT-4o / GPT-4 / GPT-3.5 / GPT-4o-mini?
    YES → use AzureChatOpenAI from langchain-openai
    NO  → use AzureAIOpenAIApiChatModel from langchain-azure-ai
```

| Model | Package | Class |
|---|---|---|
| `gpt-4o`, `gpt-4o-mini`, `gpt-4` | `langchain-openai` | `AzureChatOpenAI` |
| `Kimi-K2`, `Mistral`, `Llama`, `Phi` | `langchain-azure-ai` | `AzureAIOpenAIApiChatModel` |
| Any Foundry model (recommended) | `langchain-azure-ai` | `AzureAIOpenAIApiChatModel` |

> ⚠️ Using `AzureChatOpenAI` with a non-OpenAI model (like Kimi) gives a **401 AuthenticationError**. It's not an auth problem — it's the wrong class.

---

## 3. Correct LLM Initialization

### For Azure AI Foundry (any model) — RECOMMENDED
```python
from langchain_azure_ai.chat_models import AzureAIOpenAIApiChatModel
from azure.identity import AzureCliCredential

llm = AzureAIOpenAIApiChatModel(
    project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=AzureCliCredential(),
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
)
```

### For Azure OpenAI specifically (OpenAI models only)
```python
from langchain_openai import AzureChatOpenAI
from azure.identity import AzureCliCredential, get_bearer_token_provider

token_provider = get_bearer_token_provider(
    AzureCliCredential(),
    "https://cognitiveservices.azure.com/.default"
)

llm = AzureChatOpenAI(
    azure_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    azure_deployment=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
    api_version="2024-02-01",
    azure_ad_token_provider=token_provider,
)
```

> ⚠️ Never pass a raw token string as `api_key` — tokens expire. Use `azure_ad_token_provider` which auto-refreshes.

---

## 4. Correct Agent Creation — LangChain v1.x

### The current API (v1.x) — use `create_agent`
```python
from langchain.agents import create_agent

agent = create_agent(
    model=llm,
    tools=[your_tool],
    system_prompt="You are a helpful assistant.",
)
```

### Deprecated patterns — DO NOT USE
```python
# ❌ DEPRECATED — AgentExecutor removed in v1.x
from langchain.agents import AgentExecutor, create_tool_calling_agent

# ❌ DEPRECATED — initialize_agent removed
from langchain.agents import initialize_agent

# ❌ DEPRECATED — set_entry_point removed in LangGraph v1.0
graph.set_entry_point("node")
```

---

## 5. Correct Tool Definition

```python
from langchain_core.tools import tool

@tool
def get_destinations() -> list[str]:
    """Get a list of popular vacation destinations."""  # ← docstring is the tool description
    return ["Barcelona", "Paris", "Tokyo", "Bali"]
```

> ⚠️ The docstring is mandatory — it's what the LLM reads to decide whether to call the tool.

---

## 6. Running the Agent

### Invoke (single run)
```python
response = agent.invoke({
    "messages": [{"role": "user", "content": "Your question here"}]
})
print(response["messages"][-1].content)
```

### Stream (token by token)
```python
for chunk in agent.stream({
    "messages": [{"role": "user", "content": "Your question here"}]
}):
    if "agent" in chunk:
        content = chunk["agent"]["messages"][-1].content
        if content:
            print(content, end="", flush=True)
```

---

## 7. LangGraph v1.0 — Key API Changes

| Old (v0.x) | New (v1.0) | Notes |
|---|---|---|
| `from langgraph.prebuilt import create_react_agent` | `from langchain.agents import create_agent` | `create_react_agent` still works but deprecated |
| `from langgraph.graph.message import MessageState` | `from langgraph.graph.message import MessagesState` | Renamed |
| `set_entry_point("node")` | `add_edge(START, "node")` | Removed |
| `set_finish_point("node")` | `add_edge("node", END)` | Removed |
| `from langgraph.prebuilt import ToolExecutor` | `from langgraph.prebuilt import ToolNode` | Removed |

> ⚠️ Most tutorials online still use old v0.x patterns. Always check the version.

---

## 8. Environment Variables

```bash
# .env file
AZURE_AI_PROJECT_ENDPOINT=https://<hub>.services.ai.azure.com/api/projects/<project>
AZURE_AI_MODEL_DEPLOYMENT_NAME=<deployment-name>
```

**Always load with explicit path in notebooks:**
```python
from dotenv import load_dotenv
import os
load_dotenv("/workspaces/<your-repo>/.env")

# Verify
print(os.environ.get("AZURE_AI_PROJECT_ENDPOINT"))
print(os.environ.get("AZURE_AI_MODEL_DEPLOYMENT_NAME"))
```

> ⚠️ `load_dotenv()` without a path may not find the file in Codespaces. Always use the absolute path.

---

## 9. Authentication — How it Works

Azure AI Foundry uses **Microsoft Entra ID (formerly Azure AD)** for authentication — not API keys.

```
AzureCliCredential()     ← uses your az login session (best for local/Codespaces)
DefaultAzureCredential() ← tries multiple methods, best for production
ManagedIdentityCredential() ← for Azure-hosted apps
```

**The scope `https://cognitiveservices.azure.com/.default` explained:**
- `.default` means "give me all permissions I've been granted for this service"
- Different Azure services have different scopes — this one is for Azure AI/Cognitive Services
- You only need this when manually fetching tokens — `AzureCliCredential()` handles it automatically

---

## 10. Complete Working Template — Lesson 01

```python
import warnings
warnings.filterwarnings("ignore")
import logging
logging.getLogger("langchain").setLevel(logging.ERROR)

import os
from dotenv import load_dotenv
load_dotenv("/workspaces/<your-repo>/.env")

from langchain.agents import create_agent
from langchain_azure_ai.chat_models import AzureAIOpenAIApiChatModel
from langchain_core.tools import tool
from azure.identity import AzureCliCredential

# Tool
@tool
def get_destinations() -> list[str]:
    """Get a list of popular vacation destinations."""
    return [
        "Barcelona", "Paris", "Berlin", "Tokyo", "Sydney",
        "New York City", "Cairo", "Cape Town", "Rio de Janeiro", "Bali",
    ]

# LLM
llm = AzureAIOpenAIApiChatModel(
    project_endpoint=os.environ["AZURE_AI_PROJECT_ENDPOINT"],
    credential=AzureCliCredential(),
    model=os.environ["AZURE_AI_MODEL_DEPLOYMENT_NAME"],
)

# Agent
agent = create_agent(
    model=llm,
    tools=[get_destinations],
    system_prompt=(
        "You are a helpful travel agent. Help users find their perfect vacation "
        "destination based on their preferences. Use the get_destinations tool "
        "to see available destinations."
    ),
)

# Run
response = agent.invoke({
    "messages": [{"role": "user", "content": "I'm looking for a warm beach destination. What do you recommend?"}]
})
print(response["messages"][-1].content)
```

---

## 11. Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `ImportError: cannot import name 'AgentExecutor'` | Removed in LangChain v1.x | Use `create_agent` instead |
| `ImportError: cannot import name 'MessageState'` | Renamed | Use `MessagesState` |
| `AuthenticationError 401` | Wrong LLM class for model type | Use `AzureAIOpenAIApiChatModel` for non-OpenAI models |
| `KeyError: AZURE_AI_PROJECT_ENDPOINT` | .env not loaded | Call `load_dotenv("/workspaces/<repo>/.env")` |
| `ModuleNotFoundError` in notebook | Kernel pointing to wrong Python | Reinstall kernel with `.venv/bin/python -m ipykernel install ...` |
| `create_react_agent deprecated warning` | LangGraph v1.0 renamed it | Use `from langchain.agents import create_agent` |
| `Agent TravelAgent not found` | `FoundryAgent` needs pre-deployed agent | Use `FoundryChatClient` + `Agent` instead |

---

## 12. Model Choice Recommendation

| Use case | Recommended model | Why |
|---|---|---|
| Learning / tutorials | `gpt-4o-mini` | Cheap, fast, every tutorial uses it |
| Production / non-OpenAI | `Kimi-K2`, `Mistral` | Use `langchain-azure-ai` |
| Interviews | `gpt-4o` | What interviewers expect |

> ✅ For courseware and learning — always use `gpt-4o-mini`. It avoids authentication complexity and works with the most examples online.