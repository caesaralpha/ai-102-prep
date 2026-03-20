# 06 - Knowledge Mining with Azure AI Search (10–15%)

## 6.1 Azure AI Search Overview

Azure AI Search (formerly Azure Cognitive Search) is a cloud search service that provides AI-powered enrichment and full-text search.

### Core Components

```
Data Source → Indexer → Skillset → Index → Query
                          ↓
                    AI Enrichment
                    (Skills Pipeline)
```

| Component | Description |
|-----------|-------------|
| **Data Source** | Connection to source data (Blob, SQL, Cosmos DB, Table Storage) |
| **Indexer** | Automated crawler that reads data and populates the index |
| **Skillset** | Pipeline of AI enrichment steps |
| **Index** | Searchable data store with defined schema |
| **Knowledge Store** | Optional — persist enriched data back to storage |

## 6.2 Supported Data Sources

| Source | Type |
|--------|------|
| **Azure Blob Storage** | Files (PDF, DOCX, images, etc.) |
| **Azure Data Lake Storage Gen2** | Files |
| **Azure Table Storage** | Structured rows |
| **Azure SQL Database** | Tables/views |
| **Azure Cosmos DB** | JSON documents (SQL API, MongoDB API) |
| **Azure MySQL** | Tables |

## 6.3 Index Schema

### Field Attributes

| Attribute | Description |
|-----------|-------------|
| **searchable** | Full-text search is applied |
| **filterable** | Can be used in `$filter` expressions |
| **sortable** | Can be used in `$orderby` |
| **facetable** | Can be used for faceted navigation |
| **retrievable** | Returned in search results |
| **key** | Unique document identifier (one per index) |

### Index Definition Example

```json
{
  "name": "hotels-index",
  "fields": [
    { "name": "hotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "hotelName", "type": "Edm.String", "searchable": true, "retrievable": true },
    { "name": "description", "type": "Edm.String", "searchable": true, "retrievable": true, "analyzer": "en.microsoft" },
    { "name": "category", "type": "Edm.String", "filterable": true, "facetable": true, "sortable": true },
    { "name": "rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true },
    { "name": "location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true },
    { "name": "tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "facetable": true },
    { "name": "imageText", "type": "Edm.String", "searchable": true }
  ],
  "suggesters": [
    { "name": "sg", "searchMode": "analyzingInfixMatching", "sourceFields": ["hotelName"] }
  ]
}
```

### Field Types

| Type | Description |
|------|-------------|
| `Edm.String` | Text |
| `Edm.Int32` / `Edm.Int64` | Integer |
| `Edm.Double` | Floating point |
| `Edm.Boolean` | True/false |
| `Edm.DateTimeOffset` | Date/time |
| `Edm.GeographyPoint` | Latitude/longitude |
| `Collection(Edm.String)` | String array |
| `Edm.ComplexType` | Nested object |
| `Collection(Edm.Single)` | Vector field (for vector search) |

## 6.4 AI Enrichment (Skillsets)

### Built-in Skills

| Skill | Category | Description |
|-------|----------|-------------|
| **OCR** | Vision | Extract text from images |
| **Image Analysis** | Vision | Generate captions, tags |
| **Key Phrase Extraction** | Language | Extract key phrases |
| **Language Detection** | Language | Detect document language |
| **Sentiment Analysis** | Language | Detect sentiment |
| **Entity Recognition** | Language | Extract named entities |
| **Entity Linking** | Language | Link to Wikipedia |
| **PII Detection** | Language | Detect PII |
| **Text Split** | Utility | Split text into chunks |
| **Text Merge** | Utility | Merge text from multiple sources |
| **Shaper** | Utility | Shape complex types for knowledge store |
| **Custom Web API** | Custom | Call your own REST endpoint |
| **Azure ML** | Custom | Call Azure ML model endpoint |

### Skillset Definition Example

```json
{
  "name": "my-skillset",
  "description": "Enrich documents with AI",
  "skills": [
    {
      "@odata.type": "#Microsoft.Skills.Text.V3.EntityRecognitionSkill",
      "name": "entity-recognition",
      "context": "/document",
      "inputs": [
        { "name": "text", "source": "/document/content" }
      ],
      "outputs": [
        { "name": "persons", "targetName": "people" },
        { "name": "organizations", "targetName": "orgs" }
      ],
      "categories": ["Person", "Organization", "Location"]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
      "name": "key-phrases",
      "context": "/document",
      "inputs": [
        { "name": "text", "source": "/document/content" }
      ],
      "outputs": [
        { "name": "keyPhrases", "targetName": "keyPhrases" }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "name": "ocr",
      "context": "/document/normalized_images/*",
      "inputs": [
        { "name": "image", "source": "/document/normalized_images/*" }
      ],
      "outputs": [
        { "name": "text", "targetName": "ocrText" }
      ]
    }
  ]
}
```

### Enrichment Pipeline Flow

```
Document
├── /document/content  (raw text)
├── /document/normalized_images/*  (extracted images)
│   └── OCR Skill → /document/normalized_images/*/ocrText
├── Text Merge → /document/merged_content
├── Entity Recognition → /document/people, /document/orgs
├── Key Phrase Extraction → /document/keyPhrases
└── Output to Index fields
```

### Custom Skills

```json
{
  "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
  "name": "custom-skill",
  "uri": "https://my-function-app.azurewebsites.net/api/enrich",
  "httpMethod": "POST",
  "timeout": "PT30S",
  "batchSize": 10,
  "context": "/document",
  "inputs": [
    { "name": "text", "source": "/document/content" }
  ],
  "outputs": [
    { "name": "customResult", "targetName": "customEnrichment" }
  ]
}
```

Custom skill expected request/response:

```json
// Request
{
  "values": [
    {
      "recordId": "1",
      "data": { "text": "Some document text" }
    }
  ]
}

// Response
{
  "values": [
    {
      "recordId": "1",
      "data": { "customResult": "enriched value" },
      "errors": [],
      "warnings": []
    }
  ]
}
```

## 6.5 Knowledge Store

Persist enriched data to Azure Storage for further analysis.

### Projection Types

| Type | Storage | Use Case |
|------|---------|----------|
| **Table** | Azure Table Storage | Structured data, Power BI |
| **Object** | Blob Storage (JSON) | Full enrichment tree |
| **File** | Blob Storage (binary) | Normalized images |

```json
{
  "knowledgeStore": {
    "storageConnectionString": "<connection-string>",
    "projections": [
      {
        "tables": [
          {
            "tableName": "documentsTable",
            "generatedKeyName": "docKey",
            "source": "/document/enriched"
          }
        ],
        "objects": [
          {
            "storageContainer": "documents-json",
            "source": "/document/enriched"
          }
        ],
        "files": [
          {
            "storageContainer": "images",
            "source": "/document/normalized_images/*"
          }
        ]
      }
    ]
  }
}
```

## 6.6 Querying

### Search Query Types

| Type | Syntax | Example |
|------|--------|---------|
| **Simple** | Keywords and operators | `wifi +luxury -pool` |
| **Full Lucene** | Advanced query syntax | `hotelName:ritz~ AND rating:[4 TO 5]` |

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `search` | Search text | `luxury hotel` |
| `$filter` | OData filter expression | `rating ge 4` |
| `$orderby` | Sort results | `rating desc` |
| `$select` | Fields to return | `hotelName,rating` |
| `$top` | Number of results | `10` |
| `$skip` | Offset for paging | `20` |
| `$count` | Include total count | `true` |
| `facet` | Faceted navigation | `category,count:5` |
| `highlight` | Search hit highlighting | `description` |
| `searchMode` | `any` (OR) or `all` (AND) | `all` |
| `queryType` | `simple` or `full` | `full` |

### Filter Examples

```
# Exact match
$filter=category eq 'Luxury'

# Range
$filter=rating ge 4 and rating le 5

# Collection contains
$filter=tags/any(t: t eq 'wifi')

# Geo-distance
$filter=geo.distance(location, geography'POINT(-122.12 47.67)') le 10

# Null check
$filter=description ne null
```

### Autocomplete and Suggestions

```http
# Suggestions
GET /indexes/hotels/docs/suggest?search=lux&suggesterName=sg&$select=hotelName

# Autocomplete
GET /indexes/hotels/docs/autocomplete?search=lux&suggesterName=sg&autocompleteMode=twoTerms
```

## 6.7 Semantic Ranking

| Feature | Description |
|---------|-------------|
| **Semantic Ranker** | Re-ranks results using deep learning models |
| **Semantic Captions** | Extracts relevant passages from documents |
| **Semantic Answers** | Extracts direct answers from documents |

```http
POST /indexes/hotels/docs/search?api-version=2024-07-01

{
  "search": "what hotels have good views",
  "queryType": "semantic",
  "semanticConfiguration": "my-semantic-config",
  "captions": "extractive",
  "answers": "extractive|count-3"
}
```

## 6.8 Vector Search

```json
// Index field definition for vectors
{
  "name": "contentVector",
  "type": "Collection(Edm.Single)",
  "searchable": true,
  "dimensions": 1536,
  "vectorSearchProfile": "my-vector-profile"
}

// Vector query
{
  "vectorQueries": [
    {
      "kind": "vector",
      "vector": [0.01, 0.02, ...],
      "fields": "contentVector",
      "k": 5
    }
  ]
}
```

### Hybrid Search = Vector + Text
```json
{
  "search": "luxury hotels with views",
  "vectorQueries": [
    {
      "kind": "vector",
      "vector": [0.01, 0.02, ...],
      "fields": "contentVector",
      "k": 5
    }
  ]
}
```

## 6.9 Indexer Schedule and Change Detection

```json
{
  "schedule": {
    "interval": "PT2H",
    "startTime": "2025-01-01T00:00:00Z"
  },
  "parameters": {
    "configuration": {
      "dataToExtract": "contentAndMetadata",
      "imageAction": "generateNormalizedImages"
    }
  }
}
```

| Interval Format | Duration |
|----------------|----------|
| `PT5M` | Every 5 minutes |
| `PT1H` | Every hour |
| `PT24H` | Every 24 hours |

## Key Takeaways

1. **Pipeline**: Data Source → Indexer → Skillset → Index → Query
2. **Field attributes**: searchable, filterable, sortable, facetable, retrievable
3. **Custom skills** use Web API format with `recordId`, `data`, `errors`, `warnings`
4. **Knowledge Store** projects to tables, objects, or files
5. **Semantic ranking** re-ranks with deep learning; provides captions and answers
6. **Vector search** uses `Collection(Edm.Single)` field type
7. **Hybrid search** = text query + vector query combined

## References

- [Azure AI Search Documentation](https://learn.microsoft.com/en-us/azure/search/)

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Azure AI Search Architecture

### The Big Picture

Azure AI Search is a search-as-a-service platform that:
1. **Ingests** content from various sources
2. **Enriches** content with AI (optional)
3. **Indexes** content for fast retrieval
4. **Queries** content with powerful search capabilities

### The Pipeline Flow

```
┌─────────────┐     ┌──────────┐     ┌───────────┐     ┌─────────┐
│ Data Source │ ──► │ Indexer  │ ──► │ Skillset  │ ──► │  Index  │
│             │     │(crawler) │     │(AI enrich)│     │         │
└─────────────┘     └──────────┘     └───────────┘     └─────────┘
      │                                    │                │
      │                                    ▼                │
      │                            ┌──────────────┐        │
      │                            │ Knowledge    │        │
      │                            │ Store        │        │
      │                            │(projections) │        │
      │                            └──────────────┘        │
      │                                                    │
      │                                                    ▼
      │                                             ┌────────────┐
      │                                             │   Search   │
      │                                             │   Queries  │
      │                                             └────────────┘
```

## Deep Dive: Data Sources

### Supported Data Sources

| Source | What It Indexes | Connection |
|--------|-----------------|------------|
| **Azure Blob Storage** | Files (PDF, DOCX, images) | Storage connection string |
| **Azure Data Lake Gen2** | Files with hierarchical namespace | ADLS connection string |
| **Azure SQL Database** | Table/view rows | SQL connection string |
| **Azure Cosmos DB** | JSON documents | Cosmos connection string |
| **Azure Table Storage** | Table rows | Storage connection string |

### Data Source Configuration

```json
{
  "name": "my-blob-source",
  "type": "azureblob",
  "credentials": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=..."
  },
  "container": {
    "name": "documents",
    "query": "subfolder/"
  }
}
```

### Document Cracking

When indexing from Blob Storage, Azure AI Search "cracks" documents:

| File Type | What's Extracted |
|-----------|------------------|
| PDF | Text (per page), metadata |
| DOCX/DOC | Text, embedded images |
| PPTX | Text from slides |
| XLSX | Cell contents |
| Images | Normalized images (for OCR skills) |
| JSON | Properties as fields |
| HTML | Text content |

## Deep Dive: Index Schema

### Understanding Field Attributes

Each field in an index has attributes that control behavior:

| Attribute | What It Enables | Example Use |
|-----------|----------------|-------------|
| **key** | Unique document identifier | `hotelId` |
| **searchable** | Full-text search | `description` for keyword search |
| **filterable** | `$filter` expressions | `category eq 'Luxury'` |
| **sortable** | `$orderby` expressions | Sort by `rating desc` |
| **facetable** | Faceted navigation | Category drill-down |
| **retrievable** | Returned in results | Which fields to show |

### Choosing Attributes

**Scenario**: Hotel search application

```json
{
  "name": "hotels-index",
  "fields": [
    {
      "name": "hotelId",
      "type": "Edm.String",
      "key": true,          // Unique identifier
      "retrievable": true   // Need to show it
    },
    {
      "name": "hotelName",
      "type": "Edm.String",
      "searchable": true,   // Search by name
      "retrievable": true   // Show in results
    },
    {
      "name": "description",
      "type": "Edm.String",
      "searchable": true,   // Full-text search
      "retrievable": true,
      "analyzer": "en.microsoft"  // English analysis
    },
    {
      "name": "category",
      "type": "Edm.String",
      "filterable": true,   // Filter: $filter=category eq 'Luxury'
      "facetable": true,    // Facet: count by category
      "sortable": true,     // Sort alphabetically
      "retrievable": true
    },
    {
      "name": "rating",
      "type": "Edm.Double",
      "filterable": true,   // Filter: $filter=rating ge 4
      "sortable": true,     // Sort: $orderby=rating desc
      "facetable": true,    // Facet: count by rating
      "retrievable": true
    },
    {
      "name": "internalNotes",
      "type": "Edm.String",
      "searchable": false,  // Don't search this
      "retrievable": false  // Don't expose to users
    }
  ]
}
```

### Analyzers Explained

Analyzers process text for searching:

```
Input: "The quick brown FOXES jumped!"
        ↓ (Analyzer)
Tokens: ["quick", "brown", "fox", "jump"]
```

**What Analyzers Do**:
1. Lowercase
2. Remove stop words ("the", "a", "is")
3. Stem words ("foxes" → "fox", "jumped" → "jump")
4. Handle language-specific rules

**Built-in Analyzers**:
| Analyzer | Language | Use |
|----------|----------|-----|
| `en.microsoft` | English | Best for English text |
| `en.lucene` | English | Alternative English |
| `standard` | Generic | Language-agnostic |
| `keyword` | None | Exact match (no tokenization) |

## Deep Dive: Skillsets (AI Enrichment)

### How Skillsets Work

Skills transform content during indexing:

```
Original Document: PDF with text and images
        ↓
┌─────────────────────────────────────────────┐
│ Skillset Pipeline                            │
│  1. OCR Skill → Extract text from images    │
│  2. Text Merge → Combine all text           │
│  3. Language Detection → "en"               │
│  4. Key Phrase Extraction → ["AI", "ML"]    │
│  5. Entity Recognition → ["Microsoft", "Seattle"] │
│  6. Sentiment → "positive" (0.85)           │
└─────────────────────────────────────────────┘
        ↓
Enriched Document (indexed with all extracted data)
```

### Understanding Context and Inputs/Outputs

Skills operate on a **document tree**:

```
/document
├── /document/content              (original text)
├── /document/normalized_images/0  (first embedded image)
├── /document/normalized_images/1  (second embedded image)
├── /document/merged_content       (created by skill)
├── /document/keyPhrases           (created by skill)
└── /document/people               (created by skill)
```

**Skill Input/Output Example**:
```json
{
  "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
  "name": "key-phrases",
  "context": "/document",  // Run once per document
  "inputs": [
    {
      "name": "text",
      "source": "/document/content"  // Input from here
    }
  ],
  "outputs": [
    {
      "name": "keyPhrases",
      "targetName": "keyPhrases"  // Store at /document/keyPhrases
    }
  ]
}
```

### Processing Images with Skills

```
Document (PDF with images)
        ↓ (Document cracking)
/document/normalized_images/*  (extracted images)
        ↓ (OCR Skill)
/document/normalized_images/*/ocrText  (text from each image)
        ↓ (Text Merge Skill)
/document/merged_content  (all text combined)
```

### Custom Skills

When built-in skills aren't enough, create your own:

```
Azure AI Search ←→ Your Azure Function
                   (Custom processing)
```

**Request Format** (sent to your function):
```json
{
  "values": [
    {
      "recordId": "abc123",
      "data": {
        "text": "Content to process",
        "otherField": "..."
      }
    }
  ]
}
```

**Response Format** (your function returns):
```json
{
  "values": [
    {
      "recordId": "abc123",
      "data": {
        "customResult": "Processed output"
      },
      "errors": [],
      "warnings": []
    }
  ]
}
```

**Critical**: 
- `recordId` must match input
- Return `errors` array for failures
- Return `warnings` array for non-fatal issues

## Deep Dive: Knowledge Store

### What is Knowledge Store?

While the Index is for searching, Knowledge Store saves enrichments for OTHER purposes:
- Power BI dashboards
- Data science analysis
- Archive enriched content
- Feed other systems

### Projection Types

| Type | Storage | Best For |
|------|---------|----------|
| **Table** | Azure Table Storage | Structured querying, Power BI |
| **Object** | Blob (JSON) | Preserving full enrichment tree |
| **File** | Blob (binary) | Extracted images |

### Knowledge Store Example

```json
{
  "knowledgeStore": {
    "storageConnectionString": "<connection>",
    "projections": [
      {
        "tables": [
          {
            "tableName": "DocumentsTable",
            "generatedKeyName": "Documentid",
            "source": "/document/enriched"
          },
          {
            "tableName": "KeyPhrasesTable",
            "generatedKeyName": "KeyPhraseId",
            "source": "/document/keyPhrases/*"
          }
        ],
        "objects": [
          {
            "storageContainer": "enriched-documents",
            "source": "/document"
          }
        ],
        "files": [
          {
            "storageContainer": "extracted-images",
            "source": "/document/normalized_images/*"
          }
        ]
      }
    ]
  }
}
```

## Deep Dive: Querying

### Query Types

#### Simple Query Syntax
Default, user-friendly syntax:

```
wifi +luxury -pool
```
- `wifi` = must contain "wifi"
- `+luxury` = MUST contain "luxury"
- `-pool` = must NOT contain "pool"

#### Full Lucene Query Syntax
Advanced, powerful syntax:

```
hotelName:ritz~2 AND rating:[4 TO 5] AND NOT category:budget
```
- `hotelName:ritz~2` = "ritz" with fuzzy matching (2 edits allowed)
- `rating:[4 TO 5]` = range query
- `NOT category:budget` = exclusion

### Filter Expressions

Filters use OData syntax and operate on filterable fields:

```
# Exact match
$filter=category eq 'Luxury'

# Range
$filter=rating ge 4 and rating le 5

# Collection contains
$filter=tags/any(t: t eq 'wifi')

# Geographic distance
$filter=geo.distance(location, geography'POINT(-122.12 47.67)') le 10

# Combining conditions
$filter=rating ge 4 and (category eq 'Luxury' or category eq 'Boutique')
```

### Faceted Navigation

```
GET /indexes/hotels/docs/search?api-version=2024-07-01
  &search=*
  &facet=category,count:10
  &facet=rating
```

**Response**:
```json
{
  "@search.facets": {
    "category": [
      { "value": "Luxury", "count": 45 },
      { "value": "Budget", "count": 32 },
      { "value": "Boutique", "count": 28 }
    ],
    "rating": [
      { "value": 5, "count": 20 },
      { "value": 4, "count": 35 },
      { "value": 3, "count": 30 }
    ]
  }
}
```

## Deep Dive: Semantic Search

### What is Semantic Ranking?

Traditional search: Match keywords, rank by frequency/relevance scores

Semantic search: Understand MEANING, re-rank using AI

```
Query: "what hotels have nice views"

Traditional ranking might return:
1. "Pleasant View Hotel" (keyword match)
2. "Nice Place Inn" (keyword match)

Semantic ranking re-ranks:
1. "Oceanfront Resort - panoramic ocean views" (actually about views)
2. "Mountain Lodge - stunning mountain vistas" (actually about views)
```

### Semantic Captions and Answers

```json
{
  "search": "what is the wifi policy",
  "queryType": "semantic",
  "semanticConfiguration": "my-config",
  "captions": "extractive",
  "answers": "extractive|count-3"
}
```

**Response**:
```json
{
  "@search.answers": [
    {
      "key": "doc123",
      "text": "Free WiFi is available in all rooms and public areas.",
      "score": 0.95
    }
  ],
  "value": [
    {
      "@search.captions": [
        {
          "text": "...guests can enjoy complimentary WiFi throughout...",
          "highlights": "...guests can enjoy <em>complimentary WiFi</em>..."
        }
      ]
    }
  ]
}
```

## Deep Dive: Vector Search

### What is Vector Search?

Traditional: Search by keywords
Vector: Search by meaning (semantic similarity)

```
Document: "The cat sat on the mat"
         ↓ (Embedding model)
Vector: [0.1, 0.5, -0.2, 0.8, ...]  (1536 dimensions)

Query: "feline resting on rug"
       ↓ (Same embedding model)
Vector: [0.11, 0.48, -0.19, 0.79, ...]

Similarity: Very high (similar meaning!)
```

### Vector Field Configuration

```json
{
  "name": "contentVector",
  "type": "Collection(Edm.Single)",
  "searchable": true,
  "dimensions": 1536,
  "vectorSearchProfile": "my-vector-profile"
}
```

### Vector Search Algorithm

```json
{
  "vectorSearch": {
    "algorithms": [
      {
        "name": "my-hnsw",
        "kind": "hnsw",
        "hnswParameters": {
          "m": 4,
          "efConstruction": 400,
          "efSearch": 500,
          "metric": "cosine"
        }
      }
    ],
    "profiles": [
      {
        "name": "my-vector-profile",
        "algorithm": "my-hnsw"
      }
    ]
  }
}
```

### Hybrid Search (Text + Vector)

Combines keyword matching with semantic similarity:

```json
{
  "search": "comfortable beds",
  "vectorQueries": [
    {
      "kind": "vector",
      "vector": [0.1, 0.2, ...],
      "fields": "descriptionVector",
      "k": 10
    }
  ]
}
```

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Create an Azure AI Search solution](https://learn.microsoft.com/en-us/training/modules/create-azure-cognitive-search-solution/)
- [Create a custom skill for Azure AI Search](https://learn.microsoft.com/en-us/training/modules/create-enrichment-pipeline-azure-cognitive-search/)
- [Create a knowledge store](https://learn.microsoft.com/en-us/training/modules/create-knowledge-store-azure-cognitive-search/)
- [Implement advanced search features](https://learn.microsoft.com/en-us/training/modules/implement-advanced-search-features-azure-cognitive-search/)
- [Build an Azure AI Search solution with vector search](https://learn.microsoft.com/en-us/training/modules/improve-search-results-vector-search/)

### Official Documentation
- [Azure AI Search overview](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
- [Indexer overview](https://learn.microsoft.com/en-us/azure/search/search-indexer-overview)
- [Skillset concepts](https://learn.microsoft.com/en-us/azure/search/cognitive-search-working-with-skillsets)
- [Knowledge store](https://learn.microsoft.com/en-us/azure/search/knowledge-store-concept-intro)
- [Semantic search](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
- [Vector search](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)
- [Full Lucene query syntax](https://learn.microsoft.com/en-us/azure/search/query-lucene-syntax)
- [AI Enrichment Overview](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-intro)
- [Built-in Skills Reference](https://learn.microsoft.com/en-us/azure/search/cognitive-search-predefined-skills)
- [Custom Skills](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-web-api)
- [Knowledge Store](https://learn.microsoft.com/en-us/azure/search/knowledge-store-concept-intro)
- [Query Syntax](https://learn.microsoft.com/en-us/azure/search/query-simple-syntax)
- [OData Filter Syntax](https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter)
- [Semantic Search](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
- [Vector Search](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)
