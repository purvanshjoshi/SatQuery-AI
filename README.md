# SatQuery AI

**An Interactive Vision-Language Assistant for Multimodal Remote Sensing Image Analysis through Text Queries**

> Smart India Hackathon 2026 | Problem Statement ID: 26167
> Organization: Indian Space Research Organisation (ISRO), Department of Space
> Category: Software | Theme: Space Technology

---

## Overview

SatQuery AI is an agentic vision-language assistant that enables users to analyze single and paired remote-sensing images through natural-language queries. Instead of applying a single generic VLM, the system selects and executes suitable remote-sensing specialist models, validates inputs, combines their outputs, and returns an evidence-grounded response.

## Key Capabilities

- **Single-Image VQA** -- Answer natural-language questions about optical, multispectral, or SAR imagery.
- **Scene Captioning** -- Generate natural-language descriptions of remote sensing scenes.
- **Visual Grounding** -- Highlight and segment regions referenced in text queries.
- **Bi-Temporal Change Analysis** -- Detect, describe, and answer questions about changes between two temporal observations.
- **Cross-Modal Optical-SAR Analysis** -- Extract complementary information by jointly reasoning over co-registered optical and SAR image pairs.
- **Agentic Orchestration** -- Automatically select, sequence, and execute specialist models based on the query and input configuration.

## Architecture

```
User Query + Image(s)
        |
   [Input Validator] -- Format, CRS, bands, co-registration
        |
   [Intent Classifier] -- VQA / Captioning / Grounding / Change / Fusion
        |
   [Task Planner] -- DAG of specialist tool executions
        |
   [Model Executor] -- GeoChat VQA | Grounding-DINO+SAM | BIT ChangeFormer | Optical-SAR Fusion
        |
   [Output Synthesizer] -- Combine text + spatial evidence + confidence
        |
   Evidence-Grounded Response + Execution Trace
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14, TypeScript, Tailwind CSS, Leaflet |
| Backend | FastAPI (Python 3.11+) |
| ML Framework | PyTorch 2.x, Hugging Face Transformers, PEFT (QLoRA) |
| Geospatial | rasterio, GDAL, pyproj, shapely |
| Agent Orchestration | LangGraph |
| Task Queue | Celery + Redis |
| Database | PostgreSQL + PostGIS |
| Deployment | Docker + NVIDIA Container Toolkit |

## Documentation

All production documents are available in the [`docs/`](./docs/) directory:

| # | Document | Description |
|---|----------|-------------|
| 0 | [Document Index](docs/00_document_index.md) | Master navigation and reading order |
| 1 | [Solution Overview](docs/01_solution_overview.md) | Problem analysis, solution philosophy, requirement traceability |
| 2 | [System Architecture](docs/02_system_architecture.md) | Component diagrams, data flows, deployment architecture |
| 3 | [Functional Requirements](docs/03_functional_requirements.md) | 50+ requirements with IDs and acceptance criteria |
| 4 | [Non-Functional Requirements](docs/04_non_functional_requirements.md) | Performance, scalability, security targets |
| 5 | [Technical Design](docs/05_technical_design.md) | Model registry, AI pipelines, agentic orchestration |
| 6 | [Data Strategy](docs/06_data_strategy.md) | Datasets, preprocessing, training strategy |
| 7 | [API Specification](docs/07_api_specification.md) | REST API endpoints, WebSocket protocol, schemas |
| 8 | [UI/UX Design](docs/08_ui_ux_design.md) | Page layouts, user flows, design system |
| 9 | [Evaluation Strategy](docs/09_evaluation_strategy.md) | Benchmark protocols, testing plan, SIH criteria mapping |
| 10 | [Feasibility and Risk Analysis](docs/10_feasibility_risk_analysis.md) | Risk register, SWOT analysis |
| 11 | [Innovation Statement](docs/11_innovation_statement.md) | Novelty dimensions, prior art comparison |
| 12 | [Project Timeline](docs/12_project_timeline.md) | 9-week sprint plan, RACI matrix |
| 13 | [Sustainability Roadmap](docs/13_sustainability_roadmap.md) | Future roadmap, impact assessment, SDG alignment |

## Datasets

| Dataset | Purpose |
|---------|---------|
| [BigEarthNet.txt](https://arxiv.org/abs/2603.29630) | Primary adaptation dataset (Sentinel-1 SAR + Sentinel-2 multispectral + text) |
| VRSBench | Evaluation: captioning, grounding, VQA |
| RSVQA | Evaluation: visual question answering |
| CDVQA | Evaluation: change detection VQA |

## Team

Smart India Hackathon 2026 -- Team SatQuery AI

## License

This project is licensed under the Apache License 2.0.
