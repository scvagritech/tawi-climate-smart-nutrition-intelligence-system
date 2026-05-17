# Tawi Fresh Quality AI

**Open-source, edge-deployable produce quality assessment for school feeding supply chains in climate-vulnerable regions.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status: Pre-release](https://img.shields.io/badge/Status-Pre--release-orange.svg)](#release-timeline)
[![UNICEF Venture Fund](https://img.shields.io/badge/UNICEF-Venture%20Fund%20Applicant-00AEEF.svg)](#acknowledgements)

---

## The Problem

Kenya's school feeding programmes serve millions of children daily, but the fresh produce supply chains behind them operate without any real-time intelligence on food quality, safety, or nutritional adequacy. Climate stress — rising temperatures, extended transit without cold chain, and erratic rainfall — has made quality degradation at delivery a near-daily operational reality.

No scalable, data-driven system currently bridges climate exposure, produce quality, and child nutrition outcomes in school feeding supply chains.

## What This Project Will Contain

Tawi Fresh Quality AI is the open-source inference and assessment module extracted from Tawi Fresh's production platform. It provides:

| Component | Description | Status |
|-----------|-------------|--------|
| **Quality Grading Model** | TensorFlow/TFLite computer vision model for produce freshness classification across five quality indicators (size specification, pest infestation, rot, dehydration, discolouration) | 🔨 In development |
| **Edge Inference Engine** | Offline-capable inference pipeline using OpenCV and TensorFlow Lite, designed for low-connectivity school environments | 🔨 In development |
| **Climate-Quality Correlation Engine** | Rules-based engine linking quality scores to climate exposure variables (ambient temperature, transit duration, source-region weather) to generate predictive spoilage risk scores | 🔨 In development |
| **Training Data Toolkit** | Labelling utilities, augmentation pipelines, and data preparation scripts for building produce quality datasets | 🔨 In development |
| **Anonymised Training Dataset** | Labelled produce images across seasonal and climate variations, published as an open public good | 📋 Planned (Month 7–9) |

### Supported Crops (Initial Release)

- Bananas
- Cabbages
- Carrots

Additional crop varieties will follow via transfer learning (Month 7–9 onwards).

### Quality Indicators

Each assessment evaluates five tags per delivery:

1. **Size Specification** — meets procurement standard (current precision: 90.9%)
2. **Pest Infestation** — visible pest damage detected (current precision: 89.2%)
3. **Rotten Produce** — rot or fungal signatures present (current precision: 83.3%)
4. **Dehydrated Produce** — moisture loss beyond acceptable threshold (current precision: 56.1%)
5. **Discolouration** — abnormal colour indicating quality degradation (current precision: 33.3%)

> **Note:** Precision figures are from the current production model (Azure Custom Vision). The open-source rebuild targets 90%+ accuracy across all quality tags by Month 10–12.

## Architecture Overview

The system is designed as three decoupled layers that can run independently or together:

```
┌─────────────────────────────────────────────────────┐
│                   Mobile Client                      │
│         (Camera capture → image submission)           │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │   Edge Inference Engine  │  ◄── Offline-capable
          │   (TFLite + OpenCV)      │      TFLite model
          └────────────┬────────────┘
                       │
          ┌────────────▼────────────┐
          │   Climate Correlation    │  ◄── Open weather APIs
          │   Engine                 │      + transit metadata
          └────────────┬────────────┘
                       │
          ┌────────────▼────────────┐
          │   Assessment Output      │
          │   (Quality score +       │
          │    spoilage risk +       │
          │    climate context)      │
          └─────────────────────────┘
```

For the full technical architecture, see **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)**.

## Current Status

This repository is a **pre-release placeholder**. The open-source module is being extracted and rebuilt from Tawi Fresh's production system, which is live and operational today:

**What is running in production now:**
- Computer vision quality assessment processing real school feeding deliveries
- .NET backend on Azure Kubernetes Service (AKS) with Kong API gateway
- Azure Custom Vision prediction endpoint trained on 3 crop types, 5 quality tags
- Climate exposure logging (temperature, transit duration, source-region weather) against every assessment
- CI/CD via Azure DevOps with automated deployment pipelines
- Active school feeding programme client generating real transaction data

**What this open-source release will deliver:**
- The inference layer rebuilt on TensorFlow/TFLite and OpenCV (replacing Azure Custom Vision)
- Edge-deployable, offline-capable model that runs without cloud dependency
- Climate-quality correlation engine as a standalone module
- Training data toolkit and anonymised dataset
- Full documentation for replication by agritech platforms and NGOs

## Release Timeline

| Phase | Timeline | Deliverables |
|-------|----------|-------------|
| **Phase 1** | Month 1–3 | Repository scaffolding, training data pipeline, initial model architecture. Expand dataset to 5,000+ labelled images. MIT-licensed quality grading module published. |
| **Phase 2** | Month 4–6 | Edge-deployable TFLite model. Climate-quality data dashboard (public). Offline inference pipeline. Uganda proof-of-concept deployment. |
| **Phase 3** | Month 7–9 | Anonymised training image dataset published as open data. Cross-crop transfer learning for additional produce varieties. |
| **Phase 4** | Month 10–12 | 90%+ model accuracy across all crop types. Replication package for agritech platforms and NGOs across East Africa. Impact report and methodology publication. |

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Model Training | TensorFlow / Keras | Mature ecosystem, strong mobile/edge deployment path |
| Edge Inference | TensorFlow Lite | Optimised for mobile and low-resource devices |
| Image Processing | OpenCV | Industry-standard, no cloud dependency |
| Model Architecture | MobileNet / EfficientNet | Designed for mobile-first, resource-constrained inference |
| Climate Data | Open Weather APIs | Free, reliable, covers East African regions |
| Training Pipeline | Python, NumPy, Pillow | Standard ML data preparation stack |

## Project Context

This module is part of the broader **Tawi Fresh** platform — a climate-smart nutrition intelligence system for school feeding supply chains. The parent platform handles:

- Produce aggregation and distribution to institutional buyers
- Ordering, sourcing, logistics, and payment processing
- Seasonal availability intelligence from operational transaction data
- Supplier management and farmer demand signalling

The quality assessment module published here is designed to function both as an integrated component of the Tawi platform and as a standalone tool adoptable by other organisations.

## Contributing

This project is in pre-release. Contribution guidelines will be published alongside the first code release (Phase 1).

In the meantime, if you are:
- A **school feeding programme operator** interested in piloting the system
- An **agritech platform or NGO** interested in integrating quality assessment
- A **researcher** working on climate-food quality linkages
- A **developer** with experience in edge ML deployment

We'd like to hear from you. See [CONTRIBUTING.md](CONTRIBUTING.md) for contact details.

## License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for the full text.

We chose MIT to maximise adoption by agritech platforms, NGOs, and government programmes across East Africa and beyond.

## Acknowledgements

This project is supported by a grant application to the [UNICEF Venture Fund](https://www.unicef.org/innovation/venturefund), which invests in open-source technology solutions that benefit children worldwide.

Tawi Fresh's production platform and engineering team are funded by **Tawi Fresh Limited** (part of Tawi Innovex), based in Nairobi, Kenya.

School feeding programme partnership through **Food4Education (F4E)**.

---

**Tawi Fresh Limited** · Nairobi, Kenya · [tawifresh.com](https://tawifresh.com)
