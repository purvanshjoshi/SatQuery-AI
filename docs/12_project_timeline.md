# Project Timeline and Resource Plan

## 1. DOCUMENT CONTROL

| Property | Description |
| :--- | :--- |
| **Document Name** | Project Timeline and Resource Plan |
| **Project** | SatQuery AI - ISRO SIH (PS ID 26167) |
| **Version** | 1.0 |
| **Status** | Approved |

## 2. PROJECT PHASES

*   **Phase 1: Research and Planning** (Week 1-2)
*   **Phase 2: Data Preparation and Model Adaptation** (Week 2-4)
*   **Phase 3: Core Backend Development** (Week 3-5)
*   **Phase 4: Frontend Development** (Week 4-6)
*   **Phase 5: Integration and Agentic Pipeline** (Week 5-7)
*   **Phase 6: Testing and Evaluation** (Week 7-8)
*   **Phase 7: Demo Preparation and Polish** (Week 8-9)

## 3. DETAILED TASK BREAKDOWN

| Phase | Task | Estimated Hours |
| :--- | :--- | :--- |
| **Phase 1** | Finalize architecture and model selection. | 15 |
| | Setup project repositories and CI/CD. | 10 |
| **Phase 2** | Download and preprocess BigEarthNet.txt & CDVQA datasets. | 40 |
| | Fine-tune GeoChat (QLoRA) and adapt Grounding-DINO/SAM. | 60 |
| | Implement Optical-SAR fusion cross-attention module. | 35 |
| **Phase 3** | Setup FastAPI backend, PostgreSQL database, and Celery/Redis. | 30 |
| | Implement tool wrappers (ML inference & Non-ML tools). | 45 |
| **Phase 4** | Develop Next.js UI components (chat interface, map overlays). | 40 |
| | Integrate Leaflet for GeoTIFF visualization. | 25 |
| **Phase 5** | Implement LangGraph agent planner and routing logic. | 50 |
| | Integrate backend APIs with frontend UI. | 30 |
| **Phase 6** | End-to-end system testing against evaluation metrics. | 35 |
| | Debugging, optimization, and edge-case handling. | 40 |
| **Phase 7** | Prepare presentation, final documentation, and video demo. | 25 |

## 4. TEAM ROLE ASSIGNMENTS

**Team Composition (6 Members):**
*   **ML1 & ML2:** Machine Learning Engineers
*   **GEO:** Geospatial Engineer
*   **BE:** Backend Engineer
*   **FE:** Frontend Engineer
*   **FS/Lead:** Full-Stack/Integration Lead

**RACI Matrix (Responsible, Accountable, Consulted, Informed):**

| Task | ML1/ML2 | GEO | BE | FE | FS/Lead |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Architecture Design | C | C | C | C | R/A |
| Model Fine-tuning | R/A | C | I | I | I |
| GeoTIFF Processing | I | R/A | C | I | C |
| API Development | I | I | R/A | C | C |
| UI/Map Development | I | C | I | R/A | C |
| Agent Integration | C | I | C | I | R/A |
| End-to-End Testing | R | R | R | R | A |

## 5. MILESTONE TABLE

| Milestone | Deliverable | Target Date |
| :--- | :--- | :--- |
| M1: Architecture Frozen | Finalized tech stack and design docs. | End of Week 2 |
| M2: Models Adapted | Weights saved for VQA, Captioning, and Grounding. | End of Week 4 |
| M3: Core Backend Live | FastAPI endpoints and Celery workers operational. | End of Week 5 |
| M4: Frontend MVP | Functional chat and map rendering. | End of Week 6 |
| M5: Agent Pipeline Complete | LangGraph orchestrating full query lifecycle. | End of Week 7 |
| M6: Final Release Candidate | Fully tested, bug-free deployment package. | End of Week 8 |

## 6. CRITICAL PATH ANALYSIS

The project's critical path hinges on the ML adaptation and Backend integration:
1.  **Dataset Preprocessing (Phase 2):** Cannot train models without prepared data.
2.  **Model Fine-tuning (Phase 2):** Backend cannot serve models until weights are finalized.
3.  **Agent Integration (Phase 5):** The core novelty relies on LangGraph working flawlessly with the backend tool wrappers. Any delays in ML endpoints will directly delay the agent pipeline development.

## 7. RESOURCE REQUIREMENTS

### Hardware
*   **Development:** Minimum NVIDIA T4 (16 GB) instances (Colab/Kaggle).
*   **Production/Demo:** NVIDIA A100 (40/80 GB) or RTX 3090/4090 to support concurrent model loading in the agentic pipeline.

### Software Licenses
*   All software is open-source (Apache 2.0, MIT). No licensing costs.

### Cloud Credits
*   Estimated requirement: $500 - $800 in AWS/GCP credits or equivalent Colab Pro subscriptions for the duration of the hackathon development phase.

## 8. SPRINT PLAN

*   **Sprint 1 (Weeks 1-2):** Design freeze, environment setup, data acquisition.
*   **Sprint 2 (Weeks 3-4):** ML model fine-tuning, basic backend API structure.
*   **Sprint 3 (Weeks 5-6):** LangGraph integration, frontend chat UI and map visualization.
*   **Sprint 4 (Weeks 7-8):** System integration, rigorous testing, optimization, and final polish.
