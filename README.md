# Tawi Fresh Climate-Smart Nutrition Intelligence System

**Open-source, edge-deployable produce loss measurement for school feeding supply chains in climate-vulnerable regions — captures nutrient loss between farm and delivery, and weighs it against climate exposure, rather than estimating nutrient content from a single image.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status: Pre-release](https://img.shields.io/badge/Status-Pre--release-orange.svg)](#release-timeline)
[![UNICEF Venture Fund](https://img.shields.io/badge/UNICEF-Venture%20Fund%20Applicant-00AEEF.svg)](#acknowledgements)

---

## The Problem

Kenya's school feeding programmes serve millions of children daily, but the fresh produce supply chains behind them operate without any real-time intelligence on food quality, safety, or nutritional adequacy. Climate stress — rising temperatures, extended transit without cold chain, and erratic rainfall — has made quality degradation between farm and plate a near-daily operational reality, and nobody measures how much nutritional value is lost along the way.

No scalable, data-driven system currently links climate exposure, produce quality, and nutrient loss to the point a child actually eats.

## Our Position

Nutrient content cannot be read reliably from a single photograph of produce. Pigments such as chlorophyll and carotenoids are only partially predictable from colour, and ascorbic acid — the nutrient that matters most in leafy and fresh produce — has no direct optical signal at all. Any tool claiming to measure vitamin C from one image is overstating what the physics allows.

So this project does not measure content. It measures **loss**. By identifying the same crate at multiple points — from collection at the farm, through dispatch, to school delivery — and logging thermal exposure between each point, the system turns nutrient loss into a measurable, testable quantity instead of a single-image guess. Cultivar, soil, harvest maturity and lighting largely cancel out in the difference, which is a far better-posed problem than absolute estimation ever was.

## What This Project Will Contain

**Tawi Nutrient Loss Ledger** is the name of this specific solution: the open-source inference and assessment module extracted from Tawi Fresh's production platform, funding Layers 1 and 2 of the Climate-Smart Nutrition Intelligence System. It provides:

| Component | Description | Status |
|-----------|-------------|--------|
| **Quality Grading Model** | TensorFlow/TFLite computer vision model for produce freshness classification across five quality indicators (size specification, pest infestation, rot, dehydration, discolouration) | 🔨 In development |
| **Edge Inference Engine** | Offline-capable inference pipeline using OpenCV and TensorFlow Lite, designed for low-connectivity school and farm-gate environments | 🔨 In development |
| **Crate Identity & Multi-Point Capture** | Printed crate-ID tagging and standardised capture (with colour reference) scanned at collection from the farm, at dispatch, and at delivery, so every observation is of the same produce | 🔨 In development |
| **Climate-Quality Correlation Engine** | Links quality and senescence scores to climate exposure variables (in-crate thermal logging, transit duration, source-region weather) to estimate nutrient retention as a confidence band, validated against laboratory ascorbic acid assay | 🔨 In development |
| **Training Data Toolkit** | Labelling utilities, augmentation pipelines, and data preparation scripts for building produce quality datasets | 🔨 In development |
| **Anonymised Training Dataset** | Labelled produce images and paired crate/exposure/assay records across seasonal and climate variations, published as an open public good | 📋 Planned (Month 7–9) |

### Supported Crops (Initial Release)

- Bananas
- Cabbages
- Carrots

Additional crop varieties will follow via transfer learning (Month 7–9 onwards).

### Quality Indicators

Each assessment evaluates five tags per delivery:

1. **Size Specification** — meets procurement standard (current precision: 90.9%, n=11)
2. **Pest Infestation** — visible pest damage detected (current precision: 89.2%, n=37)
3. **Rotten Produce** — rot or fungal signatures present (current precision: 83.3%, n=6)
4. **Dehydrated Produce** — moisture loss beyond acceptable threshold (current precision: 56.1%, n=41)
5. **Discolouration** — abnormal colour indicating quality degradation (current precision: 33.3%, n=3)

> **Note:** Precision figures are early, directional results from the current production model (Azure Custom Vision) on a small validation set (~500 labelled images total). The open-source rebuild targets 90%+ recall on safety-critical tags (rot, pest infestation) by Month 10–12, alongside a validated nutrient-retention model tested against laboratory assay.

## Architecture Overview

The system tracks a crate across the chain rather than inferring content at one point:

```
┌─────────────────────────────────────────────────────┐
│           Mobile Capture Client (.NET MAUI)          │
│   Farm collection → dispatch → school delivery       │
│   Crate-ID scan + standardised photo + colour ref     │
└──────────────────────┬──────────────────────────────┘
                        │
           ┌────────────▼────────────┐
           │   Edge Inference Engine  │  ◄── Offline-capable
           │   (TFLite + OpenCV)      │      TFLite model
           └────────────┬────────────┘
                        │
           ┌────────────▼────────────┐
           │  Climate & Exposure      │  ◄── In-crate thermal logger
           │  Correlation Engine      │      + open weather APIs
           └────────────┬────────────┘
                        │
           ┌────────────▼────────────┐
           │   Assessment Output      │
           │   (Quality score +       │
           │    nutrient retention    │
           │    band + climate        │
           │    context)              │
           └─────────────────────────┘
```

This module funds Layers 1 and 2 of the four-layer Climate-Smart Nutrition Intelligence System we were selected on: nutrition-safety computer vision and climate-predictive degradation modelling. Seasonal availability intelligence (Layer 3) and AI meal recommendation (Layer 4) build on the validated retention measure this module produces.

For the full technical architecture, see **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)**.

## Current Status

This repository is a **pre-release placeholder**. The open-source module is being extracted and rebuilt from Tawi Fresh's production system, which is live and operational today:

**What is running in production now:**
- Computer vision quality assessment processing real school feeding deliveries
- Django 4.x / Django REST Framework API (Python 3.11) on Azure Kubernetes Service (AKS), with Nginx and Gunicorn
- Azure Custom Vision prediction endpoint trained on 3 crop types, 5 quality tags
- PostgreSQL as the system of record; Celery and Redis for asynchronous jobs (image post-processing, climate enrichment)
- Climate exposure logging (ambient temperature, transit duration, source-region weather) against every assessment
- CI/CD via GitHub Actions, building to Azure Container Registry
- Active school feeding programme client generating real transaction data

**What this open-source release will deliver:**
- The inference layer rebuilt on TensorFlow/TFLite and OpenCV (replacing Azure Custom Vision), running fully offline on sub-USD-150 Android hardware
- Crate identity and multi-point capture (farm collection, dispatch, delivery), so loss is measured between controlled observations of the same produce, not inferred from a single image
- A climate-quality correlation module producing a validated, confidence-banded nutrient retention estimate, tested against laboratory ascorbic acid assay
- Training data toolkit and an anonymised, paired-observation dataset
- Full documentation for replication by agritech platforms and NGOs

## Release Timeline

| Phase | Timeline | Deliverables |
|-------|----------|-------------|
| **Phase 1** | Month 1–3 | Repository scaffolding, crate-identity tagging, training data pipeline, initial model architecture. Expand dataset to 5,000+ labelled images. MIT-licensed quality grading module published. |
| **Phase 2** | Month 4–6 | Edge-deployable TFLite model, migrated off Azure Custom Vision. Climate-quality data dashboard (public), showing observed quality and measured exposure. Offline inference pipeline. |
| **Phase 3** | Month 7–9 | First laboratory-assay-validated nutrient retention results published. Anonymised training image and paired-observation dataset published as open data. Cross-crop transfer learning for additional produce varieties. |
| **Phase 4** | Month 10–12 | 90%+ recall on safety-critical tags across all crop types. Retention overlays added to the public dashboard. Replication package for agritech platforms and NGOs across East Africa. Impact report and methodology publication. |

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Mobile Capture Client | .NET MAUI (C#) | Cross-platform capture from a single codebase, targeting Android first with iOS extensibility |
| API / Application | Django 4.x, Django REST Framework (Python 3.11) | Mature ecosystem for the assessment, batch, supplier and climate endpoints |
| Model Training | TensorFlow / Keras | Mature ecosystem, strong mobile/edge deployment path |
| Edge Inference | TensorFlow Lite (MobileNetV3-Small / EfficientNet-Lite0) | Optimised for mobile and low-resource devices, sub-second inference offline |
| Image Processing | OpenCV (via OpenCvSharp) | Industry-standard, no cloud dependency |
| Climate Data | Open weather APIs, by source region | Free, reliable, covers East African regions |
| Data / Infra | PostgreSQL, Celery + Redis, Azure Kubernetes Service | Async processing and reproducible infrastructure as code |
| Training Pipeline | Python, CVAT, Albumentations, MLflow, DVC | Standard ML data preparation, tracking and versioning stack |

## Project Context

This module is part of the broader **Tawi Fresh** platform — Tawi Fresh Kenya Limited is itself a produce aggregator, buying from registered farmers and selling to institutional buyers including school feeding programmes. The parent platform handles:

- Produce aggregation and distribution to institutional buyers
- Ordering, sourcing, logistics, and payment processing
- Supplier management and farmer demand signalling

The quality and nutrient-loss assessment module published here is designed to function both as an integrated part of Tawi Fresh's own aggregation business and as a standalone tool other aggregators, buyers and schools can adopt and self-host.

## Contributing

This project is in pre-release. Contribution guidelines will be published alongside the first code release (Phase 1).

In the meantime, if you are:
- A **school feeding programme operator** interested in piloting the system
- An **agritech platform or NGO** interested in integrating quality and nutrient-loss assessment
- A **researcher** working on climate-food quality linkages
- A **developer** with experience in edge ML deployment

We'd like to hear from you. See [CONTRIBUTING.md](CONTRIBUTING.md) for contact details.

## License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for the full text.

We chose MIT to maximise adoption by agritech platforms, NGOs, and government programmes across East Africa and beyond.

## Acknowledgements

This project is supported by a grant application to the [UNICEF Venture Fund](https://www.unicef.org/innovation/venturefund), which invests in open-source technology solutions that benefit children worldwide.

Tawi Fresh's production platform and engineering team are funded by **Tawi Fresh Kenya Limited**, based in Nairobi, Kenya.

School feeding programme partnership through **Food4Education (F4E)**.

---

**Tawi Fresh Kenya Limited** · Nairobi, Kenya · [tawifresh.com](https://tawifresh.com)
