# 10 - AI-102 Practice Questions & Answers

> **Format**: These questions mimic the actual AI-102 exam format including multiple choice, multiple select, drag-and-drop scenarios, and hot area–style questions.

---

## Section A: Plan and Manage an Azure AI Solution

### Question 1
You need to deploy an Azure AI Services resource that provides access to Azure AI Language, Azure AI Vision, and Azure AI Speech using a single endpoint and a single key in a development environment.

Which resource **Kind** should you specify when creating the resource?

- A. `TextAnalytics`
- B. `CognitiveServices`
- C. `AllInOne`
- D. `AIServices`

<details>
<summary>Show Answer</summary>

**Answer: B**

`CognitiveServices` is the Kind value for a multi-service resource that provides a single endpoint and key for multiple Azure AI services. `TextAnalytics` is for the Language service only. `AllInOne` and `AIServices` are not valid Kind values.

**Reference**: [Azure AI Services multi-service resource](https://learn.microsoft.com/en-us/azure/ai-services/multi-service-resource)
</details>

---

### Question 2
Your company requires that API keys for Azure AI Services be stored securely and not embedded in application code. The application runs on an Azure App Service.

Which TWO approaches meet this requirement? (Choose two)

- A. Store the keys in Azure Key Vault and access them using the app's managed identity
- B. Store the keys in environment variables of the App Service
- C. Use a system-assigned managed identity to authenticate directly to Azure AI Services without keys
- D. Store the keys in the application's `appsettings.json` file
- E. Hard-code the keys in the application code and use code obfuscation

<details>
<summary>Show Answer</summary>

**Answer: A, C**

**A** is correct because Azure Key Vault is the recommended way to securely store secrets, and managed identity provides passwordless access to Key Vault. **C** is correct because managed identity eliminates the need for keys entirely — it's the most secure approach. **B** is not ideal because environment variables are accessible from the App Service configuration blade. **D** stores keys in code/config files. **E** is a security anti-pattern.

**Reference**: [Azure AI Services Authentication](https://learn.microsoft.com/en-us/azure/ai-services/authentication)
</details>

---

### Question 3
You need to rotate the API keys for an Azure AI Services resource without causing downtime to your production application.

Which sequence of steps should you follow?

- A. Regenerate Key 1 → Update app to Key 1 → Regenerate Key 2
- B. Update app to Key 2 → Regenerate Key 1 → Update app to Key 1 → Regenerate Key 2
- C. Regenerate Key 2 → Update app to Key 2 → Regenerate Key 1
- D. Stop the application → Regenerate both keys → Update app → Start the application

<details>
<summary>Show Answer</summary>

**Answer: C**

The correct approach for zero-downtime key rotation when the app currently uses Key 1:
1. Regenerate Key 2 (app still works with Key 1)
2. Update application to use Key 2
3. Regenerate Key 1

This ensures the application always has a valid key throughout the process.

**Reference**: [Azure AI Services Security](https://learn.microsoft.com/en-us/azure/ai-services/security-features)
</details>

---

### Question 4
You deploy Azure AI Services as a Docker container on-premises. You configure the `Eula`, `Billing`, and `ApiKey` parameters.

The container processes data locally. What is STILL required for the container to function?

- A. The container must have internet connectivity to send billing/usage data to Azure
- B. The container must synchronize model updates every 24 hours
- C. The container must upload all processed data to Azure for storage
- D. No internet connectivity is required after initial deployment

<details>
<summary>Show Answer</summary>

**Answer: A**

Azure AI Services containers process data locally but must maintain internet connectivity to send metering/billing information to Azure. They do not upload processed data. For fully offline scenarios, disconnected containers (requiring separate approval) are available.

**Reference**: [Azure AI Services containers](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-container-support)
</details>

---

### Question 5
You need to restrict access to your Azure AI Services resource so that it can only be accessed from resources within your Azure virtual network.

Which TWO configurations should you implement? (Choose two)

- A. Configure a Private Endpoint for the AI Services resource
- B. Set the default network action to **Deny** in the resource's firewall settings
- C. Enable CORS on the AI Services resource
- D. Create a service principal for each client application
- E. Enable diagnostic logging to a Log Analytics workspace

<details>
<summary>Show Answer</summary>

**Answer: A, B**

**A** creates a private endpoint that gives the AI Services resource a private IP address within your VNet. **B** sets the default action to Deny, blocking all traffic except from explicitly allowed networks/IPs. Together, they ensure only VNet traffic can reach the resource. CORS, service principals, and diagnostic logging don't restrict network access.

**Reference**: [Azure AI Services virtual networks](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-virtual-networks)
</details>

---

## Section B: Computer Vision

### Question 6
You are building an application that needs to classify images of manufactured products as "defective" or "non-defective." You have 200 labeled images of each category.

Which Azure service should you use?

- A. Azure AI Vision Image Analysis with tags
- B. Azure AI Custom Vision with multiclass classification
- C. Azure AI Custom Vision with multilabel classification
- D. Azure AI Vision Image Analysis with dense captions

<details>
<summary>Show Answer</summary>

**Answer: B**

This is a binary classification problem (defective vs. non-defective) where each image belongs to exactly one class, making it a **multiclass** classification task. Custom Vision is the right service because you need to train a custom model for your specific product defects. The prebuilt Image Analysis service doesn't know about your specific products. Multilabel would be used if an image could have multiple categories simultaneously.

**Reference**: [Custom Vision classification](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/getting-started-build-a-classifier)
</details>

---

### Question 7
You need to implement face identification in your application. Users' faces need to be identified from a group of known employees.

Which sequence of API calls should you make?

- A. Create PersonGroup → Create Person → Add Face → Identify
- B. Create PersonGroup → Create Person → Add Face → Train PersonGroup → Identify
- C. Detect Face → Create PersonGroup → Identify
- D. Create PersonGroup → Add Face → Train PersonGroup → Verify

<details>
<summary>Show Answer</summary>

**Answer: B**

The correct workflow is:
1. **Create PersonGroup** — container for people
2. **Create Person** — add individuals to the group
3. **Add Face** — add face images to each person
4. **Train PersonGroup** — train the model to recognize faces
5. **Identify** — identify unknown faces against the trained group

The training step is essential and cannot be skipped. **Verify** is for 1:1 comparison, not 1:N identification.

**Reference**: [Face API identification](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/identity-detect-faces)
</details>

---

### Question 8
Your application uses the Azure AI Vision Image Analysis API to analyze images. You need to extract text from photographs of street signs.

Which visual feature should you use?

- A. `CAPTION`
- B. `TAGS`
- C. `READ`
- D. `OBJECTS`

<details>
<summary>Show Answer</summary>

**Answer: C**

The `READ` visual feature performs OCR (Optical Character Recognition) to extract printed and handwritten text from images. `CAPTION` generates a description of the image, `TAGS` returns content tags, and `OBJECTS` detects objects with bounding boxes but doesn't extract text.

**Reference**: [Image Analysis Read feature](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-ocr)
</details>

---

## Section C: Natural Language Processing

### Question 9
You are building a chatbot that needs to understand user requests for a travel booking system. The chatbot should determine what the user wants to do (book a flight, check status, cancel reservation) and extract relevant information (destination, date, booking reference).

Which Azure service should you use?

- A. Azure AI Language — Sentiment Analysis
- B. Azure AI Language — Conversational Language Understanding (CLU)
- C. Azure AI Language — Custom Question Answering
- D. Azure AI Language — Named Entity Recognition

<details>
<summary>Show Answer</summary>

**Answer: B**

CLU is designed for understanding conversational user input by classifying **intents** (what the user wants to do) and extracting **entities** (relevant data). This is exactly the scenario: intents = BookFlight, CheckStatus, CancelReservation; entities = destination, date, bookingReference. Sentiment Analysis detects emotion, Custom Question Answering is for FAQ, and NER extracts general entity types, not domain-specific custom entities with intent classification.

**Reference**: [CLU overview](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
</details>

---

### Question 10
You are training a CLU model. You need to add an entity that matches order IDs in the format "ORD-" followed by exactly 6 digits (e.g., ORD-123456).

Which entity type should you use?

- A. Learned entity
- B. List entity
- C. Prebuilt entity
- D. Regex entity

<details>
<summary>Show Answer</summary>

**Answer: D**

A **Regex entity** uses a regular expression pattern to match text. The pattern `ORD-\d{6}` would match "ORD-" followed by exactly 6 digits. A Learned entity requires labeled examples, a List entity requires exact enumeration of all possible values (impractical for 1 million combinations), and no Prebuilt entity matches this custom format.

**Reference**: [CLU entity components](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/concepts/entity-components)
</details>

---

### Question 11
You need to analyze customer reviews and determine not only the overall sentiment but also what specific aspects of the product customers are positive or negative about.

Which feature should you use?

- A. Sentiment Analysis only
- B. Sentiment Analysis with Opinion Mining enabled
- C. Key Phrase Extraction followed by Sentiment Analysis
- D. Named Entity Recognition with Sentiment Analysis

<details>
<summary>Show Answer</summary>

**Answer: B**

**Opinion Mining** (aspect-based sentiment analysis) is a feature of Sentiment Analysis that identifies specific targets (aspects) and their associated assessments (opinions). For example, from "The camera quality is great but battery life is poor," it extracts: target="camera quality", sentiment=positive; target="battery life", sentiment=negative. You enable it with `show_opinion_mining=True`.

**Reference**: [Opinion Mining](https://learn.microsoft.com/en-us/azure/ai-services/language-service/sentiment-opinion-mining/overview)
</details>

---

### Question 12
You are implementing Text-to-Speech and need the AI voice to sound cheerful when greeting the user, then switch to a normal speaking style for the rest of the message. You also need a 500-millisecond pause between the greeting and the main message.

Which SSML elements should you use?

- A. `<prosody>` and `<break>`
- B. `<mstts:express-as>` and `<break>`
- C. `<emphasis>` and `<break>`
- D. `<voice>` and `<break>`

<details>
<summary>Show Answer</summary>

**Answer: B**

`<mstts:express-as style="cheerful">` controls the speaking style (cheerful, sad, angry, etc.), which is a Microsoft-specific SSML extension. `<break time="500ms"/>` inserts a pause. `<prosody>` controls rate, pitch, and volume but not emotional style. `<emphasis>` controls stress level. `<voice>` selects the voice but doesn't control style.

**Reference**: [SSML reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
</details>

---

### Question 13
You are implementing Custom Question Answering for a company's IT helpdesk. Users are asking "How do I reset my password?" and the correct answer depends on whether they're using Windows, Mac, or a mobile device.

Which feature should you configure?

- A. Active Learning
- B. Metadata filtering
- C. Multi-turn conversations with follow-up prompts
- D. Alternate questions

<details>
<summary>Show Answer</summary>

**Answer: C**

**Multi-turn conversations** allow you to create a hierarchical Q&A structure with follow-up prompts. When the user asks about password reset, the system can ask "Which platform are you using?" and then provide the correct answer based on the follow-up response. Metadata filtering requires the client to send filter values. Active Learning suggests alternate questions. Alternate questions are for different phrasings of the same question.

**Reference**: [Multi-turn conversations](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/how-to/multi-turn)
</details>

---

### Question 14
You need to translate a large batch of PDF and DOCX files from English to French and German. The files are stored in Azure Blob Storage.

Which Translator feature should you use?

- A. Text Translation API with each file's content
- B. Document Translation
- C. Custom Translator
- D. Transliterate API

<details>
<summary>Show Answer</summary>

**Answer: B**

**Document Translation** is the asynchronous, batch document translation feature that translates entire files while preserving the original format and structure. It works directly with Azure Blob Storage. The Text Translation API is for translating text strings, not entire documents. Custom Translator is for training custom models. Transliterate converts scripts (e.g., Latin to Cyrillic).

**Reference**: [Document Translation](https://learn.microsoft.com/en-us/azure/ai-services/translator/document-translation/overview)
</details>

---

## Section D: Document Intelligence

### Question 15
You need to extract data from invoices received by your company. The invoices come from many different vendors with varying layouts but all contain standard invoice fields like vendor name, invoice date, and total amount.

Which model should you use?

- A. `prebuilt-layout`
- B. `prebuilt-document`
- C. `prebuilt-invoice`
- D. Custom template model

<details>
<summary>Show Answer</summary>

**Answer: C**

The `prebuilt-invoice` model is specifically trained to extract common invoice fields (VendorName, InvoiceDate, InvoiceTotal, Items, etc.) from invoices regardless of layout variations. `prebuilt-layout` only extracts text, tables, and structure without field semantics. `prebuilt-document` extracts key-value pairs but doesn't have invoice-specific knowledge. A custom template model would require fixed layouts.

**Reference**: [Prebuilt invoice model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-invoice)
</details>

---

### Question 16
Your organization processes three types of tax forms: W-2, 1099, and custom state forms. You have trained custom models for each type and want a single API call to correctly process any of the three.

What should you create?

- A. A single custom neural model trained on all three types
- B. A composed model combining the three custom models
- C. Three separate API endpoints
- D. A prebuilt document model

<details>
<summary>Show Answer</summary>

**Answer: B**

A **composed model** combines multiple custom models into a single model ID. When you submit a document, it automatically classifies and routes it to the correct sub-model. This gives you a single API endpoint that handles all three form types. The maximum number of component models is 200.

**Reference**: [Composed models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-composed-models)
</details>

---

## Section E: Knowledge Mining

### Question 17
You are configuring an Azure AI Search indexer to process PDF documents stored in Azure Blob Storage. The PDFs contain embedded images with text that also needs to be searchable.

Which indexer configuration settings should you specify?

- A. `dataToExtract: contentAndMetadata` and `imageAction: none`
- B. `dataToExtract: contentAndMetadata` and `imageAction: generateNormalizedImages`
- C. `dataToExtract: storageMetadata` and `imageAction: generateNormalizedImages`
- D. `dataToExtract: contentOnly` and `imageAction: generateNormalizedImagePerPage`

<details>
<summary>Show Answer</summary>

**Answer: B**

To extract both text content and metadata from PDFs AND process embedded images, you need `dataToExtract: contentAndMetadata` (to get the document text and metadata) and `imageAction: generateNormalizedImages` (to extract images so that an OCR skill in the skillset can process them). With `imageAction: none`, images would be ignored.

**Reference**: [Indexer configuration](https://learn.microsoft.com/en-us/azure/search/search-howto-indexing-azure-blob-storage)
</details>

---

### Question 18
You need to create a custom skill for your Azure AI Search skillset that calls an Azure Function to perform custom data enrichment.

Which of the following represents the correct response format that your Azure Function must return?

- A.
```json
{ "result": "enriched value" }
```

- B.
```json
{ "values": [{ "recordId": "1", "data": { "result": "enriched value" }, "errors": [], "warnings": [] }] }
```

- C.
```json
[{ "id": "1", "output": "enriched value" }]
```

- D.
```json
{ "documents": [{ "id": "1", "enrichment": "enriched value" }] }
```

<details>
<summary>Show Answer</summary>

**Answer: B**

Custom Web API skills require a specific contract. The response must include a `values` array where each object has `recordId` (matching the input), `data` (the enrichment output), `errors` (array of any errors), and `warnings` (array of any warnings). This contract is mandatory for the search indexer to process the results.

**Reference**: [Custom Web API skill](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-web-api)
</details>

---

### Question 19
You want to configure Azure AI Search to provide direct answers to user queries, not just ranked document results.

Which query configuration should you use?

- A. `queryType: simple` with `searchMode: all`
- B. `queryType: full` with Lucene syntax
- C. `queryType: semantic` with `answers: extractive`
- D. `queryType: simple` with `$select` to include the answer field

<details>
<summary>Show Answer</summary>

**Answer: C**

**Semantic search** with `answers: extractive` uses deep learning models to re-rank results and extract direct answers from documents. This goes beyond traditional search ranking by understanding the intent of the query and returning precise answer passages. Simple and full Lucene queries only rank documents based on term matching.

**Reference**: [Semantic search](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
</details>

---

## Section F: Generative AI

### Question 20
You are building a customer support chatbot using Azure OpenAI. The bot should only answer questions using your company's knowledge base and should not generate responses from its general training data.

Which configuration should you use?

- A. Set `temperature` to 0
- B. Configure Azure OpenAI on Your Data with `in_scope: true`
- C. Use a system message saying "Only answer from the knowledge base"
- D. Set `max_tokens` to a small value

<details>
<summary>Show Answer</summary>

**Answer: B**

**Azure OpenAI on Your Data** with `in_scope: true` is the correct approach. This connects Azure OpenAI to Azure AI Search (containing your knowledge base) and the `in_scope` parameter ensures the model only generates responses based on the retrieved documents. Setting temperature to 0 reduces randomness but doesn't restrict the knowledge source. A system message alone can be bypassed. Limiting tokens doesn't restrict the source.

**Reference**: [Azure OpenAI on Your Data](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
</details>

---

### Question 21
You need to generate vector embeddings for documents to store in Azure AI Search for vector search. The documents are in English and vary in length from a few sentences to several pages.

Which Azure OpenAI model should you deploy?

- A. gpt-4o
- B. dall-e-3
- C. text-embedding-ada-002
- D. gpt-3.5-turbo

<details>
<summary>Show Answer</summary>

**Answer: C**

**text-embedding-ada-002** (or the newer text-embedding-3-small/large) is the embeddings model that converts text into vector representations. These vectors can be stored in Azure AI Search's vector fields for similarity search. GPT-4o and GPT-3.5-turbo are chat/completion models. DALL-E is for image generation.

**Reference**: [Azure OpenAI embeddings](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/understand-embeddings)
</details>

---

### Question 22
Your Azure OpenAI deployment uses the default content filter. A user submits a prompt that contains moderately violent content. What happens?

- A. The request is processed normally because the default filter only blocks high-severity content
- B. The request is blocked and an error is returned indicating the content filter was triggered
- C. The request is processed but the violent content is automatically removed from the prompt
- D. The request is logged for review but processed normally

<details>
<summary>Show Answer</summary>

**Answer: B**

The **default content filter** in Azure OpenAI blocks content at **Medium** severity and above for all four categories (Hate, Sexual, Violence, Self-Harm). Since the content is "moderately violent" (Medium severity), the input filter will block the request and return an error with content filter information. The filter works on both input and output.

**Reference**: [Azure OpenAI content filtering](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter)
</details>

---

## Section G: Content Safety

### Question 23
You are implementing content moderation for a user-generated content platform. You need to block specific brand names and competitor product names that violate your platform's terms of service.

Which Azure AI Content Safety feature should you use?

- A. Text analysis with custom severity thresholds
- B. Custom blocklists
- C. Prompt Shields
- D. Groundedness Detection

<details>
<summary>Show Answer</summary>

**Answer: B**

**Custom blocklists** allow you to define specific terms that should always be flagged or blocked, regardless of the content safety severity analysis. This is ideal for platform-specific rules like blocking competitor names or brand violations. Text analysis detects harmful content categories, Prompt Shields detect jailbreak attempts, and Groundedness Detection checks if AI responses are supported by source material.

**Reference**: [Content Safety blocklists](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/how-to/use-blocklist)
</details>

---

### Question 24
You need to detect whether an AI-generated response contains information that is NOT supported by the source documents provided to the AI model.

Which Azure AI Content Safety feature should you use?

- A. Text Analysis
- B. Prompt Shields
- C. Groundedness Detection
- D. Protected Material Detection

<details>
<summary>Show Answer</summary>

**Answer: C**

**Groundedness Detection** checks whether AI-generated text is factually consistent with (grounded in) provided source documents. It identifies ungrounded sentences and provides a percentage score. Text Analysis detects harmful content. Prompt Shields detect jailbreak attempts. Protected Material Detection looks for copyrighted content.

**Reference**: [Groundedness Detection](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/groundedness)
</details>

---

## Section H: Mixed / Integration Questions

### Question 25
You are building a RAG (Retrieval-Augmented Generation) solution. You need to:
1. Store company documents in a searchable index
2. Enrich documents with AI during indexing
3. Generate responses using the indexed data

Which THREE Azure services should you use? (Choose three)

- A. Azure AI Search
- B. Azure AI Language
- C. Azure OpenAI Service
- D. Azure AI Document Intelligence
- E. Azure Cosmos DB
- F. Azure AI Services (for skillset enrichment)

<details>
<summary>Show Answer</summary>

**Answer: A, C, F**

A typical RAG architecture requires:
- **Azure AI Search (A)** — for indexing, enrichment pipeline, and retrieval
- **Azure OpenAI Service (C)** — for generating responses based on retrieved documents
- **Azure AI Services (F)** — attached to the Azure AI Search skillset for AI enrichment (NER, key phrases, OCR, etc.)

Azure AI Language and Document Intelligence could be part of the enrichment but are accessed through the AI Services skillset attachment. Cosmos DB is a database, not directly part of the standard RAG pipeline.

**Reference**: [Azure OpenAI on Your Data](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data)
</details>

---

### Question 26
You have deployed a Conversational Language Understanding (CLU) model. Your speech-enabled application needs to recognize user intent from spoken input.

Which approach should you use?

- A. Use Speech-to-Text to transcribe, then send text to CLU prediction endpoint
- B. Use the Speech SDK's IntentRecognizer with CLU integration
- C. Use the Translator service to convert speech to text, then send to CLU
- D. Use Speech-to-Text with a custom speech model trained on intents

<details>
<summary>Show Answer</summary>

**Answer: A**

The recommended approach is to use Speech-to-Text to transcribe the spoken input into text, then send that text to the CLU prediction endpoint for intent and entity extraction. While the Speech SDK has IntentRecognizer capabilities, CLU integration typically works through the text transcription pipeline. The Translator service is for language translation, not speech-to-text.

**Reference**: [CLU with Speech](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
</details>

---

### Question 27
A user makes the following request to Azure AI Translator:

```http
POST https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=fr&to=de
```

The request body contains: `[{"text": "Good morning"}]`

How many translations will be in the response?

- A. 1
- B. 2
- C. 3
- D. The request will fail because you can't specify multiple target languages

<details>
<summary>Show Answer</summary>

**Answer: B**

The request specifies two target languages (`to=fr&to=de`), so the response will contain **2 translations** — one in French and one in German. The Translator API supports multiple target languages in a single request by repeating the `to` query parameter.

**Reference**: [Translator API reference](https://learn.microsoft.com/en-us/azure/ai-services/translator/reference/v3-0-translate)
</details>

---

### Question 28
You are configuring an Azure AI Search index. A field called `category` needs to support:
- Filtering results to a specific category
- Showing category counts in the search UI for navigation
- Sorting search results by category

Which THREE field attributes must you set to `true`? (Choose three)

- A. searchable
- B. filterable
- C. sortable
- D. facetable
- E. retrievable

<details>
<summary>Show Answer</summary>

**Answer: B, C, D**

- **filterable (B)** — needed for `$filter` expressions
- **sortable (C)** — needed for `$orderby`
- **facetable (D)** — needed for faceted navigation (showing category counts)

The question doesn't mention full-text search on the category (searchable) or returning it in results (retrievable), though you'd likely want retrievable too. The question specifically asks about the three stated requirements.

**Reference**: [Azure AI Search index attributes](https://learn.microsoft.com/en-us/azure/search/search-what-is-an-index)
</details>

---

### Question 29
You are developing a text-to-speech application. The following SSML is used:

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="http://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-JennyNeural">
    <prosody rate="-20%" pitch="+10%">
      Welcome to our service.
    </prosody>
    <break time="1s"/>
    How can I help you?
  </voice>
</speak>
```

What is the effect of this SSML?

- A. "Welcome to our service" is spoken 20% slower and at a higher pitch, followed by a 1-second pause, then "How can I help you?" at normal rate and pitch
- B. The entire message is spoken 20% slower and at a higher pitch with a 1-second pause
- C. "Welcome to our service" is spoken 20% faster, then "How can I help you?" after 1 second
- D. The SSML is invalid because `prosody` and `break` cannot be used together

<details>
<summary>Show Answer</summary>

**Answer: A**

The `<prosody>` element with `rate="-20%"` (slower) and `pitch="+10%"` (higher) only applies to the text within it: "Welcome to our service." The `<break time="1s"/>` inserts a 1-second pause. "How can I help you?" is outside the prosody element, so it uses the default rate and pitch of the JennyNeural voice.

**Reference**: [SSML reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
</details>

---

### Question 30
You are using Azure OpenAI to build a chat application. You want the model to generate highly creative marketing copy.

Which parameter configuration is most appropriate?

- A. `temperature: 0.0, max_tokens: 100`
- B. `temperature: 0.9, max_tokens: 800`
- C. `temperature: 0.0, top_p: 0.1`
- D. `temperature: 0.3, frequency_penalty: -2.0`

<details>
<summary>Show Answer</summary>

**Answer: B**

For creative content like marketing copy, you want **higher temperature** (more randomness/creativity) and sufficient `max_tokens` to generate longer content. `temperature: 0.9` provides high creativity. `temperature: 0.0` would be too deterministic. Setting both temperature and top_p is not recommended. A negative frequency_penalty would encourage repetition, which is undesirable.

**Reference**: [Azure OpenAI parameters](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference)
</details>

---

## Scoring Guide

| Score | Assessment |
|-------|-----------|
| 27–30 correct | Excellent — Ready for the exam |
| 22–26 correct | Good — Review weak areas |
| 17–21 correct | Fair — More study needed |
| Below 17 | Needs significant preparation |

## References

- [AI-102 Exam Skills Outline](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102)
- [AI-102 Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-engineer/practice/assessment?assessment-type=practice&assessmentId=61)
- [Microsoft Learn: AI-102 Learning Paths](https://learn.microsoft.com/en-us/training/paths/prepare-for-ai-engineering/)
