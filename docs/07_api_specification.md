# API Specification

## 1. DOCUMENT CONTROL

**System Name:** SatQuery AI  
**Problem Statement:** PS ID 26167 — SatQuery AI - An Interactive Vision-Language Assistant for Multimodal Remote Sensing Image Analysis through Text Queries  
**Organization:** Indian Space Research Organisation (ISRO), Department of Space  
**Document Status:** Final Draft  
**Target Audience:** Frontend Developers, Integration Specialists, Backend Engineers  

---

## 2. API OVERVIEW

The SatQuery AI application exposes a robust, RESTful API layer built with FastAPI (Python 3.11+). This API serves as the primary interface for frontend applications and authorized third-party consumers to interact with the underlying multimodal remote sensing intelligence systems.

### 2.1 Base URL
All API requests must be prefixed with the designated base URL. For production deployments, this will typically follow the pattern:
`https://api.satquery.isro.gov.in/api/v1`

### 2.2 Versioning Strategy
The API utilizes URI-based versioning to ensure backward compatibility and smooth transitions during future system upgrades. The current active version is `v1`. Deprecation of older versions will be communicated with a minimum of six months lead time.

### 2.3 Authentication
Authentication is managed via HTTP Bearer tokens (JSON Web Tokens) or designated API keys for machine-to-machine communication. 
- **Header format:** `Authorization: Bearer <token_or_api_key>`
- Tokens are issued upon successful user login via a separate Identity Provider (e.g., Keycloak or ISRO internal IAM).
- Unauthenticated requests will return a `401 Unauthorized` status code.

### 2.4 Rate Limiting
To ensure equitable resource distribution and protect backend GPU infrastructure, rate limits are enforced on a per-user and per-organization basis.
- **Image Uploads:** 100 requests per hour.
- **Query Processing:** 50 requests per hour (due to heavy GPU consumption).
- **Metadata Retrieval:** 1000 requests per hour.
Headers included in responses:
- `X-RateLimit-Limit`: Maximum requests allowed in the time window.
- `X-RateLimit-Remaining`: Remaining requests in the current window.
- `X-RateLimit-Reset`: Unix timestamp when the limit resets.

---

## 3. ENDPOINT SPECIFICATIONS

### 3.1 Image Management

#### POST /api/v1/images/upload
Uploads a multimodal image file (optical, multispectral, or SAR) to the secure storage environment. Supported formats include GeoTIFF, TIFF, PNG, and JPEG.

- **Method:** POST
- **Path:** `/api/v1/images/upload`
- **Description:** Accepts multipart form data. Initiates asynchronous extraction of spatial metadata (CRS, bounds, resolution).
- **Request Body (Multipart Form):**
  - `file`: The image file binary.
  - `metadata`: (Optional) JSON string detailing instrument type, acquisition date, etc.
- **Response Schema:**
  ```json
  {
    "image_id": "img_f7a9b8c2d1e0",
    "status": "PROCESSING",
    "message": "Upload successful. Metadata extraction initiated."
  }
  ```
- **Status Codes:**
  - `202 Accepted`: File accepted for processing.
  - `400 Bad Request`: Invalid file format or missing required fields.
  - `413 Payload Too Large`: File size exceeds the maximum allowed (e.g., 2GB).

#### GET /api/v1/images/{image_id}
Retrieves metadata and basic properties for a given image identifier.

- **Method:** GET
- **Path:** `/api/v1/images/{image_id}`
- **Description:** Returns the Extracted spatial metadata, instrument parameters, and processing status.
- **Response Schema:**
  ```json
  {
    "image_id": "img_f7a9b8c2d1e0",
    "filename": "sentinel2_mumbai_2024.tif",
    "crs": "EPSG:32643",
    "dimensions": [4096, 4096],
    "bands": 13,
    "upload_time": "2024-09-09T14:30:00Z",
    "status": "READY"
  }
  ```
- **Status Codes:**
  - `200 OK`: Request successful.
  - `404 Not Found`: Image ID does not exist.

#### GET /api/v1/images/{image_id}/preview
Fetches a dynamically rendered RGB preview of the image for web rendering.

- **Method:** GET
- **Path:** `/api/v1/images/{image_id}/preview`
- **Description:** For multispectral images, a false-color or true-color RGB composite is rendered on the fly. For SAR, a grayscale visualization with speckle filtering is applied.
- **Response Type:** `image/jpeg` or `image/png`
- **Status Codes:**
  - `200 OK`: Image returned successfully.
  - `404 Not Found`: Image ID does not exist.

#### DELETE /api/v1/images/{image_id}
Permanently removes the image and its associated metadata from the system.

- **Method:** DELETE
- **Path:** `/api/v1/images/{image_id}`
- **Description:** Deletes the physical file from the object store and metadata from the PostgreSQL database.
- **Status Codes:**
  - `204 No Content`: Deletion successful.
  - `404 Not Found`: Image ID does not exist.

#### POST /api/v1/images/validate-pair
Validates two images to ensure they are properly co-registered for bi-temporal or cross-modal analysis.

- **Method:** POST
- **Path:** `/api/v1/images/validate-pair`
- **Description:** Checks CRS compatibility, bounding box intersection, and spatial resolution parity.
- **Request Body Schema:**
  ```json
  {
    "image_id_1": "img_f7a9b8c2d1e0",
    "image_id_2": "img_a1b2c3d4e5f6"
  }
  ```
- **Response Schema:**
  ```json
  {
    "is_valid": true,
    "intersection_percentage": 98.5,
    "crs_match": true,
    "resolution_match": true,
    "message": "Images are highly co-registered and suitable for bi-temporal analysis."
  }
  ```
- **Status Codes:**
  - `200 OK`: Validation completed.

### 3.2 Query Processing

#### POST /api/v1/query
Submits a natural language query directed at one or more images.

- **Method:** POST
- **Path:** `/api/v1/query`
- **Description:** This endpoint parses the text query, instantiates the agentic LangGraph workflow, determines task classification, and enqueues tasks in Celery.
- **Request Body Schema:**
  ```json
  {
    "query": "Identify new residential developments in this area.",
    "image_ids": ["img_f7a9b8c2d1e0", "img_a1b2c3d4e5f6"],
    "preferences": {
      "focus_area": "URBAN",
      "sensitivity": "HIGH"
    }
  }
  ```
- **Response Schema:**
  ```json
  {
    "query_id": "qry_88bb99aa77cc",
    "status": "QUEUED",
    "estimated_time_seconds": 45,
    "websocket_url": "wss://api.satquery.isro.gov.in/ws/v1/query/qry_88bb99aa77cc/stream"
  }
  ```
- **Status Codes:**
  - `202 Accepted`: Query enqueued for processing.
  - `400 Bad Request`: Invalid image references or malformed request.

#### GET /api/v1/query/{query_id}/status
Polls the execution status of an ongoing query.

- **Method:** GET
- **Path:** `/api/v1/query/{query_id}/status`
- **Description:** Returns the current state (QUEUED, PROCESSING, COMPLETED, FAILED) and progress percentage.
- **Response Schema:**
  ```json
  {
    "query_id": "qry_88bb99aa77cc",
    "status": "PROCESSING",
    "progress": 65,
    "current_step": "Executing Bi-Temporal Change Detection Model"
  }
  ```
- **Status Codes:**
  - `200 OK`: Status retrieved successfully.
  - `404 Not Found`: Query ID does not exist.

#### GET /api/v1/query/{query_id}/result
Retrieves the final synthesized output for a completed query.

- **Method:** GET
- **Path:** `/api/v1/query/{query_id}/result`
- **Description:** Returns the natural language response, bounding box coordinates, mask references, and overall confidence score.
- **Status Codes:**
  - `200 OK`: Result retrieved successfully.
  - `404 Not Found`: Query ID does not exist.
  - `409 Conflict`: Query is still processing.

#### WebSocket /ws/v1/query/{query_id}/stream
Provides a real-time bidirectional stream of execution progress and intermediate tool outputs.

- **Protocol:** WebSocket
- **Path:** `/ws/v1/query/{query_id}/stream`
- **Description:** Subscribes the client to updates from the agentic orchestration layer, passing step-by-step trace information.

### 3.3 Results and Reports

#### GET /api/v1/results/{result_id}
Fetches a detailed analysis result object.

- **Method:** GET
- **Path:** `/api/v1/results/{result_id}`
- **Description:** Used to retrieve historical results. Similar payload to `/query/{query_id}/result`.

#### GET /api/v1/results/{result_id}/trace
Retrieves the full execution audit trail for explainability.

- **Method:** GET
- **Path:** `/api/v1/results/{result_id}/trace`
- **Description:** Returns a detailed log of the Agent graph execution, showing intent classification, tool selection, raw tool outputs, and confidence intervals.
- **Response Schema:**
  ```json
  {
    "trace_id": "trc_55443322",
    "steps": [
      {
        "step_order": 1,
        "action": "Query Intent Classification",
        "model": "LLM-Classifier-v2",
        "output": "CHANGE_DETECTION",
        "latency_ms": 120
      },
      {
        "step_order": 2,
        "action": "Change Detection Execution",
        "model": "BIT-ChangeFormer",
        "output": "Mask generated with 45 changed regions.",
        "latency_ms": 4500
      }
    ]
  }
  ```
- **Status Codes:**
  - `200 OK`: Trace retrieved successfully.

#### GET /api/v1/results/{result_id}/report
Generates a downloadable document summarizing the analysis.

- **Method:** GET
- **Path:** `/api/v1/results/{result_id}/report`
- **Query Parameters:** `format` (enum: pdf, html)
- **Description:** Returns a strictly formatted ISRO-branded report encompassing the query, imagery metadata, methodology trace, generated maps, and textual conclusions.
- **Response Type:** `application/pdf` or `text/html`
- **Status Codes:**
  - `200 OK`: Report generated and returned.

#### GET /api/v1/results/{result_id}/overlays/{overlay_id}
Retrieves a specific spatial overlay image (e.g., change map, segmentation mask) generated during the analysis.

- **Method:** GET
- **Path:** `/api/v1/results/{result_id}/overlays/{overlay_id}`
- **Description:** Returns PNG images with transparent backgrounds designed for rendering over base maps via Leaflet.
- **Status Codes:**
  - `200 OK`: Image returned successfully.

### 3.4 Model Registry (Admin)

#### GET /api/v1/models
Lists all specialized models registered in the orchestration layer.

- **Method:** GET
- **Path:** `/api/v1/models`
- **Description:** Returns information on available models, their domain (e.g., Optical, SAR, Multi-modal), and versioning.

#### GET /api/v1/models/{model_id}
Retrieves detailed metadata for a specific model.

- **Method:** GET
- **Path:** `/api/v1/models/{model_id}`
- **Description:** Details input requirements, memory footprint, hardware target (T4/A100), and performance benchmarks on standard datasets (e.g., RSVQA, BigEarthNet).

#### GET /api/v1/health
System health monitoring endpoint.

- **Method:** GET
- **Path:** `/api/v1/health`
- **Description:** Returns the operational status of the FastAPI server, PostgreSQL database, Redis task queue, Celery workers, and GPU availability.
- **Response Schema:**
  ```json
  {
    "status": "healthy",
    "components": {
      "database": "up",
      "redis": "up",
      "gpu_0": "available",
      "gpu_1": "in_use"
    },
    "timestamp": "2024-09-09T14:45:00Z"
  }
  ```
- **Status Codes:**
  - `200 OK`: System healthy.
  - `503 Service Unavailable`: Critical component failure.

---

## 4. DATA MODELS / SCHEMAS

### ImageMetadata Schema
```json
{
  "type": "object",
  "properties": {
    "image_id": { "type": "string" },
    "filename": { "type": "string" },
    "sensor_type": { "type": "string", "enum": ["OPTICAL", "SAR", "MULTISPECTRAL"] },
    "crs": { "type": "string" },
    "bounds": {
      "type": "array",
      "items": { "type": "number" },
      "minItems": 4,
      "maxItems": 4
    },
    "resolution_meters": { "type": "number" },
    "acquisition_date": { "type": "string", "format": "date-time" }
  },
  "required": ["image_id", "filename", "sensor_type"]
}
```

### QueryRequest Schema
```json
{
  "type": "object",
  "properties": {
    "query": { "type": "string" },
    "image_ids": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 1
    },
    "preferences": { "type": "object" }
  },
  "required": ["query", "image_ids"]
}
```

### QueryResult Schema
```json
{
  "type": "object",
  "properties": {
    "result_id": { "type": "string" },
    "query_id": { "type": "string" },
    "answer_text": { "type": "string" },
    "confidence_score": { "type": "number", "minimum": 0, "maximum": 1 },
    "overlays": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "overlay_id": { "type": "string" },
          "type": { "type": "string", "enum": ["BBOX", "MASK", "HEATMAP"] },
          "description": { "type": "string" }
        }
      }
    }
  },
  "required": ["result_id", "query_id", "answer_text", "confidence_score"]
}
```

### ExecutionTrace Schema
```json
{
  "type": "object",
  "properties": {
    "trace_id": { "type": "string" },
    "steps": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "step_order": { "type": "integer" },
          "action": { "type": "string" },
          "tool_name": { "type": "string" },
          "input_parameters": { "type": "object" },
          "output_summary": { "type": "string" },
          "execution_time_ms": { "type": "integer" }
        }
      }
    }
  },
  "required": ["trace_id", "steps"]
}
```

---

## 5. ERROR CODES TABLE

| HTTP Status | Application Code | Description | Corrective Action |
|-------------|------------------|-------------|-------------------|
| 400 | `ERR_BAD_REQUEST` | Malformed request syntax or missing parameters. | Review request body schema. |
| 400 | `ERR_UNSUPPORTED_FILE` | Uploaded image format is not recognized. | Upload GeoTIFF, TIFF, PNG, or JPEG. |
| 401 | `ERR_UNAUTHORIZED` | Invalid or missing authentication token. | Provide a valid JWT Bearer token. |
| 403 | `ERR_FORBIDDEN` | Insufficient permissions for requested resource. | Request elevated access from administrator. |
| 404 | `ERR_NOT_FOUND` | Resource (image, query, result) does not exist. | Verify the identifier. |
| 409 | `ERR_COREGISTRATION` | Bi-temporal images do not spatially overlap. | Upload overlapping image pairs. |
| 413 | `ERR_PAYLOAD_LARGE` | Request payload exceeds configured limits. | Reduce image size or crop extent. |
| 422 | `ERR_UNPROCESSABLE` | Valid request syntax, but unprocessable instructions. | Rephrase natural language query. |
| 429 | `ERR_RATE_LIMIT` | API request quota exceeded. | Wait for the rate limit window to reset. |
| 500 | `ERR_INTERNAL` | Unhandled backend exception or orchestration failure. | Contact system administrator. |
| 503 | `ERR_GPU_UNAVAILABLE`| Insufficient GPU resources for inference task. | Retry request during off-peak hours. |

---

## 6. WEBSOCKET MESSAGE PROTOCOL

The WebSocket endpoint for query streams utilizes JSON payloads formatted as discrete events.

### Client-to-Server Messages
Presently, the connection is read-only from the client perspective, except for connection keep-alive (ping/pong).

### Server-to-Client Events

**1. CONNECTED Event**
```json
{
  "event": "CONNECTED",
  "timestamp": "2024-09-09T14:46:00Z",
  "message": "Stream initialized for query qry_88bb99aa77cc"
}
```

**2. AGENT_STEP_START Event**
```json
{
  "event": "AGENT_STEP_START",
  "step_name": "Task Planning",
  "description": "Analyzing intent to determine required ML tools."
}
```

**3. AGENT_STEP_COMPLETE Event**
```json
{
  "event": "AGENT_STEP_COMPLETE",
  "step_name": "Task Planning",
  "result": "Selected models: [RS-VQA, Visual Grounding Model]",
  "latency_ms": 340
}
```

**4. PROGRESS_UPDATE Event**
```json
{
  "event": "PROGRESS_UPDATE",
  "percentage": 50,
  "message": "Executing Visual Grounding inference on GPU_0..."
}
```

**5. FINAL_RESULT Event**
```json
{
  "event": "FINAL_RESULT",
  "result_id": "res_998877665544",
  "message": "Analysis completed successfully. Retrieve full result payload from REST endpoint."
}
```

**6. ERROR Event**
```json
{
  "event": "ERROR",
  "code": "ERR_GPU_UNAVAILABLE",
  "message": "Model execution failed due to memory exhaustion.",
  "fatal": true
}
```

---
*End of Document*
