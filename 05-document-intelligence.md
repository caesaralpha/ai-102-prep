# 05 - Document Intelligence Solutions (10–15%)

## 5.1 Azure AI Document Intelligence (formerly Form Recognizer)

### Overview

Azure AI Document Intelligence extracts text, key-value pairs, tables, and structures from documents using machine learning.

### Prebuilt Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **prebuilt-invoice** | Extract invoice fields | Invoices |
| **prebuilt-receipt** | Extract receipt fields | Receipts |
| **prebuilt-idDocument** | Extract ID card/passport fields | Identity verification |
| **prebuilt-businessCard** | Extract business card fields | Contact management |
| **prebuilt-tax.us.w2** | Extract W-2 tax form fields | Tax processing |
| **prebuilt-healthInsuranceCard.us** | Health insurance card fields | Healthcare |
| **prebuilt-layout** | Extract text, tables, selection marks, structure | General document layout |
| **prebuilt-read** | Extract text (OCR only) | Pure text extraction |
| **prebuilt-document** | Extract key-value pairs from generic documents | General documents |
| **prebuilt-contract** | Extract contract clauses | Legal |

### Model Comparison

| Feature | Read | Layout | General Document | Prebuilt | Custom |
|---------|------|--------|-------------------|----------|--------|
| Text extraction | ✅ | ✅ | ✅ | ✅ | ✅ |
| Tables | ❌ | ✅ | ✅ | ✅ | ✅ |
| Selection marks | ❌ | ✅ | ✅ | ✅ | ✅ |
| Key-value pairs | ❌ | ❌ | ✅ | ✅ | ✅ |
| Specific fields | ❌ | ❌ | ❌ | ✅ | ✅ |
| Signatures | ❌ | ✅ | ❌ | ❌ | ✅ |

## 5.2 API Calls

### Analyze Document (Python SDK)

```python
from azure.ai.formrecognizer import DocumentAnalysisClient
from azure.core.credentials import AzureKeyCredential

client = DocumentAnalysisClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<key>")
)

# Using prebuilt-invoice model
with open("invoice.pdf", "rb") as f:
    poller = client.begin_analyze_document("prebuilt-invoice", f)
result = poller.result()

for document in result.documents:
    print(f"Document type: {document.doc_type}")
    for name, field in document.fields.items():
        print(f"  {name}: {field.value} (confidence: {field.confidence})")
        
# Common invoice fields
invoice = result.documents[0]
vendor_name = invoice.fields.get("VendorName")
invoice_total = invoice.fields.get("InvoiceTotal")
invoice_date = invoice.fields.get("InvoiceDate")
```

### REST API

```http
POST https://<endpoint>/formrecognizer/documentModels/prebuilt-invoice:analyze?api-version=2023-07-31
Content-Type: application/pdf
Ocp-Apim-Subscription-Key: <key>

<binary PDF data>
```

The response is async — you get an `Operation-Location` header to poll for results.

### Layout Model Output Structure

```
AnalyzeResult
├── pages[]
│   ├── pageNumber
│   ├── width, height, unit
│   ├── words[] (text, confidence, polygon)
│   ├── lines[] (text, polygon)
│   ├── selectionMarks[] (state: selected/unselected)
│   └── barcodes[]
├── tables[]
│   ├── rowCount, columnCount
│   └── cells[] (rowIndex, columnIndex, text)
├── paragraphs[] (text, role: title/sectionHeading/etc.)
├── styles[] (isHandwritten, spans)
└── documents[]  (only for prebuilt/custom models)
    ├── docType
    └── fields{}
```

## 5.3 Custom Models

### Custom Model Types

| Type | Description | Training |
|------|-------------|----------|
| **Custom Template** | Fixed layout documents (forms) | Need labeled data |
| **Custom Neural** | Variable layout documents | Need labeled data, more flexible |
| **Composed Model** | Combines multiple custom models | Routes to correct sub-model |

### Custom Model Workflow

```
1. Collect sample documents (minimum 5 per type)
2. Upload to Azure Blob Storage
3. Create project in Document Intelligence Studio
4. Label fields in documents
5. Train custom model
6. Evaluate (accuracy, confidence)
7. Deploy and use via API
```

### Training Requirements

| Model Type | Min Documents | Layout | Best For |
|------------|---------------|--------|----------|
| **Template** | 5 per type | Fixed/structured | Tax forms, standardized invoices |
| **Neural** | 5 per type | Variable | Contracts, letters, varied layouts |

### Composed Model

```
Composed Model
├── Custom Model A (Invoice Type 1)
├── Custom Model B (Invoice Type 2)
└── Custom Model C (Receipt)

→ Automatically routes to the best matching sub-model
→ Maximum 200 component models
```

```python
# Compose models
from azure.ai.formrecognizer import DocumentModelAdministrationClient

admin_client = DocumentModelAdministrationClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<key>")
)

poller = admin_client.begin_compose_document_model(
    component_model_ids=["model-invoice-a", "model-invoice-b", "model-receipt"],
    model_id="composed-model"
)
composed_model = poller.result()
```

## 5.4 Document Intelligence Studio

| Feature | Description |
|---------|-------------|
| **Try prebuilt models** | Test with sample or uploaded documents |
| **Label and train** | Label documents for custom models |
| **Test models** | Visualize extraction results |
| **Manage models** | List, delete, compose models |

URL: [https://formrecognizer.appliedai.azure.com/](https://formrecognizer.appliedai.azure.com/)

## 5.5 Key Field Extraction Examples

### Invoice Fields

| Field | Type |
|-------|------|
| `VendorName` | String |
| `VendorAddress` | Address |
| `CustomerName` | String |
| `InvoiceId` | String |
| `InvoiceDate` | Date |
| `InvoiceTotal` | Currency |
| `DueDate` | Date |
| `Items` | Array of line items |
| `SubTotal` | Currency |
| `TotalTax` | Currency |

### Receipt Fields

| Field | Type |
|-------|------|
| `MerchantName` | String |
| `MerchantAddress` | Address |
| `TransactionDate` | Date |
| `Items` | Array |
| `Total` | Number |
| `Subtotal` | Number |
| `Tax` | Number |

### ID Document Fields

| Field | Type |
|-------|------|
| `FirstName` | String |
| `LastName` | String |
| `DateOfBirth` | Date |
| `DocumentNumber` | String |
| `ExpirationDate` | Date |
| `Address` | Address |
| `CountryRegion` | CountryRegion |

## Key Takeaways

1. **Layout model** extracts text, tables, selection marks — no specific fields
2. **Read model** is OCR only — text extraction
3. **Prebuilt models** know specific field names (InvoiceTotal, MerchantName, etc.)
4. **Custom Template** = fixed layout; **Custom Neural** = variable layout
5. **Composed model** routes to the best sub-model automatically (max 200 components)
6. **Minimum 5 documents** per type for custom model training
7. The API is **asynchronous** — submit, then poll for results

## References

- [Document Intelligence Documentation](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/)
- [Document Intelligence Models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-model-overview)
- [Prebuilt Invoice Model](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-invoice)
- [Custom Models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-custom)
- [Composed Models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-composed-models)
- [Document Intelligence Studio](https://formrecognizer.appliedai.azure.com/)
- [Document Intelligence REST API](https://learn.microsoft.com/en-us/rest/api/aiservices/document-models)

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Document Intelligence

### What is Document Intelligence?

Azure AI Document Intelligence (formerly Form Recognizer) extracts information from documents using AI. Unlike simple OCR that just reads text, Document Intelligence understands document STRUCTURE — fields, tables, headers, and relationships.

### The Model Hierarchy

```
Document Intelligence Models
├── Read Model (OCR only)
│   └── Just extracts text — no structure
├── Layout Model
│   └── Text + Tables + Selection marks + Structure
├── General Document Model
│   └── Layout + Key-value pairs (generic)
├── Prebuilt Models
│   └── Layout + Specific fields for document types
│       ├── prebuilt-invoice
│       ├── prebuilt-receipt
│       ├── prebuilt-idDocument
│       └── ... (more)
└── Custom Models
    └── Your trained models for your documents
        ├── Template (fixed layout)
        └── Neural (variable layout)
```

## Model Selection Guide

### When to Use Each Model

| You Need To... | Use This Model | Why |
|---------------|----------------|-----|
| Just read text from any document | `prebuilt-read` | Fastest, cheapest |
| Extract tables from documents | `prebuilt-layout` | Understands table structure |
| Extract data from invoices | `prebuilt-invoice` | Pre-trained on invoice fields |
| Extract data from receipts | `prebuilt-receipt` | Optimized for receipts |
| Extract from your custom forms | Custom Template | Train on your forms |
| Extract from varied documents | Custom Neural | Handles layout variations |

### Prebuilt Models Deep Dive

#### Read Model
**What It Does**: Pure OCR — extracts all text from the document.

**Output**:
```
Page 1:
  Line: "Invoice #12345"
  Line: "Date: March 20, 2026"
  Line: "Total: $500.00"
```

**Use When**: You just need the raw text, no structure.

#### Layout Model
**What It Does**: OCR + understands document structure.

**Output**:
```
Page 1:
  Paragraphs:
    - "Invoice #12345" (role: title)
    - "Date: March 20, 2026" (role: text)
  Tables:
    - Table 1 (3 rows x 4 columns)
      Row 1: ["Item", "Qty", "Price", "Total"]
      Row 2: ["Widget A", "5", "$10.00", "$50.00"]
  Selection Marks:
    - Checkbox at (100, 200): selected
```

**Use When**: You need tables, checkboxes, or want to understand document layout.

#### Prebuilt Invoice
**What It Does**: Extracts invoice-specific fields.

**Fields Extracted**:
| Field | Description |
|-------|-------------|
| `VendorName` | Company sending the invoice |
| `VendorAddress` | Vendor's address |
| `CustomerName` | Customer being billed |
| `InvoiceId` | Invoice number |
| `InvoiceDate` | Date on invoice |
| `DueDate` | Payment due date |
| `SubTotal` | Pre-tax amount |
| `TotalTax` | Tax amount |
| `InvoiceTotal` | Total amount due |
| `Items` | Line items array |

**Line Items Include**: Description, Quantity, UnitPrice, Amount, ProductCode, etc.

#### Prebuilt ID Document
**What It Does**: Extracts fields from ID cards, passports, driver's licenses.

**Fields Extracted**:
| Field | Description |
|-------|-------------|
| `FirstName` | Given name |
| `LastName` | Surname |
| `DateOfBirth` | Birth date |
| `DocumentNumber` | ID/Passport number |
| `ExpirationDate` | When document expires |
| `Address` | Address on document |
| `Sex` | Gender |
| `CountryRegion` | Issuing country |

**Important**: ID extraction has stricter data handling requirements. Review compliance.

## Custom Model Deep Dive

### Template vs Neural Models

| Aspect | Template Model | Neural Model |
|--------|---------------|--------------|
| **Layout** | Fixed positions | Variable positions |
| **Best For** | Standard forms (tax, applications) | Contracts, letters, varied layouts |
| **Training Data** | 5+ documents minimum | 5+ documents minimum |
| **Speed** | Faster | Slower |
| **Accuracy** | Very high for fixed layouts | Good for varied layouts |

### Training Custom Models

#### Step 1: Prepare Documents

**Requirements**:
- Minimum 5 documents (more is better)
- PDF, JPEG, PNG, or TIFF format
- Good quality (not blurry)
- Representative of real-world variety

**Best Practices**:
```
Good Training Set:
├── Form_filled_blue_ink.pdf
├── Form_filled_black_ink.pdf
├── Form_scanned_slightly_tilted.pdf
├── Form_with_handwritten_fields.pdf
└── Form_machine_printed.pdf

Bad Training Set:
├── Form_perfect_scan.pdf    (all identical)
├── Form_perfect_scan2.pdf
├── Form_perfect_scan3.pdf
├── Form_perfect_scan4.pdf
└── Form_perfect_scan5.pdf
```

#### Step 2: Upload to Blob Storage

Documents must be in Azure Blob Storage with appropriate access:
- Container with SAS token, OR
- Managed Identity access

#### Step 3: Label Documents

Using Document Intelligence Studio, label the fields you want to extract:

```
Document: invoice_001.pdf
Labels:
├── InvoiceNumber: "INV-2026-001"
├── Date: "March 20, 2026"
├── Total: "$1,500.00"
└── VendorName: "Contoso Ltd"
```

**Labeling Best Practices**:
- Label ALL instances of each field
- Be consistent (same field name across documents)
- Include variations (handwritten, printed, different positions)

#### Step 4: Train

```python
from azure.ai.formrecognizer import DocumentModelAdministrationClient
from azure.core.credentials import AzureKeyCredential

admin_client = DocumentModelAdministrationClient(
    endpoint="https://...",
    credential=AzureKeyCredential("<key>")
)

# Start training
poller = admin_client.begin_build_document_model(
    build_mode="template",  # or "neural"
    blob_container_url="https://storage.blob.core.windows.net/training-data",
    model_id="my-custom-invoice-model"
)

model = poller.result()
print(f"Model ID: {model.model_id}")
```

### Composed Models Explained

**Problem**: You have different document types (Invoice A, Invoice B, Receipt).

**Solution**: Create individual models, then compose them into one.

```
Composed Model: "all-documents"
├── Model: "invoice-type-a"
├── Model: "invoice-type-b"
└── Model: "receipt-model"

When you analyze a document:
1. Composed model routes to best matching sub-model
2. You get results with the correct fields
```

**Code Example**:
```python
poller = admin_client.begin_compose_document_model(
    component_model_ids=[
        "invoice-type-a",
        "invoice-type-b", 
        "receipt-model"
    ],
    model_id="composed-all-documents"
)
```

**Limits**:
- Maximum 200 component models per composed model

## API Patterns

### Async Processing Pattern

Document Intelligence uses async processing for reliability:

```
1. Submit Document
   POST /documentModels/{model}:analyze
   ↓
   Response: 202 Accepted
   Header: Operation-Location: https://...operations/{id}

2. Poll for Status
   GET /operations/{id}
   ↓
   { "status": "running" }
   { "status": "running" }
   { "status": "succeeded", "analyzeResult": {...} }
```

**Python SDK (handles polling automatically)**:
```python
with open("document.pdf", "rb") as f:
    poller = client.begin_analyze_document("prebuilt-invoice", f)

# This blocks until done (SDK handles polling)
result = poller.result()
```

### Handling Multiple Pages

```python
result = poller.result()

for page in result.pages:
    print(f"Page {page.page_number}")
    print(f"  Width: {page.width}, Height: {page.height}")
    print(f"  Lines: {len(page.lines)}")
    
    for line in page.lines:
        print(f"    {line.content}")
```

### Accessing Fields by Confidence

```python
result = poller.result()
document = result.documents[0]

for field_name, field in document.fields.items():
    if field.confidence >= 0.9:
        print(f"✓ {field_name}: {field.value}")
    elif field.confidence >= 0.7:
        print(f"? {field_name}: {field.value} (review recommended)")
    else:
        print(f"✗ {field_name}: {field.value} (low confidence)")
```

## Document Intelligence Studio

### What You Can Do

| Feature | Description |
|---------|-------------|
| **Try Prebuilt** | Test prebuilt models with sample or your documents |
| **Train Custom** | Label documents and train models |
| **Compose** | Combine multiple models |
| **Test** | Visualize results with bounding boxes |
| **Deploy** | Manage model deployments |

**URL**: https://documentintelligence.ai.azure.com/

### Studio Workflow

```
1. Create Resource (if not exists)
   ↓
2. Create Project
   ↓
3. Connect Storage Account (for training data)
   ↓
4. Upload Documents
   ↓
5. Label Fields (draw boxes, assign field names)
   ↓
6. Train Model
   ↓
7. Test Model
   ↓
8. Use via API (model ID)
```

## Real-World Scenarios

### Invoice Processing Pipeline

```
Incoming Invoice (email/upload)
        ↓
Azure Blob Storage (temporary)
        ↓
Azure Function (triggered)
        ↓
Document Intelligence (prebuilt-invoice)
        ↓
Extract: VendorName, InvoiceId, Total, LineItems
        ↓
Validate against PO Database
        ↓
If matches: Auto-approve
If discrepancy: Human review queue
        ↓
Update ERP System
```

### Multi-Document Type Processing

```
Unknown Document
        ↓
Composed Model (Invoice + Receipt + Contract)
        ↓
Routing: Identifies as "Invoice Type B"
        ↓
Extract fields using Invoice Type B model
        ↓
Process based on document type
```

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Get started with Document Intelligence](https://learn.microsoft.com/en-us/training/modules/get-started-document-intelligence/)
- [Extract data from forms with Document Intelligence](https://learn.microsoft.com/en-us/training/modules/work-form-recognizer/)
- [Create a composed Document Intelligence model](https://learn.microsoft.com/en-us/training/modules/create-composed-form-recognizer-model/)

### Official Documentation
- [Document Intelligence overview](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview)
- [Model overview](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-model-overview)
- [Prebuilt models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/prebuilt-overview)
- [Custom models](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept-custom)
- [REST API reference](https://learn.microsoft.com/en-us/rest/api/aiservices/document-models)
- [Python SDK reference](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-documentintelligence-readme)
