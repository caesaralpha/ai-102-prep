# 11 - AI-102 Case Study Questions & Answers

> **Case studies on the AI-102 exam** present a detailed business scenario, then ask 4–6 questions about the same scenario. You can revisit the case study while answering. These practice case studies mimic the real exam format.

---

## Case Study 1: Contoso Travel Agency

### Background

Contoso Travel is a global travel agency with offices in the United States, Europe, and Asia. The company serves customers who speak English, French, German, Spanish, and Japanese. Contoso Travel wants to modernize its customer service operations using Azure AI services.

### Current Environment

- Customer service agents handle booking requests, cancellations, and travel inquiries via phone and web chat
- The company has a knowledge base of 5,000+ FAQ documents stored as PDFs and Word documents in Azure Blob Storage
- Customer reviews are collected from the website and need to be analyzed for product improvement
- The company website receives 500,000 monthly visitors across 5 language versions

### Requirements

**Business Requirements:**
- BR1: Implement an AI-powered chatbot that understands customer booking requests in all five supported languages
- BR2: Provide automated answers to common FAQ questions with the ability to ask follow-up questions
- BR3: Analyze customer reviews to identify specific aspects of their travel experience (hotel, flight, service) and the sentiment around each
- BR4: Translate customer support emails across all five languages while maintaining domain-specific travel terminology
- BR5: Enable voice-based customer service with natural-sounding responses in English

**Technical Requirements:**
- TR1: All Azure AI resources must use private endpoints for network security
- TR2: Authentication must use Microsoft Entra ID managed identities where possible
- TR3: Solutions must be deployed to the West Europe Azure region for GDPR compliance
- TR4: All API keys must be stored in Azure Key Vault
- TR5: The chatbot must identify the user's language automatically before routing to the appropriate language model

**Security Requirements:**
- SR1: Customer PII in reviews must be redacted before storage
- SR2: The voice service must not allow generation of celebrities' voices
- SR3: AI-generated content must be moderated before delivery to customers

---

### Question 1.1

To fulfill **BR1** (AI-powered chatbot that understands booking requests), which TWO Azure services should you implement? (Choose two)

- A. Azure AI Language — Conversational Language Understanding (CLU)
- B. Azure AI Language — Text Summarization
- C. Azure AI Language — Custom Question Answering
- D. Azure AI Speech — Speech-to-Text
- E. Azure AI Translator — Language Detection

<details>
<summary>Show Answer</summary>

**Answer: A, D**

**CLU (A)** is needed to understand customer booking intents (BookFlight, CancelReservation, etc.) and extract entities (destination, date, traveler count). **Speech-to-Text (D)** is needed because BR1 mentions the chatbot handles phone calls (voice) in addition to web chat.

Custom Question Answering (C) is for BR2 (FAQ), not BR1. Text Summarization (B) is not mentioned in the requirements. Language Detection (E) relates to TR5 but is part of the Language service, not a separate service to implement for booking understanding.

**Reference**: [CLU overview](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
</details>

---

### Question 1.2

To fulfill **BR2** (automated FAQ answers with follow-up questions), you are implementing Custom Question Answering.

Which THREE actions should you perform? (Choose three)

- A. Import the 5,000+ PDF and Word documents as knowledge base sources
- B. Configure multi-turn conversations for questions that need clarification
- C. Train a CLU model with FAQ intents
- D. Enable active learning to improve answer quality over time
- E. Configure semantic ranking on the knowledge base
- F. Deploy the knowledge base to a Custom Vision endpoint

<details>
<summary>Show Answer</summary>

**Answer: A, B, D**

**A** — The FAQ documents (PDF, Word) are imported as knowledge base sources into Custom Question Answering. **B** — Multi-turn conversations enable follow-up questions, matching the "ability to ask follow-up questions" requirement. **D** — Active learning automatically suggests alternate questions based on user queries, continuously improving answer quality.

**C** is for intent recognition (BR1), not FAQ. **E** semantic ranking is an Azure AI Search feature, not Custom Question Answering. **F** Custom Vision is for image classification.

**Reference**: [Custom Question Answering](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/overview)
</details>

---

### Question 1.3

To fulfill **BR3** (analyze customer reviews for aspect-level sentiment about hotel, flight, and service), which API call and parameter should you use?

- A. `analyze_sentiment(documents)` with default parameters
- B. `analyze_sentiment(documents, show_opinion_mining=True)`
- C. `extract_key_phrases(documents)` followed by `analyze_sentiment()`
- D. `recognize_entities(documents)` with entity linking enabled

<details>
<summary>Show Answer</summary>

**Answer: B**

**Opinion Mining** (`show_opinion_mining=True`) provides aspect-based sentiment analysis, which identifies specific targets (hotel, flight, service) and their associated sentiment (positive/negative). This directly fulfills the requirement to "identify specific aspects of their travel experience and the sentiment around each."

Default sentiment analysis (A) only gives document/sentence-level sentiment, not aspect-level. Key phrases (C) extract topics but not sentiment about them. Entity recognition (D) identifies entities but doesn't analyze sentiment.

**Reference**: [Opinion Mining](https://learn.microsoft.com/en-us/azure/ai-services/language-service/sentiment-opinion-mining/overview)
</details>

---

### Question 1.4

To fulfill **BR4** (translate emails with domain-specific travel terminology), which approach should you use?

- A. Use the standard Translator Text API with `to` parameters for each language
- B. Train a Custom Translator model with parallel travel-domain documents, then use the custom category ID in translation calls
- C. Use Azure OpenAI GPT-4 for translation with a travel-domain system prompt
- D. Use Document Translation with automatic language detection

<details>
<summary>Show Answer</summary>

**Answer: B**

**Custom Translator** allows you to train a model with parallel documents containing your domain-specific terminology. This ensures that travel-specific terms (e.g., "layover," "connecting flight," "red-eye") are translated correctly and consistently. You use the custom model by specifying the `category` parameter in the Translate API call.

Standard translation (A) might not handle travel-specific terminology well. GPT-4 (C) could work but isn't the recommended approach for production translation. Document Translation (D) is for batch file translation, not maintaining terminology.

**Reference**: [Custom Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/custom-translator/overview)
</details>

---

### Question 1.5

Which approach fulfills **TR2** (managed identity authentication) and **TR4** (keys in Key Vault) when the chatbot application running on Azure App Service accesses the CLU endpoint?

- A. Store the CLU key in environment variables and pass it in the API header
- B. Enable a system-assigned managed identity on the App Service and assign the "Cognitive Services User" role on the Language resource
- C. Store the CLU key in Key Vault, enable managed identity on App Service, and grant the App Service access to Key Vault
- D. Use a service principal with a client secret stored in App Service configuration

<details>
<summary>Show Answer</summary>

**Answer: B**

The most secure and recommended approach is to use **managed identity** to authenticate directly to Azure AI Services without any keys. Assigning the "Cognitive Services User" role to the App Service's managed identity allows it to call the CLU endpoint using `DefaultAzureCredential()`, eliminating the need for keys entirely. While option C also works, option B eliminates keys completely, which is the preferred approach when possible.

TR2 requires managed identities "where possible" — and since Azure AI Services supports managed identity authentication, this is the correct choice. TR4 (Key Vault) applies only when keys must be used.

**Reference**: [Azure AI Services authentication with managed identity](https://learn.microsoft.com/en-us/azure/ai-services/authentication)
</details>

---

### Question 1.6

To fulfill **SR1** (redact PII from customer reviews before storage), which code approach should you use?

- A.
```python
response = client.recognize_entities(documents)
# Manually remove entities
```

- B.
```python
response = client.recognize_pii_entities(documents)
redacted = response[0].redacted_text
```

- C.
```python
response = client.analyze_sentiment(documents)
# Check for PII in sentiment
```

- D.
```python
response = client.extract_key_phrases(documents)
# Filter out PII phrases
```

<details>
<summary>Show Answer</summary>

**Answer: B**

The **PII Detection** feature (`recognize_pii_entities`) directly identifies PII entities and returns a **`redacted_text`** property where all detected PII is replaced with placeholder characters. This is the purpose-built solution for PII redaction. Standard NER (A) doesn't have PII-specific categories or redaction. Sentiment analysis (C) and key phrases (D) don't detect PII.

**Reference**: [PII Detection](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/overview)
</details>

---

---

## Case Study 2: Northwind Healthcare

### Background

Northwind Healthcare is a hospital network with 15 facilities across three states. They process thousands of medical documents daily including patient intake forms, lab reports, insurance cards, and physician notes.

### Current Environment

- Paper-based forms are scanned to PDF and stored in Azure Blob Storage
- Medical staff manually transcribe key information from documents into the EHR (Electronic Health Record) system
- The hospital has an existing Azure AI Search index for searching patient records
- Physicians dictate clinical notes that are recorded as audio files

### Requirements

**Business Requirements:**
- BR1: Automatically extract patient information from intake forms (name, DOB, address, insurance ID, medical history)
- BR2: Extract data from insurance cards (member ID, group number, plan type, copay amounts)  
- BR3: Build a searchable knowledge base from all medical documents with AI enrichment
- BR4: Transcribe physician audio notes to text with medical terminology accuracy
- BR5: Generate summaries of long patient histories for quick physician review

**Technical Requirements:**
- TR1: Intake forms have a fixed layout used across all 15 facilities
- TR2: Insurance cards vary significantly in layout across different insurance providers
- TR3: The search index must support both keyword search and semantic understanding
- TR4: Audio transcription must handle medical jargon accurately
- TR5: All extracted medical data must remain within the Azure region for HIPAA compliance

**Data Requirements:**
- DR1: Extracted key-value pairs from forms must be stored in Azure Table Storage for reporting
- DR2: Normalized images from PDFs must be stored back to Blob Storage
- DR3: The enrichment pipeline must extract medical entities (conditions, medications, procedures) from document text

---

### Question 2.1

To fulfill **BR1** (extract patient info from intake forms) given **TR1** (forms have a fixed layout across all facilities), which Document Intelligence approach should you use?

- A. `prebuilt-document` model
- B. Custom template model trained with labeled examples from the intake forms
- C. Custom neural model trained with labeled examples
- D. `prebuilt-layout` model with post-processing

<details>
<summary>Show Answer</summary>

**Answer: B**

Since the intake forms have a **fixed layout** (TR1) used across all 15 facilities, a **custom template model** is the best choice. Template models are optimized for fixed/structured layouts and provide exact field extraction. The form has specific fields (name, DOB, address, insurance ID, medical history) that don't match any prebuilt model.

Custom neural (C) is for variable layouts (not needed here since layout is fixed). `prebuilt-document` (A) extracts generic key-value pairs without domain-specific field understanding. `prebuilt-layout` (D) only extracts structure, not named fields.

**Reference**: [Custom template model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-custom-template)
</details>

---

### Question 2.2

To fulfill **BR2** (extract data from insurance cards) given **TR2** (cards vary significantly in layout), which approach should you use?

- A. `prebuilt-healthInsuranceCard.us` model
- B. Custom template model
- C. Custom neural model
- D. `prebuilt-read` model with regex post-processing

<details>
<summary>Show Answer</summary>

**Answer: A**

The `prebuilt-healthInsuranceCard.us` model is specifically designed to extract fields from US health insurance cards including member ID, group number, plan type, and copay information. Since this is a healthcare scenario with standard US insurance cards, the prebuilt model is the most efficient choice — no training required.

While custom neural (C) could handle varying layouts, it would require collecting and labeling training data unnecessarily when a prebuilt model exists for this exact use case.

**Reference**: [Health insurance card model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-health-insurance-card)
</details>

---

### Question 2.3

To fulfill **BR3** (searchable knowledge base with AI enrichment) along with **DR1** (store key-value pairs in Table Storage), **DR2** (store images in Blob Storage), and **DR3** (extract medical entities), which Azure AI Search configuration elements do you need?

Select **Yes** or **No** for each:

| Component | Required? |
|-----------|-----------|
| A. Skillset with `EntityRecognitionSkill` configured for health entities | |
| B. Skillset with `OcrSkill` for extracting text from embedded images | |
| C. Knowledge Store with table projections and file projections | |
| D. Indexer with `imageAction: none` | |
| E. Semantic configuration on the index | |

<details>
<summary>Show Answer</summary>

| Component | Required? | Explanation |
|-----------|-----------|-------------|
| A. Skillset with `EntityRecognitionSkill` for health entities | **Yes** | DR3 requires extracting medical entities (conditions, medications, procedures) |
| B. Skillset with `OcrSkill` for embedded images | **Yes** | PDFs may contain scanned images; OCR extracts text from them |
| C. Knowledge Store with table projections and file projections | **Yes** | DR1 (Table Storage for key-value pairs) + DR2 (Blob Storage for images) = table + file projections |
| D. Indexer with `imageAction: none` | **No** | Images need to be processed, so `imageAction: generateNormalizedImages` is needed, not `none` |
| E. Semantic configuration on the index | **Yes** | TR3 requires "semantic understanding" which needs semantic ranking configured |

**Reference**: [Knowledge Store projections](https://learn.microsoft.com/en-us/azure/search/knowledge-store-projection-overview)
</details>

---

### Question 2.4

To fulfill **BR4** (transcribe physician notes with medical terminology), which approach provides the best accuracy for medical jargon?

- A. Standard Azure AI Speech-to-Text with the base model
- B. Azure AI Speech-to-Text with a custom speech model trained on medical audio data
- C. Azure OpenAI Whisper model
- D. Azure AI Language text analytics on the audio file

<details>
<summary>Show Answer</summary>

**Answer: B**

A **custom speech model** trained on medical audio data and terminology will provide the best accuracy for medical jargon. Custom Speech allows you to upload audio+transcript training data specific to your domain, improving recognition of medical terms, drug names, and clinical vocabulary that the base model might not handle well.

The standard model (A) struggles with domain-specific terminology. Whisper (C) is a general-purpose model without customization for medical terms. Language text analytics (D) works on text, not audio.

**Reference**: [Custom Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/custom-speech-overview)
</details>

---

### Question 2.5

To fulfill **BR5** (generate summaries of patient histories), which service and feature should you use?

- A. Azure AI Language — Extractive Text Summarization
- B. Azure AI Language — Abstractive Text Summarization
- C. Azure OpenAI — GPT-4 with a summarization prompt
- D. Azure AI Search — Semantic captions

<details>
<summary>Show Answer</summary>

**Answer: C**

For generating **summaries of long patient histories** for physician review, **Azure OpenAI GPT-4** with a summarization prompt is the most appropriate choice. Patient histories are complex, spanning multiple encounters and specialists, and require abstractive understanding that goes beyond simple extraction. GPT-4 can generate coherent, clinically relevant summaries with appropriate context.

Extractive summarization (A) only selects existing sentences, which may miss important connections. Abstractive summarization from Language (B) is more limited in context length and clinical understanding. Semantic captions (D) are for search result snippets, not document summarization.

**Reference**: [Azure OpenAI for summarization](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)
</details>

---

---

## Case Study 3: Adventure Works Retail

### Background

Adventure Works is a global outdoor equipment retailer with an e-commerce platform, 200 physical stores, and a customer support center. They sell products in 30 countries and support 12 languages on their website.

### Current Environment

- Product catalog with 50,000 SKUs including images and descriptions
- Customer support agents handle 10,000 cases per day via chat, email, and phone
- 5 million product reviews in a Cosmos DB database
- Return/refund process requires manual review of product images submitted by customers
- Content moderation for user-generated product reviews is done manually

### Requirements

**Business Requirements:**
- BR1: Implement visual product search — customers upload a photo and find similar products
- BR2: Automate quality inspection of product images submitted with return requests (detect damage, defects)
- BR3: Build an AI-powered support chatbot that:
  - (a) Understands customer intents (track order, return item, product question)
  - (b) Answers product FAQs
  - (c) Handles 12 languages
  - (d) Uses company product data to answer, not general knowledge
- BR4: Automatically moderate product reviews for inappropriate content before publication
- BR5: Analyze all product reviews to identify trending issues per product category
- BR6: Enable search across all product data with AI-enhanced ranking

**Technical Requirements:**
- TR1: The visual search system must generate vector representations of product images
- TR2: The chatbot must be accessible through the website and Microsoft Teams
- TR3: Product review moderation must block custom terms specific to Adventure Works competitors
- TR4: Cost optimization — use the most cost-effective service tier for each component
- TR5: The chatbot's AI responses must be checked for hallucinations before delivery

---

### Question 3.1

To fulfill **BR1** (visual product search using uploaded photos) and **TR1** (vector representations), which architecture should you implement?

- A. Azure AI Vision Image Analysis to tag uploaded images, then search by tags in Azure AI Search
- B. Azure AI Vision for image vectorization (multimodal embeddings) + Azure AI Search vector search
- C. Azure AI Custom Vision to classify uploaded images into product categories
- D. Azure OpenAI DALL-E to generate product images similar to the upload

<details>
<summary>Show Answer</summary>

**Answer: B**

Azure AI Vision's **multimodal embeddings** generate vector representations of both images and text, enabling visual similarity search. Storing these vectors in Azure AI Search's vector search capability allows you to find products with visually similar images. This directly fulfills TR1 (vector representations) and BR1 (visual product search).

Tag-based search (A) loses visual similarity information. Custom Vision classification (C) puts images into categories but doesn't find visually similar products. DALL-E (D) generates images, not searches.

**Reference**: [Azure AI Vision multimodal embeddings](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-image-retrieval)
</details>

---

### Question 3.2

To fulfill **BR2** (detect damage and defects in return images), which approach should you use?

- A. Azure AI Vision Image Analysis with object detection
- B. Azure AI Custom Vision with an Object Detection model trained on defect images
- C. Azure AI Content Safety image analysis
- D. Azure AI Document Intelligence layout model

<details>
<summary>Show Answer</summary>

**Answer: B**

**Custom Vision Object Detection** is the correct choice because detecting product damage and defects requires a model trained specifically on your product images showing various types of damage. The prebuilt Image Analysis (A) can detect general objects but not specific product defects. Content Safety (C) detects inappropriate content, not product damage. Document Intelligence (D) is for document processing.

You would need to collect and label images of damaged products (minimum 15 images per tag with bounding boxes for object detection) to train the model.

**Reference**: [Custom Vision object detection](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/get-started-build-detector)
</details>

---

### Question 3.3

For **BR3** (AI-powered support chatbot), you need to design the architecture. Match each sub-requirement to the correct service:

| Sub-Requirement | Service |
|----------------|---------|
| (a) Understand intents (track order, return, question) | ? |
| (b) Answer product FAQs | ? |
| (c) Handle 12 languages | ? |
| (d) Use company data only, not general knowledge | ? |

Choose from:
1. Conversational Language Understanding (CLU)
2. Custom Question Answering
3. Azure AI Translator
4. Azure OpenAI on Your Data with `in_scope: true`

<details>
<summary>Show Answer</summary>

| Sub-Requirement | Service | Explanation |
|----------------|---------|-------------|
| (a) Understand intents | **1 — CLU** | CLU classifies user intents and extracts entities from conversational input |
| (b) Answer product FAQs | **2 — Custom Question Answering** | Purpose-built for FAQ matching with multi-turn support |
| (c) Handle 12 languages | **3 — Azure AI Translator** | Translates user input and bot responses across 12 languages |
| (d) Use company data only | **4 — Azure OpenAI on Your Data** | `in_scope: true` ensures responses come only from your indexed data |

The complete chatbot architecture routes user input through Translator → Language Detection → CLU (intent classification) → the appropriate handler (Q&A for FAQs, OpenAI for complex questions, custom logic for order tracking).

**Reference**: [Build a chatbot architecture](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
</details>

---

### Question 3.4

To fulfill **BR4** (moderate reviews) and **TR3** (block competitor terms), which combination of features should you use?

- A. Azure AI Content Safety text analysis only
- B. Azure AI Content Safety text analysis + custom blocklist
- C. Azure AI Language sentiment analysis + PII detection
- D. Azure OpenAI content filters with a custom filter policy

<details>
<summary>Show Answer</summary>

**Answer: B**

**Azure AI Content Safety text analysis** handles the general moderation (detecting hate, violence, sexual content, self-harm). The **custom blocklist** feature addresses TR3 by allowing you to define competitor brand names and terms that should be blocked. Together, they provide comprehensive moderation.

Sentiment analysis (C) detects sentiment, not inappropriate content. PII detection finds personal information but not offensive content or competitor mentions. Azure OpenAI content filters (D) only apply to OpenAI model inputs/outputs, not to user-generated reviews.

**Reference**: [Content Safety with blocklists](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/how-to/use-blocklist)
</details>

---

### Question 3.5

To fulfill **TR5** (check chatbot responses for hallucinations), which feature should you implement?

- A. Azure AI Content Safety — Text Analysis
- B. Azure AI Content Safety — Groundedness Detection
- C. Azure AI Content Safety — Prompt Shields
- D. Azure OpenAI — Content Filter with custom policy

<details>
<summary>Show Answer</summary>

**Answer: B**

**Groundedness Detection** checks whether AI-generated responses are factually consistent with the provided source material. It identifies ungrounded (hallucinated) statements in the response and provides both a percentage score and specific ungrounded segments. This directly addresses TR5's requirement to check for hallucinations.

Text Analysis (A) checks for harmful content, not factual accuracy. Prompt Shields (C) detect jailbreak attempts. Content filters (D) block harmful content but don't verify factual grounding.

**Reference**: [Groundedness Detection](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/groundedness)
</details>

---

### Question 3.6

To fulfill **BR6** (search across product data with AI-enhanced ranking) and support both text queries and visual similarity, which index configuration should you create?

- A. An index with searchable text fields and a semantic configuration
- B. An index with vector fields (`Collection(Edm.Single)`) only
- C. An index with searchable text fields, vector fields, and a semantic configuration for hybrid + semantic search
- D. An index with filterable and facetable fields only

<details>
<summary>Show Answer</summary>

**Answer: C**

To support both **text search** and **visual similarity** (from BR1) with **AI-enhanced ranking**, you need:
- **Searchable text fields** — for keyword/full-text search on product descriptions
- **Vector fields** (`Collection(Edm.Single)`) — for storing image and text embeddings for vector similarity search
- **Semantic configuration** — for AI-enhanced re-ranking that understands query intent

This enables **hybrid search** (text + vector) with **semantic ranking**, providing the most comprehensive and relevant search experience. Option A lacks vector support. Option B lacks text search. Option D lacks search and vector capabilities.

**Reference**: [Azure AI Search hybrid search](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)
</details>

---

---

## Case Study Exam Tips

1. **Read the ENTIRE case study first** before answering any questions — requirements interrelate
2. **Pay attention to requirement IDs** (BR1, TR2, etc.) — questions reference them specifically
3. **Look for constraints** in Technical Requirements that eliminate answer choices
4. **Fixed layout → Template model; Variable layout → Neural model** — this distinction is critical
5. **`in_scope: true`** is the answer when "only company data" is mentioned
6. **Multi-turn** is the answer when "follow-up questions" are mentioned
7. **Opinion Mining** is the answer when "aspect-level sentiment" is mentioned
8. **Groundedness Detection** is the answer when "hallucination checking" is mentioned
9. **Custom blocklist** is the answer when "specific terms to block" is mentioned
10. Don't overthink — the simplest service that meets ALL stated requirements is usually correct

## References

- [AI-102 Exam Skills Outline](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102)
- [AI-102 Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/azure-ai-engineer/practice/assessment?assessment-type=practice&assessmentId=61)
- [Azure AI Services Documentation](https://learn.microsoft.com/en-us/azure/ai-services/)
- [Azure AI Search Documentation](https://learn.microsoft.com/en-us/azure/search/)
- [Azure OpenAI Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [Azure AI Content Safety Documentation](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/)
