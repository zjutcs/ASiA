#  Inferring Smartphone Application Types Via Inaudible Charging Sound: A New Acoustic Side-Channel Attack

## IEEE Franklin V. Taylor Memorial Award - Nominees   in IEEE SMC 2025, Vienna. 

## Overview

This repository contains the dataset and description for our research on acoustic side-channel attacks targeting smartphone charging scenarios. By analyzing high-frequency acoustic emissions during charging sessions, we can infer the type of application being used on the device.

## Dataset

The dataset contains 36 audio recordings captured during smartphone charging sessions across different application scenarios. 

Address: ![](https://zenodo.org/uploads/17339457)

### Recording Specifications

- **Sampling Rate**: 192 kHz (high-fidelity acoustic capture)
- **Recording**: Continuous full-session audio (no pre-sliced segments; slicing done during preprocessing)
- **Format**: WAV (uncompressed)
- **Microphone**: Double X0 pickup microphone
- **Battery Level**: All recordings at 100% capacity
- **Environment**: Controlled ambient conditions for reproducible acoustic feature extraction

### Device Specifications

**Smartphones:**

- **Redmi Note 12T**: Mid-range Android device
- **Huawei Nova 6**: Legacy device for cross-generational analysis

**Chargers:**

- **Xiaomi 67W Charger**: Fast charging (QC/PD protocols)
- **Ugreen Dual-Port 20W**: Baseline charging (17W single-port mode)

### Application Categories

| Category                | Applications                                       | Protocol                              |
| ----------------------- | -------------------------------------------------- | ------------------------------------- |
| **Games**         | Minecraft, Sky: Children of the Light, PUBG Mobile | 3D scene navigation and gameplay      |
| **Social**        | QQ, DingTalk                                       | Text messaging and transmission       |
| **Entertainment** | Douyin (TikTok), Kuaishou (Kwai)                   | Video playback with content switching |
| **Shopping**      | JD, Pinduoduo                                      | Product browsing and selection        |

### File Structure

```
dataset/
├── 01.wav ~ 36.wav    # Audio recordings
└── labels.json        # Metadata with app names and device info
```

Each entry in `labels.json` contains:

```json
{
  "app_name": "Application name",
  "device": {
    "phone": "Smartphone model",
    "charger": "Charger specification"
  }
}
```

## Prerequisites

- Python 3.7+
- NumPy
- SciPy
- librosa
- TensorFlow 2.x / Keras
- scikit-learn
- Matplotlib

## Installation

```bash
pip install numpy scipy librosa tensorflow scikit-learn matplotlib
```

## Usage

### Loading Audio Files

```python
import librosa

# Load audio file with original sampling rate
audio_path = "dataset/01.wav"
y, sr = librosa.load(audio_path, sr=None)
print(f"Sample rate: {sr} Hz")
print(f"Audio shape: {y.shape}")
```

### Audio Preprocessing

```python
import numpy as np

# Normalize audio signal
def normalize_audio(y):
    return (y - np.mean(y)) / np.std(y)

# Apply normalization
y_normalized = normalize_audio(y)
```

### Sliding Window Segmentation

```python
def slice_audio_sliding_window(y, sr, slice_duration=2, hop_duration=0.5):
    """
    Segment audio into overlapping slices
  
    Parameters:
    - y: Audio signal
    - sr: Sampling rate
    - slice_duration: Duration of each slice (seconds)
    - hop_duration: Hop size between slices (seconds)
    """
    slice_samples = sr * slice_duration
    hop_samples = int(sr * hop_duration)
  
    slices = []
    for i in range(0, len(y) - slice_samples + 1, hop_samples):
        slices.append(y[i:i + slice_samples])
  
    return slices

# Create 2-second slices with 0.5-second hop
audio_slices = slice_audio_sliding_window(y, sr, slice_duration=2, hop_duration=0.5)
print(f"Number of slices: {len(audio_slices)}")
```

### STFT Features (Transformer)

Note: Used as input to Transformer model.

```python
import librosa

def extract_stft_features(y, sr, n_fft=256, hop_length=1024):
    """
    Extract STFT magnitude spectrum
  
    Parameters:
    - y: Audio signal
    - sr: Sampling rate
    - n_fft: FFT window size
    - hop_length: Hop size for STFT
  
    Returns:
    - magnitude: 2D array (time_steps, frequency_bins)
    """
    # Normalize audio
    y_normalized = (y - np.mean(y)) / np.std(y)
  
    # Compute STFT
    stft = librosa.stft(y_normalized, n_fft=n_fft, hop_length=hop_length)
  
    # Get magnitude spectrum
    magnitude = np.abs(stft)
  
    # Transpose to (time_steps, frequency_bins)
    magnitude = magnitude.T
  
    return magnitude

# Extract STFT features
for audio_slice in audio_slices:
    features = extract_stft_features(audio_slice, sr, n_fft=256, hop_length=1024)
    print(f"Feature shape: {features.shape}")  # (time_steps, frequency_bins)
    break  # Show only first slice
```

### Handcrafted Features (HF) for RF

Note: HF features (e.g., MFCC, ZCR, RMS) are used with Random Forest (RF).

```python
# Extract MFCCs
mfccs = librosa.feature.mfcc(y=y_normalized, sr=sr, n_mfcc=13)
print(f"MFCCs shape: {mfccs.shape}")

# Extract zero-crossing rate
zcr = librosa.feature.zero_crossing_rate(y_normalized)
print(f"ZCR shape: {zcr.shape}")

# Extract RMS energy
rms = librosa.feature.rms(y=y_normalized)
print(f"RMS shape: {rms.shape}")
```

## Dataset Statistics

- **Total Files**: 36 WAV files
- **Applications**: 9 different apps
- **Device Combinations**: 2 phones × 2 chargers = 4 combinations per app
- **Categories**: 4 application types (Games, Social, Entertainment, Shopping)
- **Total Slices**: 8,014 two-second audio slices after sliding window segmentation
- **Total Duration**: 4,352 seconds (~72.5 minutes)

## Methodology

1. **Signal Acquisition**: Audio captured via USB-connected microphone at 192 kHz
2. **Preprocessing**: Signal normalization and segmentation
3. **Feature Extraction**: STFT-based spectral analysis
4. **Classification**: Deep learning models (e.g., Transformer-based architecture)

## Citation

If you use this dataset in your research, please cite:

```
"Inferring Smartphone Application Types via Inaudible Charging Sound: 
A New Acoustic Side-channel Attack"
```

## License

This dataset is provided for academic research purposes only.
