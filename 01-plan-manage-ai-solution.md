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

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Multi-Service vs Single-Service Resources

### What is a Multi-Service Resource?

A **multi-service resource** (also called "Azure AI services" resource) is a single Azure resource that gives you access to multiple AI services through one endpoint and one API key. Think of it as an "all-access pass" to Azure AI.

**When to Use Multi-Service:**
- **Prototyping and development**: Quickly test multiple services without creating separate resources
- **Simple applications**: Apps that use multiple AI capabilities (e.g., image analysis + text translation)
- **Cost tracking simplicity**: One bill for all AI services used

**Example Scenario**: You're building a travel app that:
1. Analyzes photos users upload (Vision)
2. Detects the language of user reviews (Language)
3. Translates content (Translator)

With a multi-service resource, you configure ONE endpoint and ONE key for all three capabilities.

### What is a Single-Service Resource?

A **single-service resource** is dedicated to one specific AI service. It gives you granular control over that service.

**When to Use Single-Service:**
- **Production workloads**: Need separate scaling and billing
- **Security isolation**: Different teams manage different services
- **Regional requirements**: Deploy specific services to specific regions
- **Service-specific features**: Some features require their own resource (e.g., Custom Vision training)

**Example Scenario**: Your company has:
- A computer vision team that needs isolated billing
- A chatbot team that needs to track Language service costs separately
- Compliance requirements that restrict where certain data is processed

## Deep Dive: Authentication Methods

### Subscription Keys (API Keys)

The simplest authentication method. You pass a key in every API request.

```http
Ocp-Apim-Subscription-Key: YOUR_KEY_HERE
```

**Pros:**
- Simple to implement
- Works immediately after resource creation

**Cons:**
- Keys can be leaked or stolen
- Difficult to manage at scale
- No fine-grained access control

**Best Practice**: Never hardcode keys in source code. Use environment variables or Azure Key Vault.

### Microsoft Entra ID (Azure AD) Authentication

Uses OAuth 2.0 tokens instead of API keys. Your application authenticates with Microsoft Entra ID and receives a token.

```python
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient

# No API key needed - uses your Azure identity
credential = DefaultAzureCredential()
client = TextAnalyticsClient(endpoint="https://...", credential=credential)
```

**Pros:**
- More secure (tokens expire)
- Fine-grained access control with RBAC
- Audit logging of who accessed what

**Cons:**
- More complex setup
- Requires understanding of Azure RBAC

### Managed Identity (Recommended)

A special type of Microsoft Entra ID authentication for Azure-to-Azure scenarios. Azure automatically manages the identity for your service.

**System-Assigned Managed Identity:**
- Tied to a specific Azure resource (e.g., App Service)
- Automatically deleted when the resource is deleted
- One identity per resource

**User-Assigned Managed Identity:**
- Created as a standalone Azure resource
- Can be assigned to multiple resources
- Lifecycle is independent

**Example**: Your Azure Function needs to call Azure AI services. Instead of storing a key in the Function's configuration, you:
1. Enable system-assigned managed identity on the Function
2. Grant the Function's identity "Cognitive Services User" role on the AI resource
3. Use `DefaultAzureCredential()` in your code - it automatically uses the managed identity

## Understanding Network Security

### Why Network Security Matters

By default, Azure AI Services are accessible from the public internet. Anyone with your endpoint and key can make API calls. Network security restricts WHO can even reach your service.

### IP Firewall

The simplest approach - whitelist specific IP addresses or ranges.

**Use Cases:**
- Your office has a static public IP
- Your application runs on VMs with known IPs

**Limitations:**
- Dynamic IPs make management difficult
- Doesn't protect against insider threats within allowed IPs

### Virtual Network Service Endpoints

Route traffic from your VNet to Azure AI Services through the Azure backbone (not the public internet).

```
Your VM in VNet → Service Endpoint → Azure AI Services
(Traffic stays within Azure - never hits public internet)
```

**Benefits:**
- Reduced latency
- Enhanced security (traffic never leaves Azure backbone)
- Easy to configure

**Limitations:**
- Source IP still shows as public IP to the service
- Requires VNet configuration

### Private Endpoints (Most Secure)

Creates a private IP address for the Azure AI service inside your VNet. The service now appears as if it's running inside your network.

```
Your VM (10.0.1.5) → Private Endpoint (10.0.2.10) → Azure AI Services
(Service is accessed via PRIVATE IP - can disable public access entirely)
```

**Benefits:**
- Service is completely private (can disable public endpoint)
- Works with on-premises networks via VPN/ExpressRoute
- Prevents data exfiltration

**Key Setup Steps:**
1. Create a private endpoint in your VNet
2. Set up Private DNS Zone for name resolution
3. (Optional) Disable public network access on the AI resource

## Understanding Container Deployment

### Why Use Containers?

Azure AI Services can run as Docker containers. The AI processing happens locally, but billing still occurs through Azure.

**Use Cases:**
1. **Data Sovereignty**: Sensitive data cannot leave your premises
2. **Low Latency**: Edge scenarios where network latency is unacceptable
3. **Offline Scenarios**: Intermittent connectivity (with disconnected containers)
4. **Regulatory Compliance**: Data processing must occur in specific locations

### How Container Billing Works

Even though processing is local, containers must connect to Azure periodically to report usage. Azure bills you based on this reported usage.

```
┌─────────────────────────────────────────────┐
│ Your On-Premises / Edge                      │
│  ┌─────────────────────────────────┐        │
│  │ AI Services Container            │        │
│  │ (All processing happens here)    │        │
│  │                                  │        │
│  │ Billing data sent ──────────────│───────►│Azure (Usage tracking)
│  │                                  │        │
│  └─────────────────────────────────┘        │
└─────────────────────────────────────────────┘
```

### Required Container Parameters

Every container needs these three parameters:

| Parameter | What It Does |
|-----------|--------------|
| `Eula=accept` | Accepts the license agreement - container won't start without this |
| `Billing=<endpoint>` | Azure resource endpoint for usage tracking (NOT for API calls) |
| `ApiKey=<key>` | Azure resource key for authentication to the billing endpoint |

### Disconnected Containers

For scenarios with no internet connectivity:
1. Apply for access (requires approval)
2. Download a license file
3. Container runs completely offline
4. Uses commitment tier pricing (pre-pay for capacity)

## Monitoring Best Practices

### What Should You Monitor?

| Metric | Why It Matters |
|--------|----------------|
| `TotalCalls` | Track usage trends and capacity planning |
| `TotalErrors` | Detect issues before users complain |
| `BlockedCalls` | Identify if you're hitting rate limits |
| `Latency` | Ensure acceptable user experience |
| `DataIn/DataOut` | Track data volumes for cost management |

### Creating Effective Alerts

**Example Alert Rules:**

1. **Error Rate Alert**: Notify when `ServerErrors > 10` in 5 minutes
2. **Throttling Alert**: Notify when `BlockedCalls > 50` in 1 minute  
3. **Latency Alert**: Notify when average `Latency > 5000ms`

**Action Groups** define what happens when an alert fires:
- Email notification
- SMS message
- Webhook call (to Slack, Teams, etc.)
- Azure Function trigger
- Logic App trigger

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Prepare to develop AI solutions on Azure](https://learn.microsoft.com/en-us/training/paths/prepare-for-ai-engineering/)
- [Create and manage Azure AI Services](https://learn.microsoft.com/en-us/training/modules/create-manage-ai-services/)
- [Secure Azure AI Services](https://learn.microsoft.com/en-us/training/modules/secure-ai-services/)
- [Monitor Azure AI Services](https://learn.microsoft.com/en-us/training/modules/monitor-ai-services/)
- [Deploy Azure AI Services in containers](https://learn.microsoft.com/en-us/training/modules/deploy-ai-services-containers/)

### Official Documentation
- [Azure AI Services networking](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-virtual-networks)
- [Configure customer-managed keys](https://learn.microsoft.com/en-us/azure/ai-services/encryption/cognitive-services-encryption-keys-portal)
- [Azure AI Services container documentation](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support)
- [Diagnostic logging for Azure AI Services](https://learn.microsoft.com/en-us/azure/ai-services/diagnostic-logging)
- [Azure AI Services Authentication](https://learn.microsoft.com/en-us/azure/ai-services/authentication)
