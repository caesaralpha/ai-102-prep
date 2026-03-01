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
- [AI Enrichment Overview](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-intro)
- [Built-in Skills Reference](https://learn.microsoft.com/en-us/azure/search/cognitive-search-predefined-skills)
- [Custom Skills](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-web-api)
- [Knowledge Store](https://learn.microsoft.com/en-us/azure/search/knowledge-store-concept-intro)
- [Query Syntax](https://learn.microsoft.com/en-us/azure/search/query-simple-syntax)
- [OData Filter Syntax](https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter)
- [Semantic Search](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
- [Vector Search](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)
