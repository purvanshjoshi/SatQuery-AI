# SatQuery AI: Evaluation and Testing Strategy

## 1. DOCUMENT CONTROL

| Field | Description |
|-------|-------------|
| Document Title | Evaluation and Testing Strategy |
| System Name | SatQuery AI |
| Problem Statement | PS ID 26167 — SatQuery AI - An Interactive Vision-Language Assistant for Multimodal Remote Sensing Image Analysis through Text Queries |
| Organization | Indian Space Research Organisation (ISRO), Department of Space |
| Version | 1.0.0 |
| Classification | Internal / Confidential |

---

## 2. EVALUATION FRAMEWORK OVERVIEW

The evaluation framework for SatQuery AI is constructed to ensure rigorous and reproducible validation of the multimodal interactive assistant. Given the complexity of combining agentic orchestration with specialized vision-language models, the evaluation strategy must isolate subsystem performance while validating end-to-end functionality. 

The strategy distinguishes between three primary dimensions of evaluation:

1. **AI Model Evaluation on Benchmarks**: This facet focuses on the intrinsic quantitative performance of the individual machine learning specialist models (e.g., GeoChat for VQA/Captioning, Grounding-DINO for object grounding, BIT for change detection) on standardized academic datasets. Metrics computed in this phase validate that the baseline ML capabilities meet or exceed state-of-the-art literature.
2. **System Integration Testing**: This facet examines the structural integrity, latency, fault tolerance, and orchestration accuracy of the LangGraph-based agentic layer. It evaluates how effectively the system routes user intents, fuses outputs from multiple tools (including non-ML tools like Spectral Index Calculators), handles edge cases, and scales under concurrent load on the designated GPU hardware (NVIDIA T4 / A100).
3. **SIH Judging Criteria Evaluation**: This facet serves as a qualitative and strategic self-assessment against the official Smart India Hackathon rubrics. It ensures that the technical execution maps directly to expectations regarding novelty, scalability, practical impact, and user experience.

---

## 3. BENCHMARK EVALUATION PROTOCOL

The foundation of the evaluation relies on measuring specialist model capabilities across rigorous academic datasets. The models will be fine-tuned via QLoRA and evaluated on held-out test splits.

### 3.1 VRSBench Evaluation

VRSBench provides a comprehensive framework for assessing single-image captioning, grounding, and visual question answering capabilities on high-resolution optical images.

**Test Split Specification:**
The official VRSBench validation/test splits will be utilized. Specifically, 10% of the dataset is reserved for testing, ensuring no data leakage between the training and evaluation sets.

**Captioning Metrics:**
- **BLEU-1 and BLEU-4:** Evaluates n-gram precision against ground-truth descriptions.
- **METEOR:** Computes harmonic mean of unigram precision and recall, incorporating stemming and synonym matching for better correlation with human judgment.
- **ROUGE-L:** Measures longest common subsequence to evaluate sentence structure and coherence.
- **CIDEr:** Consensus-based Image Description Evaluation; heavily penalizes generic sentences and rewards informative, descriptive captions.

**Grounding Metrics:**
- **Accuracy@0.5 IoU:** A grounding prediction is considered correct if the Intersection over Union (IoU) between the predicted bounding box and the ground-truth annotation is strictly greater than 0.5.

**VQA Metrics:**
- **Overall Accuracy (OA):** The proportion of correctly answered questions across the entire test set.
- **Per-Category Accuracy:** Accuracy computed independently for object counting, spatial relationships, color identification, and object existence.

**Baseline Comparison Targets:**
- Target BLEU-4 > 0.150
- Target CIDEr > 0.850
- Target Grounding Acc@0.5 > 65.0%
- Target VQA OA > 70.0%

### 3.2 RSVQA Evaluation

The RSVQA dataset challenges the system with natural language questions concerning low-resolution (LR) and high-resolution (HR) remote sensing imagery.

**Test Split Specification:**
Evaluations will be conducted separately on the RSVQA-LR (Sentinel-2, 10m resolution) and RSVQA-HR (aerial imagery) test splits.

**Metrics:**
- **Overall Accuracy (OA):** Total correct responses divided by total questions.
- **Average Accuracy (AA):** Mean of accuracies computed for each question category independently, preventing imbalance in question types from skewing results.

**Question Type Breakdown:**
- **Presence:** "Is there a road in this image?"
- **Comparison:** "Are there more residential buildings than industrial buildings?"
- **Count:** "How many water bodies are present?"
- **Rural/Urban:** Broad scene classification questions.

**Baseline Comparison Targets:**
- RSVQA-LR OA > 82.0%
- RSVQA-HR OA > 85.0%

### 3.3 CDVQA Evaluation

To evaluate bi-temporal change reasoning, the Change Detection Visual Question Answering (CDVQA) dataset is utilized.

**Test Split Specification:**
The standard CDVQA evaluation split will be used, comprising pre- and post-event image pairs with corresponding question-answer pairs.

**Metrics:**
- **Overall Accuracy (OA):** Standard correctness metric for the text output.
- **Per-Question-Type Accuracy:** Specific measurement of system performance across varied change-reasoning tasks.

**Question Types:**
- **Change Existence:** "Has there been any change in this area?"
- **Change Type:** "What kind of change occurred in the top-left quadrant?" (e.g., deforestation, urban expansion).
- **Change Count:** "How many new buildings were constructed?"
- **Change Location:** "Where did the flooding occur?"

**Baseline Comparison Targets:**
- Change Existence Accuracy > 88.0%
- Overall CDVQA Accuracy > 75.0%

### 3.4 BigEarthNet.txt Evaluation

BigEarthNet.txt serves as a proxy for evaluating the system's ability to handle multi-modal representations, particularly the alignment between text and Sentinel-1/Sentinel-2 paired data.

**Adaptation Quality Metrics:**
- **Multi-Label Classification F1:** Using the vision encoder outputs to predict scene classes via a linear probe, measuring macro and micro F1 scores.
- **Text-Image Retrieval Recall@K:** (Recall@1, Recall@5, Recall@10) Assesses cross-modal alignment capability by ranking images given a text query and vice-versa.

---

## 4. ISRO/SAC HIDDEN EVALUATION SET PREPARATION

The ultimate test for SatQuery AI involves evaluation against undisclosed datasets curated by ISRO/SAC, likely containing high-resolution Cartosat-2S optical imagery and RISAT Synthetic Aperture Radar (SAR) imagery. The system must generalize beyond the Sentinel-based distributions of the academic datasets.

### 4.1 Generalization Strategy
To avoid overfitting to Sentinel-1/2 spectral characteristics:
- The input pipeline enforces strict normalization (e.g., zero mean, unit variance per band) rather than absolute top-of-atmosphere reflectance values.
- Data augmentation during fine-tuning includes resolution scaling, contrast jitter, and synthetic speckle noise to simulate varied sensor modalities.
- The Agentic Orchestration layer relies on abstract tool interfaces; if a sensor format falls outside trained parameters, the orchestrator gracefully degrades to fallback models (e.g., zero-shot VLMs) or non-ML spectral index calculations.

### 4.2 Cartosat-2S Characteristics and Pipeline Adaptation
Cartosat-2S provides sub-meter panchromatic and ~2m multispectral imagery.
- **Resolution Mismatch:** The models trained on 10m Sentinel data will process Cartosat data via a sliding window / tiling mechanism configured during the Task Planning phase.
- **Band Configuration:** The metadata extraction tool reads the TIFF header to map Cartosat bands (Blue, Green, Red, NIR) to the expected input channels of the ML models.

### 4.3 RISAT SAR Characteristics and Preprocessing
RISAT operates primarily in C-band (RISAT-1) or X-band (RISAT-2).
- **Polarization Differences:** The optical-SAR fusion module dynamically adapts to Single Polarization (e.g., HH or VV) or Dual Polarization inputs.
- **Speckle Handling:** The preprocessing pipeline incorporates Lee or Frost filters prior to ingestion by the Grounding-DINO or GeoChat SAR encoders.
- **dB Conversion:** Radiometric calibration and logarithmic scaling are applied to normalize backscatter intensity ranges.

### 4.4 Expected Evaluation Tasks
The system anticipates ISRO/SAC evaluations focused on:
- Disaster assessment (flooding, landslides) requiring bi-temporal SAR change detection.
- Strategic target monitoring using Cartosat-2S high-resolution capabilities.
- Cross-modal queries: "Identify the bridges visible in the optical image and confirm their structural integrity using the SAR image."

---

## 5. AGENTIC ORCHESTRATION EVALUATION

Evaluating the LLM-based agent (LangGraph) is as critical as evaluating the specialist vision models. The orchestration layer must correctly interpret the natural language query and orchestrate the Directed Acyclic Graph (DAG) of tools.

### 5.1 Task Routing Accuracy
- **Definition:** The percentage of user queries that are routed to the correct specialist model(s).
- **Evaluation Method:** A curated validation set of 200 complex text queries (with known intent classifications) will be passed to the Query Intent Classification module.
- **Target:** > 95% routing accuracy.

### 5.2 Input Validation Accuracy
- **Definition:** The system's ability to correctly reject invalid files (e.g., corrupted GeoTIFFs, mismatched projections) and accept valid files.
- **Evaluation Method:** Subjecting the ingestion pipeline to 50 edge-case files (malformed headers, incomplete bands, extreme nodata values).
- **Target:** 100% correct rejection rate with informative user feedback.

### 5.3 Compound Query Decomposition Accuracy
- **Definition:** The planner's ability to break down multi-step queries into a correct sequence of tool calls.
- **Example:** "Calculate NDVI and outline areas where it dropped below 0.2 since last month." (Requires Spectral Index Tool -> Thresholding -> Vectorization Tool -> Text Synthesis).
- **Target:** The generated LangGraph execution path must match the human-annotated reference path for 90% of compound test queries.

### 5.4 Execution Trace Completeness and Correctness
- **Definition:** The comprehensiveness of the audit logs presented to the user.
- **Requirement:** Every task executed must log its input parameters, selected model, inference time, intermediate output summary, and a confidence score. Evaluated via manual code review and automated schema validation of the trace JSON.

### 5.5 End-to-End Latency Benchmarks
- **Target Metrics (NVIDIA T4):**
  - Single-image VQA / Captioning: < 4 seconds.
  - Object Grounding (DINO + SAM): < 8 seconds.
  - Bi-temporal Change Detection: < 12 seconds.
  - Spectral Index Calculation (Non-ML): < 2 seconds.

---

## 6. SYSTEM TESTING PLAN

To ensure robustness, reliability, and deployability, a comprehensive software testing pipeline is established.

### 6.1 Unit Tests
- **Scope:** Individual functions, isolated ML model inference scripts, preprocessing utilities, and query intent classifiers.
- **Implementation:** PyTest framework. Mock inputs (synthetic tensors) are used to test model forward passes without loading heavy weights.
- **Coverage Target:** 80% line coverage for backend logic.

### 6.2 Integration Tests
- **Scope:** The end-to-end flow from the Next.js frontend, through the FastAPI backend, to the Celery worker queue, and back to the client.
- **Implementation:** Automated test scripts that mimic API requests with actual small-scale GeoTIFFs, validating the final composite response format (text + geospatial metadata).

### 6.3 Performance Tests
- **Scope:** Latency under concurrent load and GPU memory utilization.
- **Implementation:** Locust framework to simulate multiple concurrent users submitting queries.
- **Monitoring:** Tracking CUDA memory spikes to ensure the system does not encounter Out-Of-Memory (OOM) errors during peak utilization. Batching mechanisms will be tested.

### 6.4 Edge Case Tests
The system must gracefully handle the following scenarios without catastrophic failure:
- **Corrupted GeoTIFF:** Handled by rasterio try-catch blocks during ingestion; returns a standard error artifact.
- **Mismatched CRS in pairs:** Detected during input validation; automatically reprojects to a common EPSG code using pyproj/GDAL before inference.
- **Extremely large images (>10000x10000):** Triggers automatic tiling via the orchestrator to prevent OOM.
- **Non-English queries:** The system relies on the LLM intent classifier, which inherently handles multilingual translation before passing standardized prompts to the downstream English-based vision models.
- **Ambiguous or multi-task queries:** The agent requests user clarification if intent confidence falls below 60%.
- **Single image when pair is expected:** Returns a descriptive error instructing the user to upload a secondary temporal image.
- **SAR image with severe speckle:** Validates that the preprocessing filtering smooths the image sufficiently for the downstream VLM.
- **Cloud-covered optical image:** Validates that the VQA model correctly identifies cloud presence and expresses low confidence in ground-level observations.

### 6.5 Regression Tests
- Automated test suites run on every commit to the main branch via GitHub Actions. Ensures that updates to the orchestration logic do not degrade previously validated routing accuracy or inference latency.

---

## 7. SIH JUDGING CRITERIA SELF-ASSESSMENT

To ensure alignment with the Smart India Hackathon objectives, the system is continually evaluated against the official judging criteria.

### 7.1 Novelty of the Idea
- **Assessment:** High. Traditional remote sensing applications rely on static dashboards and monolithic deep learning models for singular tasks. SatQuery AI introduces an **agentic architecture** that dynamically routes queries to specialist models, treats non-ML algorithms (spectral indices) as equal peers to ML models, and facilitates cross-modal reasoning (Optical-SAR fusion) through natural language.

### 7.2 Complexity
- **Assessment:** High. The integration of large language models, vision-language models, geospatial processing libraries (GDAL/rasterio), asynchronous GPU task queues (Celery/Redis), and a responsive map-based frontend constitutes a highly complex, distributed architecture.

### 7.3 Clarity and Details
- **Assessment:** Excellent. Extensive documentation (e.g., this strategy document, architecture specification, API contracts) is maintained. The execution trace feature explicitly demonstrates clarity by providing users with an auditable log of how the AI arrived at its conclusion.

### 7.4 Feasibility
- **Assessment:** High. The technology stack relies on mature, battle-tested frameworks (FastAPI, PyTorch, LangGraph, Next.js). Pre-trained foundations models (GeoChat, Grounding-DINO) are available and can be fine-tuned via parameter-efficient methods (QLoRA) within the hackathon timeframe.

### 7.5 Practicability
- **Assessment:** High. The system directly addresses the bottleneck of manual satellite imagery analysis. It allows non-technical stakeholders (e.g., disaster response managers, policymakers) to query geospatial data effortlessly, reducing the time-to-insight from days to seconds.

### 7.6 Sustainability
- **Assessment:** High. The modular agentic design means the system can be sustained long-term. New, more advanced models can be swapped into the Model Registry without rewriting the orchestration logic. Open-source deployment configurations (Docker Compose) ensure portability.

### 7.7 Scale of Impact
- **Assessment:** Massive. Can be deployed across ISRO, the National Remote Sensing Centre (NRSC), State Disaster Management Authorities, and agricultural sectors for near real-time monitoring and decision support.

### 7.8 User Experience
- **Assessment:** Excellent. The interactive Next.js interface mimics standard consumer chat applications but is augmented with Leaflet-based geospatial overlays. Users receive both textual answers and visual evidence (bounding boxes, masks, change maps) in a cohesive, intuitive view.

### 7.9 Potential for Future Work
- **Assessment:** High. The roadmap includes integration with real-time satellite telemetry APIs, incorporation of hyperspectral imagery analysis, and the deployment of federated learning architectures for edge inference on orbital platforms.

---

## 8. EVALUATION RESULTS TEMPLATE

*This section is intended to be populated during the active development and testing phases of the hackathon.*

### 8.1 Benchmark Results

| Benchmark | Metric | Target | Actual Score | Date Evaluated | Notes |
|-----------|--------|--------|--------------|----------------|-------|
| VRSBench | BLEU-4 | > 0.150 | [ ] | [ ] | |
| VRSBench | CIDEr | > 0.850 | [ ] | [ ] | |
| VRSBench | Grounding Acc@0.5 | > 65.0%| [ ] | [ ] | |
| VRSBench | VQA OA | > 70.0%| [ ] | [ ] | |
| RSVQA-LR | OA | > 82.0%| [ ] | [ ] | |
| RSVQA-HR | OA | > 85.0%| [ ] | [ ] | |
| CDVQA | OA | > 75.0%| [ ] | [ ] | |
| BigEarthNet| Multi-label F1 | > 0.80 | [ ] | [ ] | |

### 8.2 System Integration Benchmarks

| Metric | Target | Actual | Date Evaluated | Notes |
|--------|--------|--------|----------------|-------|
| Task Routing Accuracy | > 95% | [ ] | [ ] | |
| Input Validation True Positive Rate| 100% | [ ] | [ ] | |
| VQA Latency (T4 GPU) | < 4.0s | [ ] | [ ] | |
| Grounding Latency (T4 GPU) | < 8.0s | [ ] | [ ] | |
| Edge Case Survival Rate | 100% | [ ] | [ ] | |

### 8.3 ISRO/SAC Hidden Set Qualitative Feedback

| Data Modality | Task | Success (Y/N) | Feedback / Adjustments Needed |
|---------------|------|---------------|-------------------------------|
| Cartosat-2S | VQA | [ ] | |
| RISAT SAR | VQA | [ ] | |
| Cartosat + RISAT | Cross-Modal Analysis | [ ] | |

---
*End of Document*
