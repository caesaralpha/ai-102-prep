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
