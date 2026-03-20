# 03 - Computer Vision Solutions (15–20%)

## 3.1 Azure AI Vision (formerly Computer Vision)

### Image Analysis 4.0 Features

| Feature | Description | API Version |
|---------|-------------|-------------|
| **Captions** | Generate human-readable descriptions | 4.0 |
| **Dense Captions** | Captions for multiple detected regions | 4.0 |
| **Tags** | Content tags for the image | 4.0 |
| **Objects** | Detect objects with bounding boxes | 4.0 |
| **People** | Detect people with bounding boxes | 4.0 |
| **Smart Crops** | Aspect ratio–based crop suggestions | 4.0 |
| **Read (OCR)** | Extract printed/handwritten text | 4.0 |

### Image Analysis API Call

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential

client = ImageAnalysisClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<key>")
)

result = client.analyze(
    image_url="https://example.com/image.jpg",
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.READ
    ],
    language="en"
)

# Caption
print(f"Caption: {result.caption.text} (confidence: {result.caption.confidence})")

# Tags
for tag in result.tags.list:
    print(f"Tag: {tag.name} (confidence: {tag.confidence})")

# Objects detected
for obj in result.objects.list:
    print(f"Object: {obj.tags[0].name} at [{obj.bounding_box}]")

# OCR
for block in result.read.blocks:
    for line in block.lines:
        print(f"Text: {line.text}")
```

### REST API

```http
POST https://<endpoint>/computervision/imageanalysis:analyze?api-version=2024-02-01&features=caption,tags,objects,read
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
  "url": "https://example.com/image.jpg"
}
```

## 3.2 Custom Vision

### Training Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Classification** | Assign labels to entire images | "Is this a cat or dog?" |
| **Object Detection** | Detect and locate objects with bounding boxes | "Where are the products on shelf?" |

### Classification Types

| Type | Description |
|------|-------------|
| **Multiclass** | One label per image (mutually exclusive) |
| **Multilabel** | Multiple labels per image |

### Training Domain Types

| Domain | Description |
|--------|-------------|
| **General** | Broad range of images |
| **Food** | Optimized for food photographs |
| **Landmarks** | Recognizable landmarks |
| **Retail** | Retail shelf images |
| **General (compact)** | Exportable models for edge deployment |

### Custom Vision Workflow

```
1. Create Project (Classification or Object Detection)
2. Upload and Tag Images (minimum 5 images per tag, recommended 50+)
3. Train Model (Quick Training or Advanced Training)
4. Evaluate (Precision, Recall, AP)
5. Publish to Prediction Endpoint
6. Call Prediction API
```

### Prediction API

```python
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from msrest.authentication import ApiKeyCredentials

credentials = ApiKeyCredentials(in_headers={"Prediction-key": "<prediction-key>"})
predictor = CustomVisionPredictionClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credentials=credentials
)

# Classify image by URL
results = predictor.classify_image_url(
    project_id="<project-id>",
    published_name="<iteration-name>",
    url="https://example.com/test.jpg"
)

for prediction in results.predictions:
    print(f"{prediction.tag_name}: {prediction.probability:.2%}")
```

### Evaluation Metrics

| Metric | Formula | Description |
|--------|---------|-------------|
| **Precision** | TP / (TP + FP) | Of predicted positives, how many are correct |
| **Recall** | TP / (TP + FN) | Of actual positives, how many were found |
| **AP (Average Precision)** | Area under Precision-Recall curve | Overall model performance per class |
| **mAP** | Mean AP across all classes | Overall model performance |

## 3.3 Face API

### Face Detection Features

| Feature | Description |
|---------|-------------|
| **Face Detection** | Detect faces in an image with bounding boxes |
| **Face Attributes** | Age, gender, emotion, glasses, hair, head pose, etc. |
| **Face Landmarks** | 27-point facial landmarks |
| **Face Verification** | 1:1 — Are these two faces the same person? |
| **Face Identification** | 1:N — Who is this person from a group? |
| **Find Similar** | Find faces similar to a target face |
| **Group** | Group faces by visual similarity |

### Face API Responsible AI

> **Critical**: Face API has **Limited Access** requirements:
> - Face **identification** and **verification** require an application and approval
> - Emotion recognition from facial expressions is **retired**
> - Microsoft's Responsible AI Standard applies

### PersonGroup Workflow

```
1. Create a PersonGroup
2. Add Persons to the group
3. Add Faces to each Person (multiple angles/conditions)
4. Train the PersonGroup
5. Identify faces against the PersonGroup
```

```python
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials

face_client = FaceClient(
    endpoint="https://<resource>.cognitiveservices.azure.com/",
    credentials=CognitiveServicesCredentials("<key>")
)

# Detect faces
detected_faces = face_client.face.detect_with_url(
    url="https://example.com/faces.jpg",
    detection_model="detection_03",
    recognition_model="recognition_04",
    return_face_attributes=["qualityForRecognition"]
)
```

### Face Models

| Model | Version | Best For |
|-------|---------|----------|
| **Detection** | `detection_01`, `detection_03` | `detection_03` is latest and most accurate |
| **Recognition** | `recognition_01` to `recognition_04` | `recognition_04` is latest and most accurate |

## 3.4 OCR (Optical Character Recognition)

### Read API vs Image Analysis OCR

| Feature | Read API (Document Intelligence) | Image Analysis 4.0 OCR |
|---------|--------------------------------|------------------------|
| **Optimized For** | Documents (PDF, TIFF) | Natural images |
| **Async** | Yes (for large documents) | Synchronous |
| **Multi-page** | Yes | No |
| **Handwritten** | Yes | Yes |
| **Structured** | More structured output | Simpler output |

### Read API Hierarchy

```
Read Result
├── Page 1
│   ├── Line 1
│   │   ├── Word 1 (text, confidence, bounding polygon)
│   │   └── Word 2
│   └── Line 2
└── Page 2
    └── ...
```

## 3.5 Spatial Analysis

- Analyzes **video feeds** in real-time
- Requires **Azure Stack Edge** device or compatible GPU
- Use cases:
  - People counting
  - Social distancing monitoring
  - Queue management
  - Zone dwell time

### Operations

| Operation | Description |
|-----------|-------------|
| `cognitiveservices.vision.spatialanalysis-personcount` | Count people in a zone |
| `cognitiveservices.vision.spatialanalysis-personcrossingline` | Detect when people cross a line |
| `cognitiveservices.vision.spatialanalysis-personcrossingpolygon` | Detect when people enter/exit a zone |
| `cognitiveservices.vision.spatialanalysis-persondistance` | Detect social distancing violations |

## 3.6 Video Indexer (Azure AI Video Indexer)

| Feature | Description |
|---------|-------------|
| **Face Identification** | Identify people in video |
| **OCR** | Extract text from video frames |
| **Speech Transcription** | Transcribe spoken words |
| **Topic Inference** | Identify topics discussed |
| **Emotion Detection** | Detect emotions from audio tone |
| **Scene Segmentation** | Split video into scenes |
| **Keyframe Extraction** | Extract key frames |
| **Sentiment Analysis** | Analyze sentiment of speech |

## Key Takeaways

1. **Image Analysis 4.0** is the current version — know the visual features enum
2. **Custom Vision**: minimum 5 images per tag; multiclass vs multilabel
3. **Face API** has **Limited Access** — identification requires approval
4. **Detection model 03** + **Recognition model 04** are the latest Face models
5. **Spatial Analysis** requires edge hardware (GPU device)
6. **Read API** is for documents; **Image Analysis OCR** is for natural images

## References

- [Azure AI Vision Documentation](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/)
- [Image Analysis 4.0](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-analyzing-images-40)
- [Custom Vision Documentation](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/)
- [Face API Documentation](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-identity)
- [Face API Limited Access](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/identity-limited-access)
- [Spatial Analysis](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/intro-to-spatial-analysis-public-preview)
- [Azure Video Indexer](https://learn.microsoft.com/en-us/azure/azure-video-indexer/)

---

# Detailed Explanations

This section provides in-depth explanations for complex topics to support your understanding.

## Understanding Image Analysis 4.0

### What is Image Analysis 4.0?

Image Analysis 4.0 is the latest version of Azure's computer vision capabilities. It uses modern deep learning models (including Florence foundation model) to analyze images and extract information.

### Visual Features Explained

| Feature | What It Does | When to Use |
|---------|--------------|-------------|
| **Caption** | Generates a single sentence describing the main content | "A dog playing in a park" |
| **Dense Captions** | Multiple captions for different regions of the image | Complex scenes with multiple elements |
| **Tags** | List of relevant keywords | Search/indexing, categorization |
| **Objects** | Detects and locates objects with bounding boxes | Counting items, spatial analysis |
| **People** | Detects people with bounding boxes | Crowd analysis, photo organization |
| **Smart Crops** | Suggests crop regions for different aspect ratios | Thumbnail generation |
| **Read (OCR)** | Extracts text from the image | Signs, labels, documents in photos |

### Understanding Confidence Scores

Every result comes with a confidence score (0.0 to 1.0):

```python
result.caption.text = "A dog playing in a park"
result.caption.confidence = 0.85  # 85% confident
```

**Typical Thresholds**:
- **>0.9**: Very high confidence - likely accurate
- **0.7-0.9**: Good confidence - usually accurate
- **0.5-0.7**: Moderate - may need verification
- **<0.5**: Low - consider alternative interpretations

### Handling Multiple Features

You can request multiple features in one API call (more efficient):

```python
result = client.analyze(
    image_url="https://...",
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE,
        VisualFeatures.READ
    ]
)
```

**Cost Consideration**: Each feature may add to the cost. Only request what you need.

## Deep Dive: Custom Vision

### When to Use Custom Vision vs Image Analysis 4.0

| Scenario | Use | Why |
|----------|-----|-----|
| Detect common objects | Image Analysis 4.0 | Pre-trained on millions of images |
| Classify your specific products | Custom Vision | You need to train on YOUR data |
| Find where logos appear | Custom Vision (Object Detection) | Your specific brand logos |
| Identify plant diseases | Custom Vision | Domain-specific expertise |
| Get general image description | Image Analysis 4.0 | No training needed |

### Understanding Training Types

#### Classification

Answers: "What IS this image?"

```
Image of Dog → [Dog: 0.95, Cat: 0.03, Bird: 0.02]
```

**Multiclass** (Single Label):
- Image can only be ONE category
- Example: "Is this a cat, dog, or bird?"
- Categories are mutually exclusive

**Multilabel** (Multiple Labels):
- Image can have MULTIPLE categories
- Example: "What's in this photo?" → [sunny, beach, people, umbrella]
- Categories can all be true simultaneously

#### Object Detection

Answers: "What is in this image AND where is it?"

```
Image of Street → [
  Car at (100,200,300,250),
  Person at (400,100,450,300),
  Stop Sign at (50,50,100,100)
]
```

Returns bounding boxes (coordinates) for each detected object.

### Training Domain Selection

| Domain | Optimize For | Example Use Cases |
|--------|--------------|-------------------|
| General | Broad image types | Most applications |
| Food | Food photographs | Restaurant menus, nutrition apps |
| Landmarks | Famous places | Travel apps |
| Retail | Product shelves | Inventory management |
| **General (Compact)** | Edge deployment | Mobile apps, IoT devices |

**Compact Domains**: Can export the model to run on:
- TensorFlow.js (web browsers)
- CoreML (iOS)
- ONNX (various platforms)
- Docker containers

### Training Data Best Practices

**Minimum Requirements**:
- At least 5 images per tag (bare minimum)
- At least 50 images per tag (recommended)
- Balanced classes (similar counts per category)

**Quality Guidelines**:

| Good Practice | Bad Practice |
|--------------|--------------|
| Various lighting conditions | Same lighting everywhere |
| Different angles | Only straight-on shots |
| Real-world backgrounds | Always white backgrounds |
| Multiple examples of variations | Only one product variant |

**Example**: Training a "Ripe Apple" detector

```
Good Training Set:
- Red apples, green apples, yellow apples
- On trees, in bowls, on tables
- Different sizes
- Some with stems, some without
- Various lighting (sunlight, indoor)

Bad Training Set:
- Only red Fuji apples
- All photographed on white background
- All same size and angle
```

### Evaluation Metrics Explained

After training, Custom Vision shows evaluation metrics:

#### Precision

**Question**: Of all the images the model said were "cats", how many actually were cats?

```
Formula: True Positives / (True Positives + False Positives)

Example:
- Model detected 100 images as "cat"
- 90 were actually cats (True Positives)
- 10 were not cats (False Positives)
- Precision = 90/100 = 90%
```

**Interpretation**: High precision = fewer false alarms

#### Recall

**Question**: Of all the actual cat images, how many did the model correctly identify?

```
Formula: True Positives / (True Positives + False Negatives)

Example:
- There are 100 actual cat images
- Model found 80 of them (True Positives)
- Model missed 20 (False Negatives)
- Recall = 80/100 = 80%
```

**Interpretation**: High recall = finds most of what you're looking for

#### Probability Threshold

You can adjust the threshold to trade off precision vs recall:

| Threshold | Effect |
|-----------|--------|
| Higher (e.g., 0.9) | More precise but may miss some |
| Lower (e.g., 0.5) | Catches more but more false positives |

## Deep Dive: Face API

### Understanding Limited Access

Microsoft restricts certain Face API capabilities due to responsible AI concerns. This is called **Limited Access**.

#### Features Requiring Approval:
- **Face Identification**: Who is this person? (matching against enrolled faces)
- **Face Verification**: Is this the same person as the reference photo?
- **Liveness Detection**: Is this a real person or a photo/video?

#### Features Available Without Approval:
- **Face Detection**: Where are faces in this image?
- **Face Attributes**: General attributes (blur, occlusion, head pose)
- **Quality for Recognition**: Is this image good enough for face matching?

#### How to Apply:
1. Go to [Microsoft Face Recognition intake form](https://aka.ms/facerecognition)
2. Provide business justification
3. Describe your use case
4. Microsoft reviews and responds

### Face Models Explained

#### Detection Models

| Model | Characteristics |
|-------|-----------------|
| `detection_01` | Original model, good general performance |
| `detection_02` | Better for small/rotated/occluded faces |
| `detection_03` | Latest, best overall accuracy |

**Recommendation**: Use `detection_03` for new projects.

#### Recognition Models

| Model | Characteristics |
|-------|-----------------|
| `recognition_01` | Original model |
| `recognition_02` | Improved accuracy |
| `recognition_03` | Better for difficult conditions |
| `recognition_04` | Latest, best accuracy |

**Important**: Faces enrolled with one model CANNOT be matched using a different model. If you upgrade, you must re-enroll all faces.

### PersonGroup Workflow Explained

For face identification (1:N matching), you need to set up a PersonGroup:

```
Step 1: Create PersonGroup
        └── "my-company-employees"

Step 2: Add Persons
        ├── Person: "John Smith" (person-id-1)
        ├── Person: "Jane Doe" (person-id-2)
        └── Person: "Bob Wilson" (person-id-3)

Step 3: Add Faces to Each Person
        ├── John: face1.jpg, face2.jpg, face3.jpg
        ├── Jane: face1.jpg, face2.jpg
        └── Bob: face1.jpg, face2.jpg, face3.jpg

Step 4: Train the PersonGroup
        └── System learns to recognize each person

Step 5: Identify Unknown Faces
        └── Input: new photo → Output: "90% confident this is John Smith"
```

**Best Practices for Face Enrollment**:
- Multiple photos per person (3-10 recommended)
- Different angles (front, slight left, slight right)
- Different expressions (neutral, smiling)
- Good lighting
- High-quality images

## OCR: Read API vs Image Analysis

### When to Use Which

| Scenario | Use | Why |
|----------|-----|-----|
| Multi-page PDF documents | Read API (Document Intelligence) | Handles multi-page, complex layouts |
| TIFF images | Read API | Better for document formats |
| Photo of a sign | Image Analysis 4.0 OCR | Natural images with text |
| Handwritten notes | Both work | Read API slightly better for handwriting |
| Real-time single image | Image Analysis 4.0 | Synchronous, faster for single images |
| Batch processing | Read API | Better for high-volume document processing |

### Read API Async Pattern

For large documents, Read API uses async processing:

```python
# Step 1: Submit the document
poller = client.begin_read(document)

# Step 2: Wait for completion (or poll)
result = poller.result()  # Blocks until done

# OR poll manually
operation_id = poller.id
while not poller.done():
    time.sleep(1)
result = poller.result()
```

## Spatial Analysis Deep Dive

### What is Spatial Analysis?

Spatial Analysis processes **video streams** to understand physical spaces. Unlike Image Analysis (single images), it tracks **movement over time**.

### Use Cases

| Operation | Use Case | Output |
|-----------|----------|--------|
| **Person Count** | How many people in zone? | Real-time count |
| **Line Crossing** | People entering/exiting | Entry/exit events |
| **Zone Dwell** | How long do people stay? | Duration per person |
| **Social Distance** | Are people too close? | Distance violations |

### Hardware Requirements

Spatial Analysis requires:
- **Azure Stack Edge** device OR
- **NVIDIA GPU** with Docker support

This is NOT a cloud API — it runs on edge hardware sending analytics to Azure.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ Your Premises                                                │
│  ┌──────────────┐    ┌──────────────────────────────────┐   │
│  │ IP Cameras   │───►│ Azure Stack Edge / GPU Server    │   │
│  │ (RTSP feeds) │    │ ┌─────────────────────────────┐  │   │
│  └──────────────┘    │ │ Spatial Analysis Container  │  │   │
│                      │ │ - Video processing          │  │   │
│                      │ │ - AI inference              │  │   │
│                      │ │ - Event generation          │  │   │
│                      │ └─────────────────────────────┘  │   │
│                      └───────────────┬──────────────────┘   │
└──────────────────────────────────────┼──────────────────────┘
                                       │ Events/Telemetry
                                       ▼
                           ┌────────────────────┐
                           │ Azure IoT Hub      │
                           │ ↓                  │
                           │ Power BI / Apps    │
                           └────────────────────┘
```

## Additional Learning Resources

### Microsoft Learn Modules (Free)
- [Analyze images with Azure AI Vision](https://learn.microsoft.com/en-us/training/modules/analyze-images/)
- [Classify images with Custom Vision](https://learn.microsoft.com/en-us/training/modules/classify-images/)
- [Detect objects in images with Custom Vision](https://learn.microsoft.com/en-us/training/modules/detect-objects-images/)
- [Detect and analyze faces](https://learn.microsoft.com/en-us/training/modules/detect-analyze-faces/)
- [Read text in images and documents](https://learn.microsoft.com/en-us/training/modules/read-text-images-documents/)

### Official Documentation
- [Image Analysis overview](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-image-analysis)
- [Custom Vision documentation](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/overview)
- [Face service documentation](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-identity)
- [Spatial Analysis documentation](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/intro-to-spatial-analysis-public-preview)
- [Video Indexer documentation](https://learn.microsoft.com/en-us/azure/azure-video-indexer/)
