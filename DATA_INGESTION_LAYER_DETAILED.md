# Data Ingestion Layer - Detailed Breakdown

## Overview

The Data Ingestion Layer is the entry point for all customer data into Novity's AWS platform. It receives data from customer PI historians, validates it, normalizes it, and prepares it for AI processing.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    CUSTOMER ENVIRONMENT                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  PI Historian (OSIsoft)                                                │
│  ├─ Pressures, temperatures, flows, speeds                            │
│  ├─ Operating modes, configurations                                   │
│  └─ Timestamps and quality flags                                      │
│                                                                          │
│  Novity Data Connector                                                 │
│  ├─ Connection: OPC UA / REST API                                     │
│  ├─ Polling: Every 5-15 minutes                                       │
│  ├─ Batch Size: 100-1000 data points per request                      │
│  └─ Encryption: HTTPS/TLS 1.2+                                        │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
                              ↓
                [HTTPS POST Request with JSON payload]
                              ↓
┌──────────────────────────────────────────────────────────────────────────┐
│                    AWS DATA INGESTION LAYER                              │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ 1. API GATEWAY (Entry Point)                                       │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │                                                                    │ │
│  │  Endpoint: POST /api/v1/data/ingest                              │ │
│  │                                                                    │ │
│  │  Request Format:                                                 │ │
│  │  {                                                               │ │
│  │    "customer_id": "cust-12345",                                 │ │
│  │    "asset_id": "C-1402",                                        │ │
│  │    "data_points": [                                             │ │
│  │      {                                                           │ │
│  │        "tag": "C-1402_DISCH_PRESS",                            │ │
│  │        "value": 450.5,                                          │ │
│  │        "unit": "psi",                                           │ │
│  │        "timestamp": "2026-05-04T18:10:00Z",                    │ │
│  │        "quality": "good"                                        │ │
│  │      },                                                          │ │
│  │      ...                                                         │ │
│  │    ]                                                             │ │
│  │  }                                                               │ │
│  │                                                                    │ │
│  │  Functions:                                                      │ │
│  │  ├─ Receives HTTPS POST requests                                │ │
│  │  ├─ Validates API key / authentication token                    │ │
│  │  ├─ Checks request format (JSON schema validation)              │ │
│  │  ├─ Rate limiting (e.g., 1000 requests/hour per customer)       │ │
│  │  ├─ Throttling (rejects if over limit)                          │ │
│  │  ├─ Logs all requests (CloudWatch)                              │ │
│  │  ├─ Returns 200 OK if accepted                                  │ │
│  │  ├─ Returns 400/401/429 if rejected                             │ │
│  │  └─ Routes to Lambda function                                   │ │
│  │                                                                    │ │
│  │  Security:                                                       │ │
│  │  ├─ SSL/TLS encryption (HTTPS only)                             │ │
│  │  ├─ API key validation                                          │ │
│  │  ├─ JWT token verification                                      │ │
│  │  ├─ CORS policy (only from authorized domains)                  │ │
│  │  ├─ WAF (Web Application Firewall) rules                        │ │
│  │  └─ DDoS protection (AWS Shield)                                │ │
│  │                                                                    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                              ↓                                           │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ 2. LAMBDA - DATA VALIDATION & NORMALIZATION                       │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │                                                                    │ │
│  │  Trigger: API Gateway POST request                              │ │
│  │  Runtime: Python 3.11                                           │ │
│  │  Memory: 1024 MB                                                │ │
│  │  Timeout: 60 seconds                                            │ │
│  │  Concurrency: Auto-scaling (up to 1000 concurrent)              │ │
│  │                                                                    │ │
│  │  Step 1: Parse Request                                          │ │
│  │  ├─ Extract customer_id, asset_id, data_points                 │ │
│  │  ├─ Validate JSON structure                                     │ │
│  │  ├─ Check for required fields                                   │ │
│  │  └─ Log parsing errors                                          │ │
│  │                                                                    │ │
│  │  Step 2: Data Quality Checks                                    │ │
│  │  ├─ Check for missing values (NaN, null)                        │ │
│  │  ├─ Detect outliers (values > 3 std dev)                        │ │
│  │  ├─ Validate timestamps (not in future, not too old)            │ │
│  │  ├─ Check data types (numeric, string, boolean)                 │ │
│  │  ├─ Verify quality flags (good, uncertain, bad)                 │ │
│  │  └─ Flag suspicious data for review                             │ │
│  │                                                                    │ │
│  │  Step 3: Data Normalization                                     │ │
│  │  ├─ Convert units (e.g., bar → psi)                             │ │
│  │  ├─ Standardize timestamps (UTC)                                │ │
│  │  ├─ Round values to appropriate precision                       │ │
│  │  ├─ Handle missing values (interpolation or flagging)           │ │
│  │  ├─ Normalize tag names (lowercase, remove spaces)              │ │
│  │  └─ Apply calibration factors (if needed)                       │ │
│  │                                                                    │ │
│  │  Step 4: Data Enrichment                                        │ │
│  │  ├─ Add customer_id to each data point                          │ │
│  │  ├─ Add asset_id to each data point                             │ │
│  │  ├─ Add asset_type (compressor, pump, motor, etc.)              │ │
│  │  ├─ Add ingestion_timestamp (when received)                     │ │
│  │  ├─ Add data_source (PI, REST API, etc.)                        │ │
│  │  ├─ Add processing_version (for model compatibility)            │ │
│  │  └─ Add unique batch_id for tracking                            │ │
│  │                                                                    │ │
│  │  Step 5: Publish to Message Queue                               │ │
│  │  ├─ Serialize data to JSON                                      │ │
│  │  ├─ Publish to SQS queue                                        │ │
│  │  ├─ Include metadata (customer_id, asset_id, batch_id)          │ │
│  │  ├─ Set message attributes for filtering                        │ │
│  │  └─ Return success/failure status                               │ │
│  │                                                                    │ │
│  │  Error Handling:                                                │ │
│  │  ├─ Validation errors → Log and skip data point                 │ │
│  │  ├─ Parsing errors → Return 400 Bad Request                     │ │
│  │  ├─ Queue errors → Retry with exponential backoff               │ │
│  │  ├─ Timeout errors → Return 504 Gateway Timeout                 │ │
│  │  └─ Unknown errors → Log and alert ops team                     │ │
│  │                                                                    │ │
│  │  Monitoring:                                                    │ │
│  │  ├─ CloudWatch metrics (invocations, duration, errors)          │ │
│  │  ├─ X-Ray tracing (request flow)                                │ │
│  │  ├─ Custom metrics (data quality score)                         │ │
│  │  └─ Alarms (error rate > 5%, latency > 30s)                     │ │
│  │                                                                    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                              ↓                                           │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ 3. SQS - MESSAGE QUEUE (Buffering & Decoupling)                  │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │                                                                    │ │
│  │  Queue Name: novity-data-ingestion-queue                        │ │
│  │  Message Retention: 14 days                                     │ │
│  │  Visibility Timeout: 300 seconds (5 minutes)                    │ │
│  │  Max Message Size: 256 KB                                       │ │
│  │  Delivery Delay: 0 seconds (immediate)                          │ │
│  │                                                                    │ │
│  │  Message Format:                                                │ │
│  │  {                                                               │ │
│  │    "batch_id": "batch-abc123",                                  │ │
│  │    "customer_id": "cust-12345",                                 │ │
│  │    "asset_id": "C-1402",                                        │ │
│  │    "asset_type": "reciprocating_compressor",                    │ │
│  │    "data_points": [                                             │ │
│  │      {                                                           │ │
│  │        "tag": "C-1402_DISCH_PRESS",                            │ │
│  │        "value": 450.5,                                          │ │
│  │        "unit": "psi",                                           │ │
│  │        "timestamp": "2026-05-04T18:10:00Z",                    │ │
│  │        "quality": "good",                                       │ │
│  │        "normalized": true                                       │ │
│  │      },                                                          │ │
│  │      ...                                                         │ │
│  │    ],                                                            │ │
│  │    "ingestion_timestamp": "2026-05-04T18:10:05Z",              │ │
│  │    "processing_version": "v2.1.0"                               │ │
│  │  }                                                               │ │
│  │                                                                    │ │
│  │  Functions:                                                      │ │
│  │  ├─ Buffers incoming data                                       │ │
│  │  ├─ Decouples ingestion from processing                         │ │
│  │  ├─ Ensures no data loss (persistent storage)                   │ │
│  │  ├─ Enables parallel processing (multiple consumers)            │ │
│  │  ├─ Provides backpressure (queue fills if processing slow)      │ │
│  │  ├─ Allows retry logic (visibility timeout)                     │ │
│  │  └─ Scales automatically (no capacity limits)                   │ │
│  │                                                                    │ │
│  │  Scaling:                                                       │ │
│  │  ├─ Typical queue depth: 1,000-10,000 messages                  │ │
│  │  ├─ Peak throughput: 100,000+ messages/minute                   │ │
│  │  ├─ Auto-scaling: Consumers scale based on queue depth          │ │
│  │  └─ Dead Letter Queue: For messages that fail 3 times           │ │
│  │                                                                    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                              ↓                                           │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ 4. STORAGE - IMMEDIATE PERSISTENCE                               │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │                                                                    │ │
│  │  S3 (Raw Data Backup)                                           │ │
│  │  ├─ Bucket: novity-raw-data-backup                              │ │
│  │  ├─ Path: s3://novity-raw-data-backup/YYYY/MM/DD/HH/           │ │
│  │  ├─ Format: JSON (one file per batch)                           │ │
│  │  ├─ Compression: gzip                                           │ │
│  │  ├─ Retention: 90 days (then archive to Glacier)                │ │
│  │  ├─ Encryption: AES-256 (SSE-S3)                                │ │
│  │  └─ Purpose: Audit trail, replay, debugging                     │ │
│  │                                                                    │ │
│  │  CloudWatch Logs                                                │ │
│  │  ├─ Log Group: /aws/lambda/data-ingestion                       │ │
│  │  ├─ Log Stream: Per Lambda invocation                           │ │
│  │  ├─ Retention: 30 days                                          │ │
│  │  ├─ Metrics: Errors, warnings, data quality scores              │ │
│  │  └─ Purpose: Debugging, monitoring, compliance                  │ │
│  │                                                                    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
                              ↓
                    [To Data Processing Layer]
```

## Detailed Component Breakdown

### 1. API Gateway

**Purpose:** Single entry point for all data ingestion requests

**Configuration:**
```
Endpoint: POST https://api.novity.us/api/v1/data/ingest
Protocol: HTTPS (TLS 1.2+)
Authentication: API Key or JWT Token
Rate Limiting: 1000 requests/hour per customer
Throttling: 100 requests/second per customer
```

**Request Validation:**
- Content-Type: application/json
- Body size: Max 1 MB
- Schema validation: JSON schema
- Required fields: customer_id, asset_id, data_points

**Response Codes:**
- 200 OK: Data accepted
- 400 Bad Request: Invalid format
- 401 Unauthorized: Invalid credentials
- 429 Too Many Requests: Rate limit exceeded
- 500 Internal Server Error: Server error

**Security:**
- SSL/TLS encryption
- API key validation (stored in Secrets Manager)
- JWT token verification
- CORS policy enforcement
- WAF rules (SQL injection, XSS protection)
- DDoS protection (AWS Shield)

### 2. Lambda - Data Validation & Normalization

**Purpose:** Process and validate incoming data

**Execution Flow:**

```
1. Receive event from API Gateway
   ↓
2. Parse JSON payload
   ├─ Extract customer_id, asset_id, data_points
   ├─ Validate structure
   └─ Check required fields
   ↓
3. Data Quality Checks
   ├─ Check for missing values (NaN, null)
   ├─ Detect outliers (statistical analysis)
   ├─ Validate timestamps
   ├─ Check data types
   ├─ Verify quality flags
   └─ Flag suspicious data
   ↓
4. Data Normalization
   ├─ Convert units
   ├─ Standardize timestamps (UTC)
   ├─ Round values
   ├─ Handle missing values
   ├─ Normalize tag names
   └─ Apply calibration factors
   ↓
5. Data Enrichment
   ├─ Add customer_id, asset_id
   ├─ Add asset_type
   ├─ Add ingestion_timestamp
   ├─ Add data_source
   ├─ Add processing_version
   └─ Add batch_id
   ↓
6. Publish to SQS
   ├─ Serialize to JSON
   ├─ Publish message
   ├─ Include metadata
   └─ Return success
```

**Data Quality Checks:**

| Check | Method | Action |
|-------|--------|--------|
| Missing Values | Check for NaN, null | Flag or interpolate |
| Outliers | 3-sigma rule | Flag for review |
| Timestamps | Check range | Reject if invalid |
| Data Types | Type validation | Reject if wrong type |
| Quality Flags | Enum validation | Reject if invalid |
| Duplicates | Hash comparison | Deduplicate |

**Normalization Examples:**

```
Input:  {"tag": "C-1402_DISCH_PRESS", "value": 3.1, "unit": "bar"}
Output: {"tag": "c_1402_disch_press", "value": 450.5, "unit": "psi"}

Input:  {"timestamp": "2026-05-04T18:10:00+02:00"}
Output: {"timestamp": "2026-05-04T16:10:00Z"}

Input:  {"value": 450.567}
Output: {"value": 450.57}  (rounded to 2 decimals)
```

**Error Handling:**

```
Validation Error
├─ Log error details
├─ Add to error metrics
├─ Skip data point (don't fail entire batch)
└─ Continue processing

Parsing Error
├─ Log error details
├─ Return 400 Bad Request
└─ Alert customer

Queue Error
├─ Retry with exponential backoff
├─ Max 3 retries
└─ Send to Dead Letter Queue if fails

Timeout Error
├─ Log error
├─ Return 504 Gateway Timeout
└─ Alert ops team
```

### 3. SQS - Message Queue

**Purpose:** Buffer data and decouple ingestion from processing

**Queue Configuration:**
```
Queue Name: novity-data-ingestion-queue
Message Retention: 14 days
Visibility Timeout: 300 seconds (5 minutes)
Max Message Size: 256 KB
Delivery Delay: 0 seconds
Dead Letter Queue: novity-data-ingestion-dlq
```

**Message Structure:**
```json
{
  "batch_id": "batch-abc123",
  "customer_id": "cust-12345",
  "asset_id": "C-1402",
  "asset_type": "reciprocating_compressor",
  "data_points": [
    {
      "tag": "c_1402_disch_press",
      "value": 450.5,
      "unit": "psi",
      "timestamp": "2026-05-04T18:10:00Z",
      "quality": "good",
      "normalized": true
    }
  ],
  "ingestion_timestamp": "2026-05-04T18:10:05Z",
  "processing_version": "v2.1.0"
}
```

**Scaling:**
- Typical queue depth: 1,000-10,000 messages
- Peak throughput: 100,000+ messages/minute
- Auto-scaling: Consumers scale based on queue depth
- Dead Letter Queue: For messages that fail 3 times

### 4. Storage & Logging

**S3 Raw Data Backup:**
```
Bucket: novity-raw-data-backup
Path: s3://novity-raw-data-backup/2026/05/04/18/batch-abc123.json.gz
Compression: gzip
Encryption: AES-256
Retention: 90 days (then archive to Glacier)
Purpose: Audit trail, replay, debugging
```

**CloudWatch Logs:**
```
Log Group: /aws/lambda/data-ingestion
Log Stream: 2026/05/04/[$LATEST]abc123
Retention: 30 days
Metrics: Errors, warnings, data quality scores
Alarms: Error rate > 5%, latency > 30s
```

## Performance Characteristics

| Metric | Value |
|--------|-------|
| **API Latency** | <100ms (p99) |
| **Lambda Duration** | 5-15 seconds |
| **Queue Latency** | <1 second |
| **Throughput** | 100,000+ messages/minute |
| **Availability** | 99.99% |
| **Data Loss** | 0% (persistent queue) |

## Monitoring & Alerting

**CloudWatch Metrics:**
- API Gateway: Requests, latency, errors
- Lambda: Invocations, duration, errors, throttles
- SQS: Messages sent, received, deleted, age
- Custom: Data quality score, validation errors

**Alarms:**
- Lambda error rate > 5%
- Lambda duration > 30 seconds
- SQS queue depth > 10,000
- API Gateway 5xx errors > 1%
- Data quality score < 90%

**Dashboards:**
- Real-time ingestion metrics
- Data quality trends
- Error rates and types
- Queue depth and latency

## Cost Optimization

**Strategies:**
- Batch requests (reduce API calls)
- Compress data (reduce bandwidth)
- Archive old logs (reduce storage)
- Use SQS standard queue (cheaper than FIFO)
- Lambda reserved concurrency (cost predictability)

**Estimated Monthly Cost (per customer):**
- API Gateway: $10-50
- Lambda: $20-100
- SQS: $5-20
- S3: $5-20
- CloudWatch: $5-10
- **Total: $45-200**

## Summary

**Data Ingestion Layer:**
1. **API Gateway** receives HTTPS requests from customer connectors
2. **Lambda** validates, normalizes, and enriches data
3. **SQS** buffers data for processing
4. **S3** stores raw data backup
5. **CloudWatch** logs all activity

**Key Characteristics:**
- Highly scalable (auto-scaling)
- Fault-tolerant (persistent queue, retries)
- Secure (HTTPS, encryption, authentication)
- Observable (comprehensive logging and metrics)
- Cost-efficient (pay-per-use)

**Result:** Clean, validated, normalized data ready for AI processing
