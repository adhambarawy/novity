# Novity Cloud Platform - Overview

## What is the Novity Cloud Platform?

The Novity Cloud Platform is a **SaaS (Software-as-a-Service) application** that receives data from customer historians, runs TruPrognostics AI diagnostics, and delivers actionable maintenance insights via a web-based dashboard.

It's the central hub where:
- Customer data flows in (from historian connectors)
- AI models process the data
- Diagnostics and alerts are generated
- Engineers access results and make decisions

## Architecture Overview

```
Customer Environment          Novity Cloud Platform
─────────────────────────────────────────────────────────

Historian (PI/SCADA)
    ↓
[Data Connector]
    ↓
[Encrypted HTTPS]
    ↓
                          ┌─────────────────────────────┐
                          │  Data Ingestion Layer       │
                          │  - Receive data streams     │
                          │  - Validate & normalize     │
                          │  - Store in time-series DB  │
                          └─────────────────────────────┘
                                    ↓
                          ┌─────────────────────────────┐
                          │  AI Processing Layer        │
                          │  - TruPrognostics models    │
                          │  - Physics-based analysis   │
                          │  - ML fault detection       │
                          │  - RUL forecasting          │
                          └─────────────────────────────┘
                                    ↓
                          ┌─────────────────────────────┐
                          │  Results Storage            │
                          │  - Diagnostics database     │
                          │  - Alert history            │
                          │  - Trending data            │
                          └─────────────────────────────┘
                                    ↓
                          ┌─────────────────────────────┐
                          │  Web Dashboard              │
                          │  - Real-time asset health   │
                          │  - RUL estimates            │
                          │  - Alerts & notifications   │
                          │  - Reports & analytics      │
                          └─────────────────────────────┘
                                    ↓
                          [HTTPS API]
                                    ↓
Engineer Workstations
    ↓
[Web Browser]
    ↓
Novity Dashboard
```

## Core Components

### 1. Data Ingestion Layer

**Purpose:** Receive and prepare data from customer historians

**Functions:**
- Accepts data from OPC UA, REST API, or PI SDK connectors
- Validates data quality (checks for missing values, outliers)
- Normalizes units and timestamps
- Stores raw data in time-series database
- Handles data buffering and batching

**Technology:**
- Message queue (Kafka, RabbitMQ, or similar)
- Time-series database (InfluxDB, TimescaleDB, or similar)
- Data validation pipeline

**Capacity:**
- Handles thousands of tags per customer
- Processes millions of data points per day
- Supports real-time and batch ingestion

### 2. AI Processing Layer

**Purpose:** Run TruPrognostics AI models on incoming data

**Functions:**
- Loads pre-built AI models for each equipment type
- Processes normalized data through physics-based models
- Detects deviations from expected behavior
- Runs machine learning fault classifiers
- Generates RUL forecasts
- Produces maintenance recommendations

**Models Included:**
- Reciprocating compressor diagnostics
- Centrifugal compressor diagnostics
- Pump diagnostics
- Motor diagnostics
- Fan/blower diagnostics
- Heat exchanger diagnostics

**Processing:**
- Runs continuously (real-time or near real-time)
- Processes data in batches (e.g., every 15 minutes)
- Caches results for fast dashboard access
- Logs all diagnostic reasoning for audit trail

### 3. Results Storage

**Purpose:** Store diagnostics, alerts, and historical data

**Functions:**
- Stores diagnostic results (fault names, confidence scores, RUL windows)
- Maintains alert history (when alerts were triggered, acknowledged, resolved)
- Tracks trending data (efficiency over time, bearing temperature trends)
- Stores maintenance recommendations and their sources
- Maintains audit logs (who accessed what, when)

**Data Stored:**
- Per-asset health status
- Fault event history
- RUL forecasts and updates
- Alert events and acknowledgments
- User actions and access logs
- Configuration and metadata

### 4. Web Dashboard

**Purpose:** Present insights to reliability engineers

**Features:**

**Fleet Overview**
- Quick view of all assets and their health status
- Color-coded health indicators (green/yellow/red)
- Sorting by RUL, asset type, location
- Filter by asset class, site, or health status

**Per-Asset Detailed View**
- Asset name, model, serial number, location
- Current health score and RUL estimate
- Trending graph showing RUL over time
- Current fault diagnostics (if any)
- Past events and maintenance history
- Recommended next action

**Alerts & Notifications**
- Real-time alerts when new faults detected
- Email and SMS notifications (configurable)
- Alert acknowledgment and resolution tracking
- Alert history and trends

**Reporting**
- Custom report builder
- Summary reports (fleet health, upcoming maintenance)
- Detailed reports (per-asset diagnostics, RUL forecasts)
- Export to PDF, CSV, Excel
- Scheduled report delivery

**Analytics**
- Trending analysis (efficiency loss over time)
- Fault frequency analysis
- Lead time metrics (how early faults detected)
- Maintenance cost savings analysis

## Deployment Model

### Multi-Tenant SaaS

**Architecture:**
- Single cloud instance serving multiple customers
- Data isolation via customer tenant ID
- Shared infrastructure (compute, storage, networking)
- Separate databases per customer (logical isolation)

**Hosting:**
- Likely hosted on AWS, Azure, or Google Cloud
- Distributed across multiple availability zones (high availability)
- Auto-scaling to handle variable load
- CDN for dashboard performance

**Security:**
- Encryption at rest (database encryption)
- Encryption in transit (HTTPS/TLS)
- Role-based access control (RBAC)
- Multi-factor authentication (MFA) support
- Audit logging of all access

## Data Flow Example

**Scenario:** Compressor C-1402 develops a suction valve leak

**Timeline:**

```
T=0:00   Historian records pressure drop on suction side
         Data connector polls historian
         
T=0:05   Data arrives at Novity cloud platform
         Ingestion layer validates and normalizes
         Stores in time-series database
         
T=0:10   AI processing layer runs diagnostics
         Physics model detects pressure anomaly
         ML classifier identifies suction valve leak pattern
         RUL forecast: 12-20 days to failure
         Recommendation: "Inspect suction valves with ultrasound"
         
T=0:15   Results stored in database
         Alert triggered (new fault detected)
         
T=0:16   Dashboard updates in real-time
         Engineer sees alert notification
         Clicks on C-1402 to view details
         
T=0:17   Engineer sees:
         - Fault: Suction valve leak (confidence: 87%)
         - RUL: 12-20 days
         - Recommendation: Inspect with ultrasound, vibration, PV analysis
         - Source: Ariel Compressor Operating Manual, Section 4.2
         
T=0:20   Engineer schedules inspection for next week
         Acknowledges alert in dashboard
         No emergency callout needed
```

## Key Capabilities

| Capability | Description |
|------------|-------------|
| **Real-Time Monitoring** | Continuous data ingestion and processing |
| **Multi-Tenant Isolation** | Secure data separation between customers |
| **Scalability** | Handles thousands of assets and millions of data points |
| **High Availability** | Redundant infrastructure, automatic failover |
| **API Access** | REST API for integration with customer systems |
| **Mobile Access** | Responsive dashboard works on phones/tablets |
| **Audit Trail** | Complete logging of all access and actions |
| **Compliance** | Supports SOC 2, ISO 27001, GDPR requirements |

## Integration Points

### Inbound Integrations
- **Historian Systems**: PI, Wonderware, GE DigitalWorks, etc.
- **Sensor Data**: Vibration sensors, pressure transducers, current sensors
- **Configuration**: OEM manuals, SOPs, equipment metadata

### Outbound Integrations
- **Email/SMS**: Alert notifications
- **CMMS/EAM**: Maintenance management systems (Maximo, SAP, etc.)
- **REST API**: Custom integrations
- **Webhooks**: Event-driven integrations

## Performance Characteristics

| Metric | Typical Value |
|--------|---------------|
| **Data Latency** | 5-15 minutes (from historian to dashboard) |
| **Diagnostic Latency** | <1 minute (from data arrival to results) |
| **Dashboard Load Time** | <2 seconds |
| **Alert Notification** | <5 minutes from fault detection |
| **Uptime SLA** | 99.9% (typical for SaaS) |
| **Data Retention** | 2-5 years (configurable) |

## Scalability

**Per Customer:**
- Up to 1,000+ assets
- Up to 10,000+ tags
- Millions of data points per day

**Platform:**
- Hundreds of customers
- Billions of data points per day
- Auto-scaling infrastructure

## Security & Compliance

**Data Protection:**
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.2+)
- Regular security audits
- Penetration testing

**Access Control:**
- Role-based access control (RBAC)
- Multi-factor authentication (MFA)
- Single sign-on (SSO) support
- API key management

**Compliance:**
- SOC 2 Type II certified
- GDPR compliant
- HIPAA compatible (if needed)
- Data residency options (US, EU, etc.)

## Summary

**Novity Cloud Platform is:**
- A SaaS application for predictive maintenance
- Receives data from customer historians
- Runs TruPrognostics AI diagnostics
- Stores results and alerts
- Provides web dashboard for engineers
- Scales to thousands of assets
- Enterprise-grade security and reliability

**Key Value:**
- No on-premises software to install
- Automatic updates and maintenance
- Secure, encrypted data handling
- Real-time insights and alerts
- Accessible from anywhere via web browser
