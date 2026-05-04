# Novity Tier 1 Data Pull - How It Works

## What is Tier 1?

Tier 1 (Base) is Novity's entry-level offering that uses **data customers already collect** in their existing historian systems. No new sensors, no new infrastructure—just a data connection.

## Data Source

**PI System (OSIsoft)** or equivalent SCADA/historian platforms

Novity pulls time-series process data tags that are already being logged:
- Pressures (suction, discharge, intermediate stages)
- Temperatures (inlet, outlet, bearing, oil)
- Flows (inlet, outlet, bypass)
- Speeds (RPM, frequency)
- Operating mode and configuration tags

## How Novity Pulls the Data

### Step 1: Connection Setup
Customer provides Novity with **read-only access** to their historian system via:
- **OPC UA** (OLE for Process Control Unified Architecture)
- **REST API** (if historian supports it)
- **Native PI connectors** (for OSIsoft PI systems)

### Step 2: Tag Identification
Novity works with customer to identify which tags correspond to which equipment:
- Example: "C-1402_DISCH_PRESS" → Discharge pressure on Compressor C-1402
- Maps customer's tag naming convention to Novity's data model

### Step 3: Continuous Data Pull
Novity's data connector:
- **Polls the historian** at regular intervals (typically every 5-15 minutes)
- **Retrieves time-series data** for identified tags
- **Transmits encrypted data** to Novity's cloud platform
- **Stores data** for real-time and historical analysis

### Step 4: Data Processing
Once received:
1. **Validation**: Check for missing values, outliers, data quality issues
2. **Normalization**: Convert units, align timestamps
3. **Feature Extraction**: Calculate derived metrics (efficiency ratios, trends)
4. **Contextualization**: Combine with equipment metadata (model, OEM manual, operating ranges)

### Step 5: AI Analysis
TruPrognostics AI processes the normalized data:
- Compares against physics-based expected behavior models
- Detects deviations (efficiency loss, compression changes, bearing wear indicators)
- Generates diagnostics and RUL forecasts
- Produces sourced maintenance recommendations

### Step 6: Output to Dashboard
Results appear in Novity platform:
- Per-asset health status and RUL estimates
- Trending graphs showing fault development
- Alerts when new events occur (email/text)
- Detailed diagnostic reports

## Data Flow Diagram

```
Customer Historian (PI/SCADA)
    ↓
[Read-Only Connection: OPC UA / REST API]
    ↓
Novity Data Connector
    ↓
[Encrypted HTTPS Transmission]
    ↓
Novity Cloud Platform
    ↓
Data Validation & Normalization
    ↓
TruPrognostics AI Models
    ↓
Diagnostics Engine
    ↓
Novity Dashboard
    ↓
Alerts & Reports
```

## What Data is Pulled?

### Typical Tags for a Reciprocating Compressor

| Tag Category | Examples | Frequency |
|--------------|----------|-----------|
| **Pressures** | Suction, discharge, intermediate stage pressures | Every 5-15 min |
| **Temperatures** | Inlet, outlet, bearing, oil temperatures | Every 5-15 min |
| **Flows** | Inlet flow, outlet flow, bypass flow | Every 5-15 min |
| **Speed** | Compressor RPM, motor frequency | Every 5-15 min |
| **Operating Mode** | Run/stop status, load percentage, valve position | Every 5-15 min |

### Data Volume
- **Per compressor**: ~20-50 tags
- **Polling interval**: 5-15 minutes
- **Data points per day**: ~2,000-5,000 per tag
- **Storage**: Compressed and archived

## Security

- **Encryption**: HTTPS/TLS for all data in transit
- **Authentication**: Customer credentials or API tokens
- **Access Control**: Read-only access to historian
- **Data Isolation**: Customer data segregated in Novity's multi-tenant platform
- **Audit Trail**: All data access logged

## Timeline to Value

| Phase | Duration | What Happens |
|-------|----------|--------------|
| **Scoping** | 1 week | Identify critical assets and available tags |
| **Setup** | 1 week | Configure historian connection, validate data flow |
| **Historical Analysis** | 1-2 weeks | Novity runs AI on past 6-12 months of data |
| **PoV Results** | 1 week | Present findings, validate against known events |
| **Go Live** | 1 week | Enable real-time monitoring and alerts |

## What Tier 1 Can Detect

With process data alone, Novity detects:
- **Efficiency loss** (polytropic efficiency degradation)
- **Compression changes** (pressure ratio deviations)
- **Valve degradation** (suction/discharge valve leakage)
- **Bearing wear indicators** (temperature trends, pressure ripple)
- **Intercooler fouling** (temperature rise)
- **Packing wear** (leakage indicators)

## What Tier 1 Cannot Detect

Tier 1 cannot diagnose mechanical faults that require vibration data:
- Individual valve identification (requires vibration)
- Precise bearing wear (requires vibration)
- Imbalance or misalignment (requires vibration)
- Rod/crosshead wear (requires vibration)

→ These require **Tier 2 (Plus)** with vibration sensors

## Proof of Value (PoV) Process

1. **Customer provides historian access** (read-only)
2. **Novity pulls 6-12 months of historical data**
3. **AI runs diagnostics on past data**
4. **Results validated against known maintenance events**
5. **Lead time demonstrated** (e.g., "We would have detected this fault 40 days early")
6. **Customer decides to deploy** or upgrade to Tier 2

## Cost & ROI

- **Tier 1 Cost**: Typically $X per asset per year (licensing)
- **Setup Cost**: Minimal (data connector configuration)
- **ROI**: Demonstrated in PoV via lead time on avoided emergency maintenance
- **Upgrade Path**: Add Tier 2 sensors only on assets where ROI justifies

## Summary

**Tier 1 Data Pull:**
- Pulls existing historian data (PI/SCADA tags)
- No new infrastructure or sensors required
- Continuous polling every 5-15 minutes
- Encrypted transmission to Novity cloud
- AI processes data to detect process-domain faults
- Results in dashboard with alerts and RUL forecasts
- Day-one value from data customers already collect
