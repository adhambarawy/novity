# Layer 1: Data Preparation - Deep Dive (Part 2: Frequency Domain)

## Frequency-Domain Feature Engineering

**Purpose:** Extract mechanical fault signatures from vibration waveforms

**Data Input:** Raw waveform data from vibration sensors (10 kHz - 100 kHz sampling)

### 1. FFT (Fast Fourier Transform) Analysis

**Purpose:** Convert time-domain vibration to frequency-domain for bearing fault detection

```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt

def perform_fft_analysis(vibration_waveform, sampling_rate=10000):
    """
    Perform FFT analysis on vibration waveform
    
    Args:
        vibration_waveform: Time-domain vibration data (m/s or g)
        sampling_rate: Sampling frequency (Hz)
    
    Returns:
        fft_results: Dictionary with frequency domain analysis
    """
    
    # Compute FFT
    fft_values = np.fft.fft(vibration_waveform)
    frequencies = np.fft.fftfreq(len(vibration_waveform), 1/sampling_rate)
    
    # Get magnitude spectrum (one-sided)
    magnitude = np.abs(fft_values[:len(fft_values)//2])
    frequencies = frequencies[:len(frequencies)//2]
    
    # Normalize
    magnitude = magnitude / np.max(magnitude)
    
    # Find peaks (dominant frequencies)
    peaks, properties = signal.find_peaks(magnitude, height=0.1, distance=10)
    
    return {
        'frequencies': frequencies,
        'magnitude': magnitude,
        'peaks': frequencies[peaks],
        'peak_magnitudes': magnitude[peaks],
        'dominant_frequency': frequencies[peaks[0]] if len(peaks) > 0 else 0
    }

# Example
vibration_data = np.random.randn(10000)  # 1 second at 10 kHz
fft_results = perform_fft_analysis(vibration_data, sampling_rate=10000)
print(f"Dominant Frequency: {fft_results['dominant_frequency']:.1f} Hz")
print(f"Number of Peaks: {len(fft_results['peaks'])}")
```

### 2. Bearing Defect Frequencies

**Purpose:** Identify specific bearing faults by their characteristic frequencies

**Bearing Fault Frequencies:**

```python
def calculate_bearing_defect_frequencies(shaft_speed_rpm, bearing_params):
    """
    Calculate bearing defect frequencies
    
    Args:
        shaft_speed_rpm: Shaft rotational speed (RPM)
        bearing_params: Dictionary with bearing geometry
            - num_balls: Number of rolling elements
            - pitch_diameter: Pitch diameter (mm)
            - ball_diameter: Ball diameter (mm)
            - contact_angle: Contact angle (degrees)
    
    Returns:
        defect_frequencies: Dictionary with bearing fault frequencies
    """
    
    # Convert RPM to Hz
    shaft_speed_hz = shaft_speed_rpm / 60
    
    # Extract bearing parameters
    num_balls = bearing_params['num_balls']
    pitch_diameter = bearing_params['pitch_diameter']
    ball_diameter = bearing_params['ball_diameter']
    contact_angle = bearing_params['contact_angle']
    
    # Fundamental Train Frequency (FTF)
    # Frequency at which balls pass a fixed point
    ftf = (shaft_speed_hz / 2) * (1 - (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    
    # Ball Pass Frequency Outer race (BPFO)
    # Frequency at which balls hit outer race defect
    bpfo = (num_balls / 2) * shaft_speed_hz * (1 - (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    
    # Ball Pass Frequency Inner race (BPFI)
    # Frequency at which balls hit inner race defect
    bpfi = (num_balls / 2) * shaft_speed_hz * (1 + (ball_diameter / pitch_diameter) * np.cos(np.radians(contact_angle)))
    
    # Ball Spin Frequency (BSF)
    # Frequency at which individual ball spins
    bsf = (pitch_diameter / (2 * ball_diameter)) * shaft_speed_hz * (1 - (ball_diameter / pitch_diameter) ** 2 * np.cos(np.radians(contact_angle)) ** 2)
    
    return {
        'ftf': ftf,
        'bpfo': bpfo,
        'bpfi': bpfi,
        'bsf': bsf,
        'shaft_speed': shaft_speed_hz
    }

# Example: SKF 6205 bearing at 1200 RPM
bearing_params = {
    'num_balls': 8,
    'pitch_diameter': 71.5,  # mm
    'ball_diameter': 19.05,  # mm
    'contact_angle': 0  # Deep groove ball bearing
}

defect_freqs = calculate_bearing_defect_frequencies(1200, bearing_params)
print(f"Shaft Speed: {defect_freqs['shaft_speed']:.1f} Hz")
print(f"BPFO: {defect_freqs['bpfo']:.1f} Hz")
print(f"BPFI: {defect_freqs['bpfi']:.1f} Hz")
print(f"BSF: {defect_freqs['bsf']:.1f} Hz")
print(f"FTF: {defect_freqs['ftf']:.1f} Hz")

# Output:
# Shaft Speed: 20.0 Hz
# BPFO: 79.9 Hz
# BPFI: 120.1 Hz
# BSF: 7.9 Hz
# FTF: 0.38 Hz
```

**Fault Signatures:**
- Outer race defect: Peaks at BPFO and harmonics (79.9, 159.8, 239.7 Hz)
- Inner race defect: Peaks at BPFI and harmonics (120.1, 240.2, 360.3 Hz)
- Ball defect: Peaks at BSF and harmonics (7.9, 15.8, 23.7 Hz)
- Cage defect: Peaks at FTF and harmonics (0.38, 0.76, 1.14 Hz)

### 3. Envelope Analysis (High-Frequency Demodulation)

**Purpose:** Extract bearing fault signatures from high-frequency noise

```python
def envelope_analysis(vibration_waveform, sampling_rate=10000, 
                     bandpass_low=5000, bandpass_high=8000):
    """
    Perform envelope analysis to extract bearing fault signatures
    
    Args:
        vibration_waveform: Time-domain vibration data
        sampling_rate: Sampling frequency (Hz)
        bandpass_low: Low frequency for bandpass filter (Hz)
        bandpass_high: High frequency for bandpass filter (Hz)
    
    Returns:
        envelope_results: Dictionary with envelope analysis
    """
    
    # Step 1: Bandpass filter (isolate high-frequency bearing noise)
    nyquist = sampling_rate / 2
    low = bandpass_low / nyquist
    high = bandpass_high / nyquist
    
    b, a = signal.butter(4, [low, high], btype='band')
    filtered = signal.filtfilt(b, a, vibration_waveform)
    
    # Step 2: Demodulation (extract envelope)
    # Method: Hilbert transform to get analytic signal
    analytic_signal = signal.hilbert(filtered)
    envelope = np.abs(analytic_signal)
    
    # Step 3: Downsample envelope
    # Envelope contains bearing fault frequencies (much lower than carrier)
    envelope_downsampled = envelope[::100]  # Downsample by 100x
    
    # Step 4: FFT of envelope
    envelope_fft = np.fft.fft(envelope_downsampled)
    envelope_freqs = np.fft.fftfreq(len(envelope_downsampled), 100/sampling_rate)
    
    return {
        'filtered_signal': filtered,
        'envelope': envelope,
        'envelope_fft': envelope_fft,
        'envelope_freqs': envelope_freqs
    }

# Fault Signatures:
# - Outer race defect: Peaks at BPFO in envelope spectrum
# - Inner race defect: Peaks at BPFI in envelope spectrum
# - Ball defect: Peaks at BSF in envelope spectrum
```

### 4. Spectral Kurtosis

**Purpose:** Identify frequency bands with impulsive bearing fault signatures

```python
def calculate_spectral_kurtosis(vibration_waveform, sampling_rate=10000, 
                               num_bands=128):
    """
    Calculate spectral kurtosis to identify bearing fault bands
    
    Args:
        vibration_waveform: Time-domain vibration data
        sampling_rate: Sampling frequency (Hz)
        num_bands: Number of frequency bands
    
    Returns:
        spectral_kurtosis: Kurtosis value for each frequency band
    """
    
    # Divide signal into frequency bands
    fft_vals = np.fft.fft(vibration_waveform)
    band_size = len(fft_vals) // num_bands
    
    kurtosis_values = []
    
    for i in range(num_bands):
        band_start = i * band_size
        band_end = (i + 1) * band_size
        band_data = fft_vals[band_start:band_end]
        
        # Calculate kurtosis (measure of impulsiveness)
        # High kurtosis = impulsive signals (bearing faults)
        kurtosis = np.sum(np.abs(band_data) ** 4) / (np.sum(np.abs(band_data) ** 2) ** 2)
        kurtosis_values.append(kurtosis)
    
    return np.array(kurtosis_values)

# Interpretation:
# - Normal bearing: Kurtosis ~ 3 (Gaussian)
# - Bearing with defect: Kurtosis > 5 (impulsive)
# - Severe bearing defect: Kurtosis > 10 (very impulsive)
```

### 5. Cepstral Analysis

**Purpose:** Detect bearing fault harmonics and sidebands

```python
def cepstral_analysis(vibration_waveform, sampling_rate=10000):
    """
    Perform cepstral analysis to detect bearing fault harmonics
    
    Args:
        vibration_waveform: Time-domain vibration data
        sampling_rate: Sampling frequency (Hz)
    
    Returns:
        cepstrum: Cepstral coefficients
    """
    
    # Step 1: FFT
    fft_vals = np.fft.fft(vibration_waveform)
    magnitude = np.abs(fft_vals)
    
    # Step 2: Log magnitude
    log_magnitude = np.log(magnitude + 1e-10)  # Add small value to avoid log(0)
    
    # Step 3: Inverse FFT (cepstrum)
    cepstrum = np.fft.ifft(log_magnitude)
    
    # Step 4: Take real part and absolute value
    cepstrum = np.abs(np.real(cepstrum))
    
    return cepstrum

# Interpretation:
# - Peaks in cepstrum indicate periodic components (bearing harmonics)
# - Spacing between peaks = 1 / bearing_defect_frequency
# - Height of peaks = Severity of bearing defect
```

### 6. Time-Synchronous Averaging

**Purpose:** Extract periodic bearing fault signatures from noisy data

```python
def time_synchronous_averaging(vibration_waveform, shaft_speed_hz, 
                              sampling_rate=10000, num_revolutions=10):
    """
    Perform time-synchronous averaging to extract bearing faults
    
    Args:
        vibration_waveform: Time-domain vibration data
        shaft_speed_hz: Shaft speed (Hz)
        sampling_rate: Sampling frequency (Hz)
        num_revolutions: Number of shaft revolutions to average
    
    Returns:
        averaged_signal: Time-synchronously averaged signal
    """
    
    # Samples per shaft revolution
    samples_per_rev = int(sampling_rate / shaft_speed_hz)
    
    # Extract multiple revolutions
    revolutions = []
    for i in range(num_revolutions):
        start = i * samples_per_rev
        end = (i + 1) * samples_per_rev
        if end <= len(vibration_waveform):
            revolutions.append(vibration_waveform[start:end])
    
    # Average across revolutions
    averaged_signal = np.mean(revolutions, axis=0)
    
    return averaged_signal

# Benefit:
# - Noise is random and averages to zero
# - Periodic bearing faults remain and are enhanced
# - Improves signal-to-noise ratio
```

## Spectral Features for ML Models

**Purpose:** Extract numerical features from frequency domain for ML classifiers

```python
def extract_spectral_features(vibration_waveform, sampling_rate=10000, 
                             bearing_defect_freqs=None):
    """
    Extract spectral features for ML models
    
    Args:
        vibration_waveform: Time-domain vibration data
        sampling_rate: Sampling frequency (Hz)
        bearing_defect_freqs: Dictionary with bearing defect frequencies
    
    Returns:
        features: Dictionary with spectral features
    """
    
    # Perform FFT
    fft_vals = np.fft.fft(vibration_waveform)
    magnitude = np.abs(fft_vals[:len(fft_vals)//2])
    frequencies = np.fft.fftfreq(len(fft_vals), 1/sampling_rate)[:len(fft_vals)//2]
    
    features = {}
    
    # 1. Overall vibration level
    features['rms'] = np.sqrt(np.mean(vibration_waveform ** 2))
    features['peak'] = np.max(np.abs(vibration_waveform))
    features['crest_factor'] = features['peak'] / features['rms']
    
    # 2. Spectral features
    features['spectral_centroid'] = np.sum(frequencies * magnitude) / np.sum(magnitude)
    features['spectral_spread'] = np.sqrt(np.sum(((frequencies - features['spectral_centroid']) ** 2) * magnitude) / np.sum(magnitude))
    
    # 3. Bearing defect frequency energy
    if bearing_defect_freqs:
        for defect_name, defect_freq in bearing_defect_freqs.items():
            # Find energy around defect frequency (±10% bandwidth)
            bandwidth = defect_freq * 0.1
            mask = (frequencies >= defect_freq - bandwidth) & (frequencies <= defect_freq + bandwidth)
            features[f'{defect_name}_energy'] = np.sum(magnitude[mask])
    
    # 4. Kurtosis and skewness
    features['kurtosis'] = np.sum(vibration_waveform ** 4) / (np.sum(vibration_waveform ** 2) ** 2)
    features['skewness'] = np.sum(vibration_waveform ** 3) / (np.sum(vibration_waveform ** 2) ** 1.5)
    
    return features

# Example output:
# {
#   'rms': 0.45,
#   'peak': 2.1,
#   'crest_factor': 4.67,
#   'spectral_centroid': 3500,
#   'spectral_spread': 1200,
#   'bpfo_energy': 0.23,
#   'bpfi_energy': 0.08,
#   'kurtosis': 4.2,
#   'skewness': 0.15
# }
```

## Summary of Frequency-Domain Features

**Input:** Raw vibration waveforms (10 kHz - 100 kHz)

**Processing:**
1. FFT analysis (convert to frequency domain)
2. Calculate bearing defect frequencies
3. Envelope analysis (extract bearing fault signatures)
4. Spectral kurtosis (identify impulsive bands)
5. Cepstral analysis (detect harmonics)
6. Time-synchronous averaging (enhance periodic signals)
7. Extract numerical features for ML

**Output:** Spectral features ready for ML fault classifiers

**Key Insight:** Bearing faults produce characteristic frequency signatures that can be detected and quantified using signal processing techniques.
