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

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Azure OpenAI

### Azure OpenAI vs OpenAI API

| Aspect | OpenAI API | Azure OpenAI |
|--------|------------|--------------|
| **Provider** | OpenAI (company) | Microsoft Azure |
| **Compliance** | Limited | Enterprise (SOC, HIPAA, etc.) |
| **Data Privacy** | OpenAI terms | Azure terms (your data not used for training) |
| **Network** | Public internet | Private endpoints, VNet |
| **Integration** | Standalone | Azure ecosystem |
| **Content Filters** | Moderation API | Built-in filters |
| **Support** | OpenAI support | Microsoft support |

### Deployment Model

```
Azure OpenAI Resource
├── Deployment: "gpt-4o-prod" (model: gpt-4o)
├── Deployment: "gpt-35-staging" (model: gpt-3.5-turbo)
├── Deployment: "embedding-ada" (model: text-embedding-ada-002)
└── Deployment: "dalle3" (model: dall-e-3)
```

**Key Point**: API calls use DEPLOYMENT NAME, not model name:
```python
response = client.chat.completions.create(
    model="gpt-4o-prod",  # This is YOUR deployment name
    messages=[...]
)
```

### Deployment Types Explained

| Type | Billing | Capacity | Best For |
|------|---------|----------|----------|
| **Standard** | Pay-per-token | Shared, variable | Development, variable workloads |
| **Provisioned (PTU)** | Reserved units | Guaranteed | Production, consistent workloads |
| **Global Standard** | Pay-per-token | Multi-region | High availability |

## Deep Dive: Chat Completions

### Understanding Message Roles

```python
messages = [
    {
        "role": "system",
        "content": "You are a helpful assistant for Contoso products."
    },
    {
        "role": "user", 
        "content": "What's your return policy?"
    },
    {
        "role": "assistant",
        "content": "Our return policy allows returns within 30 days."
    },
    {
        "role": "user",
        "content": "What if the item is damaged?"
    }
]
```

| Role | Purpose | Who Creates It |
|------|---------|----------------|
| **system** | Set AI behavior, rules, personality | You (developer) |
| **user** | User's questions/commands | Your application |
| **assistant** | AI's previous responses | Included for context |

### System Message Best Practices

**Good System Message**:
```
You are a customer support assistant for Contoso Electronics.

Rules:
1. Only answer questions about Contoso products
2. If you don't know the answer, say "I don't know"
3. Never make up product information
4. Be concise and professional
5. If asked about competitors, politely redirect

Context:
- Contoso sells laptops, tablets, and accessories
- Current promotion: 20% off all laptops
- Support hours: 9 AM - 5 PM EST
```

**Why Structure Matters**:
- Clear rules prevent hallucinations
- Context helps accurate responses
- Constraints limit scope

### Temperature and Top_p Explained

Both control randomness, but differently:

**Temperature**:
```
Temperature = 0.0: Almost deterministic (always picks most likely)
Temperature = 0.7: Balanced (default, good for most uses)
Temperature = 1.5: Very random (creative, may be incoherent)
```

**Top_p (Nucleus Sampling)**:
```
Top_p = 0.1: Consider only top 10% probable tokens
Top_p = 0.9: Consider top 90% probable tokens
Top_p = 1.0: Consider all tokens
```

**Exam Tip**: Use ONE, not both. If setting temperature, leave top_p at default (or vice versa).

### Token Management

```
Input:  "What is the capital of France?"  ≈ 7 tokens
Output: "The capital of France is Paris." ≈ 8 tokens
Total:  15 tokens (billed)
```

**Token Limits**:
| Model | Context Window |
|-------|---------------|
| gpt-4o | 128K tokens |
| gpt-4 | 8K or 32K tokens |
| gpt-3.5-turbo | 16K tokens |

**max_tokens** limits OUTPUT length only:
```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    max_tokens=500  # Output limited to ~500 tokens
)
```

## Deep Dive: Prompt Engineering

### Zero-Shot vs Few-Shot

**Zero-Shot** (no examples):
```
System: Classify the sentiment of the text as positive, negative, or neutral.
User: The product arrived on time and works great!
```

**Few-Shot** (with examples):
```
System: Classify sentiment. Examples:
  - "I love this!" → positive
  - "Terrible experience" → negative
  - "It arrived today" → neutral

User: The product arrived on time and works great!
```

**When to Use**:
| Approach | Use When |
|----------|----------|
| Zero-shot | Simple, common tasks |
| Few-shot | Complex tasks, specific formats needed |
| Chain-of-thought | Reasoning/math problems |

### Chain-of-Thought Prompting

For complex reasoning:

```
User: If I have 15 apples and give away 3, then buy 7 more, how many do I have?

Poor prompt result: "19" (might be wrong)

Chain-of-thought prompt:
"...think step by step and explain your reasoning."

Better result:
"Step 1: Start with 15 apples
 Step 2: Give away 3: 15 - 3 = 12 apples
 Step 3: Buy 7 more: 12 + 7 = 19 apples
 Answer: 19 apples"
```

### Output Format Control

```python
messages = [
    {
        "role": "system",
        "content": """You are a product information assistant.
        Always respond in this JSON format:
        {
          "product_name": "...",
          "price": "...",
          "in_stock": true/false,
          "description": "..."
        }"""
    },
    {"role": "user", "content": "Tell me about the Surface Pro"}
]
```

**For reliable JSON**, use `response_format`:
```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    response_format={"type": "json_object"}
)
```

## Deep Dive: RAG (Retrieval-Augmented Generation)

### What is RAG?

RAG combines:
1. **Retrieval**: Find relevant documents
2. **Augmentation**: Add documents to context
3. **Generation**: Generate answer using documents

```
User Question: "What is our remote work policy?"
        ↓
Azure AI Search retrieves:
- HR Policy Document (relevant section)
- Employee Handbook (chapter 5)
        ↓
Combined prompt to Azure OpenAI:
"Using these documents, answer: What is our remote work policy?"
        ↓
Grounded Answer: "According to our HR Policy, employees may work 
remotely up to 3 days per week with manager approval..."
```

### Azure OpenAI on Your Data

Built-in RAG without custom code:

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's our PTO policy?"}],
    extra_body={
        "data_sources": [{
            "type": "azure_search",
            "parameters": {
                "endpoint": "https://search.search.windows.net",
                "index_name": "hr-policies",
                "authentication": {
                    "type": "api_key",
                    "key": "<search-key>"
                },
                "query_type": "semantic",
                "in_scope": True,
                "strictness": 3,
                "top_n_documents": 5
            }
        }]
    }
)
```

### RAG Parameters Explained

| Parameter | What It Does |
|-----------|--------------|
| `in_scope` | `true`: Only answer from data. `false`: Can use general knowledge |
| `strictness` | 1-5. Higher = stricter adherence to source documents |
| `top_n_documents` | Number of documents to retrieve (3-20) |
| `query_type` | `simple`, `semantic`, `vector`, or hybrid |

### Query Types for RAG

| Type | How It Works | Best For |
|------|--------------|----------|
| `simple` | Keyword matching | Basic search |
| `semantic` | Meaning-based re-ranking | Natural questions |
| `vector` | Embedding similarity | Semantic similarity |
| `vectorSemanticHybrid` | Keywords + vectors + semantic | Best accuracy |

## Deep Dive: Embeddings

### What are Embeddings?

Embeddings convert text to numbers (vectors) that capture meaning:

```
"cat" → [0.1, 0.5, -0.3, 0.8, ...]
"dog" → [0.12, 0.48, -0.28, 0.75, ...]  (similar to cat)
"car" → [-0.5, 0.1, 0.8, -0.2, ...]     (different from cat)
```

**Similar meanings = Similar vectors**

### Embedding Models

| Model | Dimensions | Notes |
|-------|-----------|-------|
| `text-embedding-ada-002` | 1536 | Legacy, still widely used |
| `text-embedding-3-small` | 512 or 1536 | Cost-effective |
| `text-embedding-3-large` | 256-3072 | Highest quality |

### Using Embeddings

```python
# Create embedding
response = client.embeddings.create(
    model="text-embedding-ada-002",
    input="What is machine learning?"
)

vector = response.data[0].embedding
# [0.01, -0.02, 0.05, ...] (1536 numbers)
```

### Common Embedding Use Cases

1. **Semantic Search**
   - Convert documents to vectors
   - Convert query to vector
   - Find most similar vectors

2. **Clustering**
   - Group similar documents together
   - Find document categories

3. **RAG Index**
   - Store vectors in Azure AI Search
   - Retrieve by semantic similarity

## Deep Dive: Function Calling

### What is Function Calling?

Let the model decide when to call external tools/APIs:

```
User: "What's the weather in Seattle?"
        ↓
Model: "I need to call get_weather function with location='Seattle'"
        ↓
Your code calls actual weather API
        ↓
Model uses result to respond: "It's 65°F and sunny in Seattle"
```

### Defining Functions

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
                        "description": "City name"
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
```

### Function Calling Flow

```python
# Step 1: Send message with tools
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Seattle?"}],
    tools=tools
)

# Step 2: Check if model wants to call a function
if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    
    # Step 3: Execute the function
    if tool_call.function.name == "get_weather":
        args = json.loads(tool_call.function.arguments)
        weather_result = get_weather(args["location"])  # Your code
        
        # Step 4: Send result back
        messages.append(response.choices[0].message)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": weather_result
        })
        
        # Step 5: Get final response
        final_response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools
        )
```

## Deep Dive: Image Generation (DALL-E)

### DALL-E 3 Parameters

```python
response = client.images.generate(
    model="dall-e-3",
    prompt="A serene Japanese garden with cherry blossoms",
    n=1,                    # Only 1 supported for DALL-E 3
    size="1024x1024",       # 1024x1024, 1024x1792, 1792x1024
    quality="hd",           # standard, hd
    style="natural"         # natural, vivid
)
```

### Best Practices for Prompts

| Approach | Example |
|----------|---------|
| Be specific | "A golden retriever puppy playing in autumn leaves" |
| Include style | "...in watercolor style" |
| Describe composition | "Close-up shot of..." |
| Specify lighting | "...golden hour lighting" |

### Revised Prompts

DALL-E 3 rewrites your prompt for better results:

```python
response = client.images.generate(
    prompt="a cat",
    ...
)

print(response.data[0].revised_prompt)
# "A photorealistic image of a fluffy orange tabby cat 
#  sitting on a windowsill, looking out at a garden..."
```

## Content Filtering Deep Dive

### Built-in Categories

| Category | What It Catches |
|----------|-----------------|
| **Hate** | Discrimination, slurs, hate speech |
| **Sexual** | Explicit/suggestive content |
| **Violence** | Gore, harm, violent content |
| **Self-harm** | Self-injury, suicide content |

### How Filtering Works

```
User Input    →   Input Filter   →   Model   →   Output Filter   →   Response
              ↓                             ↓
           Block/Allow                 Block/Allow
```

### Annotation Response

When filtered, you get annotations:

```json
{
  "error": {
    "code": "content_filter",
    "message": "Content filtered",
    "innererror": {
      "content_filter_result": {
        "hate": {"filtered": true, "severity": "high"},
        "sexual": {"filtered": false, "severity": "safe"},
        "violence": {"filtered": false, "severity": "safe"},
        "self_harm": {"filtered": false, "severity": "safe"}
      }
    }
  }
}
```

### Custom Content Filters

In Azure OpenAI Studio, you can:
- Adjust thresholds (allow Low/Medium, block High)
- Add custom blocklists
- Enable Prompt Shields (jailbreak detection)
- Configure per-deployment

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Introduction to Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/explore-azure-openai/)
- [Build natural language solutions with Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/build-language-solution-azure-openai/)
- [Apply prompt engineering with Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/apply-prompt-engineering-azure-openai/)
- [Implement Retrieval Augmented Generation (RAG) with Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/use-own-data-azure-openai/)
- [Generate images with Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/generate-images-azure-openai/)

### Official Documentation
- [Azure OpenAI Service documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [Models overview](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models)
- [Prompt engineering techniques](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)
- [On your data (RAG)](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
- [Content filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
- [Function calling](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/function-calling)
