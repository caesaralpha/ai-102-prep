# 02 - Azure AI Services Overview

## 2.1 Azure AI Services Map

```
┌──────────────────────────────────────────────────────────┐
│                    Azure AI Services                      │
├──────────────┬──────────────┬──────────────┬─────────────┤
│   Vision     │   Language   │   Speech     │  Decision   │
├──────────────┼──────────────┼──────────────┼─────────────┤
│ Image        │ Sentiment    │ Speech-to-   │ Content     │
│ Analysis     │ Analysis     │ Text         │ Safety      │
│              │              │              │             │
│ Custom       │ Key Phrase   │ Text-to-     │ Anomaly     │
│ Vision       │ Extraction   │ Speech       │ Detector    │
│              │              │              │ (retired)   │
│ Face API     │ NER          │ Translation  │             │
│              │              │              │             │
│ OCR          │ Language     │ Speaker      │             │
│              │ Understanding│ Recognition  │             │
│              │ (CLU)        │              │             │
│ Spatial      │              │ Intent       │             │
│ Analysis     │ QnA / Custom │ Recognition  │             │
│              │ Question     │              │             │
│ Video        │ Answering    │ Pronunciation│             │
│ Indexer      │              │ Assessment   │             │
│              │ Summarization│              │             │
│              │              │              │             │
│              │ Translator   │              │             │
└──────────────┴──────────────┴──────────────┴─────────────┘
```

## 2.2 Service Endpoints and SDKs

### Common SDK Pattern (Python)

```python
# Install
# pip install azure-ai-textanalytics azure-identity

from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# Using Key
client = TextAnalyticsClient(
    endpoint="https://<resource-name>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<api-key>")
)

# Using Microsoft Entra ID (Managed Identity)
from azure.identity import DefaultAzureCredential
client = TextAnalyticsClient(
    endpoint="https://<resource-name>.cognitiveservices.azure.com/",
    credential=DefaultAzureCredential()
)
```

### REST API Common Headers

```http
POST https://<endpoint>/language/:analyze-text?api-version=2023-04-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <subscription-key>
```

## 2.3 Resource Provisioning

### Azure CLI

```bash
# Create a multi-service resource
az cognitiveservices account create \
  --name my-ai-services \
  --resource-group my-rg \
  --kind CognitiveServices \
  --sku S0 \
  --location eastus \
  --yes

# Create a single-service resource (e.g., Language)
az cognitiveservices account create \
  --name my-language-service \
  --resource-group my-rg \
  --kind TextAnalytics \
  --sku S0 \
  --location eastus \
  --yes

# Get keys
az cognitiveservices account keys list \
  --name my-ai-services \
  --resource-group my-rg

# Regenerate key
az cognitiveservices account keys regenerate \
  --name my-ai-services \
  --resource-group my-rg \
  --key-name key1
```

### Resource Kinds

| Kind Value | Service |
|-----------|---------|
| `CognitiveServices` | Multi-service |
| `TextAnalytics` | Language Service |
| `ComputerVision` | Computer Vision |
| `Face` | Face API |
| `SpeechServices` | Speech Service |
| `FormRecognizer` | Document Intelligence |
| `OpenAI` | Azure OpenAI |
| `ContentSafety` | Content Safety |

## 2.4 Data Privacy and Compliance

### Data Handling

| Aspect | Detail |
|--------|--------|
| **Data at Rest** | Encrypted with Microsoft-managed keys (or CMK for some services) |
| **Data in Transit** | TLS 1.2+ enforced |
| **Data Retention** | Most services don't store customer data (stateless) |
| **Customer Managed Keys** | Supported for some services (Language, Speech, etc.) |
| **Data Residency** | Data processed in the region where the resource is deployed |

### Responsible AI Principles

1. **Fairness** - AI systems should treat all people fairly
2. **Reliability & Safety** - AI systems should perform reliably and safely
3. **Privacy & Security** - AI systems should be secure and respect privacy
4. **Inclusiveness** - AI systems should empower everyone
5. **Transparency** - AI systems should be understandable
6. **Accountability** - People should be accountable for AI systems

> **Exam Tip**: Microsoft's Responsible AI principles are frequently tested. Know all six principles and be able to apply them to scenarios.

## 2.5 Availability and Redundancy

### Region Pairing

- Azure AI Services are deployed per-region
- For high availability, deploy to **multiple regions**
- Use **Traffic Manager** or **Front Door** for failover routing

### SLA

| Tier | SLA |
|------|-----|
| Standard | 99.9% availability |

## Key Takeaways

1. Know the difference between **multi-service** and **single-service** resources
2. Understand the **SDK pattern**: endpoint + credential → client
3. Know the **Kind** values for Azure CLI provisioning
4. **Responsible AI** principles are testable: Fairness, Reliability, Privacy, Inclusiveness, Transparency, Accountability
5. **Data encryption** always: at rest (AES 256) + in transit (TLS 1.2+)

## References

- [Azure AI Services Documentation](https://learn.microsoft.com/en-us/azure/ai-services/)
- [Azure AI Services REST API Reference](https://learn.microsoft.com/en-us/rest/api/cognitiveservices/)
- [Azure AI Services SDKs](https://learn.microsoft.com/en-us/azure/ai-services/reference/sdk-package)
- [Microsoft Responsible AI Principles](https://www.microsoft.com/en-us/ai/responsible-ai)
- [Azure AI Services Pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/)
- [Azure AI Services Compliance](https://learn.microsoft.com/en-us/azure/ai-services/security-features)

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding the Azure AI Services Ecosystem

### The Evolution of Azure AI

Azure AI Services (formerly "Cognitive Services") is Microsoft's umbrella platform for pre-built AI capabilities. The naming has evolved:
- **2015-2020**: Azure Cognitive Services
- **2023+**: Azure AI Services (current branding)

The services are organized into categories based on their AI domain:

| Category | Services | What They Do |
|----------|----------|--------------|
| **Vision** | Azure AI Vision, Custom Vision, Face | Analyze images, detect objects, read text |
| **Language** | Azure AI Language, Translator | Understand text, translate, answer questions |
| **Speech** | Speech Service | Convert speech↔text, recognize speakers |
| **Decision** | Content Safety | Moderate content, ensure safety |
| **Generative** | Azure OpenAI | Generate text, code, images |
| **Search** | Azure AI Search | Index and search content with AI enrichment |

### Retired/Deprecated Services (Do Not Use)

| Service | Status | Replacement |
|---------|--------|-------------|
| **LUIS** | Retired (October 2025) | Conversational Language Understanding (CLU) |
| **QnA Maker** | Retired (October 2025) | Custom Question Answering |
| **Anomaly Detector** | Retired (October 2026) | Multivariate Anomaly Detection in Azure AI Services |
| **Personalizer** | Retired (October 2026) | No direct replacement |
| **Metrics Advisor** | Retired (October 2026) | No direct replacement |

## Deep Dive: SDK Architecture

### The Credential + Client Pattern

All Azure AI SDKs follow a consistent pattern:

```python
# 1. Create a Credential (how to authenticate)
credential = AzureKeyCredential("<your-key>")
# OR
credential = DefaultAzureCredential()  # For Entra ID

# 2. Create a Client (how to communicate)
client = TextAnalyticsClient(
    endpoint="https://your-resource.cognitiveservices.azure.com/",
    credential=credential
)

# 3. Make API calls
response = client.analyze_sentiment(["I love this product!"])
```

### Understanding `DefaultAzureCredential`

`DefaultAzureCredential` is a "smart" credential that tries multiple authentication methods in order:

```
1. Environment Variables (AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_CLIENT_SECRET)
2. Workload Identity (Kubernetes)
3. Managed Identity (System or User-assigned)
4. Azure CLI credentials
5. Azure PowerShell credentials
6. Interactive Browser login
```

**Why This Matters**: The same code works in development (uses your CLI login) and production (uses Managed Identity) without changes.

### Error Handling Best Practices

```python
from azure.core.exceptions import HttpResponseError, ServiceRequestError

try:
    response = client.analyze_sentiment(documents)
except ServiceRequestError as e:
    # Network error - no connection to Azure
    print(f"Network error: {e}")
except HttpResponseError as e:
    # Server error - Azure returned an error
    if e.status_code == 401:
        print("Authentication failed - check your key")
    elif e.status_code == 429:
        print("Rate limited - too many requests")
    elif e.status_code == 400:
        print(f"Bad request: {e.message}")
```

## Understanding Provisioning with Azure CLI

### Resource Kinds Explained

When creating resources with Azure CLI, the `--kind` parameter specifies what type of service you're creating:

```bash
# Multi-service (access to all services)
az cognitiveservices account create --kind CognitiveServices

# Single-service (specific service)
az cognitiveservices account create --kind TextAnalytics
az cognitiveservices account create --kind ComputerVision
az cognitiveservices account create --kind SpeechServices
```

### Full CLI Example with Best Practices

```bash
# Variables for reusability
RESOURCE_GROUP="rg-ai-production"
LOCATION="eastus"
AI_SERVICE_NAME="ai-services-prod"

# Create resource group
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION

# Create multi-service resource with network restrictions
az cognitiveservices account create \
    --name $AI_SERVICE_NAME \
    --resource-group $RESOURCE_GROUP \
    --kind CognitiveServices \
    --sku S0 \
    --location $LOCATION \
    --custom-domain $AI_SERVICE_NAME \
    --yes

# Configure network rules (block public access)
az cognitiveservices account network-rule update \
    --name $AI_SERVICE_NAME \
    --resource-group $RESOURCE_GROUP \
    --default-action Deny

# Add specific IP to allow list
az cognitiveservices account network-rule add \
    --name $AI_SERVICE_NAME \
    --resource-group $RESOURCE_GROUP \
    --ip-address "203.0.113.50"

# Get the keys (store securely!)
az cognitiveservices account keys list \
    --name $AI_SERVICE_NAME \
    --resource-group $RESOURCE_GROUP
```

## Deep Dive: Responsible AI Principles

### Why Responsible AI Matters for AI-102

The AI-102 exam tests your understanding of Responsible AI principles and how to apply them in real scenarios.

### 1. Fairness

**What It Means**: AI systems should treat all people fairly and not discriminate based on age, gender, race, religion, etc.

**How to Implement**:
- Test models with diverse datasets
- Check for bias in training data
- Monitor predictions across demographic groups
- Use tools like Fairlearn for bias detection

**Exam Scenario Example**: 
*"A company's hiring AI recommends fewer female candidates. What should be done?"*
Answer: Analyze training data for historical bias, test model across demographics, implement bias mitigation techniques.

### 2. Reliability & Safety

**What It Means**: AI systems should perform predictably and safely, especially in critical scenarios.

**How to Implement**:
- Extensive testing before deployment
- Graceful error handling
- Fallback mechanisms when AI confidence is low
- Human oversight for high-stakes decisions

**Exam Scenario Example**:
*"A medical AI suggests treatments. How to ensure reliability?"*
Answer: Set confidence thresholds, require human review for critical decisions, implement logging for all recommendations.

### 3. Privacy & Security

**What It Means**: AI systems should protect user data and respect privacy.

**How to Implement**:
- Use Managed Identities (no keys in code)
- Enable data encryption (at rest and in transit)
- Minimize data collection
- Implement data retention policies
- Use Customer Managed Keys for sensitive workloads

### 4. Inclusiveness

**What It Means**: AI systems should empower everyone and engage people regardless of abilities.

**How to Implement**:
- Support multiple languages (Translator)
- Provide alternatives to visual content (image descriptions)
- Support speech interfaces (Speech Service)
- Design for various accessibility needs

### 5. Transparency

**What It Means**: AI systems should be understandable. Users should know they're interacting with AI.

**How to Implement**:
- Disclose when AI is being used
- Explain AI decisions when asked
- Provide confidence scores
- Document how models were trained

**Exam Scenario Example**:
*"A chatbot uses AI. What transparency measures are needed?"*
Answer: Display "AI-generated content" disclaimers, provide ways for users to request human assistance.

### 6. Accountability

**What It Means**: People should be accountable for AI systems.

**How to Implement**:
- Enable diagnostic logging
- Implement audit trails
- Define roles and responsibilities
- Have processes to address AI harms

## Understanding Data Handling and Compliance

### Where Your Data Goes

When you make an API call:

1. **Data in Transit**: Your request travels encrypted (TLS 1.2+) to Azure
2. **Data at Rest**: Most services are stateless (data not stored after processing)
3. **Data Location**: Processed in the region where your resource is deployed

### Services with Data Storage

Some services store data for training/customization:

| Service | What's Stored |
|---------|---------------|
| Custom Vision | Training images |
| Custom Speech | Audio recordings |
| CLU / Language | Utterances and entities |
| Document Intelligence | Training documents |
| Face | Face data (enrolled faces) |

### Customer Managed Keys (CMK)

For sensitive workloads, you can encrypt AI service data with your own keys stored in Azure Key Vault:

```
Your Key Vault → Your Encryption Key → Encrypts AI Service Data
(You control the key - Microsoft cannot access your data)
```

**Supported Services**: Azure AI Language, Speech, Translator, Document Intelligence, Face

## Understanding High Availability

### Regional Deployment Strategy

Azure AI Services are regional. For high availability:

```
                    ┌─────────────────┐
                    │   Azure Front   │
                    │      Door       │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
    ┌─────────────────┐           ┌─────────────────┐
    │  East US Region │           │  West US Region │
    │  AI Services    │           │  AI Services    │
    └─────────────────┘           └─────────────────┘
              │                             │
              └──────────────┬──────────────┘
                             │
                    (Same endpoints via
                     custom domains)
```

### Implementation Pattern

```python
regions = [
    "https://eastus-ai.cognitiveservices.azure.com/",
    "https://westus-ai.cognitiveservices.azure.com/"
]

def call_with_failover(documents):
    for endpoint in regions:
        try:
            client = TextAnalyticsClient(endpoint, credential)
            return client.analyze_sentiment(documents)
        except Exception as e:
            print(f"Region failed: {endpoint}, trying next...")
    raise Exception("All regions failed")
```

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Fundamentals of Azure AI Services](https://learn.microsoft.com/en-us/training/modules/fundamentals-of-ai-services/)
- [Get started with Azure AI Services](https://learn.microsoft.com/en-us/training/modules/get-started-ai-services/)
- [Identify principles of Responsible AI](https://learn.microsoft.com/en-us/training/modules/responsible-ai-principles/)

### Official Documentation
- [Azure AI Services documentation](https://learn.microsoft.com/en-us/azure/ai-services/)
- [SDK packages and samples](https://learn.microsoft.com/en-us/azure/ai-services/reference/sdk-package)
- [Azure regions for AI Services](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/?products=cognitive-services)
- [Service-level agreement for Azure AI Services](https://azure.microsoft.com/en-us/support/legal/sla/cognitive-services/)
