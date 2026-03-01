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
