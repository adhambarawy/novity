# Novity Data Pull Strategy

## Overview

Novity integrates with customer data systems to pull operational data from existing historians and SCADA systems. The approach is designed to minimize friction—customers start with data they already collect, then optionally add sensors for deeper diagnostics.

## Data Integration Architecture

### Primary Data Source: Historian Systems

Novity's primary integration point is the **PI System** (OSIsoft) or equivalent SCADA/historian platforms that customers already use.

**Base Tier Data (Day-One Integration)**
- PI / time-series process data tags
- Pressures, temperatures, flows, speeds
- Operating mode and configuration tags
- Data already collected by existing SCADA/historian systems
- **No new infrastructure required**

### Data Flow

```
Customer Historian (PI/SCADA)
    ↓
Novity Data Connector
    ↓
Data Preparation & Normalization
    ↓
TruPrognostics AI Models
    ↓
Diagnostic Output & Alerts
    ↓
Novity Platform Dashboard
```

## Three-Tier Data Strategy

### Tier 1: Base (Historian Data Only)
- **Data Source**: Existing PI/SCADA tags
- **Integration**: Direct connection to customer historian
- **Frequency**: Continuous time-series data pull
- **Capabilities**: Process-domain diagnostics (efficiency, compression, bearing conditions)
- **Setup**: Minimal—identify relevant tags, configure connection
- **Time to Value**: Immediate (day-one value)

### Tier 2: Plus (Base + Vibration)
- **Data Source**: Base data + 1 high-frequency vibration sensor
- **Integration**: Vibration sensor → data acquisition system → historian or direct to Novity
- **Frequency**: High-frequency waveform data (raw vibration signals)
- **Capabilities**: Precise mechanical diagnostics, valve leakage, oil whip detection
- **Setup**: Add vibration sensor to critical machine, configure data stream
- **Incremental Cost**: Sensor + installation

### Tier 3: Premium (Multiple Sensors)
- **Data Source**: Multiple high-frequency sensors and modalities
- **Sensor Types**:
  - Vibration sensors (multiple locations)
  - High-frequency cylinder pressure transducers
  - Motor current sensor data
  - Crosshead/piston rod sensors
- **Capabilities**: Maximum precision—individual valve identification, crosshead wear, rider band wear
- **Setup**: Deploy multiple sensors on critical assets
- **Incremental Cost**: Multiple sensors + installation

## Data Integration Methods

### 1. Historian Connection (Primary)
- **Protocol**: OPC UA, REST API, or native PI connectors
- **Data Type**: Time-series process data
- **Frequency**: Continuous or periodic polling
- **Security**: Encrypted connections, authentication via customer credentials
- **Latency**: Near real-time (minutes to hours depending on historian configuration)

### 2. Waveform Data (Plus/Premium)
- **Source**: Vibration sensors, pressure transducers, current sensors
- **Format**: Raw waveform data (time-domain signals)
- **Transmission**: Direct to Novity platform or via customer data gateway
- **Frequency**: High-frequency sampling (kHz range)
- **Storage**: Compressed and archived for historical analysis

### 3. Configuration & Metadata
- **Equipment Details**: Asset type, model, serial number, OEM manual references
- **Operating Context**: Normal operating ranges, maintenance history, prior faults
- **Sensor Mapping**: Which tags/sensors correspond to which equipment
- **Calibration Data**: Sensor calibration factors, baseline measurements

## Data Preparation & Normalization

Once data reaches Novity:

1. **Tag Mapping**: Align customer tags to Novity's standardized data model
2. **Validation**: Check data quality, detect missing values, identify outliers
3. **Normalization**: Convert units, align timestamps, handle gaps
4. **Contextualization**: Combine with equipment metadata and operating modes
5. **Feature Engineering**: Compute derived metrics (efficiency, ratios, trends)

## Security & Privacy

- **Encryption**: HTTPS/TLS for data in transit
- **Authentication**: Customer credentials or API tokens
- **Data Isolation**: Customer data segregated in multi-tenant platform
- **Compliance**: Enterprise-grade security and performance
- **Retention**: Data retained per customer agreement

## Proof of Value (PoV) Process

Novity's go-to-market approach for new customers:

1. **30-Minute Scoping Call**
   - Identify critical assets
   - Determine available data sources
   - Assess data quality and completeness
   - Define success metrics

2. **Data Access Setup**
   - Customer provides read-only access to historian
   - Novity configures data connector
   - Historical data pulled for analysis

3. **Proof of Value Execution**
   - Novity runs TruPrognostics AI on customer's historical data
   - Generates baseline diagnostics and RUL forecasts
   - Validates against known maintenance events
   - Demonstrates lead time and accuracy

4. **Results & Decision**
   - Present findings to customer reliability team
   - Discuss upgrade path (Base → Plus → Premium)
   - Negotiate deployment and support terms

## Key Advantages of This Approach

| Advantage | Benefit |
|-----------|---------|
| **No New Infrastructure** | Uses existing historian data; no new software deployment |
| **Day-One Value** | Base tier provides immediate insights from existing data |
| **Scalable Instrumentation** | Add sensors incrementally as ROI justifies |
| **Low Barrier to Entry** | Proof of value on historical data before committing |
| **Flexible Deployment** | Start with one asset, expand to fleet |
| **Minimal IT Burden** | Simple data connector, no complex integrations |

## Data Governance

- **Ownership**: Customer retains ownership of all data
- **Access**: Novity accesses only what's necessary for diagnostics
- **Audit Trail**: All data access logged and auditable
- **Compliance**: Supports customer data governance policies
- **Termination**: Data deletion upon contract end (per agreement)

## Integration Timeline

| Phase | Duration | Activity |
|-------|----------|----------|
| **Scoping** | 1-2 weeks | Identify data sources, assess readiness |
| **Setup** | 1-2 weeks | Configure historian connection, validate data flow |
| **PoV** | 2-4 weeks | Run diagnostics on historical data, validate results |
| **Deployment** | 1-2 weeks | Go live with real-time monitoring |
| **Optimization** | Ongoing | Tune models, add sensors, expand asset coverage |

## Summary

Novity's data pull strategy is **customer-centric and low-friction**:
- Starts with data customers already have (historian)
- Minimal setup and IT involvement
- Immediate value demonstration via proof of value
- Scalable path to deeper diagnostics (add sensors as needed)
- Enterprise-grade security and data governance

This approach reduces adoption friction and allows customers to validate ROI before committing to additional instrumentation.
