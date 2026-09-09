# Non-Functional Requirements Specification

## 1. DOCUMENT CONTROL

| Document Name | Non-Functional Requirements Specification |
| :--- | :--- |
| **Project Name** | SatQuery AI |
| **Problem Statement ID** | PS ID 26167 |
| **Version** | 1.0 |
| **Date** | 2026-09-09 |
| **Status** | Final |

## 2. PERFORMANCE REQUIREMENTS

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-PER-001 | Inference Latency (VQA/Captioning) | < 5 seconds per query (using NVIDIA T4/A100). | Automated load testing measuring end-to-end response time. |
| NFR-PER-002 | Inference Latency (Grounding/Segmentation) | < 10 seconds for bounding box + mask generation. | System timing logs during evaluation phase. |
| NFR-PER-003 | Concurrent Users | Support a minimum of 10 concurrent inference requests without degradation. | Simulating 10 simultaneous API requests via load testing tools. |
| NFR-PER-004 | UI Responsiveness | The Next.js frontend shall render state changes in < 200ms. | Browser developer tools profiling. |

## 3. SCALABILITY REQUIREMENTS

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-SCA-001 | Async Queue Processing | The system shall utilize Celery+Redis to queue inference tasks exceeding immediate GPU capacity. | Submitting 50+ burst requests and verifying zero dropped tasks. |
| NFR-SCA-002 | Microservice Scalability | Worker nodes (Docker containers) running inference must be horizontally scalable. | Adding worker replicas in Docker Compose and measuring throughput increase. |

## 4. RELIABILITY AND AVAILABILITY

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-REL-001 | Graceful Degradation | The system shall fall back to generic VQA if specialized tools fail or input is partial. | Simulating tool failure and confirming alternative response generation. |
| NFR-REL-002 | Uptime | The system shall maintain 99.9% uptime during the evaluation/demonstration period. | System monitoring logs. |
| NFR-REL-003 | Error Handling | Unhandled backend exceptions shall not crash the frontend application. | Injecting faults into FastAPI endpoints and verifying UI stability. |

## 5. USABILITY REQUIREMENTS

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-USA-001 | Auditability (Execution Trace) | 100% of queries must produce an observable execution trace (tools used, decisions). | Manual inspection of the response interface. |
| NFR-USA-002 | Responsive Design | The UI must function correctly on standard desktop resolutions (1080p) and tablet displays. | Manual testing across different screen sizes. |
| NFR-USA-003 | Feedback Mechanisms | The system shall clearly indicate processing states (loading spinners, progress bars for long tasks). | User acceptance testing. |

## 6. SECURITY REQUIREMENTS

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-SEC-001 | Input Validation | All uploaded files must be verified for structural integrity (no malicious payloads disguised as TIFFs). | Security scanning of uploaded artifacts. |
| NFR-SEC-002 | Path Traversal Prevention | The system must prevent path traversal attacks when accessing saved artifacts. | Penetration testing of file access endpoints. |

## 7. MAINTAINABILITY AND EXTENSIBILITY

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-MNT-001 | Modular Tool Registry | Adding a new specialized ML model shall require zero changes to the core LangGraph orchestration logic. | Code review of the tool registration interface. |
| NFR-MNT-002 | Code Documentation | 100% of core backend functions and classes shall contain docstrings. | Static analysis tools. |

## 8. PORTABILITY AND DEPLOYMENT

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-DEP-001 | Containerization | The entire stack (Frontend, Backend, Database, Redis, Workers) must be deployable via a single `docker-compose up` command. | Fresh deployment test on a clean host machine. |
| NFR-DEP-002 | Hardware Agnosticism | The system must detect available GPUs (via NVIDIA Container Toolkit) or fall back to CPU (with warning) without failing to start. | Deployment testing on CPU-only and GPU-enabled hosts. |

## 9. DATA REQUIREMENTS

| ID | Description | Measurable Target | Verification Method |
| :--- | :--- | :--- | :--- |
| NFR-DAT-001 | Storage Management | Temporary inference artifacts and user uploads must be stored in MinIO/local filesystem and tracked via PostgreSQL. | Database schema review and storage inspection. |
| NFR-DAT-002 | Spatial Data Handling | Geospatial metadata shall be managed using PostGIS extensions for potential future spatial querying. | Database schema validation ensuring PostGIS usage. |
