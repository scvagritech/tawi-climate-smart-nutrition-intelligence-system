# Contributing to Tawi Fresh Quality AI

Thank you for your interest in contributing to open-source produce quality assessment for school feeding programmes. This project exists because no child should receive degraded food that a system could have caught — and because the tools to prevent that should be freely available to every programme that needs them.

This guide covers how to contribute at every stage, whether the project is in pre-release documentation or active development.

---

## Table of Contents

- [Project Status](#project-status)
- [Ways to Contribute](#ways-to-contribute)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Data Contributions](#data-contributions)
- [Reporting Issues](#reporting-issues)
- [Pull Request Process](#pull-request-process)
- [Architecture Decisions](#architecture-decisions)
- [Community and Communication](#community-and-communication)
- [Code of Conduct](#code-of-conduct)
- [License](#license)

---

## Project Status

This project is currently in **pre-release**. The repository contains documentation, architecture specifications, and the project scaffold. Source code is being extracted and rebuilt from Tawi Fresh's production system and will be published incrementally according to the [release timeline](README.md#release-timeline).

**What this means for contributors:**

- Documentation improvements, architecture feedback, and issue discussions are welcome now
- Code contributions will be accepted once the initial codebase is published (Phase 1)
- Pilot partnership and integration interest can be expressed at any time via [GitHub Issues](https://github.com/tawi-fresh/tawi-fresh-quality-ai/issues) using the provided templates

---

## Ways to Contribute

### Non-Code Contributions (Available Now)

These are just as valuable as code, especially in the early stages of an open-source project:

**Documentation**
- Improve clarity, fix errors, or add examples to existing docs
- Translate documentation into Swahili, French, or other languages relevant to East African deployment
- Write integration guides for specific platforms or use cases

**Domain Expertise**
- Review the [architecture](docs/ARCHITECTURE.md) from the perspective of school feeding operations, food safety standards, or agricultural supply chains
- Suggest quality indicators or crop types that should be prioritised
- Share knowledge about cold chain gaps, delivery logistics, or seasonal patterns in specific regions

**Research Collaboration**
- Propose improvements to model architecture or training methodology
- Share relevant published research on agricultural quality grading or climate-food linkages
- Review dataset design and annotation schema for completeness

**Testing and Feedback**
- Test documentation for clarity and completeness (can someone unfamiliar with the project follow the setup guide?)
- Report gaps in the architecture or API design
- Suggest edge cases the system should handle

### Code Contributions (Phase 1 Onwards)

Once the initial codebase is published, contributions are welcome across these areas:

| Area | Skills Needed | Priority |
|------|---------------|----------|
| Model training and evaluation | Python, TensorFlow/Keras, ML fundamentals | High |
| Edge inference optimisation | TFLite, model quantisation, OpenCV | High |
| Climate data integration | Python, REST APIs, data engineering | Medium |
| Assessment API | Python, FastAPI, REST API design | Medium |
| Mobile integration | Android (Kotlin/Java), TFLite Android SDK | Medium |
| Data pipeline tooling | Python, image processing, COCO format | Medium |
| CI/CD and testing | GitHub Actions, pytest, model validation | Medium |
| iOS integration | Swift, TFLite iOS SDK | Lower |

### Pilot Partnerships

If you operate a school feeding programme, agritech platform, or food distribution network and want to pilot the quality assessment system, open an issue using the **Pilot Interest** template. Early pilot partners directly shape the tool's design.

---

## Getting Started

### Prerequisites

The following will be required once code is published:

- Python 3.10+
- TensorFlow 2.x
- OpenCV 4.x
- Git and Git LFS (for model files and image data)

### Setup (Coming in Phase 1)

```bash
# Clone the repository
git clone https://github.com/tawi-fresh/tawi-fresh-quality-ai.git
cd tawi-fresh-quality-ai

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt

# Run tests
pytest
```

### Repository Structure

```
tawi-fresh-quality-ai/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── requirements.txt              # (Phase 1)
├── requirements-dev.txt          # (Phase 1)
├── docs/
│   └── ARCHITECTURE.md
├── src/
│   ├── inference/                # TFLite inference engine + OpenCV preprocessing
│   ├── climate/                  # Climate-quality correlation engine
│   ├── api/                      # REST API wrapper (FastAPI)
│   ├── training/                 # Model training scripts and pipelines
│   └── utils/                    # Shared utilities
├── models/                       # Pre-trained TFLite models (via Git LFS)
├── data/
│   ├── sample/                   # Small sample dataset for testing
│   └── schema/                   # COCO annotation schema and label definitions
├── tests/                        # (Phase 1)
└── .github/
    └── ISSUE_TEMPLATE/
```

---

## Development Workflow

We use a **fork and branch** workflow:

1. **Fork** the repository to your GitHub account
2. **Clone** your fork locally
3. **Create a branch** from `main` for your work
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. **Make your changes** in small, focused commits
5. **Test** your changes locally
6. **Push** to your fork and open a **Pull Request** against `main`

### Branch Naming

Use a prefix that describes the type of change:

| Prefix | Use For |
|--------|---------|
| `feature/` | New functionality |
| `fix/` | Bug fixes |
| `docs/` | Documentation changes |
| `data/` | Dataset or annotation changes |
| `model/` | Model architecture or training changes |
| `infra/` | CI/CD, tooling, or build configuration |

Examples: `feature/cabbage-augmentation-pipeline`, `fix/tflite-quantisation-overflow`, `docs/swahili-translation`

---

## Coding Standards

### Python

- **Style:** PEP 8, enforced via `ruff` (configured in `pyproject.toml`)
- **Type hints:** Required on all public function signatures
- **Docstrings:** Google style on all public modules, classes, and functions
- **Line length:** 100 characters maximum
- **Imports:** Sorted via `isort` (configured in `pyproject.toml`)

```python
def classify_produce(
    image: np.ndarray,
    crop_type: CropType,
    confidence_threshold: float = 0.5,
) -> AssessmentResult:
    """Classify produce quality from a preprocessed image.

    Args:
        image: Preprocessed image as NumPy array (224x224x3, float32, normalised).
        crop_type: The type of crop being assessed.
        confidence_threshold: Minimum confidence to flag a quality tag.

    Returns:
        AssessmentResult containing quality tags with confidence scores.

    Raises:
        ValueError: If image dimensions don't match model input requirements.
    """
```

### Tests

- All new functionality must include tests
- Use `pytest` as the test framework
- Test files mirror the source structure: `src/inference/engine.py` → `tests/inference/test_engine.py`
- Model accuracy tests should use the sample dataset in `data/sample/`
- Target: 80%+ code coverage on non-model code

### Commits

Write clear, descriptive commit messages:

```
Add temperature-weighted decay curve for cabbage spoilage model

The previous linear decay assumption underestimated spoilage rate
at ambient temperatures above 28°C. This adds a temperature-weighted
exponential curve fitted against 3 months of field assessment data
from the Nyandarua-Nairobi transit corridor.

Closes #42
```

- First line: imperative mood, under 72 characters
- Body: explain *why*, not just *what*
- Reference related issues

---

## Data Contributions

Training data is critical to this project's success. If you can contribute labelled produce images, here's how:

### Image Requirements

| Requirement | Specification |
|-------------|--------------|
| Format | JPEG or PNG |
| Minimum resolution | 640 × 480 pixels |
| Lighting | Natural lighting preferred (field conditions) |
| Background | As captured at delivery point (no studio setup needed) |
| Crop types | Bananas, cabbages, carrots (initial); other crops welcome |
| Metadata | Crop type, capture date, region (GPS optional), ambient conditions |

### Labelling

We use **COCO format** annotations. Each image should be labelled with the applicable quality tags:

- `SIZE_SPEC` — pass / fail
- `PEST` — present / absent
- `ROT` — present / absent
- `DEHYDRATION` — present / absent
- `DISCOLOURATION` — present / absent

A labelling guide with visual examples for each tag and crop type will be published in Phase 1.

### Privacy and Consent

- **No images containing identifiable people.** Crop the image to show only the produce.
- **No supplier or location PII.** Region-level location is acceptable (e.g., "Nyandarua County"), GPS coordinates will be rounded to district level in the public dataset.
- **Consent:** By contributing images, you confirm you have the right to share them and agree to their publication under the MIT License.

### How to Submit

Once the data pipeline is established:

1. Open an issue using the **Data Contribution** template
2. We'll provide upload instructions and the labelling guide
3. Contributed images will be reviewed, deduplicated, and incorporated into the training set
4. Contributors will be credited in the dataset release notes

---

## Reporting Issues

### Bug Reports

Use the **Bug Report** issue template. Include:

- What you expected to happen
- What actually happened
- Steps to reproduce
- Environment details (OS, Python version, TensorFlow version, device if relevant)
- Logs or error output (redact any sensitive information)

### Feature Requests

Use the **Feature Request** issue template. Describe:

- The problem or gap you've identified
- Your proposed solution or approach
- Who benefits (operators, farmers, developers, researchers)
- Any relevant context from field experience

### Security Vulnerabilities

**Do not open a public issue for security vulnerabilities.** Email the maintainers directly at [TBD — update before publishing] with a description of the vulnerability. We will acknowledge receipt within 48 hours and provide a timeline for resolution.

---

## Pull Request Process

1. **Ensure your PR addresses a single concern.** One feature, one fix, or one improvement per PR. If your change touches multiple areas, split it.

2. **Update documentation** if your change affects the public API, architecture, or setup process.

3. **All tests must pass.** Run the full test suite locally before submitting:
   ```bash
   pytest
   ruff check src/ tests/
   ```

4. **Fill out the PR template.** Describe what changed, why, and how to test it.

5. **Review process:**
   - At least one maintainer review is required
   - For model architecture or training changes: two reviews required
   - For data schema changes: maintainer + domain expert review required
   - Reviewers may request changes — this is normal and collaborative

6. **Merge:** Maintainers will squash-merge approved PRs into `main`.

### What We Look For in Reviews

- Does it solve the stated problem?
- Is it tested?
- Is it documented?
- Does it follow the coding standards?
- Will it work in low-resource / offline environments? (Edge deployment is a first-class concern, not an afterthought.)
- Does it introduce unnecessary cloud dependencies?

---

## Architecture Decisions

Significant technical decisions are documented as **Architecture Decision Records (ADRs)** in `docs/decisions/` (coming in Phase 1). If you want to propose a significant change to the architecture, model approach, or technology stack:

1. Open an issue labelled `architecture` describing the proposal
2. Include context, alternatives considered, and your recommendation
3. Discussion happens on the issue before any implementation begins

Examples of changes that warrant an ADR:
- Switching model architecture (e.g., MobileNet → EfficientNet)
- Adding a new deployment mode
- Changing the annotation format
- Adding a cloud dependency

---

## Community and Communication

- **GitHub Issues:** Primary channel for technical discussions, bug reports, and feature proposals
- **GitHub Discussions:** General questions, ideas, and community conversation (will be enabled with first code release)
- **Email:** [TBD — update before publishing] for partnership enquiries, security reports, and anything that shouldn't be public

We aim to respond to issues and PRs within 5 business days. For time-sensitive security reports, within 48 hours.

---

## Code of Conduct

### Our Standards

This project is committed to providing a welcoming, inclusive, and harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

**Expected behaviour:**
- Use welcoming and inclusive language
- Respect differing viewpoints and experiences
- Accept constructive criticism gracefully
- Focus on what is best for the community and the children this project serves
- Show empathy toward other community members

**Unacceptable behaviour:**
- Trolling, insulting or derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Any conduct which could reasonably be considered inappropriate in a professional setting

### Enforcement

Instances of unacceptable behaviour may be reported to the project maintainers at [TBD — update before publishing]. All complaints will be reviewed and investigated and will result in a response that is deemed necessary and appropriate to the circumstances. Maintainers are obligated to maintain confidentiality with regard to the reporter of an incident.

### Attribution

This Code of Conduct is adapted from the [Contributor Covenant](https://www.contributor-covenant.org/), version 2.1.

---

## License

By contributing to this project, you agree that your contributions will be licensed under the [MIT License](LICENSE). This ensures maximum reuse by agritech platforms, NGOs, government programmes, and researchers worldwide.

---

**Thank you for helping improve nutrition outcomes for children through open-source technology.**
