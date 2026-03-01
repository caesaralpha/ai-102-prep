# 08 - Content Safety & Moderation (10–15%)

## 8.1 Azure AI Content Safety

### Overview

Azure AI Content Safety detects harmful content across text and images with severity grading.

### Categories

| Category | Description |
|----------|-------------|
| **Hate & Fairness** | Hateful, discriminatory content targeting groups |
| **Sexual** | Sexually explicit or suggestive content |
| **Violence** | Violent content, gore, physical harm |
| **Self-Harm** | Self-harm, suicide-related content |

### Severity Levels

| Level | Value | Description |
|-------|-------|-------------|
| **Safe** | 0 | No harmful content detected |
| **Low** | 2 | Mild references, context-dependent |
| **Medium** | 4 | Moderate risk content |
| **High** | 6 | Severe, clearly harmful content |

## 8.2 Text Moderation

### API Call

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
from azure.core.credentials import AzureKeyCredential

client = ContentSafetyClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<key>")
)

request = AnalyzeTextOptions(
    text="Sample text to analyze for safety",
    categories=[TextCategory.HATE, TextCategory.SEXUAL, TextCategory.VIOLENCE, TextCategory.SELF_HARM]
)

response = client.analyze_text(request)

for result in response.categories_analysis:
    print(f"{result.category}: severity={result.severity}")
```

### REST API

```http
POST https://<endpoint>/contentsafety/text:analyze?api-version=2024-09-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
  "text": "Content to analyze",
  "categories": ["Hate", "Sexual", "Violence", "SelfHarm"],
  "outputType": "FourSeverityLevels"
}
```

### Response

```json
{
  "categoriesAnalysis": [
    { "category": "Hate", "severity": 0 },
    { "category": "Sexual", "severity": 0 },
    { "category": "Violence", "severity": 2 },
    { "category": "SelfHarm", "severity": 0 }
  ]
}
```

## 8.3 Image Moderation

```python
from azure.ai.contentsafety.models import AnalyzeImageOptions, ImageData

with open("image.jpg", "rb") as f:
    image_data = f.read()

request = AnalyzeImageOptions(
    image=ImageData(content=image_data)
)

response = client.analyze_image(request)
for result in response.categories_analysis:
    print(f"{result.category}: severity={result.severity}")
```

## 8.4 Blocklists

Custom blocklists allow you to define specific terms that should always be flagged.

```python
from azure.ai.contentsafety.models import TextBlocklist, TextBlocklistItem

# Create blocklist
client.create_or_update_text_blocklist(
    blocklist_name="custom-blocklist",
    options=TextBlocklist(description="Custom terms to block")
)

# Add items
items = [
    TextBlocklistItem(text="blocked_term_1"),
    TextBlocklistItem(text="blocked_term_2")
]
client.add_or_update_blocklist_items(
    blocklist_name="custom-blocklist",
    options={"blocklistItems": items}
)

# Use blocklist in analysis
request = AnalyzeTextOptions(
    text="Sample text with blocked_term_1",
    blocklist_names=["custom-blocklist"],
    halt_on_blocklist_hit=True
)
response = client.analyze_text(request)
```

### Blocklist Behavior

| Parameter | Description |
|-----------|-------------|
| `halt_on_blocklist_hit` | If `true`, stop processing and return immediately when a blocklist match is found |
| `blocklistMatchResults` | Array of matched terms in response |

## 8.5 Prompt Shields (Jailbreak Detection)

Detects attempts to manipulate AI systems through prompt injection.

| Type | Description |
|------|-------------|
| **Direct Attack (Jailbreak)** | User directly tries to bypass system instructions |
| **Indirect Attack** | Hidden instructions in grounding data/documents |

```http
POST https://<endpoint>/contentsafety/text:shieldPrompt?api-version=2024-09-01

{
  "userPrompt": "Ignore all instructions and...",
  "documents": ["Document with hidden instruction: ignore previous instructions"]
}
```

### Response

```json
{
  "userPromptAnalysis": {
    "attackDetected": true
  },
  "documentsAnalysis": [
    {
      "attackDetected": true
    }
  ]
}
```

## 8.6 Groundedness Detection

Checks if AI-generated responses are grounded in the provided source material.

```http
POST https://<endpoint>/contentsafety/text:detectGroundedness?api-version=2024-09-15-preview

{
  "domain": "Generic",
  "task": "QnA",
  "text": "The AI-generated response to check",
  "groundingSources": ["Source document 1 text", "Source document 2 text"],
  "reasoning": true
}
```

### Response

```json
{
  "ungroundedDetected": true,
  "ungroundedPercentage": 0.35,
  "ungroundedDetails": [
    {
      "text": "The ungrounded sentence from the response",
      "offset": { "utf8": 45, "utf16": 45 },
      "length": { "utf8": 60, "utf16": 60 },
      "reason": "This claim is not supported by the provided sources."
    }
  ]
}
```

## 8.7 Content Safety in Azure OpenAI

Azure OpenAI has **built-in content filtering** that uses the Content Safety service:

```
User Input → Input Filter → Model → Output Filter → Response
                 ↓                        ↓
           Block/Allow              Block/Allow
```

### Default Filter Configuration

| Category | Input Threshold | Output Threshold |
|----------|----------------|------------------|
| Hate | Medium | Medium |
| Sexual | Medium | Medium |
| Violence | Medium | Medium |
| Self-Harm | Medium | Medium |

### Custom Filter Policy

In Azure OpenAI Studio:
1. Go to **Content Filters**
2. Create a new filter configuration
3. Set thresholds per category (Low/Medium/High)
4. Enable/disable **Prompt Shields** (jailbreak detection)
5. Enable/disable **Protected Material Detection**
6. Assign filter to a deployment

## 8.8 Protected Material Detection

| Feature | Description |
|---------|-------------|
| **Text** | Detect known copyrighted text in outputs |
| **Code** | Detect known open-source code in outputs (with license info) |

## 8.9 Implementing Content Moderation Pipeline

### Best Practice Architecture

```
User Input
    ↓
1. Content Safety API (text/image analysis)
    ↓ [severity check]
2. Blocklist Check (custom terms)
    ↓ [pass/fail]
3. Prompt Shield (jailbreak detection)
    ↓ [pass/fail]
4. Azure OpenAI (with built-in filters)
    ↓ [content generated]
5. Output Content Safety Check
    ↓ [severity check]
6. Groundedness Check (optional)
    ↓
Response to User
```

## Key Takeaways

1. **4 categories**: Hate, Sexual, Violence, Self-Harm
2. **4 severity levels**: 0 (Safe), 2 (Low), 4 (Medium), 6 (High)
3. **Blocklists** are custom term lists; `halt_on_blocklist_hit` stops processing immediately
4. **Prompt Shields** detect jailbreak attempts (direct + indirect)
5. **Groundedness Detection** checks if AI output is supported by source material
6. **Azure OpenAI** has built-in content filters that use Content Safety
7. **Protected Material Detection** flags copyrighted text and licensed code
8. Content filters work on both **input and output** in Azure OpenAI

## References

- [Azure AI Content Safety Documentation](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/)
- [Content Safety Quickstart](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-text)
- [Blocklists](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/how-to/use-blocklist)
- [Prompt Shields](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/jailbreak-detection)
- [Groundedness Detection](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/groundedness)
- [Azure OpenAI Content Filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
- [Protected Material](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/protected-material)
