# 09 - AI-102 Quick Reference Cheatsheet

## Service Name Mapping (Old → New)

| Old Name | New Name |
|----------|----------|
| Computer Vision | Azure AI Vision |
| Cognitive Services | Azure AI Services |
| Form Recognizer | Azure AI Document Intelligence |
| Cognitive Search | Azure AI Search |
| LUIS | Conversational Language Understanding (CLU) |
| QnA Maker | Custom Question Answering |
| Content Moderator | Azure AI Content Safety |
| Text Analytics | Azure AI Language |
| Anomaly Detector | (Retired) |
| Personalizer | (Retired) |

---

## Authentication Quick Reference

| Method | Header/Approach | Best For |
|--------|----------------|----------|
| API Key | `Ocp-Apim-Subscription-Key: <key>` | Dev/test |
| Entra ID | `Authorization: Bearer <token>` | Production |
| Managed Identity | `DefaultAzureCredential()` | Azure-to-Azure |
| Key Vault | Store + retrieve keys | Key management |

**Key Rotation**: App uses Key1 → Regen Key2 → Switch to Key2 → Regen Key1 = **zero downtime**

---

## Endpoint URL Patterns

| Service | Endpoint Pattern |
|---------|-----------------|
| AI Services (multi) | `https://<name>.cognitiveservices.azure.com/` |
| Language | `https://<name>.cognitiveservices.azure.com/language/` |
| Vision | `https://<name>.cognitiveservices.azure.com/computervision/` |
| Speech | `https://<region>.api.cognitive.microsoft.com/` |
| Translator | `https://api.cognitive.microsofttranslator.com/` |
| Document Intelligence | `https://<name>.cognitiveservices.azure.com/formrecognizer/` |
| OpenAI | `https://<name>.openai.azure.com/` |
| Content Safety | `https://<name>.cognitiveservices.azure.com/contentsafety/` |
| AI Search | `https://<name>.search.windows.net/` |

---

## Azure AI Language - Features at a Glance

| Feature | Method | Key Output |
|---------|--------|------------|
| Sentiment | `analyze_sentiment()` | positive/negative/neutral/mixed + scores |
| Opinion Mining | `analyze_sentiment(show_opinion_mining=True)` | target + assessment |
| Key Phrases | `extract_key_phrases()` | string array |
| NER | `recognize_entities()` | entity, category, confidence |
| Entity Linking | `recognize_linked_entities()` | entity + Wikipedia URL |
| PII | `recognize_pii_entities()` | entities + `redacted_text` |
| Language Detection | `detect_language()` | language name, ISO code, score |
| Summarization | `begin_analyze_actions()` | extractive/abstractive summary |

---

## CLU (Conversational Language Understanding) Cheatsheet

```
Project → Intents + Entities + Utterances → Train → Deploy → Predict
```

| Entity Type | Description | Example |
|-------------|-------------|---------|
| **Learned** | ML from labeled data | Product names |
| **List** | Exact match dictionary | Sizes: S, M, L |
| **Prebuilt** | Built-in types | DateTime, Number |
| **Regex** | Pattern match | `ORD-\d{6}` |

**Prediction URL**: `POST /language/:analyze-conversations?api-version=2023-04-01`

---

## Custom Question Answering Cheatsheet

| Source | Format |
|--------|--------|
| URLs | FAQ web pages |
| Files | .docx, .pdf, .xlsx, .tsv, .txt |
| Manual | Direct Q&A pairs |
| Chitchat | Pre-built personality |

- **Active Learning**: Auto-suggests alternate questions
- **Multi-turn**: Follow-up prompts for clarification
- **Metadata**: Filter Q&A pairs by key-value tags

---

## Computer Vision Quick Reference

### Image Analysis 4.0 Visual Features
`CAPTION | DENSE_CAPTIONS | TAGS | OBJECTS | PEOPLE | SMART_CROPS | READ`

### Custom Vision
- **Min images**: 5 per tag (recommend 50+)
- **Types**: Classification (multiclass/multilabel) | Object Detection
- **Compact domains**: Exportable to edge (ONNX, TensorFlow, CoreML)

### Face API
- **Detection model**: `detection_03` (latest)
- **Recognition model**: `recognition_04` (latest)
- **Limited Access**: Identification + Verification need approval
- **PersonGroup** → add Persons → add Faces → Train → Identify

---

## Document Intelligence Quick Reference

| Model | What It Extracts |
|-------|-----------------|
| `prebuilt-read` | Text only (OCR) |
| `prebuilt-layout` | Text + Tables + Selection marks |
| `prebuilt-document` | Text + Tables + Key-value pairs |
| `prebuilt-invoice` | Invoice-specific fields |
| `prebuilt-receipt` | Receipt-specific fields |
| `prebuilt-idDocument` | ID card/passport fields |

### Custom Models
- **Template**: Fixed layout (min 5 docs)
- **Neural**: Variable layout (min 5 docs)
- **Composed**: Combines up to 200 sub-models with auto-routing

---

## Azure AI Search Quick Reference

### Pipeline
```
Data Source → Indexer → Skillset (AI enrichment) → Index → Query
```

### Field Attributes
**S**earchable | **F**ilterable | **S**ortable | **F**acetable | **R**etrievable

### Key Built-in Skills
| Skill | Input | Output |
|-------|-------|--------|
| OCR | image | text |
| Entity Recognition | text | entities |
| Key Phrase | text | keyPhrases |
| Language Detection | text | languageCode |
| Sentiment | text | score |
| Image Analysis | image | tags, captions |
| Text Split | text | text chunks |
| Shaper | any | shaped output for knowledge store |

### Custom Skill Contract
```json
Request:  { "values": [{ "recordId": "1", "data": {...} }] }
Response: { "values": [{ "recordId": "1", "data": {...}, "errors": [], "warnings": [] }] }
```

### Knowledge Store Projections
- **Table** → Azure Table Storage (for Power BI)
- **Object** → Blob Storage (JSON)
- **File** → Blob Storage (images)

### Query Types
- **Simple**: `wifi +luxury -pool`
- **Full Lucene**: `hotelName:ritz~ AND rating:[4 TO 5]`
- **Semantic**: Re-ranks with captions/answers
- **Vector**: `Collection(Edm.Single)` field + k-NN
- **Hybrid**: Text + Vector combined

---

## Azure OpenAI Quick Reference

### API Pattern
```
POST /openai/deployments/{deployment-name}/chat/completions?api-version=2024-06-01
Headers: api-key: <key>
```

### Message Roles: `system` → `user` → `assistant`

### Key Parameters
| Param | Default | Note |
|-------|---------|------|
| `temperature` | 1.0 | 0 = deterministic, 2 = creative |
| `max_tokens` | varies | Max output tokens |
| `top_p` | 1.0 | Don't use with temperature |
| `frequency_penalty` | 0 | Penalize repetition |
| `presence_penalty` | 0 | Encourage new topics |

### Prompt Engineering
| Technique | When |
|-----------|------|
| Zero-shot | Simple tasks, no examples needed |
| Few-shot | Need specific output format |
| Chain-of-thought | Complex reasoning |
| System message | Always — define behavior/constraints |

### RAG (On Your Data)
- `in_scope: true` → Only answer from provided data
- `strictness: 1-5` → How strictly to limit
- Uses Azure AI Search as retrieval backend

### Embedding Dimensions
| Model | Dimensions |
|-------|-----------|
| text-embedding-ada-002 | 1536 |
| text-embedding-3-small | 512/1536 |
| text-embedding-3-large | 256/1024/3072 |

---

## Content Safety Quick Reference

### 4 Categories × 4 Severity Levels
| Category | Levels |
|----------|--------|
| Hate | 0, 2, 4, 6 |
| Sexual | 0, 2, 4, 6 |
| Violence | 0, 2, 4, 6 |
| Self-Harm | 0, 2, 4, 6 |

### Features
| Feature | Purpose |
|---------|---------|
| **Text/Image Analysis** | Detect harmful content |
| **Blocklists** | Custom term blocking |
| **Prompt Shields** | Jailbreak detection |
| **Groundedness** | Verify AI output grounding |
| **Protected Material** | Detect copyrighted content |

---

## Speech Service Quick Reference

### STT: `SpeechRecognizer` + `SpeechConfig`
### TTS: `SpeechSynthesizer` + `SpeechConfig`
### Translation: `TranslationRecognizer` + `SpeechTranslationConfig`

### SSML Key Elements
| Element | Purpose |
|---------|---------|
| `<voice>` | Select voice |
| `<prosody>` | Rate, pitch, volume |
| `<break>` | Pause |
| `<mstts:express-as>` | Style (cheerful, sad, angry) |
| `<say-as>` | Interpret type (date, number, spell-out) |
| `<phoneme>` | Custom pronunciation |

---

## Translator Quick Reference

- **Header**: `Ocp-Apim-Subscription-Key` + `Ocp-Apim-Subscription-Region`
- **Endpoint**: `https://api.cognitive.microsofttranslator.com/`
- **Translate**: `/translate?api-version=3.0&to=fr&to=de`
- **Transliterate**: Script conversion
- **Document Translation**: Async batch via Blob Storage
- **Custom Translator**: Parallel documents, custom model category ID

---

## Container Deployment Cheatsheet

| Required Param | Value |
|----------------|-------|
| `Eula` | `accept` |
| `Billing` | Azure resource endpoint URI |
| `ApiKey` | Azure resource API key |

- Containers **must connect to Azure for billing** (except disconnected containers)
- Disconnected containers require **approval + commitment tier**

---

## Numbers to Remember

| Item | Value |
|------|-------|
| Custom Vision min images per tag | **5** (recommend 50+) |
| Custom model min documents | **5** per type |
| Composed model max sub-models | **200** |
| Cosmos DB max item size | **2 MB** |
| Content Safety severity levels | **0, 2, 4, 6** |
| DALL-E 3 max images per request | **1** |
| text-embedding-ada-002 dimensions | **1536** |
| Face PersonGroup max persons | **10,000** (free), **1,000,000** (S0) |
| Exam passing score | **700 / 1000** |
| Exam duration | **120 minutes** |

---

## References

- [AI-102 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102)
- [Azure AI Services Documentation](https://learn.microsoft.com/en-us/azure/ai-services/)
- [Azure OpenAI Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [Azure AI Search Documentation](https://learn.microsoft.com/en-us/azure/search/)
