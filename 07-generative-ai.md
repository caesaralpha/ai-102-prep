# 07 - Generative AI Solutions (10–15%)

## 7.1 Azure OpenAI Service Overview

### Available Models

| Model Family | Models | Use Case |
|-------------|--------|----------|
| **GPT-4o** | gpt-4o, gpt-4o-mini | Multimodal (text + image), chat, reasoning |
| **GPT-4** | gpt-4, gpt-4-turbo | Advanced reasoning, complex tasks |
| **GPT-3.5** | gpt-3.5-turbo | Cost-effective chat and completions |
| **DALL-E** | dall-e-3 | Image generation from text |
| **Whisper** | whisper | Speech-to-text transcription |
| **Embeddings** | text-embedding-ada-002, text-embedding-3-small/large | Vector embeddings for search/RAG |

### Deployment Models

| Deployment Type | Description |
|----------------|-------------|
| **Standard** | Pay-per-token, shared capacity |
| **Provisioned** | Reserved throughput units (PTU) |
| **Global Standard** | Routes to best available region |

## 7.2 Azure OpenAI API

### Chat Completions

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key="<key>",
    api_version="2024-06-01",
    azure_endpoint="https://<resource>.openai.azure.com/"
)

response = client.chat.completions.create(
    model="gpt-4o",  # This is the deployment name
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Azure AI Search?"}
    ],
    temperature=0.7,
    max_tokens=800,
    top_p=0.95
)

print(response.choices[0].message.content)
```

### REST API

```http
POST https://<resource>.openai.azure.com/openai/deployments/<deployment>/chat/completions?api-version=2024-06-01
Content-Type: application/json
api-key: <key>

{
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is Azure?"}
  ],
  "temperature": 0.7,
  "max_tokens": 800
}
```

### Message Roles

| Role | Description |
|------|-------------|
| **system** | Sets behavior, personality, and constraints |
| **user** | The user's input/question |
| **assistant** | Model's response (or prior response for context) |

### Key Parameters

| Parameter | Range | Description |
|-----------|-------|-------------|
| **temperature** | 0.0 – 2.0 | Randomness (lower = more deterministic) |
| **max_tokens** | 1 – model max | Maximum response length |
| **top_p** | 0.0 – 1.0 | Nucleus sampling (alternative to temperature) |
| **frequency_penalty** | -2.0 – 2.0 | Penalize repeated tokens |
| **presence_penalty** | -2.0 – 2.0 | Encourage talking about new topics |
| **stop** | Array | Sequences where model stops generating |

> **Exam Tip**: Don't set both `temperature` and `top_p` at the same time. Use one or the other.

## 7.3 Prompt Engineering

### Prompt Engineering Techniques

| Technique | Description | Example |
|-----------|-------------|---------|
| **Zero-shot** | No examples provided | "Classify this text as positive or negative" |
| **Few-shot** | Provide examples | "positive: great movie! negative: terrible film. classify: pretty good →" |
| **Chain-of-thought** | Ask model to reason step by step | "Let's think step by step..." |
| **System message** | Define behavior in system role | "You are a JSON-only response bot" |

### System Message Best Practices

```json
{
  "role": "system",
  "content": "You are an AI assistant for Contoso Electronics. You help customers with product questions. Rules: 1) Only answer about Contoso products. 2) If unsure, say 'I don't know'. 3) Be concise and professional. 4) Never make up product information."
}
```

### Grounding with Your Data

```
System: You are a helpful assistant. Use the following context to answer questions.
If the answer is not in the context, say "I don't have that information."

Context:
{retrieved_documents}

User: {question}
```

## 7.4 Azure OpenAI on Your Data (RAG)

### Architecture

```
User Query
    ↓
Azure OpenAI Service
    ↓ (query rewriting)
Azure AI Search (retrieval)
    ↓ (relevant documents)
Azure OpenAI (generation with context)
    ↓
Grounded Response
```

### Configuration

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What are the company policies on remote work?"}
    ],
    extra_body={
        "data_sources": [
            {
                "type": "azure_search",
                "parameters": {
                    "endpoint": "https://<search>.search.windows.net",
                    "index_name": "company-policies",
                    "authentication": {
                        "type": "api_key",
                        "key": "<search-key>"
                    },
                    "query_type": "semantic",
                    "semantic_configuration": "default",
                    "in_scope": True,
                    "strictness": 3,
                    "top_n_documents": 5
                }
            }
        ]
    }
)
```

### Key RAG Parameters

| Parameter | Description |
|-----------|-------------|
| `in_scope` | Only answer from provided data (true/false) |
| `strictness` | 1-5, how strictly to limit to provided data |
| `top_n_documents` | Number of documents to retrieve |
| `query_type` | `simple`, `semantic`, `vector`, `vectorSimpleHybrid`, `vectorSemanticHybrid` |

## 7.5 Embeddings

```python
response = client.embeddings.create(
    model="text-embedding-ada-002",  # deployment name
    input="What is Azure AI Search?"
)

embedding_vector = response.data[0].embedding
print(f"Vector dimensions: {len(embedding_vector)}")  # 1536 for ada-002
```

### Embedding Models

| Model | Dimensions | Description |
|-------|-----------|-------------|
| `text-embedding-ada-002` | 1536 | Legacy, widely used |
| `text-embedding-3-small` | 512 or 1536 | Efficient, cheaper |
| `text-embedding-3-large` | 256, 1024, or 3072 | Highest quality |

## 7.6 Image Generation (DALL-E)

```python
response = client.images.generate(
    model="dall-e-3",
    prompt="A futuristic city with flying cars at sunset",
    n=1,
    size="1024x1024",
    quality="hd",
    style="natural"
)

image_url = response.data[0].url
revised_prompt = response.data[0].revised_prompt
```

### DALL-E Parameters

| Parameter | Options |
|-----------|---------|
| `size` | `1024x1024`, `1024x1792`, `1792x1024` |
| `quality` | `standard`, `hd` |
| `style` | `natural`, `vivid` |
| `n` | 1 (DALL-E 3 only supports 1) |

## 7.7 Function Calling

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name, e.g., 'Seattle, WA'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"]
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Seattle?"}],
    tools=tools,
    tool_choice="auto"
)

# Check if model wants to call a function
if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    function_name = tool_call.function.name
    arguments = tool_call.function.arguments  # JSON string
```

## 7.8 Content Filtering

Azure OpenAI includes built-in content filters:

| Category | Description |
|----------|-------------|
| **Hate** | Hate speech and discrimination |
| **Sexual** | Sexual content |
| **Violence** | Violent content |
| **Self-harm** | Self-harm content |

### Severity Levels

| Level | Description | Default Action |
|-------|-------------|----------------|
| **Safe** | Content is safe | Allow |
| **Low** | Minor risk | Allow (configurable) |
| **Medium** | Moderate risk | Filter (configurable) |
| **High** | High risk | Filter (always) |

### Custom Content Filters

- Custom filters can be created in Azure OpenAI Studio
- Adjust thresholds per category
- Add **blocklists** for specific terms
- Requires approval for some configurations

## 7.9 Responsible AI for Generative AI

| Principle | Application |
|-----------|------------|
| **Transparency** | Disclose AI-generated content to users |
| **Grounding** | Use RAG to reduce hallucinations |
| **Metaprompt** | System message to constrain behavior |
| **Content Filtering** | Block harmful outputs |
| **Human Oversight** | Review AI outputs before publication |
| **Red Teaming** | Test for adversarial inputs |

## Key Takeaways

1. **Azure OpenAI** uses deployment names, not model names in API calls
2. **System messages** define behavior and constraints
3. **RAG (On Your Data)** = Azure OpenAI + Azure AI Search for grounded answers
4. **`in_scope: true`** restricts answers to provided data only
5. **Embeddings** produce vectors; ada-002 = 1536 dimensions
6. **Content filters** are built-in with 4 categories and 4 severity levels
7. **Function calling** lets models request external data/actions
8. **Temperature** controls randomness; lower = more deterministic

## References

- [Azure OpenAI Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [Azure OpenAI Models](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models)
- [Chat Completions API](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
- [Prompt Engineering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)
- [Azure OpenAI on Your Data](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
- [Embeddings](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/understand-embeddings)
- [Content Filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
- [Function Calling](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/function-calling)
- [DALL-E](https://learn.microsoft.com/en-us/azure/ai-services/openai/dall-e-quickstart)
- [Responsible AI for Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/responsible-ai)
