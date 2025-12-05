# Colorimetry Project

## Overview

This project reconstructs true-color images from grayscale photographs taken with a monochromatic camera using three optical filters.  Since the filters are not pure RGB and the camera has its own spectral sensitivity, we apply spectral corrections to produce accurate color output.

## Problem

Simply assigning the three filtered images to R, G, and B channels produces incorrect colors because:

1. **Filter transmission** — The filters do not match ideal red, green, and blue wavelengths
2.  **Camera sensitivity** — The sensor responds differently across the spectrum

## Solution

We characterize the imaging system by measuring:

- **Camera spectral sensitivity** — How the sensor responds at each wavelength
- **Reference light spectrum** — The known spectral output of a calibration light source

By comparing perceived vs. reference light, we build a **sensitivity map** to correct the captured images.

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                      CALIBRATION PHASE                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Measure reference light spectrum (spectrometer)             │
│  2. Capture reference light with camera                         │
│  3. Compute camera spectral sensitivity curve                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      ACQUISITION PHASE                          │
├─────────────────────────────────────────────────────────────────┤
│  4. Capture 3 grayscale images with filters (F1, F2, F3)        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      PROCESSING PHASE                           │
├─────────────────────────────────────────────────────────────────┤
│  5. Apply spectral correction using sensitivity map             │
│  6. Transform filter responses to CIE XYZ color space           │
│  7. Convert XYZ to sRGB for display                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    [ Final Color Image ]
```

## Required Data

| File | Description |
|------|-------------|
| `reference_light_spectrum.csv` | Spectral power distribution of calibration light |
| `camera_response. csv` | Measured camera output under calibration light |
| `filter_X_transmission.csv` | Transmission curve for each filter (×3) |

## Color Reconstruction

The corrected pixel values are computed as:

```
C_corrected = M × [I_F1, I_F2, I_F3]^T
```

Where:
- `I_Fn` = Intensity from filter n (corrected for camera sensitivity)
- `M` = 3×3 transformation matrix (filter space → XYZ or RGB)

## Output

A single true-color image reconstructed from the three filtered grayscale captures, with spectral corrections applied. 

---

## Bee Vision Extension

This pipeline can simulate bee vision by adapting the hardware.  Bees see **UV, Blue, and Green** instead of R, G, B.  We map this to human-visible colors while preserving UV information as luminance.

### Approach

Instead of false-color, we display a **human-readable image enhanced with bee UV perception**:

- **Green, Blue** → Mapped to visible color channels
- **UV** → Mapped to **luminance**

This lets us see the world normally, but with UV-reflective areas appearing **brighter** — just as bees perceive them. For example, flower centers that reflect UV will "glow," revealing the navigational patterns bees use. 

### Hardware Requirements

| Component | Specification | Why |
|-----------|---------------|-----|
| **Camera** | UV-sensitive sensor (no UV/IR cut filter) | Standard cameras block UV; bees see down to ~300 nm |
| **Lens** | Quartz or UV-transmissive glass | Regular glass absorbs UV light |
| **UV Filter** | Bandpass ~320–400 nm | Captures UV reflectance for luminance mapping |
| **Blue Filter** | Bandpass ~400–500 nm | Matches bee "medium" receptor (~436 nm) |
| **Green Filter** | Bandpass ~500–600 nm | Matches bee "long" receptor (~544 nm) |
| **Calibration Light** | UV-inclusive source (xenon/deuterium) | Must emit UV for proper calibration |
| **Spectrometer** | UV-capable (≥300 nm) | Measure calibration light in UV range |

### Output Mapping

| Captured Channel | Mapped To |
|------------------|-----------|
| Green | Color (Green) |
| Blue | Color (Blue) |
| UV | Luminance boost |

### Result

A natural-looking image where **UV-reflective areas appear brighter**, revealing hidden patterns visible to bees but invisible to humans — like floral nectar guides. 

The processing pipeline remains identical — only the inputs and final mapping change. 
