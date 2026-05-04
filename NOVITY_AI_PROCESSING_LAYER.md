# Novity AI Processing Layer - Complete Technical Analysis

## Overview

The AI Processing Layer is the core of Novity's TruPrognostics™ platform. It combines three AI approaches (physics-based models, machine learning, and contextual AI) to diagnose faults, forecast remaining useful life (RUL), and recommend maintenance actions.

## Architecture: The Hybrid Stack

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    NORMALIZED DATA FROM INGESTION LAYER                 │
│  (PI/SCADA tags, waveform data, timestamps, quality flags)              │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 1: DATA PREPARATION                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Feature Engineering                                                   │
│  ├─ Polytropic efficiency (for compressors)                           │
│  ├─ Pressure ratios (discharge/suction)                               │
│  ├─ Temperature relationships (inlet/outlet)                          │
│  ├─ Flow consistency (inlet/outlet flow)                              │
│  ├─ Hydraulic performance metrics                                     │
│  ├─ Spectral features (from vibration waveforms)                      │
│  ├─ Bearing condition indicators                                      │
│  └─ Trending analysis (rate of change)                                │
│                                                                         │
│  Data Windowing                                                        │
│  ├─ 1-hour windows (for real-time analysis)                           │
│  ├─ 24-hour windows (for daily trends)                                │
│  ├─ 7-day windows (for weekly patterns)                               │
│  └─ 30-day windows (for long-term degradation)                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 2: EXPECTED BEHAVIOR MODELS                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Physics-Based Models (Encodes how machines fail)                      │
│                                                                         │
│  For Reciprocating Compressors:                                        │
│  ├─ Thermodynamic models (polytropic process)                         │
│  ├─ Valve dynamics (suction/discharge valve behavior)                 │
│  ├─ Piston dynamics (rod, rings, packing)                             │
│  ├─ Bearing mechanics (journal bearing wear)                          │
│  ├─ Lubrication models (oil film thickness)                           │
│  └─ Mechanical resonance (natural frequencies)                        │
│                                                                         │
│  For Centrifugal Compressors:                                          │
│  ├─ Aerodynamic models (stage efficiency)                             │
│  ├─ Bearing wear models (radial/thrust)                               │
│  ├─ Seal leakage models (interstage, labyrinth)                       │
│  ├─ Surge/stall dynamics                                              │
│  └─ Vibration modes (critical speeds)                                 │
│                                                                         │
│  For Pumps:                                                            │
│  ├─ Cavitation models (NPSH, vapor pressure)                          │
│  ├─ Impeller wear models (efficiency loss)                            │
│  ├─ Bearing degradation models                                        │
│  └─ Seal leakage models                                               │
│                                                                         │
│  Expected Behavior Calculation:                                        │
│  ├─ Given: Operating conditions (pressure, temperature, flow, speed)  │
│  ├─ Calculate: Expected efficiency, pressures, temperatures           │
│  ├─ Compare: Actual vs. expected                                      │
│  └─ Output: Deviation score (0-100, where 100 = normal)               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 3: FAULT-SPECIFIC DIAGNOSTIC MODELS            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Parallel Diagnostic Engine                                            │
│  (Runs multiple fault classifiers simultaneously)                      │
│                                                                         │
│  For Reciprocating Compressors (Example):                              │
│                                                                         │
│  Model 1: Loss of Efficiency (by stage)                                │
│  ├─ Input: Stage pressures, temperatures, flows                       │
│  ├─ Calculation: Polytropic efficiency per stage                      │
│  ├─ Threshold: Efficiency < 85% of baseline                           │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: Forecast based on degradation rate                           │
│                                                                         │
│  Model 2: Suction Valve Leak                                           │
│  ├─ Input: Discharge temp, interstage pressure, flow                  │
│  ├─ Signature: High discharge temp + low interstage pressure          │
│  ├─ ML Classifier: Trained on known valve leak events                 │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: 2-8 weeks (based on leak rate)                               │
│                                                                         │
│  Model 3: Discharge Valve Leak                                         │
│  ├─ Input: Discharge pressure, flow, temperature                      │
│  ├─ Signature: Low discharge pressure + high flow                     │
│  ├─ ML Classifier: Trained on known discharge valve leaks             │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: 1-4 weeks (based on leak rate)                               │
│                                                                         │
│  Model 4: Loose Piston                                                 │
│  ├─ Input: Frame vibration, pressure ripple, temperature              │
│  ├─ Signature: Increased frame vibration + pressure ripple            │
│  ├─ ML Classifier: Trained on loose piston signatures                 │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: 1-3 weeks (high risk of catastrophic failure)                │
│                                                                         │
│  Model 5: Main Bearing Wear                                            │
│  ├─ Input: Frame vibration (if available), temperature trends         │
│  ├─ Signature: Increasing frame vibration + bearing temp rise         │
│  ├─ ML Classifier: Trained on bearing wear progression                │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: 4-12 weeks (depends on wear rate)                            │
│                                                                         │
│  Model 6: Packing Wear                                                 │
│  ├─ Input: Lube oil temperature, pressure ripple                      │
│  ├─ Signature: Rising lube oil temp + increased ripple                │
│  ├─ ML Classifier: Trained on packing degradation                     │
│  ├─ Output: Confidence score (0-100)                                  │
│  └─ RUL: 2-6 weeks                                                    │
│                                                                         │
│  ... (Many more models for other fault modes)                          │
│                                                                         │
│  Output: Ranked list of fault candidates with confidence scores       │
│  Example:                                                              │
│  ├─ Suction valve leak: 87% confidence                                │
│  ├─ Flow restriction: 45% confidence                                  │
│  ├─ Bearing wear: 12% confidence                                      │
│  └─ Packing wear: 8% confidence                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 4: AGENTIC CAUSAL REASONING                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Problem: Multiple fault candidates with overlapping evidence          │
│  Example:                                                              │
│  ├─ Suction valve leak: 87% confidence                                │
│  ├─ Flow restriction: 45% confidence                                  │
│  └─ Interstage seal leak: 52% confidence                              │
│                                                                         │
│  Question: Are these three separate faults or one root cause?          │
│                                                                         │
│  Agentic Reasoning Process:                                            │
│                                                                         │
│  Step 1: Analyze Causal Relationships                                  │
│  ├─ Suction valve leak → Low suction pressure                         │
│  ├─ Low suction pressure → Low flow                                   │
│  ├─ Low flow → Looks like flow restriction                            │
│  └─ Conclusion: Suction valve leak is root cause of flow restriction  │
│                                                                         │
│  Step 2: Check Physical Consistency                                    │
│  ├─ If suction valve leaks, discharge temp should rise                │
│  ├─ Actual discharge temp: 180°F (elevated)                           │
│  ├─ Expected for suction leak: 175-185°F                              │
│  └─ Consistent: YES                                                   │
│                                                                         │
│  Step 3: Evaluate Alternative Hypotheses                               │
│  ├─ Hypothesis A: Suction valve leak (root cause)                     │
│  │  └─ Explains: High discharge temp, low flow, low suction pressure  │
│  ├─ Hypothesis B: Interstage seal leak (root cause)                   │
│  │  └─ Explains: Low interstage pressure, but NOT high discharge temp │
│  └─ Conclusion: Hypothesis A is more consistent with data             │
│                                                                         │
│  Step 4: Generate Root Cause Hypotheses                                │
│  ├─ Primary: Suction valve leak (87% confidence)                      │
│  │  └─ Evidence: High discharge temp, low flow, low suction pressure  │
│  ├─ Secondary: Interstage seal leak (52% confidence)                  │
│  │  └─ Evidence: Low interstage pressure                              │
│  └─ Tertiary: Packing wear (15% confidence)                           │
│     └─ Evidence: Slight lube oil temp rise                            │
│                                                                         │
│  Step 5: Synthesize into Coherent Diagnosis                            │
│  ├─ Most Likely: Suction valve leak on throw 4                        │
│  ├─ Contributing: Possible interstage seal degradation                │
│  ├─ Confidence: 87%                                                   │
│  └─ RUL: 12-20 days (based on leak rate progression)                  │
│                                                                         │
│  Output: Coherent diagnosis with multiple hypotheses ranked by        │
│  likelihood, each with supporting evidence                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 5: RUL FORECASTING                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Remaining Useful Life (RUL) Calculation                               │
│                                                                         │
│  For Suction Valve Leak:                                               │
│                                                                         │
│  Step 1: Measure Current Degradation                                   │
│  ├─ Current discharge temp: 180°F                                     │
│  ├─ Normal discharge temp: 160°F                                      │
│  ├─ Degradation: 20°F above normal                                    │
│  └─ Severity: 50% of failure threshold (40°F above normal)            │
│                                                                         │
│  Step 2: Estimate Degradation Rate                                     │
│  ├─ 7 days ago: Discharge temp was 175°F                              │
│  ├─ Today: Discharge temp is 180°F                                    │
│  ├─ Rate: 5°F per 7 days = 0.71°F per day                             │
│  └─ Trend: Linear degradation (or accelerating)                       │
│                                                                         │
│  Step 3: Project to Failure                                            │
│  ├─ Failure threshold: 200°F discharge temp                           │
│  ├─ Current: 180°F                                                    │
│  ├─ Remaining margin: 20°F                                            │
│  ├─ At 0.71°F/day: 20 / 0.71 = 28 days to failure                    │
│  └─ With uncertainty: 12-20 days (conservative estimate)              │
│                                                                         │
│  Step 4: Account for Uncertainty                                       │
│  ├─ Degradation may accelerate (as leak grows)                        │
│  ├─ Degradation may plateau (if leak stabilizes)                      │
│  ├─ Confidence interval: 12-20 days (80% confidence)                  │
│  └─ Output: "12-20 days to failure"                                   │
│                                                                         │
│  RUL Output:                                                           │
│  ├─ Point estimate: 16 days                                           │
│  ├─ Confidence interval: 12-20 days                                   │
│  ├─ Confidence level: 80%                                             │
│  └─ Trend: Degradation accelerating (recommend earlier action)        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 6: RECOMMENDATION ENGINE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Sourced Recommendations (Every action cites its source)               │
│                                                                         │
│  For Suction Valve Leak Diagnosis:                                     │
│                                                                         │
│  Recommended Action:                                                   │
│  "Non-intrusive inspection of throw 4 suction valves with:             │
│   - Ultrasound (to detect valve chatter)                              │
│   - Vibration analysis (to confirm mechanical signature)              │
│   - PV (pressure-volume) analysis (to quantify leakage)"              │
│                                                                         │
│  Source: Ariel Compressor Operating Manual, Section 4.2               │
│  (Valve Inspection Procedures)                                        │
│                                                                         │
│  Timing: Schedule within 7 days (before degradation accelerates)      │
│                                                                         │
│  If Confirmed:                                                         │
│  "Replace suction valve assembly on throw 4.                          │
│   Estimated downtime: 4-6 hours                                       │
│   Estimated cost: $2,000-3,000 (parts + labor)"                       │
│                                                                         │
│  Source: Ariel Compressor Maintenance Manual, Section 5.1             │
│  (Valve Replacement Procedures)                                       │
│                                                                         │
│  Knowledge Sources:                                                    │
│  ├─ OEM Manuals (Ariel, Worthington, etc.)                            │
│  ├─ Customer SOPs (Standard Operating Procedures)                     │
│  ├─ FMECAs (Failure Mode & Effects Analysis)                          │
│  ├─ Prior Work Orders (historical maintenance)                        │
│  └─ Industry Best Practices                                           │
│                                                                         │
│  Constraint: Reasoning space is constrained to these sources           │
│  (Cannot hallucinate or recommend unsupported actions)                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                    FINAL OUTPUT                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Diagnostic Report:                                                    │
│  ├─ Asset: Reciprocating Compressor C-1402                            │
│  ├─ Fault: Suction valve leak (throw 4)                               │
│  ├─ Confidence: 87%                                                   │
│  ├─ RUL: 12-20 days                                                   │
│  ├─ Severity: HIGH (risk of catastrophic failure)                     │
│  ├─ Recommended Action: Inspect suction valves with ultrasound        │
│  ├─ Action Source: Ariel Manual Section 4.2                           │
│  ├─ Timing: Within 7 days                                             │
│  ├─ Supporting Evidence:                                              │
│  │  ├─ Discharge temperature: 180°F (20°F above normal)              │
│  │  ├─ Suction pressure: 85 psi (5 psi below normal)                 │
│  │  ├─ Flow: 450 CFM (50 CFM below normal)                           │
│  │  └─ Trend: Degrading over past 7 days                             │
│  └─ Alternative Hypotheses:                                           │
│     ├─ Interstage seal leak (52% confidence)                          │
│     └─ Packing wear (15% confidence)                                  │
│                                                                         │
│  Dashboard Display:                                                    │
│  ├─ Health Score: 35/100 (CRITICAL)                                   │
│  ├─ RUL Gauge: 12-20 days (RED)                                       │
│  ├─ Fault Trend: Degrading (downward arrow)                           │
│  ├─ Alert: HIGH PRIORITY                                              │
│  └─ Notification: Email + SMS to maintenance team                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Three Analytical Domains

### 1. Process-Domain Analytics (Tier 1: Base)

**Data Input:**
- PI/SCADA tags (pressures, temperatures, flows, speeds)
- Operating mode and configuration tags
- Historical data (6-12 months)

**Metrics Calculated:**
- Polytropic efficiency (per stage)
- Pressure ratios (discharge/suction)
- Temperature relationships (inlet/outlet)
- Flow consistency (inlet/outlet)
- Hydraulic performance

**Faults Detected:**
- Valve degradation (suction/discharge)
- Intercooler fouling
- Leakage (interstage, packing)
- Packing wear
- Efficiency loss
- Bearing condition indicators

**Advantages:**
- Uses data already in historian
- No new sensors required
- Day-one value
- Cost-effective

**Limitations:**
- Cannot detect mechanical faults that don't affect process variables
- Lower precision than vibration data
- Slower to detect bearing wear

### 2. Frequency-Domain Analytics (Tier 2+: Plus/Premium)

**Data Input:**
- Raw waveform data from vibration sensors
- High-frequency cylinder pressure transducers
- Motor current sensor data
- Sampling rate: 10 kHz - 100 kHz

**Spectral Analysis:**
- FFT (Fast Fourier Transform) to convert time-domain to frequency-domain
- Identify spectral peaks at bearing frequencies
- Detect sidebands (modulation patterns)
- Analyze harmonics and sub-harmonics

**Faults Detected:**
- Bearing wear (radial/thrust)
- Journal bearing instability (oil whirl)
- Individual valve identification
- Crosshead wear
- Rod wear
- Imbalance
- Misalignment

**Advantages:**
- Detects mechanical faults early
- High precision
- Can identify specific components
- Faster detection than process data

**Limitations:**
- Requires additional sensors
- Higher cost
- More complex signal processing

### 3. Contextual AI (Agentic Reasoning)

**Purpose:** Synthesize multiple fault candidates into coherent diagnosis

**Process:**
1. Receive ranked list of fault candidates from diagnostic models
2. Analyze causal relationships between faults
3. Check physical consistency with operating data
4. Evaluate alternative hypotheses
5. Generate root cause hypotheses
6. Produce sourced recommendations

**Output:**
- Primary diagnosis with confidence
- Alternative hypotheses ranked by likelihood
- Supporting evidence for each hypothesis
- Recommended actions with sources
- RUL forecast with uncertainty bounds

## Fault Coverage by Equipment Type

### Reciprocating Compressors
**Faults Detected (92% true positive rate):**
- Loss of efficiency (by stage)
- Suction valve leak
- Discharge valve leak
- Loose piston
- Main bearing wear
- Packing wear
- Intercooler fouling
- Strainer clogging
- Rod packing leakage
- Crosshead bushing wear
- Oil whirl
- Rider band wear

**Data Requirements:**
- Base: Pressures, temperatures, flows, RPM
- Plus: + High-frequency frame vibration
- Premium: + Cylinder pressure, crosshead vibration, rod drop

### Centrifugal Compressors
**Faults Detected:**
- Stage efficiency loss
- Bearing wear (radial/thrust)
- Seal leakage (interstage, labyrinth)
- Oil whirl/whip
- Surge/stall conditions
- Impeller wear
- Cavitation (for pumps)

### Centrifugal Pumps
**Faults Detected:**
- Cavitation
- Impeller wear
- Bearing decline
- Seal leakage
- Cavitation erosion

### Electric Motors
**Faults Detected:**
- Winding faults
- Eccentricity
- Bearing faults
- Rotor bar breakage

### Industrial Fans & Blowers
**Faults Detected:**
- Imbalance
- Blade wear
- Bearing wear
- Misalignment

## Performance Metrics

**Validation Results:**
- True Positive Rate: 92-93% (across validated fault modes)
- False Alarm Rate: Low (validated against customer-confirmed events)
- Lead Time: 40+ days (case study example)
- Multiple operators, multiple sites, upstream and midstream

**Case Studies:**
1. **7,800 HP Cylinder Fault**: Identified 4-6 weeks before failure
2. **4,000 HP Loose Piston**: Detected 11 days before catastrophic failure

## Model Training & Deployment

**Training Data:**
- Historical customer data (6-12 months minimum)
- Known fault events (confirmed by maintenance)
- Operating conditions (normal and abnormal)
- Equipment specifications (OEM manuals)

**Model Types:**
- Physics-based models (deterministic)
- Machine learning classifiers (trained on historical data)
- Ensemble methods (combining multiple models)
- Agentic reasoning (causal inference)

**Deployment:**
- Pre-built models for each equipment type
- Tier-specific models (Base, Plus, Premium)
- Continuous learning (models improve with more data)
- Model versioning (track performance over time)

## Integration with AWS

**SageMaker Endpoints:**
- Host pre-trained TruPrognostics models
- Real-time inference (< 1 second latency)
- Batch processing for historical data
- Model versioning and A/B testing

**Lambda Functions:**
- Invoke SageMaker endpoints
- Process results
- Generate alerts
- Store diagnostics in RDS

**Processing Pipeline:**
```
Normalized Data (from SQS)
    ↓
Lambda (Feature Engineering)
    ↓
SageMaker (Model Inference)
    ↓
Lambda (Results Processing)
    ↓
RDS (Store Diagnostics)
    ↓
SNS (Send Alerts)
    ↓
Dashboard (Display Results)
```

## Key Differentiators

1. **Physics-Based**: Encodes how machines fail (not just pattern matching)
2. **Hybrid Approach**: Combines physics, ML, and contextual AI
3. **Agentic Reasoning**: Synthesizes multiple fault candidates into coherent diagnosis
4. **Sourced Recommendations**: Every action cites its source (OEM manual, SOP, etc.)
5. **Transparent**: Every diagnosis shows its evidence and reasoning
6. **Validated**: 92-93% true positive rate in production
7. **Tier-Based**: Scales with data (Base → Plus → Premium)

## Summary

**Novity's AI Processing Layer:**
- Combines physics-based models, ML, and contextual AI
- Processes both process data and waveform data
- Generates fault diagnosis, RUL forecast, and sourced recommendations
- Achieves 92-93% true positive rate in production
- Provides transparent, explainable diagnostics
- Scales from Base (historian data only) to Premium (multiple sensors)

**Result:** Actionable insights that engineers can trust and act on immediately.
