# 13 - AI-102 Comprehensive Braindump: Practice Questions & Answers

> **Compiled from publicly available sources** including Whizlabs, ExamTopics community discussions, Microsoft Learn practice assessments, GitHub repos, DEV Community blogs, and certification community forums. Questions synthesized and updated to align with the latest AI-102 exam objectives (2025/2026).
>
> **Sources**: [Whizlabs Free Questions](https://www.whizlabs.com/blog/ai-102-exam-questions/) | [Microsoft Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-engineer/practice/assessment?assessment-type=practice&assessmentId=61) | [ExamTopics Community](https://www.examtopics.com/exams/microsoft/ai-102/view/) | [Study4Exam](https://www.study4exam.com/microsoft/free-ai-102-questions) | [GitHub: maeiwalk12/AI-102-exam-tips](https://github.com/maeiwalk12/AI-102-exam-tips-resources-practice-tests) | [DEV Community](https://dev.to/alextheluchador/how-i-passed-the-azure-ai-102-exam-4gg7)
>
> **Total**: 75 questions across all 6 exam domains

---

## Domain 1: Plan and Manage an Azure AI Solution (15–20%)

---

### Q1. Multi-Service vs Single-Service Resource

You need to deploy an Azure AI Services resource. Your application requires access to Azure AI Language, Azure AI Vision, and Azure AI Speech. You want to use a **single endpoint and single key** and simplify billing.

Which resource Kind should you create?

- A. `TextAnalytics`
- B. `ComputerVision`
- C. `CognitiveServices`
- D. `AIServices`

<details><summary>Answer</summary>

**C. `CognitiveServices`**

A multi-service resource (Kind = `CognitiveServices`) provides a single endpoint and single API key for accessing multiple Azure AI services. Individual service Kinds like `TextAnalytics` or `ComputerVision` only provide access to that specific service. `AIServices` is not a valid Kind value.

**Ref**: [Multi-service resource](https://learn.microsoft.com/en-us/azure/ai-services/multi-service-resource)
</details>

---

### Q2. Key Rotation Strategy

Your production application currently uses Key 1 of your Azure AI Services resource. You need to rotate keys with **zero downtime**.

What should you do?

- A. Regenerate Key 1, update the app to use Key 1, then regenerate Key 2
- B. Regenerate Key 2, update the app to use Key 2, then regenerate Key 1
- C. Regenerate both keys simultaneously, then update the app
- D. Create a new resource with new keys, then switch the app

<details><summary>Answer</summary>

**B. Regenerate Key 2, update the app to use Key 2, then regenerate Key 1**

Since the app currently uses Key 1, regenerating Key 2 first has no impact. Then update the app to use Key 2. Finally, regenerate Key 1 to complete rotation. This ensures the app always has a valid key.

**Ref**: [Azure AI Services security](https://learn.microsoft.com/en-us/azure/ai-services/security-features)
</details>

---

### Q3. Container Deployment Required Parameters

You deploy an Azure AI Services container using Docker. Which THREE parameters are **required** for the container to function?

- A. `Eula=accept`
- B. `Billing={endpoint_URI}`
- C. `ApiKey={api_key}`
- D. `Logging:Disk:Format=json`
- E. `HTTP_PROXY={proxy_url}`

<details><summary>Answer</summary>

**A, B, C**

The three required parameters for all Azure AI Services containers are:
- `Eula=accept` — Must accept the license
- `Billing` — The endpoint URI for billing
- `ApiKey` — The subscription key

`Logging` and `HTTP_PROXY` are optional configuration settings.

**Ref**: [Container configuration](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support)
</details>

---

### Q4. Network Security

You need to restrict your Azure AI Services resource to only accept traffic from your Azure Virtual Network. Which TWO actions should you perform?

- A. Set the default network action to `Deny`
- B. Create a Private Endpoint for the resource
- C. Enable diagnostic logging
- D. Create a service principal
- E. Enable CORS

<details><summary>Answer</summary>

**A, B**

Setting the default action to `Deny` blocks all public traffic. Creating a Private Endpoint provides a private IP address within your VNet, ensuring all traffic stays on the Microsoft backbone. CORS, service principals, and diagnostic logging don't restrict network access.

**Ref**: [Virtual networks for AI Services](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-virtual-networks)
</details>

---

### Q5. Authentication — Managed Identity

Your application runs on Azure App Service and needs to call Azure AI Language. Your company policy mandates no credentials in code or config files.

Which approach should you use?

- A. Store the API key in the app's `appsettings.json`
- B. Store the API key in an environment variable
- C. Enable system-assigned managed identity and assign "Cognitive Services User" RBAC role
- D. Hard-code the API key and use code obfuscation

<details><summary>Answer</summary>

**C**

Managed identity with RBAC completely eliminates the need for keys. Assign the "Cognitive Services User" role to the App Service's managed identity on the Language resource. The app uses `DefaultAzureCredential()` to authenticate. All other options involve storing or embedding keys.

**Ref**: [Authentication with managed identity](https://learn.microsoft.com/en-us/azure/ai-services/authentication)
</details>

---

### Q6. Monitoring — Diagnostic Settings

You need to query logs for your Azure AI Services resource using KQL and create alerts when error rates exceed a threshold.

Which diagnostic setting destination should you configure?

- A. Azure Storage Account
- B. Azure Event Hub
- C. Log Analytics Workspace
- D. Azure Cosmos DB

<details><summary>Answer</summary>

**C. Log Analytics Workspace**

Log Analytics Workspace supports KQL queries and alert rule creation through Azure Monitor. Storage Account is for archival. Event Hub is for streaming to external SIEM. Cosmos DB is not a valid diagnostic destination.

**Ref**: [Diagnostic logging](https://learn.microsoft.com/en-us/azure/ai-services/diagnostic-logging)
</details>

---

### Q7. Container Billing

After deploying an Azure AI Services container on-premises, you notice it stops working after losing internet connectivity for 48 hours. Why?

- A. The container needs to download model updates
- B. The container must connect to Azure for billing/metering
- C. The container license expires after 24 hours
- D. The container needs to sync with the Azure resource configuration

<details><summary>Answer</summary>

**B**

Azure AI Services containers process data locally but must maintain internet connectivity to send billing/metering information to Azure. Without connectivity, the container will stop functioning after a grace period. Disconnected containers (requiring separate approval) are available for fully offline scenarios.

**Ref**: [Container support](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support)
</details>

---

### Q8. Azure Monitor Costs

Identify the TRUE statements about Azure Monitor Log Analytics costs: (Select TWO)

Statement 1: The customer is charged for **Data Ingestion**.
Statement 2: The customer is charged for **Data Retention**.

- A. Statement 1 → True
- B. Statement 1 → False
- C. Statement 2 → True
- D. Statement 2 → False

<details><summary>Answer</summary>

**A and C**

Azure Monitor Log Analytics charges for both data ingestion (the volume of data sent to the workspace) and data retention (storing data beyond the default free retention period of 31 days).

**Ref**: [Azure Monitor pricing](https://azure.microsoft.com/en-us/pricing/details/monitor/)
</details>

---

### Q9. RBAC — Custom Vision Roles

Match each Azure role to the correct permission level:

| Role | Permission |
|------|-----------|
| R1: Cognitive Services Custom Vision **Reader** | P1: View projects, export/publish models |
| R2: Cognitive Services Custom Vision **Deployment** | P2: View projects, delete training images, add tags |
| R3: Cognitive Services Custom Vision **Trainer** | P3: Edit projects, train models (cannot create/delete projects) |
| R4: Cognitive Services Custom Vision **Labeler** | P4: View projects, cannot make changes |

- A. R1→P4; R2→P1; R3→P3; R4→P2
- B. R1→P1; R2→P3; R3→P2; R4→P4
- C. R1→P4; R2→P2; R3→P3; R4→P1
- D. R1→P4; R2→P3; R3→P2; R4→P1

<details><summary>Answer</summary>

**A. R1→P4; R2→P1; R3→P3; R4→P2**

- **Reader** = view only, no changes (P4)
- **Deployment** = view + export/publish models (P1)
- **Trainer** = edit projects + train models, but cannot create/delete projects (P3)
- **Labeler** = view + manage training images/tags (P2)

**Ref**: [Custom Vision RBAC](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/role-based-access-control)
</details>

---

### Q10. Responsible AI

Which of the following is NOT one of Microsoft's six Responsible AI principles?

- A. Fairness
- B. Transparency
- C. Profitability
- D. Accountability
- E. Inclusiveness

<details><summary>Answer</summary>

**C. Profitability**

Microsoft's six Responsible AI principles are: **Fairness**, **Reliability & Safety**, **Privacy & Security**, **Inclusiveness**, **Transparency**, and **Accountability**. Profitability is not one of them.

**Ref**: [Microsoft Responsible AI](https://www.microsoft.com/en-us/ai/responsible-ai)
</details>

---

## Domain 2: Content Moderation / Decision Support (10–15%)

---

### Q11. Content Safety Categories

Azure AI Content Safety analyzes text and images across four harm categories with four severity levels. What are the four severity level values?

- A. 0, 1, 2, 3
- B. 0, 2, 4, 6
- C. Low, Medium, High, Critical
- D. 1, 2, 3, 4

<details><summary>Answer</summary>

**B. 0, 2, 4, 6**

Content Safety severity levels are: **0** (Safe), **2** (Low), **4** (Medium), **6** (High). The four categories are Hate, Sexual, Violence, and Self-Harm.

**Ref**: [Content Safety concepts](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/harm-categories)
</details>

---

### Q12. Custom Blocklists

You run a user-generated content platform and need to block specific competitor brand names that violate your terms of service. The content doesn't qualify as hate, violence, or other standard categories.

Which Content Safety feature should you use?

- A. Increase severity threshold for all categories
- B. Custom blocklists
- C. Prompt Shields
- D. Groundedness Detection

<details><summary>Answer</summary>

**B. Custom blocklists**

Custom blocklists allow you to define specific terms that should always be flagged, regardless of the built-in category analysis. This is ideal for platform-specific business rules like blocking competitor names. You can set `halt_on_blocklist_hit: true` to immediately block matched content.

**Ref**: [Blocklists](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/how-to/use-blocklist)
</details>

---

### Q13. Prompt Shields

You are building an AI chatbot that uses Azure OpenAI. You need to protect against users trying to manipulate the model by injecting instructions like "Ignore all previous instructions and..."

Which feature should you implement?

- A. Content Safety Text Analysis
- B. Custom Blocklists
- C. Prompt Shields (Jailbreak Detection)
- D. Groundedness Detection

<details><summary>Answer</summary>

**C. Prompt Shields (Jailbreak Detection)**

Prompt Shields detect both direct attacks (user jailbreak attempts) and indirect attacks (hidden instructions in grounding documents). It analyzes the user prompt and grounding documents for manipulation attempts.

**Ref**: [Prompt Shields](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/jailbreak-detection)
</details>

---

### Q14. Groundedness Detection

Your RAG-based chatbot occasionally generates information not found in the source documents. You need to detect these hallucinations before sending responses to users.

Which feature should you use?

- A. Content Safety Text Analysis
- B. Prompt Shields
- C. Groundedness Detection
- D. Protected Material Detection

<details><summary>Answer</summary>

**C. Groundedness Detection**

Groundedness Detection checks whether AI-generated responses are factually consistent with provided source material. It returns an `ungroundedPercentage` and identifies specific ungrounded sentences with reasons.

**Ref**: [Groundedness Detection](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/groundedness)
</details>

---

### Q15. Content Filtering in Azure OpenAI

What is the default input/output filter threshold for all four harm categories in Azure OpenAI?

- A. Low severity and above
- B. Medium severity and above
- C. High severity only
- D. No filtering by default

<details><summary>Answer</summary>

**B. Medium severity and above**

By default, Azure OpenAI's content filters block content at **Medium** severity and above for all four categories (Hate, Sexual, Violence, Self-Harm) on both input and output. Custom filter policies can adjust these thresholds.

**Ref**: [Content filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
</details>

---

### Q16. Image Moderation

You need to scan user-uploaded images for inappropriate content. Which approach detects harmful content by category with severity levels?

- A. Azure AI Vision Image Analysis with tags
- B. Azure AI Content Safety Image Analysis
- C. Azure AI Custom Vision with a moderation model
- D. Azure OpenAI DALL-E content filter

<details><summary>Answer</summary>

**B. Azure AI Content Safety Image Analysis**

Content Safety's `analyze_image` endpoint scans images across the four harm categories (Hate, Sexual, Violence, Self-Harm) and returns severity levels (0, 2, 4, 6) for each. Vision Image Analysis provides content tags, not moderation scores.

**Ref**: [Image moderation quickstart](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-image)
</details>

---

## Domain 3: Computer Vision Solutions (15–20%)

---

### Q17. Custom Vision — Domain Conversion

You need to export a Custom Vision model to run locally for real-time classification. The project currently uses a standard domain.

What is the correct sequence of steps?

- A. Select & save new domain → Retrain → Select project → Export
- B. Select project → Select & save compact domain → Retrain → Export
- C. Select & save new domain → Select project → Export → Retrain
- D. Select project → Export → Select compact domain → Retrain

<details><summary>Answer</summary>

**B. Select project → Select & save compact domain → Retrain → Export**

You must first select the project, change to a compact domain (which supports export), retrain the model with the new domain, and then export it in the desired format (ONNX, TensorFlow, CoreML, etc.).

**Ref**: [Export Custom Vision model](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/export-your-model)
</details>

---

### Q18. Face API — Adding Faces

You need to add faces to persons in a PersonGroup using file streams. Complete the code:

```csharp
await faceClient.______________(personGroupId, personId, stream);
```

- A. `PersonGroupPerson.AddFaceFromUrlAsync`
- B. `PersonGroupPerson.CreateAsync`
- C. `PersonGroupPerson.UpdateFaceAsync`
- D. `PersonGroupPerson.AddFaceFromStreamAsync`

<details><summary>Answer</summary>

**D. `PersonGroupPerson.AddFaceFromStreamAsync`**

Since the input is a file stream (not a URL), `AddFaceFromStreamAsync` is correct. `AddFaceFromUrlAsync` requires a URL. `CreateAsync` creates a new person. `UpdateFaceAsync` updates existing face data.

**Ref**: [Add faces to a group](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/identity-detect-faces)
</details>

---

### Q19. Video Indexer

A developer needs to extract insights from videos for deep search and social media content creation. Which service should they use?

- A. Face API
- B. Computer Vision
- C. Azure Video Indexer
- D. Bing Video Search

<details><summary>Answer</summary>

**C. Azure Video Indexer**

Video Indexer extracts insights from videos including transcription, face identification, OCR, scene segmentation, and topic inference. These insights enable deep search and content creation. Face API only handles faces. Computer Vision handles images/short video frames. Bing Video Search searches the web.

**Ref**: [Video Indexer overview](https://learn.microsoft.com/en-us/azure/azure-video-indexer/video-indexer-overview)
</details>

---

### Q20. Image Analysis 4.0

You need to extract text from photographs of street signs using Azure AI Vision. Which visual feature should you specify?

- A. `VisualFeatures.CAPTION`
- B. `VisualFeatures.TAGS`
- C. `VisualFeatures.READ`
- D. `VisualFeatures.OBJECTS`

<details><summary>Answer</summary>

**C. `VisualFeatures.READ`**

The `READ` feature performs OCR to extract printed and handwritten text from images. `CAPTION` generates descriptions. `TAGS` returns content labels. `OBJECTS` detects objects with bounding boxes.

**Ref**: [Image Analysis 4.0](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-ocr)
</details>

---

### Q21. Face API Models

You need the most accurate face detection and recognition. Which model combination should you use?

- A. `detection_01` + `recognition_01`
- B. `detection_03` + `recognition_04`
- C. `detection_02` + `recognition_03`
- D. `detection_03` + `recognition_01`

<details><summary>Answer</summary>

**B. `detection_03` + `recognition_04`**

`detection_03` is the latest and most accurate detection model. `recognition_04` is the latest and most accurate recognition model. Always use the latest models for best accuracy.

**Ref**: [Face detection models](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/specify-detection-model)
</details>

---

### Q22. Custom Vision — Classification Types

You need to classify product images where each image can have MULTIPLE defect types simultaneously (scratch, dent, discoloration). Which classification type should you use?

- A. Multiclass
- B. Multilabel
- C. Object Detection
- D. Binary Classification

<details><summary>Answer</summary>

**B. Multilabel**

**Multilabel** classification allows assigning multiple labels to each image simultaneously (scratch AND dent AND discoloration). **Multiclass** assigns exactly one label per image. **Object Detection** locates objects with bounding boxes.

**Ref**: [Custom Vision classification](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/getting-started-build-a-classifier)
</details>

---

### Q23. Face API — Limited Access

Which Face API features require a Limited Access application and approval from Microsoft? (Select TWO)

- A. Face Detection with bounding boxes
- B. Face Identification (1:N matching)
- C. Face Verification (1:1 matching)
- D. Face attributes (age estimate)
- E. Face landmarks (27-point)

<details><summary>Answer</summary>

**B and C**

Face **Identification** (who is this person from a group?) and Face **Verification** (are these two faces the same person?) require Limited Access approval. Face Detection, basic attributes, and landmarks are available without special approval.

**Ref**: [Face API Limited Access](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/identity-limited-access)
</details>

---

## Domain 4: Natural Language Processing Solutions (30–35%)

---

### Q24. Sentiment Analysis — Document Level

Given these sentiment conditions, what is the document-level sentiment?

| Condition | Result |
|-----------|--------|
| At least one sentence is negative AND at least one is positive | ? |
| At least one sentence is negative AND rest are neutral | ? |
| At least one sentence is positive AND rest are neutral | ? |

- A. Neutral, Negative, Positive
- B. Mixed, Negative, Positive
- C. Mixed, Neutral, Neutral
- D. Negative, Negative, Positive

<details><summary>Answer</summary>

**B. Mixed, Negative, Positive**

- Negative + Positive sentences → **Mixed** document sentiment
- Negative + Neutral (rest) → **Negative** document sentiment
- Positive + Neutral (rest) → **Positive** document sentiment

**Ref**: [Sentiment Analysis](https://learn.microsoft.com/en-us/azure/ai-services/language-service/sentiment-opinion-mining/overview)
</details>

---

### Q25. Opinion Mining

You need aspect-level sentiment from customer reviews. Complete the code:

```python
response = client.analyze_sentiment(documents, {______________})
```

- A. `includeOpinionMining: true`
- B. `includeOpinionMining: enable`
- C. `show_opinion_mining=True`
- D. `getConfidenceScores: true`

<details><summary>Answer</summary>

**C. `show_opinion_mining=True`** (Python SDK)

In the Python SDK, the parameter is `show_opinion_mining=True`. In the JavaScript SDK it's `includeOpinionMining: true`. The option flag enables aspect-based sentiment analysis.

**Ref**: [Opinion Mining](https://learn.microsoft.com/en-us/azure/ai-services/language-service/sentiment-opinion-mining/how-to/call-api)
</details>

---

### Q26. Language Detection — Unknown Language

When the Language Detection API returns a confidence score of 0.0, what are the `iso6391Name` and `name` values?

- A. `"en"` and `"English"`
- B. `""` and `""`
- C. `"(Unknown)"` and `"(Unknown)"`
- D. `null` and `null`

<details><summary>Answer</summary>

**C. `"(Unknown)"` and `"(Unknown)"`**

When the API cannot determine the language (confidence 0.0), it returns `"(Unknown)"` for both the ISO code and the language name.

**Ref**: [Language Detection](https://learn.microsoft.com/en-us/azure/ai-services/language-service/language-detection/overview)
</details>

---

### Q27. NER Endpoint

Which endpoint is used for Named Entity Recognition (general entities)?

- A. `.../entities/recognition/general`
- B. `.../entities/recognition/pii`
- C. `.../entities/recognition/pii?domain=phi`
- D. `.../entities/linking`

<details><summary>Answer</summary>

**A. `.../entities/recognition/general`**

- `/entities/recognition/general` — Named Entity Recognition (person, org, location, etc.)
- `/entities/recognition/pii` — Personally Identifiable Information
- `/entities/recognition/pii?domain=phi` — Protected Health Information
- `/entities/linking` — Entity Linking (Wikipedia)

**Ref**: [NER in Text Analytics](https://learn.microsoft.com/en-us/azure/ai-services/language-service/named-entity-recognition/overview)
</details>

---

### Q28. CLU — Entity Types

You need a CLU entity that matches order IDs in the format `ORD-123456`. Which entity type should you use?

- A. Learned entity
- B. List entity
- C. Prebuilt entity
- D. Regex entity

<details><summary>Answer</summary>

**D. Regex entity**

A Regex entity uses a pattern like `ORD-\d{6}` to match specific formats. Learned entities require training examples. List entities need exact enumeration. No prebuilt entity matches this custom format.

**Ref**: [CLU entity components](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/concepts/entity-components)
</details>

---

### Q29. CLU — Data Imbalance

You're training a CLU model. After deleting utterances from several intents, the quantity of example utterances varies significantly across intents. What issue will the dashboard show?

- A. Incorrect predictions
- B. Unclear predictions
- C. Data imbalance
- D. None of the above

<details><summary>Answer</summary>

**C. Data imbalance**

Data imbalance occurs when the quantity of example utterances varies significantly across intents. To fix this, add more utterances to under-represented intents. Incorrect predictions occur when utterances are predicted for the wrong intent. Unclear predictions happen when top intent scores are too close.

**Ref**: [LUIS/CLU dashboard analysis](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/how-to/view-model-evaluation)
</details>

---

### Q30. Custom Question Answering — Multi-Turn

Users asking "How do I reset my password?" need different answers depending on their platform (Windows, Mac, Mobile). Which feature should you configure?

- A. Active Learning
- B. Metadata filtering
- C. Multi-turn conversations with follow-up prompts
- D. Alternate questions

<details><summary>Answer</summary>

**C. Multi-turn conversations with follow-up prompts**

Multi-turn conversations create a hierarchical Q&A with follow-up prompts. The system asks "Which platform?" and provides the correct answer based on the follow-up. Metadata filtering requires the client to send filters. Active Learning suggests alternate questions.

**Ref**: [Multi-turn conversations](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/how-to/multi-turn)
</details>

---

### Q31. Speech Service — Batch Transcription REST API

Match each batch transcription operation to its REST API call:

| Operation | API Call |
|-----------|---------|
| Create transcription | ? |
| Get transcription | ? |
| Update transcription | ? |
| Delete transcription | ? |

- A. POST, GET, PATCH, DELETE
- B. GET, POST, PATCH, DELETE
- C. POST, GET, DELETE, PATCH
- D. PUT, GET, PATCH, DELETE

<details><summary>Answer</summary>

**A. POST, GET, PATCH, DELETE**

- **Create** → `POST speechtotext/v3.0/transcriptions`
- **Get** → `GET speechtotext/v3.0/transcriptions/{id}`
- **Update** → `PATCH speechtotext/v3.0/transcriptions/{id}`
- **Delete** → `DELETE speechtotext/v3.0/transcriptions/{id}`

**Ref**: [Batch transcription](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/batch-transcription)
</details>

---

### Q32. Intent Recognition — SpeechConfig

When using the Speech SDK for intent recognition with a LUIS/CLU model, which key and region do you use for `SpeechConfig.FromSubscription()`?

- A. Speech service key and Speech service region
- B. LUIS/CLU primary key and LUIS/CLU service region
- C. Azure subscription ID and tenant ID
- D. Any Cognitive Services key and `global` region

<details><summary>Answer</summary>

**B. LUIS/CLU primary key and LUIS/CLU service region**

For intent recognition, the `SpeechConfig` uses the Language Understanding (LUIS/CLU) prediction key and its hosting region, not the Speech service key.

**Ref**: [Intent recognition with Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/get-started-intent-recognition)
</details>

---

### Q33. SSML — Speaking Style

You want the AI voice to sound cheerful when greeting users and insert a 500ms pause before the main message. Which SSML elements do you use?

- A. `<prosody>` and `<break>`
- B. `<mstts:express-as style="cheerful">` and `<break time="500ms"/>`
- C. `<emphasis>` and `<break>`
- D. `<voice>` and `<break>`

<details><summary>Answer</summary>

**B. `<mstts:express-as style="cheerful">` and `<break time="500ms"/>`**

`<mstts:express-as>` is the Microsoft-specific SSML extension for speaking styles (cheerful, sad, angry, etc.). `<break>` inserts a pause. `<prosody>` controls rate/pitch/volume but not emotional style.

**Ref**: [SSML reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
</details>

---

### Q34. Translator — Multiple Target Languages

How many translations will this request return?

```http
POST https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=fr&to=de&to=ja
Body: [{"text": "Hello"}]
```

- A. 1
- B. 2
- C. 3
- D. Error — cannot specify multiple `to` parameters

<details><summary>Answer</summary>

**C. 3**

The Translator API supports multiple target languages in a single request by repeating the `to` parameter. This request will return three translations: French, German, and Japanese.

**Ref**: [Translator API](https://learn.microsoft.com/en-us/azure/ai-services/translator/reference/v3-0-translate)
</details>

---

### Q35. Translator — Required Headers

Which headers are required for the Azure AI Translator API? (Select TWO)

- A. `Ocp-Apim-Subscription-Key`
- B. `Ocp-Apim-Subscription-Region`
- C. `Authorization: Bearer`
- D. `X-ClientTraceId`

<details><summary>Answer</summary>

**A and B**

The Translator API requires the subscription key header (`Ocp-Apim-Subscription-Key`) and the region header (`Ocp-Apim-Subscription-Region`). The region is needed because the Translator uses a global endpoint. `Authorization: Bearer` is an alternative to the key header. `X-ClientTraceId` is optional.

**Ref**: [Translator authentication](https://learn.microsoft.com/en-us/azure/ai-services/translator/reference/v3-0-reference)
</details>

---

### Q36. Document Translation

You need to translate large batches of PDF and DOCX files from English to French while preserving formatting. Files are in Azure Blob Storage. Which feature should you use?

- A. Translator Text API with file content
- B. Document Translation (batch)
- C. Custom Translator
- D. Azure OpenAI with translation prompt

<details><summary>Answer</summary>

**B. Document Translation (batch)**

Document Translation is the asynchronous batch translation feature that can translate entire documents while preserving their original format. It works directly with Azure Blob Storage containers as source and target.

**Ref**: [Document Translation](https://learn.microsoft.com/en-us/azure/ai-services/translator/document-translation/overview)
</details>

---

### Q37. Speech Service — Which is NOT a Language service?

Which of the following is NOT a Language cognitive service?

- A. Immersive Reader
- B. Language Understanding
- C. QnA Maker
- D. Speech Service
- E. Translator

<details><summary>Answer</summary>

**D. Speech Service**

Speech Service is categorized as a **Speech cognitive service**, not a Language cognitive service. Immersive Reader, Language Understanding (CLU), QnA Maker (Custom Question Answering), and Translator are all Language services.

**Ref**: [Azure AI Services categories](https://learn.microsoft.com/en-us/azure/ai-services/)
</details>

---

### Q38. PII Detection

You need to redact personally identifiable information from customer reviews before storing them. Which approach gives you the redacted text directly?

- A. `client.recognize_entities(documents)`
- B. `client.recognize_pii_entities(documents)` → use `redacted_text`
- C. `client.analyze_sentiment(documents)`
- D. `client.extract_key_phrases(documents)`

<details><summary>Answer</summary>

**B. `client.recognize_pii_entities(documents)` → use `redacted_text`**

The PII detection feature returns a `redacted_text` property where all PII entities are replaced with placeholder characters (e.g., "John Smith" → "***********"). Standard NER doesn't provide redaction.

**Ref**: [PII Detection](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/overview)
</details>

---

## Domain 5: Knowledge Mining & Document Intelligence (10–15%)

---

### Q39. Azure AI Search — Pipeline

What is the correct order of the Azure AI Search enrichment pipeline?

- A. Index → Indexer → Skillset → Data Source → Query
- B. Data Source → Skillset → Indexer → Index → Query
- C. Data Source → Indexer → Skillset → Index → Query
- D. Skillset → Data Source → Indexer → Query → Index

<details><summary>Answer</summary>

**C. Data Source → Indexer → Skillset → Index → Query**

The pipeline flows: **Data Source** (where data lives) → **Indexer** (reads data) → **Skillset** (AI enrichment) → **Index** (searchable store) → **Query** (search).

**Ref**: [AI enrichment overview](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-intro)
</details>

---

### Q40. Search Index — Field Attributes

A `category` field needs to support filtering, faceted navigation, and sorting. Which THREE attributes must be `true`?

- A. searchable
- B. filterable
- C. sortable
- D. facetable
- E. retrievable

<details><summary>Answer</summary>

**B, C, D**

- **filterable** — for `$filter` expressions
- **sortable** — for `$orderby`
- **facetable** — for faceted navigation (category counts)

**Ref**: [Index attributes](https://learn.microsoft.com/en-us/azure/search/search-what-is-an-index)
</details>

---

### Q41. Custom Web API Skill — Response Format

Your Azure Function custom skill for Azure AI Search must return responses in a specific format. Which is correct?

- A. `{ "result": "value" }`
- B. `{ "values": [{ "recordId": "1", "data": { "result": "value" }, "errors": [], "warnings": [] }] }`
- C. `[{ "id": "1", "output": "value" }]`
- D. `{ "documents": [{ "id": "1", "enrichment": "value" }] }`

<details><summary>Answer</summary>

**B**

Custom Web API skills require a `values` array where each object has `recordId`, `data`, `errors`, and `warnings`. This contract is mandatory for the indexer to process results correctly.

**Ref**: [Custom Web API skill](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-web-api)
</details>

---

### Q42. Knowledge Store Projections

You need to persist enriched data to Azure Table Storage for Power BI reporting AND store normalized images to Blob Storage. Which projection types do you need?

- A. Table projections only
- B. Object projections only
- C. Table projections and File projections
- D. Object projections and File projections

<details><summary>Answer</summary>

**C. Table projections and File projections**

- **Table projections** → Azure Table Storage (structured data for Power BI)
- **File projections** → Blob Storage (binary files like normalized images)
- Object projections → Blob Storage (JSON documents)

**Ref**: [Knowledge Store projections](https://learn.microsoft.com/en-us/azure/search/knowledge-store-projection-overview)
</details>

---

### Q43. Semantic Search

You want Azure AI Search to provide direct answers to user questions, not just ranked documents. Which configuration do you use?

- A. `queryType: simple` with `searchMode: all`
- B. `queryType: full` with Lucene syntax
- C. `queryType: semantic` with `answers: extractive`
- D. `queryType: simple` with `$select`

<details><summary>Answer</summary>

**C. `queryType: semantic` with `answers: extractive`**

Semantic search with extractive answers uses deep learning to re-rank results and extract direct answer passages from documents.

**Ref**: [Semantic search](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
</details>

---

### Q44. Azure AI Search — Improving Query Performance

You need to improve Azure AI Search query performance while keeping costs low. Which TWO actions should you take?

- A. Reduce content to maintain smaller indexes
- B. Select all properties for all fields
- C. Add more storage disks
- D. Map complex data types to simpler type fields
- E. Upgrade to Standard S2 and add search units

<details><summary>Answer</summary>

**A and D**

Smaller indexes improve query speed. Mapping complex types to simpler fields reduces storage overhead and improves performance. Selecting all properties (B) slows queries. Adding disks (C) and upgrading tiers (E) increase costs.

**Ref**: [Search performance tips](https://learn.microsoft.com/en-us/azure/search/search-performance-tips)
</details>

---

### Q45. Indexer — Image Processing

You're indexing PDFs with embedded images from Blob Storage. You need OCR to extract text from those images. Which indexer configuration is correct?

- A. `dataToExtract: contentAndMetadata`, `imageAction: none`
- B. `dataToExtract: contentAndMetadata`, `imageAction: generateNormalizedImages`
- C. `dataToExtract: storageMetadata`, `imageAction: generateNormalizedImages`
- D. `dataToExtract: contentOnly`, `imageAction: generateNormalizedImagePerPage`

<details><summary>Answer</summary>

**B**

`contentAndMetadata` extracts document text and metadata. `generateNormalizedImages` extracts embedded images so OCR skills can process them. Without `generateNormalizedImages`, embedded images are ignored.

**Ref**: [Indexing Blob Storage](https://learn.microsoft.com/en-us/azure/search/search-howto-indexing-azure-blob-storage)
</details>

---

### Q46. Document Intelligence — Prebuilt Invoice

Which prebuilt Document Intelligence model extracts fields like VendorName, InvoiceDate, and InvoiceTotal from invoices of varying layouts?

- A. `prebuilt-layout`
- B. `prebuilt-document`
- C. `prebuilt-invoice`
- D. `prebuilt-read`

<details><summary>Answer</summary>

**C. `prebuilt-invoice`**

The `prebuilt-invoice` model is trained specifically for invoices and knows field names like VendorName, InvoiceDate, InvoiceTotal, Items, etc. `prebuilt-layout` extracts structure only. `prebuilt-document` extracts generic key-value pairs. `prebuilt-read` is OCR only.

**Ref**: [Prebuilt invoice model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-invoice)
</details>

---

### Q47. Document Intelligence — Composed Models

You process three document types (W-2, 1099, custom state form) with separate custom models. You want a single API call to handle all three. What should you create?

- A. A single neural model trained on all types
- B. A composed model combining the three custom models
- C. Three separate API endpoints with routing logic
- D. A prebuilt-document model

<details><summary>Answer</summary>

**B. A composed model combining the three custom models**

A composed model combines up to 200 custom models. When a document is submitted, it automatically classifies and routes to the correct sub-model, providing a single API endpoint for all types.

**Ref**: [Composed models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-composed-models)
</details>

---

### Q48. Document Intelligence — Custom Template vs Neural

Your intake forms have a **fixed layout** used across all facilities. Which custom model type should you use?

- A. Custom template model
- B. Custom neural model
- C. Composed model
- D. Prebuilt-layout model

<details><summary>Answer</summary>

**A. Custom template model**

Custom **template** models are optimized for fixed/structured layouts (forms, tax documents). Custom **neural** models handle variable layouts (contracts, letters). Since the forms have a consistent layout, template is the better choice.

**Ref**: [Custom models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-custom)
</details>

---

### Q49. Azure AI Search — API Keys vs RBAC

You granted Reader role to users for Azure AI Search operations. They cannot perform index queries or content operations. What should you do?

- A. Use API keys to grant access for content operations
- B. Grant Contributor role instead
- C. Use a Service Principal with RBAC
- D. Grant Owner role instead

<details><summary>Answer</summary>

**A. Use API keys to grant access for content operations**

Azure AI Search uses **API keys** as the primary mechanism for authenticating inbound requests to the search endpoint. RBAC roles (Reader, Contributor, Owner) control management operations (create/delete service) but content operations (querying, indexing) require API keys.

**Ref**: [Search security API keys](https://learn.microsoft.com/en-us/azure/search/search-security-api-keys)
</details>

---

### Q50. Vector Search Field Type

What field type is used for storing vector embeddings in an Azure AI Search index?

- A. `Edm.String`
- B. `Edm.Double`
- C. `Collection(Edm.Single)`
- D. `Edm.Binary`

<details><summary>Answer</summary>

**C. `Collection(Edm.Single)`**

Vector fields use `Collection(Edm.Single)` to store floating-point embedding arrays. You also specify `dimensions` and a `vectorSearchProfile`.

**Ref**: [Vector search](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)
</details>

---

## Domain 6: Generative AI Solutions (10–15%)

---

### Q51. Azure OpenAI — Deployment Names

When calling Azure OpenAI's chat completions API, what do you specify as the "model" parameter?

- A. The OpenAI model name (e.g., `gpt-4`)
- B. The Azure deployment name you created
- C. The Azure resource name
- D. The model version string

<details><summary>Answer</summary>

**B. The Azure deployment name you created**

Azure OpenAI uses your custom deployment name, not the underlying model name. When you deploy a model, you give it a deployment name which is used in API calls: `/openai/deployments/{deployment-name}/chat/completions`.

**Ref**: [Azure OpenAI reference](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
</details>

---

### Q52. Temperature Parameter

You need the model to generate highly creative and varied marketing copy. Which parameter setting is most appropriate?

- A. `temperature: 0.0`
- B. `temperature: 0.9`
- C. `temperature: 0.0, top_p: 0.1`
- D. `temperature: 0.3, frequency_penalty: -2.0`

<details><summary>Answer</summary>

**B. `temperature: 0.9`**

Higher temperature = more randomness/creativity. Temperature 0.0 is deterministic. Don't set both temperature and top_p together. Negative frequency_penalty encourages repetition, which is undesirable.

**Ref**: [Chat completions](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
</details>

---

### Q53. RAG — On Your Data

Your chatbot should ONLY answer from company data, not from the model's general knowledge. Which configuration ensures this?

- A. Set `temperature` to 0
- B. Add a system message saying "Only use provided data"
- C. Configure Azure OpenAI on Your Data with `in_scope: true`
- D. Set `max_tokens` to a small number

<details><summary>Answer</summary>

**C. Azure OpenAI on Your Data with `in_scope: true`**

`in_scope: true` ensures the model only generates responses based on retrieved documents from Azure AI Search. Temperature 0 reduces randomness but doesn't restrict knowledge source. System messages can be bypassed.

**Ref**: [Azure OpenAI on Your Data](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
</details>

---

### Q54. Embeddings — Model Dimensions

You deploy `text-embedding-ada-002` for vector embeddings. How many dimensions does each vector have?

- A. 256
- B. 512
- C. 1024
- D. 1536

<details><summary>Answer</summary>

**D. 1536**

`text-embedding-ada-002` produces 1536-dimensional vectors. Newer models like `text-embedding-3-small` (512/1536) and `text-embedding-3-large` (256/1024/3072) offer flexible dimensions.

**Ref**: [Embeddings](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/understand-embeddings)
</details>

---

### Q55. Prompt Engineering — Techniques

You need the model to solve a complex math problem step by step. Which technique should you use?

- A. Zero-shot prompting
- B. Few-shot prompting
- C. Chain-of-thought prompting
- D. System message only

<details><summary>Answer</summary>

**C. Chain-of-thought prompting**

Chain-of-thought prompting asks the model to "think step by step" or "show your reasoning," which significantly improves performance on complex reasoning tasks like math and logic problems.

**Ref**: [Prompt engineering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)
</details>

---

### Q56. Function Calling

Your Azure OpenAI application needs to get real-time weather data when users ask about weather. The model should request this data from your API. Which feature enables this?

- A. System messages with weather data
- B. RAG with weather documents
- C. Function calling (tools)
- D. Fine-tuning with weather data

<details><summary>Answer</summary>

**C. Function calling (tools)**

Function calling lets the model generate structured function call requests (JSON) with parameters. Your app executes the function (weather API call) and returns results to the model. The model then incorporates the real-time data into its response.

**Ref**: [Function calling](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/function-calling)
</details>

---

### Q57. DALL-E 3 — Constraints

Which of the following is TRUE about DALL-E 3 in Azure OpenAI?

- A. It can generate up to 4 images per request
- B. It only supports `1024x1024` resolution
- C. It can generate only 1 image per request
- D. It supports animated GIF output

<details><summary>Answer</summary>

**C. It can generate only 1 image per request**

DALL-E 3 supports only `n=1` per request. It supports three sizes: `1024x1024`, `1024x1792`, and `1792x1024`. It does not produce animated outputs.

**Ref**: [DALL-E quickstart](https://learn.microsoft.com/en-us/azure/ai-services/openai/dall-e-quickstart)
</details>

---

### Q58. Message Roles

In Azure OpenAI's chat completions API, which role defines the AI's behavior, personality, and constraints?

- A. `user`
- B. `assistant`
- C. `system`
- D. `function`

<details><summary>Answer</summary>

**C. `system`**

The `system` message defines the AI's behavior, personality, rules, and constraints. `user` is the human's input. `assistant` is the AI's prior responses (for context). `function` returns tool call results.

**Ref**: [Chat completions API](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
</details>

---

### Q59. RAG — Query Types

In Azure OpenAI on Your Data, which `query_type` combines keyword search AND vector search with deep learning re-ranking?

- A. `simple`
- B. `vector`
- C. `semantic`
- D. `vectorSemanticHybrid`

<details><summary>Answer</summary>

**D. `vectorSemanticHybrid`**

`vectorSemanticHybrid` combines text (keyword) search + vector similarity search + semantic re-ranking for the best retrieval quality. Options include: `simple`, `semantic`, `vector`, `vectorSimpleHybrid`, `vectorSemanticHybrid`.

**Ref**: [On Your Data query types](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
</details>

---

### Q60. Protected Material Detection

Azure OpenAI includes a feature to detect known copyrighted text and open-source code (with license info) in outputs. What is this feature called?

- A. Content filtering
- B. Groundedness detection
- C. Protected material detection
- D. Prompt shields

<details><summary>Answer</summary>

**C. Protected material detection**

Protected material detection identifies known copyrighted text and licensed open-source code in model outputs, helping ensure compliance. Content filtering detects harmful content. Groundedness checks factual accuracy. Prompt shields detect jailbreak.

**Ref**: [Protected material](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/protected-material)
</details>

---

## Mixed Domain: Scenario-Based Questions

---

### Q61. Bot Framework — Card Types

Interaction 1: Your bot displays a large image, one button, and a few lines of text.
Interaction 2: Your bot renders a JSON object for a rich adaptive experience.

Which cards should you use?

- A. Interaction 1 → Hero Card; Interaction 2 → Adaptive Card
- B. Interaction 1 → Thumbnail Card; Interaction 2 → Adaptive Card
- C. Interaction 1 → Adaptive Card; Interaction 2 → Thumbnail Card
- D. Interaction 1 → Adaptive Card; Interaction 2 → Hero Card

<details><summary>Answer</summary>

**A. Interaction 1 → Hero Card; Interaction 2 → Adaptive Card**

Hero Cards display a large image with buttons and text. Adaptive Cards render JSON objects for flexible, rich interactive experiences. Thumbnail Cards use small images.

**Ref**: [Bot Framework cards](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards)
</details>

---

### Q62. Bot Authentication

In a bot architecture, the user must be authenticated at Step 2 before making requests. Which Azure service handles this?

- A. LUIS/CLU
- B. Microsoft Entra ID (Azure AD)
- C. Application Insights
- D. Azure Bot Authenticator Service

<details><summary>Answer</summary>

**B. Microsoft Entra ID (Azure AD)**

Microsoft Entra ID (formerly Azure Active Directory) provides identity and access management, handling user authentication. LUIS/CLU handles natural language understanding. Application Insights monitors performance. "Azure Bot Authenticator Service" is not a real service.

**Ref**: [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/fundamentals/whatis)
</details>

---

### Q63. Image Moderation API

A meme-design app needs to: 1) Scan text content from images, 2) Extract text from images. Which API should they use?

- A. Text Moderation API
- B. Content Moderator Review Tool
- C. Custom Term API
- D. Image Moderation API

<details><summary>Answer</summary>

**D. Image Moderation API**

Image Moderation API can both scan images for text content and extract text from images. Text Moderation API works with text input, not images. The Review Tool combines human review with ML. Custom Term API creates custom term lists.

**Ref**: [Image Moderation API](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/)
</details>

---

### Q64. Bot Publish Sequence

You want to publish a bot to Azure Web App by importing existing resources. Order these steps:

1. In Composer menu, select Publish
2. Select the publish tab
3. Select the bot to publish
4. Add profile (name + target = "Publish bot to Azure")
5. Select "Manage profiles"
6. Select import existing Azure resources
7. Select "Publish selected bots"

- A. 3→2→5→4→6→1→7
- B. 3→2→6→5→4→1→7
- C. 1→2→3→4→5→6→7
- D. 1→2→6→3→4→5→7

<details><summary>Answer</summary>

**C. 1→2→3→4→5→6→7**

The correct sequence: Open Publish in Composer → Select publish tab → Choose bot → Add publish profile → Manage profiles → Import existing resources → Publish.

**Ref**: [Publish bot to Azure](https://learn.microsoft.com/en-us/composer/how-to-publish-bot)
</details>

---

### Q65. Hybrid Search Architecture

You're building a RAG solution. Which THREE services form the core architecture?

- A. Azure AI Search
- B. Azure AI Language
- C. Azure OpenAI Service
- D. Azure AI Document Intelligence
- E. Azure Cosmos DB
- F. Azure AI Services (for skillset enrichment)

<details><summary>Answer</summary>

**A, C, F**

- **Azure AI Search** — indexing, enrichment, and retrieval
- **Azure OpenAI** — response generation
- **Azure AI Services** — attached to skillsets for AI enrichment (OCR, NER, etc.)

**Ref**: [RAG architecture](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
</details>

---

### Q66. Custom Translator

You need to translate content with domain-specific travel terminology that standard translation handles poorly. Which feature should you use?

- A. Standard Translator API
- B. Custom Translator with parallel documents
- C. Azure OpenAI with translation prompt
- D. Document Translation with glossary

<details><summary>Answer</summary>

**B. Custom Translator with parallel documents**

Custom Translator trains a model with your parallel documents (source + target language pairs) containing domain-specific terminology. You reference the custom model via a `category` parameter in translation calls.

**Ref**: [Custom Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/custom-translator/overview)
</details>

---

### Q67. Cognitive Search — OData Filter

Which filter expression correctly finds hotels with rating >= 4 that have "wifi" in their tags?

- A. `$filter=rating ge 4 and tags eq 'wifi'`
- B. `$filter=rating ge 4 and tags/any(t: t eq 'wifi')`
- C. `$filter=rating >= 4 AND tags CONTAINS 'wifi'`
- D. `$filter=rating ge 4 or tags/all(t: t eq 'wifi')`

<details><summary>Answer</summary>

**B. `$filter=rating ge 4 and tags/any(t: t eq 'wifi')`**

OData syntax uses `ge` (greater than or equal), `and` for logical AND, and `tags/any(t: t eq 'wifi')` to check if the collection contains a value. `any` returns true if at least one element matches.

**Ref**: [OData filter syntax](https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter)
</details>

---

### Q68. Prebuilt Entity — DateTime

In a NER response, the entity "last week" has `type: "DateTime"`. What is the correct `subType`?

- A. Duration
- B. DateRange
- C. Date
- D. Time

<details><summary>Answer</summary>

**B. DateRange**

"Last week" represents a range of dates (a full week), so the subType is `DateRange`. `Date` would be a specific day. `Duration` measures time units. `Time` represents clock time.

**Ref**: [DateTime entity types](https://learn.microsoft.com/en-us/azure/ai-services/language-service/named-entity-recognition/concepts/named-entity-categories)
</details>

---

### Q69. Document Intelligence — Async API Pattern

When you call the Document Intelligence analyze endpoint, what is returned in the initial response?

- A. The complete extraction result
- B. An `Operation-Location` header with a URL to poll for results
- C. A status code 200 with partial results
- D. A redirect to the Document Intelligence Studio

<details><summary>Answer</summary>

**B. An `Operation-Location` header with a URL to poll for results**

Document Intelligence uses an asynchronous pattern. The initial POST returns HTTP 202 with an `Operation-Location` header. You poll this URL until the status is `succeeded`, then retrieve the results.

**Ref**: [Document Intelligence REST API](https://learn.microsoft.com/en-us/rest/api/aiservices/document-models/analyze-document)
</details>

---

### Q70. SSML — Prosody Scope

Given this SSML, what happens?

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-US-JennyNeural">
    <prosody rate="-20%" pitch="+10%">Welcome!</prosody>
    <break time="1s"/>
    How can I help?
  </voice>
</speak>
```

- A. Both sentences are slow with high pitch
- B. "Welcome!" is slow+high pitch, 1s pause, "How can I help?" at normal
- C. "Welcome!" is fast, "How can I help?" is slow
- D. Invalid SSML

<details><summary>Answer</summary>

**B. "Welcome!" is slow+high pitch, 1s pause, "How can I help?" at normal**

`<prosody>` only applies to its enclosed text. "Welcome!" gets rate=-20% (slower) and pitch=+10%. After a 1-second break, "How can I help?" uses the default voice settings since it's outside the prosody element.

**Ref**: [SSML reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
</details>

---

### Q71. Azure AI Search — Suggesters

You need to implement autocomplete as users type in the search box. Which component must you define in the index?

- A. Analyzer
- B. Suggester
- C. Scoring profile
- D. Synonym map

<details><summary>Answer</summary>

**B. Suggester**

A **suggester** enables autocomplete and suggestions. It's defined in the index with `searchMode: "analyzingInfixMatching"` and source fields. Analyzers process text. Scoring profiles influence relevance ranking. Synonym maps expand query terms.

**Ref**: [Suggesters](https://learn.microsoft.com/en-us/azure/search/index-add-suggesters)
</details>

---

### Q72. Custom Speech Model

You need to transcribe medical audio recordings with specialized terminology. The base STT model doesn't recognize domain-specific terms. What should you do?

- A. Use the standard Speech-to-Text with the latest base model
- B. Train a Custom Speech model with medical audio + transcripts
- C. Use Azure OpenAI Whisper
- D. Pre-process audio to remove medical terms

<details><summary>Answer</summary>

**B. Train a Custom Speech model with medical audio + transcripts**

Custom Speech allows you to train a model with domain-specific audio data and transcripts, significantly improving recognition of specialized terminology. The base model lacks medical vocabulary.

**Ref**: [Custom Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/custom-speech-overview)
</details>

---

### Q73. Azure OpenAI — Content Filter Pipeline

What is the correct flow of content filtering in Azure OpenAI?

- A. User Input → Model → Output Filter → Response
- B. User Input → Input Filter → Model → Output Filter → Response
- C. User Input → Model → Response (no filtering)
- D. User Input → Input Filter → Model → Response

<details><summary>Answer</summary>

**B. User Input → Input Filter → Model → Output Filter → Response**

Azure OpenAI applies content filters on both **input** (before the model processes it) and **output** (before delivering to the user). Both filters check the same 4 categories at configured severity thresholds.

**Ref**: [Content filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
</details>

---

### Q74. Azure AI Vision — Multimodal Embeddings

You need visual product search — users upload a photo and find visually similar products. Which approach enables this?

- A. Image Analysis with tags → search by tags
- B. Azure AI Vision multimodal embeddings → Azure AI Search vector search
- C. Custom Vision classification → match categories
- D. DALL-E → generate similar images

<details><summary>Answer</summary>

**B. Azure AI Vision multimodal embeddings → Azure AI Search vector search**

Multimodal embeddings generate vectors from both images and text, enabling visual similarity search. Store vectors in Azure AI Search's vector fields for k-NN similarity search. Tag-based search loses visual nuance.

**Ref**: [Image retrieval with multimodal embeddings](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-image-retrieval)
</details>

---

### Q75. Document Intelligence — Model Comparison

Match each model to what it extracts:

| Model | Extracts |
|-------|---------|
| `prebuilt-read` | ? |
| `prebuilt-layout` | ? |
| `prebuilt-document` | ? |

- A. Text only → Text + Tables + Selection marks → Text + Tables + Key-value pairs
- B. Text + Tables → Text only → Key-value pairs
- C. Key-value pairs → Text only → Text + Tables
- D. Text + Tables + Key-value pairs → Text + Tables → Text only

<details><summary>Answer</summary>

**A. Text only → Text + Tables + Selection marks → Text + Tables + Key-value pairs**

- `prebuilt-read` = OCR only (text extraction)
- `prebuilt-layout` = text + tables + selection marks + structure
- `prebuilt-document` = text + tables + key-value pairs

**Ref**: [Model overview](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-model-overview)
</details>

---

## Scoring Guide

| Score | Assessment |
|-------|-----------|
| 68–75 correct (90%+) | Excellent — Highly likely to pass |
| 56–67 correct (75–89%) | Good — Review weak domains |
| 45–55 correct (60–74%) | Fair — Significant study needed |
| Below 45 (< 60%) | More preparation required |

> **Passing score on the real exam**: 700 / 1000

---

## Source References

| Source | URL | Notes |
|--------|-----|-------|
| **Whizlabs Free Questions** | [whizlabs.com/blog/ai-102-exam-questions](https://www.whizlabs.com/blog/ai-102-exam-questions/) | 25+ free practice questions with explanations |
| **Microsoft Practice Assessment** | [learn.microsoft.com](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-engineer/practice/assessment?assessment-type=practice&assessmentId=61) | Official free practice test |
| **ExamTopics Community** | [examtopics.com/exams/microsoft/ai-102](https://www.examtopics.com/exams/microsoft/ai-102/view/) | Community-contributed Q&A with discussions |
| **Study4Exam** | [study4exam.com/microsoft/free-ai-102-questions](https://www.study4exam.com/microsoft/free-ai-102-questions) | Free practice questions updated Feb 2026 |
| **CertSteps** | [certsteps.com/exams/microsoft/ai-102/test](https://www.certsteps.com/exams/microsoft/ai-102/test/) | Free Q&A updated Feb 2026 |
| **GitHub: maeiwalk12** | [github.com/maeiwalk12/AI-102-exam-tips](https://github.com/maeiwalk12/AI-102-exam-tips-resources-practice-tests) | Exam tips from someone who passed |
| **DEV Community** | [dev.to — How I Passed AI-102](https://dev.to/alextheluchador/how-i-passed-the-azure-ai-102-exam-4gg7) | Pass experience with study resources |
| **Medium** | [I Passed AI-102 — Here's How](https://medium.com/@jeslurrahman/i-passed-ai-102-azure-ai-engineer-exam-heres-how-you-can-too-2025-guide-3131db145c) | June 2025 pass story with resources |
| **AI-102 Skills Outline** | [learn.microsoft.com/study-guides/ai-102](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102) | Official exam skills measured |

---

> **Disclaimer**: This braindump is compiled from publicly available free practice questions, community discussions, and official Microsoft resources. It is intended for educational purposes to help you prepare for the AI-102 exam. Always cross-reference answers with the [official Microsoft documentation](https://learn.microsoft.com/en-us/azure/ai-services/) as services and APIs evolve. No copyrighted exam content has been reproduced.
