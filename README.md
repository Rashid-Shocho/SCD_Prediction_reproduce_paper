# Implementation Notes, Assumptions & Methodological Decisions

> This document describes how the implementation was performed, which parts exactly follow the original paper, where assumptions were required due to insufficient methodological details, and which engineering decisions were taken during development.

---

# Project Status

Current Progress:

- ✅ Notebook 1 — Dataset Download
- ✅ Notebook 2 — ECG Extraction
- ✅ Notebook 3 — ECG Segmentation
- ✅ Notebook 4 — Dataset Preparation
- ✅ Notebook 5 — Hilbert Spectrum Generation
- ✅ Notebook 6 — Wavelet Scalogram Generation
- ✅ Notebook 7 — Multimodal Model (Paper Reproduction)

---

# Original Paper Pipeline

The original paper proposes a multimodal deep learning framework using three modalities:

1. Raw ECG Signal
2. Hilbert Spectrum
3. Wavelet Scalogram

These three modalities are fused inside a multimodal deep learning architecture for Sudden Cardiac Death prediction.

---

# Our Reproduction Pipeline

```
Notebook 1
    ↓
Dataset Download

Notebook 2
    ↓
ECG Extraction

Notebook 3
    ↓
Signal Segmentation

Notebook 4
    ↓
Prepared Dataset (.npy)

Notebook 5
    ↓
Hilbert Spectrum Images

Notebook 6
    ↓
Wavelet Scalogram Images

Notebook 7
    ↓
Multimodal Deep Learning Model
```

---

# Reproduced Components

The following parts were implemented according to the paper as closely as possible.

## Dataset

- Same ECG records
- Same prediction horizons
- Same balanced dataset

---

## ECG Segment Length

Paper:

- 500 samples

Implementation:

- 500 samples

Status:

✅ Same

---

## Hilbert Spectrum

Implemented using

- Empirical Mode Decomposition (EMD)
- Hilbert Transform
- Instantaneous Frequency
- Instantaneous Amplitude

Status:

✅ Same methodology

---

## Wavelet Scalogram

Implemented using

- Continuous Wavelet Transform
- Sym4 Wavelet
- Jet Color Mapping

Status:

✅ Same methodology

---

## Multimodal Network

Three input branches

- Raw ECG
- Hilbert Spectrum
- Scalogram

Feature fusion follows the architecture described in the paper.

Status:

✅ Same concept

---

## Training

Implemented

- Adam Optimizer
- Learning Rate Scheduler
- Early Stopping
- Best Model Saving

Status:

Minor engineering improvements only.

---

# Engineering Decisions

Several implementation details were not fully described inside the paper.

Those decisions are documented below.

---

## 1. Missing EMD Implementation Details

The paper does not specify

- EMD library
- Stopping criterion
- Boundary handling
- Maximum IMF count

Decision:

- Python EMD implementation
- Maximum IMF = 6

Reason:

Provides reproducible decomposition while limiting computation.

---

## 2. Hilbert Spectrum Resolution

The paper does not specify

- Frequency bin count
- Image resolution
- Spectrum normalization

Decision

- 224×224 output images
- Gaussian smoothing
- Frequency clipping

Reason

Compatible with CNN architectures and reproducible.

---

## 3. Wavelet Parameters

The paper mentions wavelet processing but does not completely specify

- Continuous vs Discrete Wavelet
- Image generation pipeline
- Plotting configuration

Decision

Continuous Wavelet Transform

Reason

Produces scalograms compatible with CNN input.

---

## 4. ECG Denoising

Paper references wavelet denoising but omits implementation details.

Decision

- Sym4
- Threshold = 0.04

Reason

Matches available description from the paper.

---

## 5. Image Resolution

Paper does not specify image size.

Decision

224 × 224

Reason

Standard ImageNet resolution.

---

## 6. Training Improvements

Additional engineering features added

- Early Stopping
- Automatic Best Model Saving
- Learning Rate Scheduler

These additions do not modify the proposed architecture.

They only improve reproducibility.

---

# Assumptions

The following assumptions were necessary due to insufficient methodological details.

| Component | Assumption |
|------------|------------|
| Maximum IMF | 6 |
| Wavelet plotting | Matplotlib |
| Hilbert normalization | Gaussian smoothing |
| Image size | 224×224 |
| Data loading | NumPy arrays |
| Training scheduler | ReduceLROnPlateau |
| Early stopping | Patience = 10 |

---

# Problems Encountered During Implementation

## Patient 41

One ECG record contained corrupted values.

Implemented automatic signal repair before EMD.

This issue was not discussed in the paper.

---

## Long Hilbert Generation Time

Original implementation was computationally expensive.

Pipeline optimized while preserving generated output.

---

## Kaggle Runtime Interruptions

Long preprocessing occasionally exceeded runtime limits.

Generation pipeline modified to support resumable execution.

---

# Methodological Concern Identified

During reproduction, an important methodological concern was observed.

Current Notebook 7 performs

```
Random Sample Split
```

instead of

```
Patient-wise Split
```

This produces unrealistically high validation performance.

Example

```
Validation Accuracy = 100%
Validation F1 = 100%
```

after only a few epochs.

This strongly suggests that ECG segments originating from the same patient may appear in both training and validation datasets.

This evaluation protocol may overestimate real-world performance because ECG morphology is highly patient-specific.

---

# Planned Improvement

A second experimental pipeline will be implemented.

Instead of changing the reproduced implementation, a new experiment will preserve patient identity throughout preprocessing.

Pipeline

```
Notebook 4 (Patient IDs)
        ↓
Notebook 5
        ↓
Notebook 6
        ↓
Notebook 8
```

Training will use

- GroupShuffleSplit

or

- GroupKFold

to ensure

```
No patient appears in multiple dataset splits.
```

This provides a more realistic estimate of generalization performance.

---

# Reproducibility

Random seeds are fixed for

- Python
- NumPy
- PyTorch

CUDA deterministic mode enabled.

Best model checkpoints are automatically saved.

---

# Current Conclusion

The implementation successfully reproduces the proposed multimodal architecture.

However, the observed validation performance suggests that the evaluation protocol should be further investigated using patient-wise dataset partitioning.

The reproduced implementation is therefore preserved as the baseline experiment, while a second patient-independent evaluation will be developed for comparison.
