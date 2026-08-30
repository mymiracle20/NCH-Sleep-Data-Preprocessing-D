# NCH-Sleep-Data-Preprocessing-D
Desaturated vs. undesaturated patients를 대상으로 Sleep Stage 분포를 matching한 후, patient-level 4-fold cross-validation을 통해 EEG 기반 분류 성능을 평가하는 실험. 60/120 Hz noise 제거 및 STFT 전처리 후 ResNet18을 적용하고, Balanced Accuracy와 AUC를 기준으로 성능을 평가한다.


# Experiment A — Stage-Matched Patient-Level Classification

## Overview

This experiment evaluates whether EEG signals can distinguish **desaturated patients (Positive)** from **undesaturated patients (Negative)** while controlling for differences in sleep-stage distribution.

To reduce potential confounding caused by different sleep-stage compositions between the two groups, Negative EEG epochs are **stage-matched to the Positive EEG epoch distribution**.

The classification is performed at the **patient level using 4-fold cross-validation**, with ResNet18 applied to STFT-transformed multi-channel EEG signals.

## Experimental Design

### Positive

* Desaturated patients
* EEG epochs corresponding to oxygen desaturation
* Label: `1`
* Number of patients: **64**

### Negative

* Undesaturated patients
* Normal sleep EEG epochs
* Label: `0`
* Number of patients: **7**

### Sleep Stages

The following sleep stages are included:

* N1
* N2
* N3
* REM

Negative epochs are sampled to match the sleep-stage distribution of the corresponding Positive epochs.

## Preprocessing

Each EEG epoch is processed as follows:

1. **60 Hz notch filtering**

   * 3rd-order Butterworth band-stop filter
   * Frequency range: 59–61 Hz

2. **120 Hz notch filtering**

   * 3rd-order Butterworth band-stop filter
   * Frequency range: 119–121 Hz

3. **STFT**

   * Sampling frequency: 256 Hz
   * Window: Hann
   * `nperseg = 256`
   * `noverlap = 128`

The resulting STFT magnitude is used as the model input.

## EEG Channels

Six EEG channels are used:

* F3-M2
* F4-M1
* C3-M2
* C4-M1
* O1-M2
* O2-M1

## Patient-Level 4-Fold Cross-Validation

Patient IDs are split into four folds separately for Positive and Negative subjects.

For each fold:

* **Test:** current fold
* **Validation:** next fold
* **Train:** remaining two folds

Patient-level splitting is used to prevent epochs from the same patient from appearing across different datasets.

### Epoch Sampling

For training:

* Maximum **30 epochs per patient** for Positive subjects
* Negative candidate epochs are also limited to 30 per patient
* Negative epochs are then sampled according to the Positive sleep-stage distribution

For validation and testing:

* All Positive epochs are used
* Negative epochs are sampled to match the Positive sleep-stage distribution

## Model

### ResNet18

A standard ResNet18 architecture is adapted for six-channel EEG input.

* Input channels: `6`
* Output classes: `2`
* Optimizer: Adam
* Learning rate: `1e-3`
* Weight decay: `1e-4`
* Batch size: `16`
* Maximum epochs: `100`

Class-weighted cross-entropy loss is used to account for class imbalance.

## Model Selection

The model checkpoint with the highest **Validation Balanced Accuracy (BA)** is selected for each fold.

The selected model is then evaluated on the corresponding test set.

## Evaluation Metrics

The following metrics are reported:

* **Balanced Accuracy**
* **AUC**
* **Confusion Matrix**

Final performance is summarized by the mean and standard deviation across the four folds.

## Output

Fold-level results are saved as:

`experiment_A_stage_matched_results.csv`

The output contains:

* Fold number
* Best epoch
* Validation Balanced Accuracy
* Validation AUC
* Test Balanced Accuracy
* Test AUC
* Test negative patients
* Number of Positive/Negative training epochs
* Number of Positive/Negative validation epochs
* Number of Positive/Negative test epochs
* Confusion Matrix

## Summary

**Experiment A** tests EEG-based patient classification while explicitly controlling for **sleep-stage distribution differences** between desaturated and undesaturated subjects.

The key experimental strategy is:

**Patient-level split → Sleep-stage matching → STFT preprocessing → ResNet18 → Validation-based model selection → Test evaluation**
