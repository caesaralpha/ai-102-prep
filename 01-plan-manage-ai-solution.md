# 01 - Plan and Manage an Azure AI Solution (15–20%)

## 1.1 Select the Appropriate Azure AI Service

### Azure AI Services Hierarchy

```
Azure AI Services (Umbrella)
├── Azure AI Vision
├── Azure AI Language
├── Azure AI Speech
├── Azure AI Translator
├── Azure AI Document Intelligence
├── Azure AI Search (Knowledge Mining)
├── Azure OpenAI Service (Generative AI)
├── Azure AI Content Safety
├── Azure AI Personalizer (retired)
├── Azure AI Anomaly Detector (retired → merged into Multivariate)
└── Azure AI Bot Service
```

### Multi-Service vs Single-Service Resources

| Feature | Multi-Service Resource | Single-Service Resource |
|---------|----------------------|------------------------|
| **Endpoint** | Single endpoint for all services | Separate endpoint per service |
| **Key** | One key for all services | One key per service |
| **Billing** | Combined billing | Separate billing per service |
| **Use Case** | Prototyping, multi-service apps | Production, granular control |
| **Resource Name** | `Microsoft.CognitiveServices/accounts` (Kind: `CognitiveServices`) | `Microsoft.CognitiveServices/accounts` (Kind: specific like `Face`, `TextAnalytics`) |

### Pricing Tiers

| Tier | Description |
|------|-------------|
| **Free (F0)** | Limited transactions/month, for development/testing |
| **Standard (S0/S1)** | Pay-as-you-go, production workloads |
| **Commitment Tier** | Discounted rate for committed usage volume |

> **Exam Tip**: Know when to use multi-service vs single-service. Multi-service = one endpoint, one key, combined billing. Single-service = separate control, separate billing.

## 1.2 Plan and Configure Security

### Authentication Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **Subscription Key** | API key passed in `Ocp-Apim-Subscription-Key` header | Quick testing, simple apps |
| **Azure Active Directory (Microsoft Entra ID)** | OAuth 2.0 token-based auth | Production, enterprise apps |
| **Managed Identity** | System/User-assigned identity | Azure-to-Azure, no credentials in code |
| **Key Vault** | Store keys securely | When keys must be used but secured |

### Key Rotation Strategy

```
1. Application uses Key 1
2. Regenerate Key 2
3. Update application to use Key 2
4. Regenerate Key 1
5. (Zero downtime achieved)
```

### Network Security

| Feature | Description |
|---------|-------------|
| **Virtual Network (VNet)** | Restrict access to specific VNets |
| **Private Endpoints** | Access via private IP within VNet |
| **Service Endpoints** | Route traffic through Azure backbone |
| **IP Firewall** | Allow/deny specific IP addresses |

### RBAC Roles for AI Services

| Role | Permissions |
|------|------------|
| **Cognitive Services Contributor** | Create, manage, delete resources; read keys |
| **Cognitive Services User** | Read keys and make API calls |
| **Cognitive Services Custom Vision Contributor** | Full access to Custom Vision projects |
| **Cognitive Services Custom Vision Trainer** | Train and publish models |
| **Cognitive Services Custom Vision Deployment** | Publish/unpublish models |

## 1.3 Plan and Configure Networking

### Private Endpoints

```
Your VNet → Private Endpoint → Azure AI Services
(Traffic stays on Microsoft backbone network)
```

- Private endpoint uses a private IP from your VNet address space
- DNS resolution must be configured (Private DNS Zone)
- Can coexist with public endpoint (or disable public access)

### Virtual Network Rules

```json
{
  "networkAcls": {
    "defaultAction": "Deny",
    "virtualNetworkRules": [
      {
        "id": "/subscriptions/.../subnets/default"
      }
    ],
    "ipRules": [
      {
        "value": "203.0.113.0/24"
      }
    ]
  }
}
```

## 1.4 Monitor Azure AI Services

### Diagnostic Settings

| Destination | Use Case |
|-------------|----------|
| **Log Analytics Workspace** | Query logs with KQL, create alerts |
| **Storage Account** | Long-term archival |
| **Event Hub** | Stream to external SIEM |

### Key Metrics to Monitor

| Metric | Description |
|--------|-------------|
| `TotalCalls` | Total API calls |
| `SuccessfulCalls` | Successful API calls |
| `TotalErrors` | Calls with error response |
| `BlockedCalls` | Calls exceeding rate/quota |
| `ClientErrors` | 4xx errors |
| `ServerErrors` | 5xx errors |
| `Latency` | Time to complete request |
| `DataIn` / `DataOut` | Data volume |

### Creating Alerts

```
Azure Monitor → Alerts → Create Alert Rule
  → Scope: AI Services resource
  → Condition: e.g., ServerErrors > 5 in 5 minutes
  → Action Group: Email, SMS, Webhook, Logic App
```

## 1.5 Implement Container Deployment

### Supported Containers

Azure AI Services can be deployed as Docker containers for:
- **On-premises** deployment
- **Edge** computing scenarios
- **Data sovereignty** requirements
- **Low-latency** requirements

### Container Billing

> **Important**: Containers must connect to Azure for billing. They send usage info to Azure but processing happens locally.

```yaml
# Docker run example
docker run --rm -it -p 5000:5000 \
  --memory 4g --cpus 4 \
  mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest \
  Eula=accept \
  Billing={ENDPOINT_URI} \
  ApiKey={API_KEY}
```

### Required Parameters

| Parameter | Description |
|-----------|-------------|
| `Eula` | Must be set to `accept` |
| `Billing` | Endpoint URI of the Azure resource |
| `ApiKey` | Key from the Azure resource |

### Disconnected Containers

- Available for specific services with an **application and approval process**
- Allows operation without internet connectivity
- Uses a **commitment tier** pricing model
- Must periodically connect for usage reporting

## Key Takeaways

1. **Multi-service resource** = one endpoint/key for all services; **Single-service** = granular control
2. **Managed Identity** is the preferred authentication for Azure-to-Azure
3. **Key rotation** uses the two-key strategy for zero downtime
4. **Containers** still require Azure billing connection (except disconnected containers)
5. **Private Endpoints** keep traffic on the Microsoft backbone

## References

- [Azure AI Services Overview](https://learn.microsoft.com/en-us/azure/ai-services/what-are-ai-services)
- [Azure AI Services Security](https://learn.microsoft.com/en-us/azure/ai-services/security-features)
- [Azure AI Services Containers](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support)
- [Azure AI Services Diagnostic Logging](https://learn.microsoft.com/en-us/azure/ai-services/diagnostic-logging)
- [Azure AI Services Virtual Networks](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-virtual-networks)
- [Azure AI Services Authentication](https://learn.microsoft.com/en-us/azure/ai-services/authentication)
