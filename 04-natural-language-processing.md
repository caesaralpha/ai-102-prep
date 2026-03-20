# 04 - Natural Language Processing Solutions (30–35%)

> **This is the highest-weighted domain on the exam!**

## 4.1 Azure AI Language Service

### Prebuilt Features

| Feature | Description | Input |
|---------|-------------|-------|
| **Sentiment Analysis** | Detect positive/negative/mixed/neutral sentiment | Text |
| **Opinion Mining** | Aspect-level sentiment (e.g., "food was great, service was slow") | Text |
| **Key Phrase Extraction** | Extract main talking points | Text |
| **Named Entity Recognition (NER)** | Identify entities (person, org, location, date, etc.) | Text |
| **Entity Linking** | Link entities to Wikipedia | Text |
| **Language Detection** | Detect language of text | Text |
| **PII Detection** | Detect personally identifiable information | Text |
| **Text Summarization** | Extractive and abstractive summarization | Text/Conversation |
| **Text Analytics for Health** | Extract medical entities and relations | Clinical text |

### Sentiment Analysis Response

```json
{
  "documents": [
    {
      "id": "1",
      "sentiment": "mixed",
      "confidenceScores": {
        "positive": 0.6,
        "neutral": 0.1,
        "negative": 0.3
      },
      "sentences": [
        {
          "sentiment": "positive",
          "text": "The food was excellent",
          "confidenceScores": { "positive": 0.95, "neutral": 0.03, "negative": 0.02 }
        },
        {
          "sentiment": "negative",
          "text": "but the service was terrible",
          "confidenceScores": { "positive": 0.01, "neutral": 0.04, "negative": 0.95 }
        }
      ]
    }
  ]
}
```

### Python SDK for Language Service

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<key>")
)

documents = [
    "The hotel was wonderful and the staff very helpful.",
    "The room was dirty and the food was cold."
]

# Sentiment Analysis
response = client.analyze_sentiment(documents, show_opinion_mining=True)
for doc in response:
    print(f"Sentiment: {doc.sentiment}")
    for sentence in doc.sentences:
        for opinion in sentence.mined_opinions:
            target = opinion.target
            print(f"  Target: {target.text}, Sentiment: {target.sentiment}")
            for assessment in opinion.assessments:
                print(f"    Assessment: {assessment.text}, Sentiment: {assessment.sentiment}")

# NER
response = client.recognize_entities(documents)
for doc in response:
    for entity in doc.entities:
        print(f"{entity.text}: {entity.category} ({entity.confidence_score:.2f})")

# Key Phrases
response = client.extract_key_phrases(documents)
for doc in response:
    print(f"Key phrases: {doc.key_phrases}")

# PII Detection
response = client.recognize_pii_entities(documents)
for doc in response:
    print(f"Redacted: {doc.redacted_text}")
```

## 4.2 Conversational Language Understanding (CLU)

> **CLU replaces LUIS** (Language Understanding Intelligent Service, retired)

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Utterance** | A user's input text/speech |
| **Intent** | The action the user wants to perform |
| **Entity** | A data element extracted from the utterance |
| **Project** | Container for intents, entities, and utterances |

### CLU Workflow

```
1. Create a CLU Project (in Language Studio or via API)
2. Define Intents (e.g., BookFlight, GetWeather, None)
3. Define Entities (e.g., Destination, Date, Origin)
4. Add Utterances (labeled examples for each intent)
5. Train the Model
6. Evaluate (F1 Score, Precision, Recall per intent/entity)
7. Deploy to a Deployment Slot
8. Query the Prediction Endpoint
```

### Entity Types

| Type | Description | Example |
|------|-------------|---------|
| **Learned** | Machine-learned from examples | Custom product names |
| **List** | Exact match from a defined list | Sizes: "small", "medium", "large" |
| **Prebuilt** | Common entity types | `DateTime`, `Number`, `Temperature` |
| **Regex** | Regex pattern match | Order ID: `ORD-\d{6}` |

### CLU Prediction API

```json
POST https://<endpoint>/language/:analyze-conversations?api-version=2023-04-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
  "kind": "Conversation",
  "analysisInput": {
    "conversationItem": {
      "id": "1",
      "text": "Book a flight to Seattle next Friday",
      "participantId": "user1"
    }
  },
  "parameters": {
    "projectName": "FlightBooking",
    "deploymentName": "production"
  }
}
```

### CLU Response

```json
{
  "result": {
    "prediction": {
      "topIntent": "BookFlight",
      "intents": [
        { "category": "BookFlight", "confidenceScore": 0.95 },
        { "category": "None", "confidenceScore": 0.05 }
      ],
      "entities": [
        {
          "category": "Destination",
          "text": "Seattle",
          "confidenceScore": 0.92
        },
        {
          "category": "DateTime",
          "text": "next Friday",
          "resolutions": [{ "dateTimeSubKind": "Date", "value": "2026-03-06" }]
        }
      ]
    }
  }
}
```

## 4.3 Custom Text Classification

### Types

| Type | Description |
|------|-------------|
| **Single-label** | One class per document |
| **Multi-label** | Multiple classes per document |

### Workflow

```
1. Create a Language resource + Storage Account
2. Upload documents to Blob Storage
3. Create project in Language Studio
4. Label documents with classes
5. Train model
6. Evaluate (F1, Precision, Recall per class)
7. Deploy model
8. Classify new documents via API
```

### Data Requirements

- Minimum **10 labeled documents per class**
- Recommended **50+ per class** for best results
- Documents should represent real-world data distribution

## 4.4 Custom Named Entity Recognition (Custom NER)

### Workflow (similar to Custom Classification)

```
1. Upload documents to Blob Storage
2. Create project in Language Studio
3. Label entities in documents
4. Train model
5. Evaluate
6. Deploy and use
```

### Labeling Best Practices

- Label **at least 10 instances** per entity type
- Label **consistently** across all documents
- Include **negative examples** (documents without the entity)
- Label **boundary cases** (ambiguous examples)

## 4.5 Custom Question Answering (QnA)

> **Replaces QnA Maker** (retired)

### Knowledge Base Sources

| Source | Description |
|--------|-------------|
| **URLs** | Import FAQ pages from websites |
| **Files** | Upload .docx, .pdf, .xlsx, .tsv, .txt files |
| **Manual** | Add question-answer pairs manually |
| **Chitchat** | Add pre-built personality chitchat |

### Workflow

```
1. Create Language resource (with Custom Question Answering enabled)
2. Create a Knowledge Base project
3. Add sources (URLs, files, manual Q&A pairs)
4. Edit and refine Q&A pairs
5. Add alternate questions and metadata
6. Test in Language Studio
7. Deploy to production slot
8. Integrate with Bot Framework (optional)
```

### Multi-Turn Conversations

```
Q: How do I reset my password?
A: Which platform are you using?
  ├── Follow-up: Windows → Go to Settings > Accounts > Sign-in options
  ├── Follow-up: Mac → Go to System Preferences > Users & Groups
  └── Follow-up: Mobile → Go to Settings > Security
```

### Metadata Filtering

```json
{
  "question": "How to get a refund?",
  "top": 3,
  "filters": {
    "metadataFilter": {
      "metadata": [
        { "key": "product", "value": "subscription" }
      ],
      "logicalOperation": "AND"
    }
  }
}
```

### Confidence Score Thresholds

| Range | Guidance |
|-------|----------|
| **0.0 – 0.29** | Low confidence; likely wrong answer |
| **0.30 – 0.69** | Moderate; may need clarification |
| **0.70 – 1.0** | High confidence; likely correct |

### Active Learning

- System suggests **alternate questions** based on similar user queries
- Review and approve suggestions in Language Studio
- Improves matching over time

## 4.6 Azure AI Translator

### Text Translation

```python
import requests

endpoint = "https://api.cognitive.microsofttranslator.com"
path = "/translate?api-version=3.0&to=fr&to=de"
headers = {
    "Ocp-Apim-Subscription-Key": "<key>",
    "Ocp-Apim-Subscription-Region": "<region>",
    "Content-Type": "application/json"
}
body = [{"text": "Hello, how are you?"}]

response = requests.post(endpoint + path, headers=headers, json=body)
print(response.json())
```

### Translator Features

| Feature | Description |
|---------|-------------|
| **Translate** | Translate text between 100+ languages |
| **Transliterate** | Convert script (e.g., Japanese kanji → romaji) |
| **Detect** | Detect source language |
| **Dictionary Lookup** | Find alternate translations |
| **Dictionary Examples** | See translations in context |
| **Document Translation** | Translate entire documents (async, batch) |
| **Custom Translator** | Train custom translation models |

### Custom Translator

- Upload **parallel documents** (source + target language pairs)
- Build a **custom model** that understands your domain terminology
- Deploy to a **custom category ID** and use in Translate API call
- Minimum: **10,000 parallel sentences** recommended

### Document Translation

```http
POST https://<endpoint>/translator/document/batches?api-version=2024-05-01

{
  "inputs": [
    {
      "source": {
        "sourceUrl": "https://storage.blob.core.windows.net/source/...",
        "language": "en"
      },
      "targets": [
        {
          "targetUrl": "https://storage.blob.core.windows.net/target/...",
          "language": "fr"
        }
      ]
    }
  ]
}
```

## 4.7 Azure AI Speech Service

### Speech-to-Text (STT)

| Feature | Description |
|---------|-------------|
| **Real-time STT** | Transcribe audio in real-time |
| **Batch STT** | Transcribe large volumes asynchronously |
| **Custom Speech** | Train custom acoustic/language models |
| **Pronunciation Assessment** | Score pronunciation accuracy |

```python
import azure.cognitiveservices.speech as speechsdk

speech_config = speechsdk.SpeechConfig(
    subscription="<key>",
    region="<region>"
)
speech_config.speech_recognition_language = "en-US"

audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config
)

result = recognizer.recognize_once_async().get()
if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print(f"Recognized: {result.text}")
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("No speech recognized")
```

### Text-to-Speech (TTS)

| Feature | Description |
|---------|-------------|
| **Neural Voices** | Natural-sounding AI voices |
| **Custom Neural Voice** | Clone a voice with your own recordings |
| **SSML** | Fine-tune speech output (pitch, rate, volume, pauses) |
| **Audio Content Creation** | Language Studio tool for creating audio |

### SSML Example

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="http://www.w3.org/2001/mstts"
       xml:lang="en-US">
  <voice name="en-US-JennyNeural">
    <mstts:express-as style="cheerful">
      Welcome to our service!
    </mstts:express-as>
    <break time="500ms"/>
    <prosody rate="-10%" pitch="+5%">
      How can I help you today?
    </prosody>
  </voice>
</speak>
```

### SSML Key Elements

| Element | Description |
|---------|-------------|
| `<speak>` | Root element |
| `<voice>` | Select voice name |
| `<prosody>` | Control rate, pitch, volume |
| `<break>` | Insert pause |
| `<mstts:express-as>` | Set speaking style |
| `<say-as>` | Specify pronunciation type (date, number, spell-out) |
| `<phoneme>` | Custom pronunciation with IPA/SAPI |
| `<sub>` | Substitution (alias for acronyms) |

### Speech Translation

```python
translation_config = speechsdk.translation.SpeechTranslationConfig(
    subscription="<key>",
    region="<region>"
)
translation_config.speech_recognition_language = "en-US"
translation_config.add_target_language("fr")
translation_config.add_target_language("de")

recognizer = speechsdk.translation.TranslationRecognizer(
    translation_config=translation_config
)

result = recognizer.recognize_once_async().get()
if result.reason == speechsdk.ResultReason.TranslatedSpeech:
    print(f"Recognized: {result.text}")
    print(f"French: {result.translations['fr']}")
    print(f"German: {result.translations['de']}")
```

### Intent Recognition with Speech

```python
# Uses CLU (or LUIS legacy) for intent recognition from speech
intent_config = speechsdk.SpeechConfig(subscription="<key>", region="<region>")
intent_recognizer = speechsdk.intent.IntentRecognizer(speech_config=intent_config)

# Add intents from CLU model
model = speechsdk.intent.PatternMatchingModel()
model.intents.append(
    speechsdk.intent.PatternMatchingIntent("BookFlight", "Book a flight to {destination}")
)
intent_recognizer.add_intent(model)

result = intent_recognizer.recognize_once_async().get()
```

### Speaker Recognition

| Feature | Description |
|---------|-------------|
| **Speaker Verification** | 1:1 — Is this the claimed person? |
| **Speaker Identification** | 1:N — Who is this person? |
| **Text-dependent** | User must speak a specific passphrase |
| **Text-independent** | User speaks freely (min 20 sec enrollment) |

## 4.8 Azure AI Bot Service

### Bot Framework Integration

```
User → Channel (Teams, Web, Slack)
  → Azure Bot Service
    → Bot Logic (App Service / Functions)
      → Azure AI Language (CLU)
      → Custom Question Answering
      → Azure OpenAI
```

### Channels Supported

- Microsoft Teams
- Web Chat
- Direct Line
- Slack, Facebook, Twilio, Email, etc.

## Key Takeaways

1. **CLU replaces LUIS** — know intents, entities, utterances, deployment slots
2. **Custom Question Answering replaces QnA Maker** — know sources, multi-turn, active learning
3. **Sentiment + Opinion Mining** = aspect-level sentiment analysis
4. **SSML** is critical for TTS — know `<prosody>`, `<break>`, `<mstts:express-as>`
5. **Custom Translator** needs parallel documents
6. **Speech Translation** combines STT + Translation in one step
7. **Entity types in CLU**: Learned, List, Prebuilt, Regex
8. **PII Detection** returns redacted text

## References

- [Azure AI Language Documentation](https://learn.microsoft.com/en-us/azure/ai-services/language-service/)
- [Conversational Language Understanding](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
- [Custom Question Answering](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/overview)
- [Text Analytics SDK](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-textanalytics-readme)
- [Azure AI Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/)
- [Custom Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/custom-translator/overview)
- [Azure AI Speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/)
- [SSML Reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
- [Speech SDK Reference](https://learn.microsoft.com/en-us/python/api/azure-cognitiveservices-speech/)
- [Azure Bot Service](https://learn.microsoft.com/en-us/azure/bot-service/)

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Azure AI Language Service

### The Unified Language Service

Azure AI Language combines what used to be separate services into one:

| Old Service | New Feature | Status |
|-------------|-------------|--------|
| Text Analytics | Prebuilt features (sentiment, NER, etc.) | Unified |
| LUIS | Conversational Language Understanding (CLU) | LUIS retired Oct 2025 |
| QnA Maker | Custom Question Answering | QnA Maker retired Oct 2025 |
| Text Analytics for Health | Still separate due to compliance | Available |

### Prebuilt vs Custom Features

| Prebuilt (No Training) | Custom (Training Required) |
|------------------------|---------------------------|
| Sentiment Analysis | CLU (Conversational Understanding) |
| Key Phrase Extraction | Custom Text Classification |
| NER (Named Entity Recognition) | Custom NER |
| PII Detection | Custom Question Answering |
| Language Detection | |
| Summarization | |

## Deep Dive: Sentiment Analysis

### How Sentiment Works

The model analyzes text and classifies it as:
- **Positive**: Expressing approval, happiness, satisfaction
- **Negative**: Expressing disapproval, disappointment, frustration
- **Neutral**: Factual statements, no emotion
- **Mixed**: Contains both positive and negative sentiments

### Document-Level vs Sentence-Level

```python
documents = ["The hotel was amazing but the food was terrible."]
response = client.analyze_sentiment(documents)

# Document-level (overall)
print(response[0].sentiment)  # "mixed"

# Sentence-level (granular)
for sentence in response[0].sentences:
    print(f"{sentence.text} → {sentence.sentiment}")
    # "The hotel was amazing" → "positive"
    # "but the food was terrible" → "negative"
```

### Opinion Mining (Aspect-Based Sentiment)

Goes deeper than sentence-level — identifies WHAT is being talked about and how:

```python
response = client.analyze_sentiment(documents, show_opinion_mining=True)
```

**Output Structure**:
```
Sentence: "The hotel was amazing"
├── Target: "hotel" → positive
│   └── Assessment: "amazing" → positive

Sentence: "but the food was terrible"  
├── Target: "food" → negative
│   └── Assessment: "terrible" → negative
```

**Real-World Use Case**: Product reviews analysis
- Extract: "battery life" → positive, "screen" → negative, "camera" → positive
- Aggregate across thousands of reviews
- Identify product strengths/weaknesses

## Deep Dive: Conversational Language Understanding (CLU)

### Key Terminology

| Term | Simple Definition | Example |
|------|-------------------|---------|
| **Utterance** | What the user says | "Book a flight to Seattle tomorrow" |
| **Intent** | What the user wants to DO | `BookFlight` |
| **Entity** | Important data to EXTRACT | Destination: "Seattle", Date: "tomorrow" |

### CLU vs LUIS

| Feature | LUIS (Retired) | CLU |
|---------|---------------|-----|
| Interface | Separate LUIS portal | Language Studio |
| Entity Types | Hierarchical, composite | Learned, List, Prebuilt, Regex |
| Orchestration | No native support | Orchestration workflows |
| Integration | Standalone | Part of Language Service |

### Entity Types Explained

#### 1. Learned Entities
Machine learning extracts entities from training examples.

```
Training:
- "Book a flight to Seattle" → Destination: Seattle
- "I want to fly to London" → Destination: London
- "Reserve ticket to Paris" → Destination: Paris

After training, model learns pattern and extracts destinations automatically.
```

**When to Use**: Custom entities specific to your domain

#### 2. List Entities
Exact match from a predefined list with synonyms.

```json
{
  "entity": "Size",
  "list": [
    { "canonicalForm": "Small", "synonyms": ["S", "sm", "petit"] },
    { "canonicalForm": "Medium", "synonyms": ["M", "med", "regular"] },
    { "canonicalForm": "Large", "synonyms": ["L", "lg", "big"] }
  ]
}
```

**When to Use**: Closed sets of values (sizes, colors, product SKUs)

#### 3. Prebuilt Entities
Pre-trained to recognize common types:

| Prebuilt Entity | Recognizes |
|----------------|------------|
| `DateTime` | Dates, times, durations |
| `Number` | Cardinals, ordinals |
| `Temperature` | Fahrenheit, Celsius |
| `Currency` | Money amounts |
| `Email` | Email addresses |
| `URL` | Web addresses |

#### 4. Regex Entities
Pattern matching using regular expressions.

```
Pattern: ORD-\d{6}
Matches: "ORD-123456", "ORD-999888"
```

**When to Use**: Structured identifiers (order IDs, license plates, product codes)

### Training Best Practices

| Recommendation | Why |
|----------------|-----|
| 15+ utterances per intent | Minimum for decent accuracy |
| 50+ utterances per intent | Recommended for production |
| Varied phrasing | "Book a flight", "I want to fly", "Reserve a ticket" |
| Include entities | Label ALL entity instances |
| Balance intents | Similar number of utterances across intents |
| Include None intent | Examples of out-of-scope requests |

### Evaluating CLU Models

After training, evaluate with test data:

| Metric | Question It Answers |
|--------|-------------------|
| **Precision** | Of predictions made, how many were correct? |
| **Recall** | Of actual cases, how many did we find? |
| **F1 Score** | Harmonic mean of precision and recall |

**Per Intent and Per Entity**: You get metrics for each intent and each entity type separately.

## Deep Dive: Custom Question Answering

### When to Use Q&A vs CLU

| Scenario | Use |
|----------|-----|
| User asks questions expecting answers | Custom Q&A |
| User wants to DO something (book, order, schedule) | CLU |
| FAQ-style interactions | Custom Q&A |
| Task-oriented conversations | CLU |
| Simple chitchat | Custom Q&A |

### Knowledge Base Sources

| Source Type | Best For | Considerations |
|-------------|----------|----------------|
| **URLs** | Existing FAQ pages | Pages must be crawlable |
| **PDF/DOCX** | Internal documents | Extract Q&A pairs automatically |
| **Excel/TSV** | Structured Q&A lists | Direct import of Q&A pairs |
| **Manual** | Custom pairs | Full control over Q&A |
| **Chitchat** | Personality | Pre-built conversational responses |

### Multi-Turn Conversations Explained

Simple Q&A is one question, one answer. Multi-turn adds follow-up context:

```
User: "How do I return a product?"
Bot: "What type of product do you want to return?"
    ├── [Electronics] → "Electronics must be returned within 30 days..."
    ├── [Clothing] → "Clothing can be returned within 60 days..."
    └── [Other] → "Please contact customer support..."
```

This creates a decision tree starting from one question.

### Active Learning

The system learns from real user queries:

```
Original Q&A:
  Q: "How do I reset my password?"
  A: "Go to Settings > Security > Reset Password..."

Users also asked:
  - "forgot my password"
  - "change password"
  - "password recovery"

Active Learning Suggestion:
  Add these as alternate questions for the same answer
```

**Benefits**:
- Improves matching over time
- Discovers how real users phrase questions
- Reduces "I don't understand" responses

### Confidence Thresholds

```python
# Set threshold in query
response = qna_client.get_answers(
    question="How do I return?",
    confidence_threshold=0.7
)

if response.answers[0].confidence >= 0.7:
    return response.answers[0].answer
else:
    return "I'm not sure. Let me connect you to an agent."
```

## Deep Dive: Translator

### Understanding Translation API Structure

The Translator API is regional — you must specify the region:

```python
headers = {
    "Ocp-Apim-Subscription-Key": "<key>",
    "Ocp-Apim-Subscription-Region": "<region>",  # Required!
    "Content-Type": "application/json"
}
```

### Translate vs Transliterate

| Operation | Input | Output |
|-----------|-------|--------|
| **Translate** | "Hello" (English) | "Bonjour" (French) |
| **Transliterate** | "こんにちは" (Japanese kanji) | "Konnichiwa" (Latin script) |

Transliterate converts SCRIPT, not language. The meaning stays the same.

### Custom Translator Deep Dive

For domain-specific translations where generic models fail:

```
Generic Model:
"The Python script failed" → "Le script Python a échoué"
(Might translate "Python" as the snake)

Custom Model (trained on tech docs):
"The Python script failed" → "Le script Python a échoué"
(Knows Python is a programming language)
```

**Training Requirements**:
- Parallel documents (same content in both languages)
- Minimum 10,000 parallel sentences recommended
- Domain-specific terminology

### Document Translation

For batch processing of entire documents:

```
1. Upload source documents to Blob Storage
2. Specify target container for translations
3. Submit batch job
4. Poll for completion
5. Download translated documents

Supports: PDF, DOCX, PPTX, XLSX, HTML, TXT, and more
```

## Deep Dive: Speech Service

### Speech-to-Text (STT) Configurations

```python
speech_config = speechsdk.SpeechConfig(
    subscription="<key>",
    region="<region>"
)

# Language setting
speech_config.speech_recognition_language = "en-US"

# Audio configuration options
audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
# OR
audio_config = speechsdk.audio.AudioConfig(filename="audio.wav")
# OR  
audio_config = speechsdk.audio.AudioConfig(stream=push_stream)
```

### Continuous vs Single Recognition

| Mode | Use Case | Method |
|------|----------|--------|
| **Single** | Short phrases, commands | `recognize_once_async()` |
| **Continuous** | Long-form speech, dictation | `start_continuous_recognition_async()` |

```python
# Continuous recognition
def recognized_callback(evt):
    print(f"Recognized: {evt.result.text}")

recognizer.recognized.connect(recognized_callback)
recognizer.start_continuous_recognition_async()
# ... later ...
recognizer.stop_continuous_recognition_async()
```

### Text-to-Speech (TTS) Voice Selection

Azure offers 400+ neural voices across 140+ languages:

```python
speech_config.speech_synthesis_voice_name = "en-US-JennyNeural"
```

**Voice Naming Convention**: `{language}-{country}-{name}Neural`

| Example | Description |
|---------|-------------|
| `en-US-JennyNeural` | English (US), Jenny voice |
| `en-GB-SoniaNeural` | English (UK), Sonia voice |
| `zh-CN-XiaoxiaoNeural` | Chinese (Mandarin), Xiaoxiao voice |

### SSML Deep Dive

Speech Synthesis Markup Language gives fine control over speech output:

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="http://www.w3.org/2001/mstts"
       xml:lang="en-US">
  
  <!-- Voice selection -->
  <voice name="en-US-JennyNeural">
    
    <!-- Speaking style (emotion/tone) -->
    <mstts:express-as style="cheerful">
      Welcome to our service!
    </mstts:express-as>
    
    <!-- Pause -->
    <break time="500ms"/>
    
    <!-- Speed and pitch control -->
    <prosody rate="-10%" pitch="+5%">
      How may I help you today?
    </prosody>
    
    <!-- Pronunciation guidance -->
    <say-as interpret-as="date" format="mdy">3/20/2026</say-as>
    
    <!-- Custom pronunciation -->
    <phoneme alphabet="ipa" ph="ˈkɒɡnɪtɪv">cognitive</phoneme>
    
    <!-- Substitution (read differently than written) -->
    <sub alias="Azure AI">AAI</sub>
    
  </voice>
</speak>
```

**Key SSML Elements**:

| Element | Purpose | Example |
|---------|---------|---------|
| `<voice>` | Select voice | `name="en-US-JennyNeural"` |
| `<prosody>` | Speed, pitch, volume | `rate="-20%" pitch="+10%"` |
| `<break>` | Insert pause | `time="500ms"` or `strength="strong"` |
| `<mstts:express-as>` | Speaking style | `style="cheerful"`, `style="sad"` |
| `<say-as>` | Interpretation | `interpret-as="date"`, `"telephone"` |
| `<phoneme>` | Custom pronunciation | IPA symbols |
| `<sub>` | Read alias instead | Acronyms, abbreviations |

### Speaker Recognition

#### Verification (1:1)
"Is this voice the person they claim to be?"

```
Enrollment: User speaks passphrase 3 times
Verification: User speaks same passphrase
Result: Match (confidence score) or No Match
```

#### Identification (1:N)
"Who is speaking from this group of people?"

```
Enrollment: Enroll multiple speakers (minimum 20 seconds each)
Identification: New audio sample
Result: "This is Speaker #3 with 95% confidence"
```

**Text-Independent**: No specific passphrase needed — user speaks freely

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Analyze text with Azure AI Language](https://learn.microsoft.com/en-us/training/modules/analyze-text-ai-language/)
- [Build a conversational language understanding model](https://learn.microsoft.com/en-us/training/modules/build-language-understanding-model/)
- [Create a question answering solution](https://learn.microsoft.com/en-us/training/modules/create-question-answer-solution/)
- [Translate text and speech](https://learn.microsoft.com/en-us/training/modules/translate-text-speech/)
- [Create speech-enabled apps](https://learn.microsoft.com/en-us/training/modules/create-speech-enabled-apps/)

### Official Documentation
- [Azure AI Language documentation](https://learn.microsoft.com/en-us/azure/ai-services/language-service/)
- [CLU documentation](https://learn.microsoft.com/en-us/azure/ai-services/language-service/conversational-language-understanding/overview)
- [Question Answering documentation](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/overview)
- [Translator documentation](https://learn.microsoft.com/en-us/azure/ai-services/translator/)
- [Speech Service documentation](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/)
- [SSML reference](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-synthesis-markup)
