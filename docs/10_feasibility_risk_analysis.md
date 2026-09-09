# Feasibility and Risk Analysis

## 1. DOCUMENT CONTROL

| Property | Description |
| :--- | :--- |
| **Document Name** | Feasibility and Risk Analysis |
| **Project** | SatQuery AI - ISRO SIH (PS ID 26167) |
| **Version** | 1.0 |
| **Status** | Approved |

## 2. TECHNICAL FEASIBILITY ANALYSIS

### 2.1 Compute Requirements
The proposed agentic architecture orchestrates multiple specialist models. Running the full pipeline necessitates strategic resource management.
*   **Model GPU Memory Footprint:**
    *   **GeoChat (7B):** Requires ~16 GB VRAM for 8-bit quantized inference (QLoRA); ~24 GB for FP16 training.
    *   **Grounding-DINO + SAM:** Requires ~8 GB VRAM.
    *   **ChangeFormer / BIT:** Requires ~8-12 GB VRAM.
    *   **Agent Planner (LLM):** Can be handled by a quantized lightweight LLM or API integration, requiring ~8 GB if run locally.
*   **Total Inference Pipeline:** Sequential execution allows models to be loaded and unloaded from a single 16 GB GPU (e.g., NVIDIA T4) or 24 GB GPU (e.g., RTX 3090/4090). Parallel execution requires an A100 (40/80 GB).
*   **Training Requirements:** Fine-tuning GeoChat on the CDVQA dataset requires at least one 24 GB GPU (or multiple 16 GB GPUs) using gradient checkpointing and QLoRA.

### 2.2 Model Availability
All proposed models are accessible and open-source:
*   **GeoChat:** Open-source under Apache 2.0. Weights available on Hugging Face.
*   **Grounding-DINO & SAM:** Apache 2.0 license, widely supported.
*   **Change Detection Models:** Implementations available via PyTorch and open-source repositories.
*   **Orchestration:** LangGraph and LangChain are open-source.

### 2.3 Dataset Accessibility
The required datasets are publicly available for research:
*   **BigEarthNet.txt:** ~65 GB. Readily available but requires significant storage and preprocessing bandwidth.
*   **VRSBench & RSVQA:** Available via standard academic channels.
*   **CDVQA:** Open access dataset for change detection visual question answering.
*   **Licensing:** All are permissible for research and hackathon use.

### 2.4 Library and Framework Maturity
*   **PyTorch (2.x), Transformers, PEFT:** Highly mature, production-ready.
*   **FastAPI, Celery, Redis, PostgreSQL:** Industry-standard stack for robust backend services.
*   **Geospatial Libraries (rasterio, GDAL, pyproj):** Standard toolkit for geospatial analysis, though requiring careful environment configuration (e.g., via Docker) to avoid dependency conflicts.
*   **LangGraph:** A relatively new but rapidly maturing framework; robust enough for directed acyclic graph (DAG) execution.

### 2.5 Team Skill Assessment
Implementing SatQuery AI requires a balanced interdisciplinary team. A typical 6-member student team must possess or quickly acquire:
*   Deep Learning (PyTorch, Hugging Face).
*   Geospatial Data Processing (GDAL, rasterio).
*   Backend System Design (FastAPI, queues).
*   Frontend Development (React/Next.js, Leaflet).
*   The modular architecture allows parallel development, aligning with team specializations.

## 3. ECONOMIC FEASIBILITY

### 3.1 Cloud Compute Costs
Running an A100 GPU instance on AWS (p4d.24xlarge) or GCP (a2-highgpu-1) costs approximately $3.00 - $4.00 per hour. For a 2-month development cycle assuming 200 hours of intensive training/testing, compute costs would be ~$600-$800.

### 3.2 Free Tier Options
*   **Google Colab Pro:** Provides cost-effective access to V100/A100 GPUs for iterative testing.
*   **Kaggle:** Free 30 hours/week of dual T4 GPUs.
*   **University Clusters:** High Performance Computing (HPC) centers provided by academic institutions can completely offset training costs.

### 3.3 Open-Source vs Proprietary Considerations
SatQuery AI relies entirely on open-source software, eliminating licensing costs. Utilizing a local, self-hosted LLM for the planning agent avoids recurring API token costs associated with proprietary models (e.g., GPT-4).

## 4. OPERATIONAL FEASIBILITY

### 4.1 Deployment Complexity
The architecture is containerized using Docker and Docker Compose, encapsulating complex GDAL dependencies and GPU driver requirements (NVIDIA Container Toolkit). This ensures reproducibility across environments.

### 4.2 Maintenance Burden
The modular monolith design simplifies maintenance. Individual models in the Model Registry can be swapped or upgraded independently without refactoring the orchestration layer.

### 4.3 User Training Requirements
The natural-language interface significantly lowers the barrier to entry. End-users (ISRO scientists, analysts) do not need to learn complex GIS software syntax. Training is minimal, focused only on understanding system confidence scores and interpreting spatial overlays.

## 5. RISK REGISTER

| Risk ID | Category | Description | Probability | Impact | Score (PxI) | Mitigation Strategy | Contingency Plan | Owner |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| R01 | Technical | GPU Out of Memory (OOM) during live demo. | Medium | High | High | Implement sequential model loading/unloading; strict VRAM monitoring. | Have a pre-recorded video of the pipeline execution. | ML Lead |
| R02 | Technical | Model inference too slow for real-time demo. | High | Medium | High | Optimize with ONNX/TensorRT; use asynchronous Celery workers. | Warn users via UI; show a progress bar. | Backend Lead |
| R03 | Resource | BigEarthNet.txt dataset too large to download/process in time. | High | High | High | Download a representative subset (10%) for initial pipeline validation. | Use smaller surrogate datasets (e.g., EuroSAT). | ML Lead |
| R04 | Technical | ISRO evaluation set uses different sensor characteristics than training data. | Medium | High | High | Incorporate sensor-agnostic preprocessing (normalization, resampling). | Re-calibrate the input layer dynamically based on metadata. | Geo Engineer |
| R05 | Technical | Co-registration algorithm fails on misaligned pairs. | Medium | High | High | Utilize robust keypoint matching (SIFT/ORB) before change detection. | Prompt user to manually verify alignment or provide pre-registered pairs. | Geo Engineer |
| R06 | Technical | SAR speckle degrades model performance. | Medium | Medium | Medium | Apply Lee or Frost filters as a mandatory preprocessing step for SAR inputs. | Fallback to raw intensity scaling. | Geo Engineer |
| R07 | Technical | LangGraph agent routing errors. | Low | High | Medium | Implement strict schema validation and fallback static routing rules. | Bypass the planner and allow manual tool selection in the UI. | Full-Stack Lead |
| R08 | External | Network failure during cloud-dependent demo. | Low | High | Medium | Ensure local deployment capability on a high-end laptop with an RTX GPU. | Run the demo completely offline using pre-loaded models. | Integration Lead |
| R09 | Resource | Team member unavailability. | Medium | Medium | Medium | Cross-train members; maintain comprehensive documentation. | Reallocate critical path tasks to available members. | Project Mgr |
| R10 | Technical | GeoTIFF parsing errors on edge-case files. | Medium | Medium | Medium | Implement robust exception handling using `rasterio`; support standard TIFF fallbacks. | Convert problem files to standard PNG/JPEG for basic VQA. | Backend Lead |
| R11 | Technical | Model hallucination on out-of-distribution queries. | High | High | High | Enforce strict output parsing; implement confidence thresholding. | Append a low-confidence warning to the UI response. | ML Lead |
| R12 | Technical | WebSocket connection drops during streaming. | Low | Medium | Low | Implement automatic reconnection logic in the frontend. | Fallback to standard HTTP polling. | Frontend Lead |
| R13 | Technical | Browser memory issues with large image overlays (Leaflet). | Medium | High | High | Implement server-side tiling for large GeoTIFFs before sending to frontend. | Downsample display images; retain full resolution for backend analysis. | Frontend Lead |
| R14 | Technical | Benchmark scores below baseline. | Medium | High | High | Focus on data augmentation and hyperparameter tuning during fine-tuning. | Highlight the multi-tool orchestration aspect as the primary novelty. | ML Lead |
| R15 | Schedule | Time overrun on fine-tuning. | High | Medium | High | Start training pipelines early; use LoRA/QLoRA for faster convergence. | Use base pre-trained models with zero-shot prompting. | ML Lead |

## 6. RISK HEATMAP

| Probability \ Impact | Low | Medium | High |
| :--- | :--- | :--- | :--- |
| **High** | | R02, R15 | R03, R11 |
| **Medium** | R10 | R06, R09 | R01, R04, R05, R13, R14 |
| **Low** | | R12 | R07, R08 |

## 7. SWOT ANALYSIS

### Strengths
*   **Modular Architecture:** Easy to update individual components without disrupting the entire system.
*   **Multi-Modal Reasoning:** Seamlessly handles optical, multispectral, and SAR data.
*   **Agentic Orchestration:** Automates complex multi-step analysis dynamically based on user intent.
*   **Native GeoTIFF Support:** Retains critical metadata and multi-band information.

### Weaknesses
*   **High Resource Requirements:** Needs substantial GPU compute for parallel execution and training.
*   **System Complexity:** Integrating multiple large models introduces latency and orchestration overhead.

### Opportunities
*   **Scalability:** The tool registry allows for the integration of future state-of-the-art models.
*   **ISRO Utility:** Directly addresses the problem statement for an interactive, text-driven remote sensing analysis tool.
*   **Open-Source Contribution:** Potential to release the framework to the broader remote sensing community.

### Threats
*   **Data Domain Shift:** Models trained on European datasets (BigEarthNet) may underperform on Indian geography (ISRO test sets).
*   **Hardware Constraints:** Lack of access to high-end GPUs during the hackathon finale could limit the live demonstration capabilities.
