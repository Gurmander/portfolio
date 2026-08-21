---
title: "Handling Distribution Shift in Audio Classification"
date: 2026-05-18
description: "What I learned from building a music genre classifier for noisy, distribution-shifted audio."
categories:
  - Machine Learning
tags:
  - PyTorch
  - Audio
  - Deep Learning
  - Librosa
draft: false
---

For the **Messy Mashup** challenge, the task was to classify audio into one of **10 music genres**.

The difficult part was that the training and test data were not distributed in the same way. A model that performed well on the clean training samples did not necessarily generalize well to the evaluation data.

![Mel spectrogram used during training](/images/blog/messy-mashup/spectrogram.webp)

## The problem

The training data mainly consisted of relatively clean instrument stems.

The test data was much messier. It contained mashups created using stems from different songs belonging to the same genre, along with changes such as environmental noise and tempo variation.

This created a clear **distribution shift**:

```text
Training data
Clean instrument stems
        ↓
      Model
        ↓
    Test data
Mashups + noise + other transformations
```

Training only on the original samples meant the model could learn patterns that worked well on clean audio but failed once those conditions changed.

## Making the training data more realistic

Instead of treating the training files as the only available data, I generated additional samples that were closer to what the model would encounter during testing.

The main idea was to create **synthetic mashups** by combining compatible stems from different samples of the same genre.

I also introduced environmental noise using samples from **ESC-50**.

The goal was not simply to create more training examples. It was to make the training distribution better represent the evaluation conditions.

Conceptually:

```text
Original audio
     │
     ├── Synthetic mashups
     ├── Environmental noise
     └── Spectrogram augmentation
              │
              ▼
        Expanded training set
```

## Audio representation

Instead of training directly on raw waveforms, I converted the audio into **Mel spectrograms**.

I also experimented with additional spectral information, including delta and delta-delta features, which represent how spectral values change over time.

The feature pipeline was roughly:

```text
Audio
  ↓
Mel Spectrogram
  ↓
Delta Features
  ↓
Delta-Delta Features
  ↓
Per-channel Normalization
  ↓
Model
```

I also used **SpecAugment** during training to randomly mask portions of the spectrogram.

This helped reduce the model's dependence on very specific time or frequency regions.

## Models I tried

I compared several architectures rather than relying on a single model.

### Custom CNN

A convolutional neural network was trained directly on the spectrogram representation.

This ended up being the strongest model.

**Macro F1: 0.945**

### CNN + Transformer

I also experimented with combining convolutional feature extraction with a Transformer-based component.

The idea was to let the CNN capture local spectrogram patterns while the Transformer modeled longer-range relationships.

It achieved approximately:

**Macro F1: 0.901**

### Audio Spectrogram Transformer

I also tested an Audio Spectrogram Transformer model.

Despite being the more sophisticated architecture, it performed considerably worse in this particular setup:

**Macro F1: 0.741**

This was a useful reminder that a larger or more complex model is not automatically the right model for a dataset.

## Tracking experiments

I used **Weights & Biases (W&B)** to keep track of training runs and compare model behaviour across experiments.

This became useful once I started changing several things at once:

* model architecture
* augmentation strategy
* preprocessing
* normalization
* spectrogram configuration

Without experiment tracking, it becomes surprisingly easy to forget which combination actually produced an improvement.

## Results

The best-performing approach was the custom CNN combined with the improved preprocessing and augmentation pipeline.

| Model                         |  Macro F1 |
| ----------------------------- | --------: |
| Custom CNN                    | **0.945** |
| CNN + Transformer             |    ~0.901 |
| Audio Spectrogram Transformer |    ~0.741 |

The final result ranked **86th out of 1,119 participants**, placing the solution in approximately the **top 8%**.

## What I learned

The biggest lesson from this project was that **understanding the data can matter more than increasing model complexity**.

My initial instinct was to experiment with increasingly sophisticated architectures. But the strongest result came from a comparatively straightforward CNN paired with a training pipeline that better represented the test conditions.

A few things stood out:

* Distribution shift should be treated as part of the problem, not just as a model limitation.
* Data augmentation is most useful when it reflects realistic variations the model may encounter.
* A more complex architecture does not guarantee better generalization.
* Keeping experiments reproducible and tracked becomes increasingly important as the number of variables grows.
* Looking carefully at how the evaluation data differs from the training data can influence the entire modeling strategy.

This project ended up being less about finding the most advanced model and more about designing a training process that matched the actual problem.
