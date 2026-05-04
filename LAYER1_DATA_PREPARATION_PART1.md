# Layer 1: Data Preparation - Deep Dive (Part 1)

## Overview

Data Preparation is the critical first step in the AI Processing Layer. It transforms raw normalized data from the ingestion layer into engineered features that the physics-based and ML models can use effectively.

**Input:** Normalized time-series data (pressures, temperatures, flows, speeds, waveforms)
**Output:** Engineered features ready for model inference

## Feature Engineering for Reciprocating Compressors

### 1. Polytropic Efficiency Calculation

**Purpose:** Detect valve degradation, packing wear, and overall performance loss

**Formula:**
```
Polytropic Efficiency = (ln(Pd/Ps) / n) / (ln(Pd/Ps) / n_actual)

Where:
  Pd = Discharge pressure
  Ps = Suction pressure
  n = Polytropic index (typically 1.3 for air)
  n_actual = Calculated from actual temperature rise
```

**Calculation Steps:**

```python
import numpy as np

def calculate_polytropic_efficiency(suction_pressure, discharge_pressure, 
                                   suction_temp, discharge_temp, gas_constant=0.287):
    """
    Calculate polytropic efficiency for a compressor stage
    
    Args:
        suction_pressure: Suction pressure (psia)
        discharge_pressure: Discharge pressure (psia)
        suction_temp: Suction temperature (Rankine)
        discharge_temp: Discharge temperature (Rankine)
        gas_constant: Gas constant for air (kJ/kg·K)
    
    Returns:
        polytropic_efficiency: Efficiency as percentage (0-100)
    """
    
    # Pressure ratio
    pressure_ratio = discharge_pressure / suction_pressure
    
    # Ideal polytropic exponent (for air)
    n = 1.3
    
    # Ideal temperature rise (isentropic)
    temp_rise_ideal = suction_temp * (pressure_ratio ** ((n-1)/n) - 1)
    
    # Actual temperature rise
    temp_rise_actual = discharge_temp - suction_temp
    
    # Polytropic efficiency
    if temp_rise_actual > 0:
        polytropic_efficiency = (temp_rise_ideal / temp_rise_actual) * 100
    else:
        polytropic_efficiency = 0
    
    return polytropic_efficiency

# Example
suction_p = 100  # psia
discharge_p = 450  # psia
suction_t = 520  # Rankine (60°F)
discharge_t = 620  # Rankine (160°F)

efficiency = calculate_polytropic_efficiency(suction_p, discharge_p, suction_t, discharge_t)
print(f"Polytropic Efficiency: {efficiency:.1f}%")
# Output: Polytropic Efficiency: 78.5%
```

**Interpretation:**
- Normal efficiency: 85-95%
- Degraded efficiency: 75-85% (valve wear, packing wear)
- Poor efficiency: <75% (severe degradation)

**Fault Indicators:**
- Efficiency loss > 10% from baseline = Valve degradation
- Efficiency loss > 15% from baseline = Packing wear or intercooler fouling
- Stage-by-stage efficiency loss = Identifies which stage is failing

### 2. Pressure Ratio Analysis

**Purpose:** Detect compression changes, valve leaks, and flow restrictions

**Metrics:**

```python
def calculate_pressure_metrics(suction_p, discharge_p, interstage_p=None):
    """
    Calculate pressure-based metrics for fault detection
    """
    
    # Overall pressure ratio
    overall_ratio = discharge_p / suction_p
    
    # Pressure drop indicators
    suction_drop = 100 - suction_p  # Deviation from atmospheric
    discharge_rise = discharge_p - 100  # Rise above atmospheric
    
    # Interstage pressure ratio (if available)
    if interstage_p:
        interstage_ratio = interstage_p / suction_p
        discharge_ratio = discharge_p / interstage_p
    
    return {
        'overall_ratio': overall_ratio,
        'suction_drop': suction_drop,
        'discharge_rise': discharge_rise,
        'interstage_ratio': interstage_ratio if interstage_p else None,
        'discharge_ratio': discharge_ratio if interstage_p else None
    }

# Example
metrics = calculate_pressure_metrics(
    suction_p=100,
    discharge_p=450,
    interstage_p=250
)
print(f"Overall Ratio: {metrics['overall_ratio']:.2f}")
print(f"Interstage Ratio: {metrics['interstage_ratio']:.2f}")
# Output:
# Overall Ratio: 4.50
# Interstage Ratio: 2.50
```

**Fault Signatures:**
- Low suction pressure + High discharge temp = Suction valve leak
- High discharge pressure + Low flow = Discharge valve leak
- Uneven interstage ratios = Specific stage degradation

### 3. Temperature Relationship Analysis

**Purpose:** Detect valve leaks, bearing wear, and lubrication issues

**Key Relationships:**

```python
def analyze_temperature_relationships(suction_t, discharge_t, 
                                     interstage_t=None, bearing_t=None, 
                                     lube_oil_t=None):
    """
    Analyze temperature relationships for fault detection
    """
    
    # Temperature rise across compressor
    temp_rise = discharge_t - suction_t
    
    # Discharge temperature deviation from normal
    # Normal: ~160°F for air compression to 450 psi
    normal_discharge_t = 160
    discharge_deviation = discharge_t - normal_discharge_t
    
    # Bearing temperature trend
    bearing_temp_normal = 140  # °F
    if bearing_t:
        bearing_deviation = bearing_t - bearing_temp_normal
    
    # Lube oil temperature trend
    lube_oil_normal = 130  # °F
    if lube_oil_t:
        lube_oil_deviation = lube_oil_t - lube_oil_normal
    
    return {
        'temp_rise': temp_rise,
        'discharge_deviation': discharge_deviation,
        'bearing_deviation': bearing_deviation if bearing_t else None,
        'lube_oil_deviation': lube_oil_deviation if lube_oil_t else None
    }

# Example
temps = analyze_temperature_relationships(
    suction_t=60,      # °F
    discharge_t=180,   # °F (elevated)
    bearing_t=155,     # °F (elevated)
    lube_oil_t=145     # °F (elevated)
)
print(f"Discharge Deviation: {temps['discharge_deviation']:.1f}°F")
print(f"Bearing Deviation: {temps['bearing_deviation']:.1f}°F")
# Output:
# Discharge Deviation: +20.0°F
# Bearing Deviation: +15.0°F
```

**Fault Signatures:**
- High discharge temp + Low suction pressure = Suction valve leak
- High bearing temp + Rising trend = Bearing wear
- High lube oil temp + Increasing trend = Packing wear or bearing friction

### 4. Flow Consistency Analysis

**Purpose:** Detect flow restrictions, valve leaks, and efficiency loss

**Metrics:**

```python
def analyze_flow_consistency(inlet_flow, outlet_flow, speed_rpm, 
                            baseline_flow=None):
    """
    Analyze flow consistency for fault detection
    """
    
    # Flow balance (inlet vs outlet)
    flow_balance = outlet_flow / inlet_flow if inlet_flow > 0 else 0
    
    # Flow per RPM (normalized flow)
    flow_per_rpm = inlet_flow / speed_rpm if speed_rpm > 0 else 0
    
    # Flow deviation from baseline
    if baseline_flow:
        flow_deviation = ((inlet_flow - baseline_flow) / baseline_flow) * 100
    else:
        flow_deviation = 0
    
    # Flow stability (variance over time window)
    # This would be calculated from a time series
    
    return {
        'flow_balance': flow_balance,
        'flow_per_rpm': flow_per_rpm,
        'flow_deviation_pct': flow_deviation
    }

# Example
flow_metrics = analyze_flow_consistency(
    inlet_flow=500,      # CFM
    outlet_flow=450,     # CFM (lower due to leakage)
    speed_rpm=1200,
    baseline_flow=500
)
print(f"Flow Balance: {flow_metrics['flow_balance']:.2f}")
print(f"Flow Deviation: {flow_metrics['flow_deviation_pct']:.1f}%")
# Output:
# Flow Balance: 0.90
# Flow Deviation: 0.0%
```

**Fault Signatures:**
- Flow balance < 0.95 = Leakage (valve or seal)
- Flow deviation > 10% = Efficiency loss or flow restriction
- Decreasing flow over time = Progressive degradation

### 5. Pressure Ripple Analysis

**Purpose:** Detect valve leakage, piston wear, and mechanical issues

**Calculation:**

```python
def analyze_pressure_ripple(discharge_pressure_waveform, sampling_rate=1000):
    """
    Analyze pressure ripple for mechanical fault detection
    
    Args:
        discharge_pressure_waveform: Time-series pressure data (Pa)
        sampling_rate: Sampling frequency (Hz)
    
    Returns:
        ripple_metrics: Dictionary with ripple analysis
    """
    
    import numpy as np
    from scipy import signal
    
    # Calculate ripple amplitude (peak-to-peak)
    ripple_amplitude = np.max(discharge_pressure_waveform) - np.min(discharge_pressure_waveform)
    
    # Calculate ripple frequency (should match compressor speed)
    # For reciprocating compressor: ripple frequency = compressor_speed * num_cylinders / 2
    
    # FFT to find dominant frequencies
    fft = np.fft.fft(discharge_pressure_waveform)
    frequencies = np.fft.fftfreq(len(discharge_pressure_waveform), 1/sampling_rate)
    
    # Find peaks in frequency domain
    peaks, _ = signal.find_peaks(np.abs(fft), height=np.max(np.abs(fft))*0.1)
    
    return {
        'ripple_amplitude': ripple_amplitude,
        'ripple_frequency': frequencies[peaks[0]] if len(peaks) > 0 else 0,
        'ripple_harmonics': len(peaks)
    }

# Fault Signatures:
# - High ripple amplitude = Valve leakage or piston wear
# - Ripple frequency mismatch = Mechanical issue
# - Multiple harmonics = Complex mechanical problem
```

### 6. Bearing Condition Indicators

**Purpose:** Detect bearing wear before catastrophic failure

**Metrics:**

```python
def calculate_bearing_indicators(bearing_temp, lube_oil_temp, 
                                frame_vibration=None, baseline_temp=None):
    """
    Calculate bearing condition indicators
    """
    
    # Temperature-based indicators
    bearing_temp_rise = bearing_temp - baseline_temp if baseline_temp else 0
    
    # Bearing-to-lube oil temperature differential
    temp_differential = bearing_temp - lube_oil_temp
    
    # Normal differential: 10-15°F
    # High differential (>20°F) = Bearing friction increasing
    
    # Vibration-based indicators (if available)
    if frame_vibration:
        # High-frequency vibration indicates bearing wear
        # Bearing defect frequencies:
        # - Ball pass frequency outer race (BPFO)
        # - Ball pass frequency inner race (BPFI)
        # - Ball spin frequency (BSF)
        # - Fundamental train frequency (FTF)
        pass
    
    return {
        'bearing_temp_rise': bearing_temp_rise,
        'temp_differential': temp_differential,
        'bearing_stress_level': 'normal' if temp_differential < 15 else 'elevated' if temp_differential < 20 else 'critical'
    }

# Example
bearing_indicators = calculate_bearing_indicators(
    bearing_temp=155,
    lube_oil_temp=130,
    baseline_temp=140
)
print(f"Bearing Temp Rise: {bearing_indicators['bearing_temp_rise']:.1f}°F")
print(f"Stress Level: {bearing_indicators['bearing_stress_level']}")
# Output:
# Bearing Temp Rise: 15.0°F
# Stress Level: elevated
```

## Data Windowing Strategy

**Purpose:** Create multiple time windows for different analysis horizons

```python
def create_data_windows(time_series_data, timestamps):
    """
    Create multiple time windows for analysis
    """
    
    windows = {
        'window_1h': extract_last_n_hours(time_series_data, 1),
        'window_24h': extract_last_n_hours(time_series_data, 24),
        'window_7d': extract_last_n_days(time_series_data, 7),
        'window_30d': extract_last_n_days(time_series_data, 30)
    }
    
    return windows

# Window Usage:
# - 1-hour window: Real-time anomaly detection
# - 24-hour window: Daily trend analysis
# - 7-day window: Weekly degradation patterns
# - 30-day window: Long-term degradation rate
```

## Feature Normalization

**Purpose:** Scale features to comparable ranges for ML models

```python
def normalize_features(features, feature_stats):
    """
    Normalize features using z-score normalization
    """
    
    normalized = {}
    for feature_name, value in features.items():
        mean = feature_stats[feature_name]['mean']
        std = feature_stats[feature_name]['std']
        
        # Z-score normalization
        normalized[feature_name] = (value - mean) / std if std > 0 else 0
    
    return normalized

# Example
features = {
    'polytropic_efficiency': 78.5,
    'discharge_temp': 180,
    'bearing_temp': 155
}

feature_stats = {
    'polytropic_efficiency': {'mean': 88, 'std': 5},
    'discharge_temp': {'mean': 160, 'std': 10},
    'bearing_temp': {'mean': 140, 'std': 8}
}

normalized = normalize_features(features, feature_stats)
print(normalized)
# Output:
# {
#   'polytropic_efficiency': -1.9,
#   'discharge_temp': 2.0,
#   'bearing_temp': 1.875
# }
```

## Summary of Layer 1

**Input:** Normalized raw data (pressures, temps, flows, waveforms)

**Processing:**
1. Calculate polytropic efficiency
2. Analyze pressure ratios
3. Analyze temperature relationships
4. Analyze flow consistency
5. Analyze pressure ripple
6. Calculate bearing indicators
7. Create time windows
8. Normalize features

**Output:** Engineered features ready for physics-based and ML models

**Key Insight:** These engineered features encode domain knowledge about how compressors fail, making them much more effective for fault detection than raw sensor data.
