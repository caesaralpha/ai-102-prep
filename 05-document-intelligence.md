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
