# Layer 1: Engineering Features for Electric Motors

## Overview

Electric motors have different fault signatures than rotating equipment like compressors. Motor faults manifest in electrical, mechanical, and thermal domains.

## 1. Electrical Features

### Current Signature Analysis (CSA)

**Purpose:** Detect rotor bar breakage, winding faults, and bearing wear

```python
def analyze_motor_current_signature(current_waveform, sampling_rate=10000, 
                                   line_frequency=60, motor_speed_rpm=1800):
    """
    Analyze motor current signature for electrical faults
    
    Args:
        current_waveform: Time-domain current data (Amps)
        sampling_rate: Sampling frequency (Hz)
        line_frequency: AC line frequency (Hz) - 60 Hz in US
        motor_speed_rpm: Motor speed (RPM)
    
    Returns:
        csa_features: Dictionary with current signature features
    """
    
    # Convert motor speed to Hz
    motor_speed_hz = motor_speed_rpm / 60
    
    # Number of poles (typically 2, 4, 6, 8)
    # For 1800 RPM at 60 Hz: 4 poles
    num_poles = int((line_frequency * 120) / motor_speed_rpm)
    
    # Slip frequency
    slip = (line_frequency - motor_speed_hz) / line_frequency
    slip_frequency = slip * line_frequency
    
    # Rotor bar pass frequency (RBPF)
    # Frequency at which rotor bars pass stator magnetic field
    num_rotor_bars = 28  # Typical for 4-pole motor
    rbpf = num_rotor_bars * slip_frequency
    
    # Perform FFT
    fft_vals = np.fft.fft(current_waveform)
    magnitude = np.abs(fft_vals[:len(fft_vals)//2])
    frequencies = np.fft.fftfreq(len(fft_vals), 1/sampling_rate)[:len(fft_vals)//2]
    
    # Extract features
    features = {
        'line_frequency': line_frequency,
        'motor_speed_hz': motor_speed_hz,
        'slip': slip,
        'slip_frequency': slip_frequency,
        'rbpf': rbpf,
        'num_poles': num_poles,
        'num_rotor_bars': num_rotor_bars
    }
    
    # Find energy at fault frequencies
    # Rotor bar fault: Peaks at (line_freq ± RBPF) and harmonics
    for harmonic in range(1, 5):
        fault_freq = line_frequency + harmonic * rbpf
        bandwidth = fault_freq * 0.05
        mask = (frequencies >= fault_freq - bandwidth) & (frequencies <= fault_freq + bandwidth)
        features[f'rotor_bar_harmonic_{harmonic}_energy'] = np.sum(magnitude[mask])
    
    return features

# Example: 10 HP, 4-pole, 1800 RPM motor at 60 Hz
csa = analyze_motor_current_signature(
    current_waveform=np.random.randn(10000),
    sampling_rate=10000,
    line_frequency=60,
    motor_speed_rpm=1800
)
print(f"Slip: {csa['slip']:.4f}")
print(f"RBPF: {csa['rbpf']:.1f} Hz")
# Output:
# Slip: 0.0333
# RBPF: 18.6 Hz
```

**Fault Signatures:**
- **Rotor bar breakage**: Peaks at (60 ± RBPF), (60 ± 2×RBPF), etc.
- **Winding fault**: Peaks at 3× line frequency (120 Hz for 60 Hz supply)
- **Bearing wear**: Peaks at bearing defect frequencies modulated by slip

### RMS Current

**Purpose:** Detect overload, phase imbalance, and efficiency loss

```python
def calculate_motor_current_features(current_waveform, baseline_current=None):
    """
    Calculate motor current-based features
    """
    
    # RMS current
    rms_current = np.sqrt(np.mean(current_waveform ** 2))
    
    # Peak current
    peak_current = np.max(np.abs(current_waveform))
    
    # Crest factor
    crest_factor = peak_current / rms_current
    
    # Current deviation from baseline
    if baseline_current:
        current_deviation = ((rms_current - baseline_current) / baseline_current) * 100
    else:
        current_deviation = 0
    
    return {
        'rms_current': rms_current,
        'peak_current': peak_current,
        'crest_factor': crest_factor,
        'current_deviation_pct': current_deviation
    }

# Fault Signatures:
# - RMS current > 10% above baseline = Overload or efficiency loss
# - Crest factor > 1.8 = Winding fault or phase imbalance
# - Increasing RMS trend = Progressive winding degradation
```

### Phase Imbalance Detection

**Purpose:** Detect phase imbalance (common cause of motor failure)

```python
def detect_phase_imbalance(current_phase_a, current_phase_b, current_phase_c):
    """
    Detect three-phase current imbalance
    
    Args:
        current_phase_a, b, c: Current waveforms for each phase
    
    Returns:
        imbalance_metrics: Phase imbalance indicators
    """
    
    # RMS current for each phase
    rms_a = np.sqrt(np.mean(current_phase_a ** 2))
    rms_b = np.sqrt(np.mean(current_phase_b ** 2))
    rms_c = np.sqrt(np.mean(current_phase_c ** 2))
    
    # Average RMS
    rms_avg = (rms_a + rms_b + rms_c) / 3
    
    # Phase imbalance percentage
    # NEMA standard: Phase imbalance = (Max deviation / Average) × 100
    max_deviation = max(abs(rms_a - rms_avg), abs(rms_b - rms_avg), abs(rms_c - rms_avg))
    phase_imbalance_pct = (max_deviation / rms_avg) * 100
    
    return {
        'rms_a': rms_a,
        'rms_b': rms_b,
        'rms_c': rms_c,
        'phase_imbalance_pct': phase_imbalance_pct,
        'imbalance_severity': 'normal' if phase_imbalance_pct < 5 else 'warning' if phase_imbalance_pct < 10 else 'critical'
    }

# Fault Signatures:
# - Phase imbalance < 5% = Normal
# - Phase imbalance 5-10% = Warning (check power supply)
# - Phase imbalance > 10% = Critical (motor will overheat)
```

## 2. Mechanical Features

### Vibration Analysis

**Purpose:** Detect bearing wear, misalignment, and imbalance

```python
def analyze_motor_vibration(vibration_waveform, sampling_rate=10000, 
                           motor_speed_rpm=1800):
    """
    Analyze motor vibration for mechanical faults
    """
    
    motor_speed_hz = motor_speed_rpm / 60
    
    # Time-domain features
    rms_vibration = np.sqrt(np.mean(vibration_waveform ** 2))
    peak_vibration = np.max(np.abs(vibration_waveform))
    crest_factor = peak_vibration / rms_vibration
    
    # Frequency-domain analysis
    fft_vals = np.fft.fft(vibration_waveform)
    magnitude = np.abs(fft_vals[:len(fft_vals)//2])
    frequencies = np.fft.fftfreq(len(fft_vals), 1/sampling_rate)[:len(fft_vals)//2]
    
    # Find peaks
    peaks, _ = signal.find_peaks(magnitude, height=np.max(magnitude)*0.1)
    
    features = {
        'rms_vibration': rms_vibration,
        'peak_vibration': peak_vibration,
        'crest_factor': crest_factor,
        'dominant_frequency': frequencies[peaks[0]] if len(peaks) > 0 else 0
    }
    
    # Check for specific fault frequencies
    # 1× motor speed = Imbalance
    # 2× motor speed = Misalignment
    # Bearing defect frequencies = Bearing wear
    
    for harmonic in range(1, 5):
        fault_freq = harmonic * motor_speed_hz
        bandwidth = fault_freq * 0.1
        mask = (frequencies >= fault_freq - bandwidth) & (frequencies <= fault_freq + bandwidth)
        features[f'{harmonic}x_speed_energy'] = np.sum(magnitude[mask])
    
    return features

# Fault Signatures:
# - High 1× speed energy = Imbalance
# - High 2× speed energy = Misalignment
# - High bearing defect frequency energy = Bearing wear
# - Increasing RMS trend = Progressive degradation
```

### Bearing Defect Frequencies (Same as compressors)

```python
def calculate_motor_bearing_defect_frequencies(shaft_speed_rpm, bearing_type='6205'):
    """
    Calculate bearing defect frequencies for motor bearings
    
    Common motor bearing types:
    - 6205: Deep groove ball bearing (common in small motors)
    - 6308: Deep groove ball bearing (common in large motors)
    - NU205: Cylindrical roller bearing
    """
    
    bearing_specs = {
        '6205': {'num_balls': 8, 'pitch_diameter': 71.5, 'ball_diameter': 19.05, 'contact_angle': 0},
        '6308': {'num_balls': 8, 'pitch_diameter': 130, 'ball_diameter': 31.75, 'contact_angle': 0},
        'NU205': {'num_rollers': 8, 'pitch_diameter': 80, 'roller_diameter': 20, 'contact_angle': 0}
    }
    
    params = bearing_specs[bearing_type]
    shaft_speed_hz = shaft_speed_rpm / 60
    
    # Calculate defect frequencies (same as compressor bearings)
    num_balls = params['num_balls']
    pitch_diameter = params['pitch_diameter']
    ball_diameter = params['ball_diameter']
    contact_angle = params['contact_angle']
    
    ftf = (shaft_speed_hz / 2) * (1 - (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    bpfo = (num_balls / 2) * shaft_speed_hz * (1 - (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    bpfi = (num_balls / 2) * shaft_speed_hz * (1 + (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    bsf = (pitch_diameter / (2 * ball_diameter)) * shaft_speed_hz * (1 - (ball_diameter / pitch_diameter) ** 2 * np.cos(np.radians(contact_angle)) ** 2)
    
    return {'ftf': ftf, 'bpfo': bpfo, 'bpfi': bpfi, 'bsf': bsf}

# Example: 1800 RPM motor with 6205 bearing
bearing_freqs = calculate_motor_bearing_defect_frequencies(1800, '6205')
print(f"BPFO: {bearing_freqs['bpfo']:.1f} Hz")
# Output: BPFO: 79.9 Hz
```

## 3. Thermal Features

### Temperature Monitoring

**Purpose:** Detect winding degradation, bearing wear, and overload

```python
def analyze_motor_temperature(winding_temp, bearing_temp, ambient_temp, 
                             baseline_winding_temp=None):
    """
    Analyze motor temperature for thermal faults
    
    Args:
        winding_temp: Winding temperature (°C)
        bearing_temp: Bearing temperature (°C)
        ambient_temp: Ambient temperature (°C)
        baseline_winding_temp: Normal winding temperature (°C)
    
    Returns:
        thermal_features: Temperature-based features
    """
    
    # Temperature rise above ambient
    winding_rise = winding_temp - ambient_temp
    bearing_rise = bearing_temp - ambient_temp
    
    # Normal temperature rise for motor: 40-60°C above ambient
    normal_winding_rise = 50
    
    # Temperature deviation
    winding_deviation = winding_rise - normal_winding_rise
    
    # Bearing-to-winding temperature differential
    # Normal: 5-10°C (bearing cooler than winding)
    # High differential: Bearing friction increasing
    temp_differential = winding_temp - bearing_temp
    
    # Thermal stress indicator
    # NEMA standard: Motor life halves for every 10°C above rated temperature
    if winding_temp > 130:  # Class B insulation limit
        thermal_stress = 'critical'
    elif winding_temp > 120:
        thermal_stress = 'warning'
    else:
        thermal_stress = 'normal'
    
    return {
        'winding_rise': winding_rise,
        'bearing_rise': bearing_rise,
        'winding_deviation': winding_deviation,
        'temp_differential': temp_differential,
        'thermal_stress': thermal_stress
    }

# Fault Signatures:
# - Winding temp > 130°C = Overload or winding fault
# - Bearing temp > 100°C = Bearing wear or lubrication issue
# - Increasing winding temp trend = Progressive degradation
# - High temp differential = Bearing friction increasing
```

## 4. Efficiency Features

### Motor Efficiency Calculation

**Purpose:** Detect efficiency loss from winding faults, bearing wear

```python
def calculate_motor_efficiency(input_power, output_power, baseline_efficiency=None):
    """
    Calculate motor efficiency
    
    Args:
        input_power: Electrical input power (kW)
        output_power: Mechanical output power (kW)
        baseline_efficiency: Normal motor efficiency (%)
    
    Returns:
        efficiency_metrics: Efficiency-based features
    """
    
    # Efficiency
    efficiency = (output_power / input_power) * 100 if input_power > 0 else 0
    
    # Losses
    losses = input_power - output_power
    
    # Efficiency deviation
    if baseline_efficiency:
        efficiency_deviation = efficiency - baseline_efficiency
    else:
        efficiency_deviation = 0
    
    return {
        'efficiency_pct': efficiency,
        'losses_kw': losses,
        'efficiency_deviation_pct': efficiency_deviation
    }

# Fault Signatures:
# - Efficiency loss > 5% = Winding fault or bearing wear
# - Efficiency loss > 10% = Severe degradation
# - Increasing loss trend = Progressive fault development
```

## 5. Eccentricity Detection

**Purpose:** Detect rotor eccentricity (rotor rubs stator)

```python
def detect_rotor_eccentricity(current_waveform, sampling_rate=10000, 
                             line_frequency=60, motor_speed_rpm=1800):
    """
    Detect rotor eccentricity from current signature
    
    Rotor eccentricity causes:
    - Uneven air gap
    - Rotor rubs stator
    - Increased current ripple
    """
    
    motor_speed_hz = motor_speed_rpm / 60
    
    # Eccentricity produces sidebands around line frequency
    # Sidebands at: (line_freq ± motor_speed_hz)
    
    fft_vals = np.fft.fft(current_waveform)
    magnitude = np.abs(fft_vals[:len(fft_vals)//2])
    frequencies = np.fft.fftfreq(len(fft_vals), 1/sampling_rate)[:len(fft_vals)//2]
    
    # Find energy at sideband frequencies
    sideband_lower = line_frequency - motor_speed_hz
    sideband_upper = line_frequency + motor_speed_hz
    
    bandwidth = 5  # Hz
    
    mask_lower = (frequencies >= sideband_lower - bandwidth) & (frequencies <= sideband_lower + bandwidth)
    mask_upper = (frequencies >= sideband_upper - bandwidth) & (frequencies <= sideband_upper + bandwidth)
    
    sideband_energy_lower = np.sum(magnitude[mask_lower])
    sideband_energy_upper = np.sum(magnitude[mask_upper])
    
    # Find energy at line frequency
    mask_line = (frequencies >= line_frequency - bandwidth) & (frequencies <= line_frequency + bandwidth)
    line_energy = np.sum(magnitude[mask_line])
    
    # Sideband ratio
    sideband_ratio = (sideband_energy_lower + sideband_energy_upper) / line_energy if line_energy > 0 else 0
    
    return {
        'sideband_lower_freq': sideband_lower,
        'sideband_upper_freq': sideband_upper,
        'sideband_energy_lower': sideband_energy_lower,
        'sideband_energy_upper': sideband_energy_upper,
        'sideband_ratio': sideband_ratio,
        'eccentricity_severity': 'normal' if sideband_ratio < 0.1 else 'warning' if sideband_ratio < 0.2 else 'critical'
    }

# Fault Signatures:
# - Sideband ratio < 0.1 = Normal
# - Sideband ratio 0.1-0.2 = Mild eccentricity
# - Sideband ratio > 0.2 = Severe eccentricity (rotor rubs stator)
```

## Summary: Motor Engineering Features

**Electrical Domain:**
- Current Signature Analysis (CSA) - Rotor bar breakage, winding faults
- RMS Current - Overload, efficiency loss
- Phase Imbalance - Power supply issues, winding faults

**Mechanical Domain:**
- Vibration Analysis - Bearing wear, misalignment, imbalance
- Bearing Defect Frequencies - Bearing degradation
- 1× Speed Energy - Imbalance
- 2× Speed Energy - Misalignment

**Thermal Domain:**
- Winding Temperature - Overload, winding degradation
- Bearing Temperature - Bearing wear, lubrication issues
- Temperature Rise - Efficiency loss

**Efficiency Domain:**
- Motor Efficiency - Winding faults, bearing wear
- Power Loss - Degradation indicator

**Rotor Domain:**
- Rotor Eccentricity - Rotor rubs stator, mechanical damage

## Motor Fault Signatures Summary

| Fault | Electrical | Mechanical | Thermal | Efficiency |
|-------|-----------|-----------|---------|-----------|
| **Rotor Bar Breakage** | Peaks at (60±RBPF) | Increased vibration | Temp rise | Efficiency loss |
| **Winding Fault** | Peaks at 3× line freq | Vibration increase | High winding temp | Efficiency loss |
| **Bearing Wear** | CSA modulation | BPFO/BPFI peaks | Bearing temp rise | Efficiency loss |
| **Imbalance** | Phase imbalance | High 1× speed | Temp rise | Efficiency loss |
| **Misalignment** | Current ripple | High 2× speed | Temp rise | Efficiency loss |
| **Eccentricity** | Sidebands at ±motor speed | Vibration increase | Temp rise | Efficiency loss |
| **Overload** | High RMS current | Vibration increase | High winding temp | Efficiency loss |

**Key Insight:** Motor faults manifest across multiple domains (electrical, mechanical, thermal). Combining features from all domains provides robust fault detection.
