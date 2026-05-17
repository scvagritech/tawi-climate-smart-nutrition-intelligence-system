# Architecture

## System Architecture — Tawi Fresh Quality AI

This document describes the technical architecture of the open-source quality assessment module, its relationship to the production Tawi Fresh platform, and the design decisions behind the edge-first approach.

---

## 1. System Context

The quality AI module sits within a broader school feeding supply chain platform. This document covers the open-source components being published in this repository.

```
┌──────────────────────────────────────────────────────────────────┐
│                    TAWI FRESH PLATFORM (Production)               │
│                                                                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐ │
│  │  Ordering   │  │  Sourcing  │  │  Logistics │  │  Payments  │ │
│  │  Service    │  │  Service   │  │  Service   │  │  Service   │ │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └────────────┘ │
│        │               │               │                          │
│  ┌─────▼───────────────▼───────────────▼──────────────────────┐  │
│  │              API Gateway (Kong)                              │  │
│  └─────────────────────┬──────────────────────────────────────┘  │
│                        │                                          │
│  ┌─────────────────────▼──────────────────────────────────────┐  │
│  │         ╔══════════════════════════════════╗                │  │
│  │         ║   QUALITY AI MODULE (this repo)  ║                │  │
│  │         ║                                  ║                │  │
│  │         ║  ┌───────────────────────────┐   ║                │  │
│  │         ║  │  Edge Inference Engine     │   ║                │  │
│  │         ║  │  (TFLite + OpenCV)        │   ║                │  │
│  │         ║  └─────────┬─────────────────┘   ║                │  │
│  │         ║            │                     ║                │  │
│  │         ║  ┌─────────▼─────────────────┐   ║                │  │
│  │         ║  │  Climate Correlation       │   ║                │  │
│  │         ║  │  Engine                    │   ║                │  │
│  │         ║  └─────────┬─────────────────┘   ║                │  │
│  │         ║            │                     ║                │  │
│  │         ║  ┌─────────▼─────────────────┐   ║                │  │
│  │         ║  │  Assessment API            │   ║                │  │
│  │         ║  └───────────────────────────┘   ║                │  │
│  │         ╚══════════════════════════════════╝                │  │
│  │                                                             │  │
│  │         Azure Kubernetes Service (AKS)                      │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌─────────────────────┐  ┌────────────────────────────────────┐ │
│  │  Azure SQL MI        │  │  Azure DevOps (CI/CD)              │ │
│  └─────────────────────┘  └────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

## 2. Module Components

### 2.1 Edge Inference Engine

The inference engine is the core component — it takes a produce image and returns quality classifications.

**Current state (production):** Azure Custom Vision prediction endpoint, cloud-dependent.

**Target state (this repo):** TensorFlow Lite model running locally on the device, with OpenCV preprocessing.

```
Input Image (JPEG/PNG from device camera)
        │
        ▼
┌───────────────────────┐
│  Image Preprocessing  │
│  (OpenCV)             │
│                       │
│  • Resize to model    │
│    input dimensions   │
│  • Colour normalisation│
│  • Background removal │
│    (optional)         │
│  • Augmentation       │
│    (training only)    │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│  Classification Model │
│  (TFLite)             │
│                       │
│  Architecture:        │
│  MobileNetV3 or       │
│  EfficientNet-Lite    │
│                       │
│  Input: 224×224×3     │
│  Output: 5 quality    │
│  tags with confidence │
│  scores               │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│  Post-Processing      │
│                       │
│  • Confidence         │
│    thresholding       │
│  • Multi-label        │
│    aggregation        │
│  • Assessment record  │
│    generation         │
└───────────────────────┘
```

**Quality Tags (5 classes, multi-label):**

| Tag | Description | Production Precision | Target |
|-----|-------------|---------------------|--------|
| `SIZE_SPEC` | Meets procurement size standard | 90.9% | 90%+ |
| `PEST` | Visible pest damage | 89.2% | 90%+ |
| `ROT` | Rot or fungal signatures | 83.3% | 90%+ |
| `DEHYDRATION` | Moisture loss beyond threshold | 56.1% | 90%+ |
| `DISCOLOURATION` | Abnormal colour change | 33.3% | 90%+ |

**Model selection rationale:**

- **MobileNetV3** — optimised for mobile inference latency, strong accuracy-efficiency tradeoff. Preferred for real-time on-device classification.
- **EfficientNet-Lite** — higher accuracy ceiling with moderate compute cost. Preferred when slight latency increase is acceptable for precision gains.

Both architectures support TFLite conversion and quantisation for edge deployment.

### 2.2 Climate-Quality Correlation Engine

This engine enriches each quality assessment with climate context and generates forward-looking spoilage risk predictions.

```
┌──────────────────────────────────────────────────────────────┐
│                  CLIMATE CORRELATION ENGINE                    │
│                                                                │
│  Inputs:                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐│
│  │ Quality Score │  │ Transit Meta │  │ Climate Data          ││
│  │ (from model)  │  │ • Duration   │  │ • Ambient temp        ││
│  │               │  │ • Distance   │  │ • Source-region       ││
│  │               │  │ • Vehicle    │  │   weather (Open       ││
│  │               │  │   type       │  │   Weather API)        ││
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘│
│         │                 │                      │             │
│         └────────────┬────┘──────────────────────┘             │
│                      │                                         │
│              ┌───────▼────────┐                                │
│              │  Rules Engine   │                                │
│              │                 │                                │
│              │  • Temperature  │                                │
│              │    × transit    │                                │
│              │    duration     │                                │
│              │    thresholds   │                                │
│              │                 │                                │
│              │  • Historical   │                                │
│              │    pattern      │                                │
│              │    matching     │                                │
│              │                 │                                │
│              │  • Crop-        │                                │
│              │    specific     │                                │
│              │    decay curves │                                │
│              └───────┬────────┘                                │
│                      │                                         │
│              ┌───────▼────────┐                                │
│              │  Spoilage Risk  │                                │
│              │  Score          │                                │
│              │  (0.0 – 1.0)   │                                │
│              │                 │                                │
│              │  + Risk level   │                                │
│              │  + Recommended  │                                │
│              │    action       │                                │
│              └────────────────┘                                │
│                                                                │
│  Output:                                                       │
│  {                                                             │
│    "spoilage_risk": 0.73,                                     │
│    "risk_level": "HIGH",                                      │
│    "factors": ["ambient_temp_32c", "transit_6h",              │
│                "source_region_drought"],                       │
│    "recommendation": "REJECT_OR_EXPEDITE",                    │
│    "confidence": 0.81                                         │
│  }                                                             │
└──────────────────────────────────────────────────────────────┘
```

**Climate variables captured per assessment:**

| Variable | Source | Update Frequency |
|----------|--------|-----------------|
| Ambient temperature at delivery | Device sensor / manual entry | Per assessment |
| Transit duration | Logistics metadata | Per delivery |
| Transit distance | GPS / route data | Per delivery |
| Source-region temperature | Open Weather API | Hourly |
| Source-region rainfall | Open Weather API | Hourly |
| Source-region humidity | Open Weather API | Hourly |
| Seasonal baseline | Historical dataset | Monthly |

### 2.3 Assessment API

A lightweight REST API that wraps the inference engine and climate correlation engine for integration into existing platforms.

```
POST /api/v1/assess
Content-Type: multipart/form-data

Fields:
  image:            JPEG/PNG file
  crop_type:        string (banana | cabbage | carrot)
  supplier_id:      string
  source_region:    string (GPS coords or region code)
  transit_duration: integer (minutes)
  ambient_temp:     float (°C, optional — device sensor)

Response:
{
  "assessment_id": "uuid",
  "timestamp": "ISO-8601",
  "crop_type": "cabbage",
  "quality_tags": {
    "size_spec":      { "pass": true,  "confidence": 0.91 },
    "pest":           { "pass": true,  "confidence": 0.89 },
    "rot":            { "pass": true,  "confidence": 0.85 },
    "dehydration":    { "pass": true,  "confidence": 0.72 },
    "discolouration": { "pass": false, "confidence": 0.61 }
  },
  "overall_grade": "CONDITIONAL_PASS",
  "spoilage_risk": {
    "score": 0.42,
    "level": "MODERATE",
    "factors": ["transit_5h", "ambient_temp_28c"],
    "recommendation": "USE_WITHIN_24H"
  },
  "climate_context": {
    "ambient_temp_c": 28.0,
    "source_region_temp_c": 22.5,
    "source_region_rainfall_mm": 0.0,
    "transit_duration_min": 300
  },
  "supplier_id": "SUP-0042",
  "source_region": "Nyandarua"
}
```

## 3. Deployment Modes

The module is designed to run in three configurations:

### Mode A: Full Edge (Offline)

For schools and delivery points with no connectivity.

```
┌─────────────────────────────────┐
│  Android Device                  │
│                                  │
│  Camera → OpenCV → TFLite Model │
│                    ↓             │
│            Local Assessment      │
│            (stored on device)    │
│                    ↓             │
│            Sync when connected   │
└─────────────────────────────────┘
```

- Model bundled in APK or downloaded once
- Assessments cached locally, synced on connectivity
- Climate data uses last-known values or manual entry
- No cloud dependency at point of use

### Mode B: API-Connected

For environments with reliable connectivity.

```
┌──────────┐     HTTPS      ┌──────────────────┐
│  Mobile   │ ────────────► │  Assessment API   │
│  Client   │ ◄──────────── │  (Cloud/Server)   │
└──────────┘                └──────────────────┘
```

- Real-time climate data from Open Weather API
- Centralised assessment logging
- Dashboard and reporting available
- Current production mode on Tawi platform

### Mode C: Hybrid

Preferred for most deployments.

- Run inference on-device (Mode A)
- Sync assessments and pull updated models when connected
- Climate correlation runs server-side when available, falls back to cached data

## 4. Data Architecture

### Assessment Record Schema

Every quality assessment generates a structured record linking the image analysis to climate and supply chain context:

```
assessment
├── assessment_id       (UUID)
├── timestamp           (ISO-8601)
├── image_hash          (SHA-256, for deduplication)
├── crop_type           (enum)
├── quality_tags[]
│   ├── tag_name        (enum)
│   ├── pass            (boolean)
│   └── confidence      (float 0–1)
├── overall_grade       (enum: PASS | CONDITIONAL_PASS | FAIL)
├── spoilage_risk
│   ├── score           (float 0–1)
│   ├── level           (enum: LOW | MODERATE | HIGH | CRITICAL)
│   ├── factors[]       (string[])
│   └── recommendation  (enum)
├── climate_context
│   ├── ambient_temp_c
│   ├── source_temp_c
│   ├── source_rainfall_mm
│   ├── source_humidity_pct
│   └── transit_duration_min
├── supplier_id
├── source_region
├── delivery_location
└── operator_id
```

### Training Data Pipeline

```
Raw Images (field collection)
        │
        ▼
┌───────────────────┐
│  Labelling        │  ← Manual annotation (5 quality tags per image)
│  (COCO format)    │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  Augmentation     │  ← Rotation, brightness, crop, noise
│  Pipeline         │    (simulate field capture conditions)
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  Train / Val /    │  ← 70/15/15 split, stratified by crop
│  Test Split       │    type and quality tag distribution
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  Model Training   │  ← Transfer learning from ImageNet
│  (TensorFlow)     │    Fine-tune on produce quality dataset
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  TFLite Export    │  ← Quantisation (int8) for edge
│  + Validation     │    Model size target: <10MB
└───────────────────┘
```

**Dataset targets:**

| Milestone | Image Count | Crop Coverage | Climate Variation |
|-----------|-------------|---------------|-------------------|
| Current | ~500 | 3 crops | Single season |
| Phase 1 (Month 3) | 5,000+ | 3 crops | Multi-season |
| Phase 3 (Month 9) | 10,000+ | 5+ crops | Multi-region, multi-season |

## 5. Integration Points

### For Platform Integrators

The module exposes a standard REST API (Section 2.3). Integration requires:

1. POST images with metadata to the assessment endpoint
2. Receive structured quality + spoilage risk response
3. Store or display results in your platform

No Tawi-specific dependencies. The module runs as a standalone service.

### For Mobile App Developers

The TFLite model can be embedded directly in Android (via TensorFlow Lite Android Support Library) or iOS (via TensorFlow Lite Swift/Obj-C API). The OpenCV preprocessing pipeline ships as a reference implementation in Python, with Android (Java/Kotlin) examples provided.

### For Researchers

The anonymised training dataset (Phase 3) will be published in COCO format with climate metadata annotations, enabling:

- Climate-food quality correlation research
- Transfer learning to other crop types or regions
- Benchmarking against other produce grading approaches

## 6. Security and Privacy

- **No personal data in assessments.** Operator IDs are system-generated, not PII.
- **Images are not published with PII.** The public dataset will be anonymised — no supplier names, operator IDs, or GPS coordinates beyond region-level.
- **Model updates are signed.** OTA model updates will use checksum verification.
- **API authentication** uses standard bearer tokens. Edge mode requires no authentication.

## 7. Open Standards and Interoperability

- **Image format:** JPEG/PNG (standard device camera output)
- **API format:** JSON over HTTPS (OpenAPI 3.0 spec will be published with first code release)
- **Dataset format:** COCO (widely supported by ML tooling)
- **Model format:** TFLite (cross-platform, open-source runtime)
- **Climate data:** OpenWeatherMap API (free tier sufficient for operational use)
- **License:** MIT (maximum reuse permissiveness)

---

*This architecture document will be updated as the implementation progresses. For questions or integration interest, see [CONTRIBUTING.md](../CONTRIBUTING.md).*
