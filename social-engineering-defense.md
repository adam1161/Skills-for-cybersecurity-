

===== FILE: skills/detecting-deepfake-audio-in-vishing-attacks/SKILL.md =====
---
name: detecting-deepfake-audio-in-vishing-attacks
description: Detect AI-generated deepfake audio used in voice phishing (vishing) by extracting spectral features (MFCC, spectral centroid, spectral contrast, zero-crossing rate) and classifying samples with machine learning models, supporting batch audio analysis, confidence scoring, and forensic reporting. Use for deepfake voice detection, vishing investigations, AI-generated speech analysis, voice cloning detection, or audio authenticity verification.
domain: cybersecurity
subdomain: social-engineering-defense
tags:
- deepfake-detection
- vishing
- audio-forensics
- MFCC
- spectral-analysis
- voice-cloning
version: 1.0.0
author: mukul975
license: Apache-2.0
atlas_techniques:
- AML.T0088
- AML.T0043
- AML.T0018
- AML.T0052
nist_ai_rmf:
- MEASURE-2.7
- GOVERN-6.2
- MAP-5.2
- MEASURE-2.5
- MAP-5.1
d3fend_techniques:
- Sender Reputation Analysis
- Content Validation
- Message Analysis
- User Behavior Analysis
- Identifier Analysis
nist_csf:
- PR.AT-01
- DE.CM-09
- RS.CO-02
mitre_attack:
- T1078
- T1190
- T1059
- T1566
- T1598
mitre_f3:
  version: '1.1'
  tactics:
  - reconnaissance
  - initial-access
  - stealth
  - monetization
  techniques:
  - id: F1032
    name: Impersonate Official
    tactic: initial-access
    source: f3
  - id: F1031
    name: Impersonate Account Holder
    tactic: initial-access
    source: f3
  - id: F1040
    name: Phone Number Spoofing
    tactic: stealth
    source: f3
  - id: F1034
    name: Interactive Voice Response Mapping
    tactic: reconnaissance
    source: f3
  - id: F1025.003
    name: 'Electronic Funds Transfer: Wire Transfer'
    tactic: monetization
    source: f3
---

# Detecting Deepfake Audio in Vishing Attacks

## When to Use

- A suspected vishing call used an AI-cloned executive voice to authorize a wire transfer
- Security operations received a voicemail that sounds like the CEO but the tone seems off
- Incident response needs to determine whether a recorded phone call contains synthetic speech
- Fraud investigation requires forensic proof that audio was AI-generated
- Red team exercises use voice cloning and blue team needs detection capability

**Do not use** for text-based phishing (email/SMS); use email header analysis or URL detonation tools instead.

## Prerequisites

- Python 3.9+ with librosa, numpy, scikit-learn, and scipy installed
- Audio samples in WAV, MP3, or FLAC format (mono or stereo, any sample rate)
- Reference corpus of known genuine voice samples for the targeted individual (optional but improves accuracy)
- FFmpeg installed for audio format conversion (librosa dependency)
- Minimum 3 seconds of audio for reliable feature extraction

## Workflow

### Step 1: Audio Preprocessing

Normalize and prepare audio samples for feature extraction:

```python
import librosa
import numpy as np

# Load audio, resample to 16kHz mono
y, sr = librosa.load("suspect_call.wav", sr=16000, mono=True)

# Trim silence from beginning and end
y_trimmed, _ = librosa.effects.trim(y, top_db=25)

# Normalize amplitude to [-1, 1]
y_norm = y_trimmed / np.max(np.abs(y_trimmed))
```

Audio preprocessing ensures consistent feature extraction across different recording conditions, microphones, and codec artifacts.

### Step 2: Extract Spectral Features

Extract the feature set that distinguishes real from synthetic speech:

**Mel-Frequency Cepstral Coefficients (MFCCs):**
```python
# Extract 20 MFCCs + delta and delta-delta
mfccs = librosa.feature.mfcc(y=y_norm, sr=sr, n_mfcc=20)
mfcc_delta = librosa.feature.delta(mfccs)
mfcc_delta2 = librosa.feature.delta(mfccs, order=2)
```

MFCCs capture the spectral envelope of speech, representing how the vocal tract shapes sound. Deepfake audio often shows unnatural smoothness in higher-order MFCCs because neural vocoders approximate but do not perfectly replicate the acoustic resonance of a physical vocal tract.

**Spectral Features:**
```python
spectral_centroid = librosa.feature.spectral_centroid(y=y_norm, sr=sr)
spectral_bandwidth = librosa.feature.spectral_bandwidth(y=y_norm, sr=sr)
spectral_contrast = librosa.feature.spectral_contrast(y=y_norm, sr=sr)
spectral_rolloff = librosa.feature.spectral_rolloff(y=y_norm, sr=sr)
zero_crossing_rate = librosa.feature.zero_crossing_rate(y_norm)
```

**Key indicators of deepfake audio:**
- Reduced spectral contrast in the 4-8 kHz range (vocoders compress high-frequency detail)
- Abnormally consistent spectral centroid over time (real speech has natural variation)
- Lower zero-crossing rate variance (synthetic speech lacks micro-perturbations)
- Missing or attenuated formant transitions during consonant-vowel boundaries

### Step 3: Build Feature Vector and Classify

Aggregate frame-level features into a fixed-length vector and classify:

```python
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.model_selection import cross_val_score

def build_feature_vector(y, sr):
    features = []
    mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=20)
    for coeff in mfccs:
        features.extend([np.mean(coeff), np.std(coeff), np.min(coeff), np.max(coeff)])
    for feat_fn in [librosa.feature.spectral_centroid,
                    librosa.feature.spectral_bandwidth,
                    librosa.feature.spectral_rolloff,
                    librosa.feature.zero_crossing_rate]:
        feat = feat_fn(y=y, sr=sr) if feat_fn != librosa.feature.zero_crossing_rate else feat_fn(y)
        features.extend([np.mean(feat), np.std(feat), np.min(feat), np.max(feat)])
    contrast = librosa.feature.spectral_contrast(y=y, sr=sr)
    for band in contrast:
        features.extend([np.mean(band), np.std(band)])
    return np.array(features)
```

Classification uses an ensemble approach: Random Forest for robustness and Gradient Boosting for accuracy, with a voting mechanism to reduce false positives.

### Step 4: Temporal Artifact Analysis

Examine time-domain artifacts that neural vocoders leave behind:

```python
# Pitch stability analysis - deepfakes often have unnaturally stable F0
f0, voiced_flag, voiced_probs = librosa.pyin(y_norm, fmin=50, fmax=500, sr=sr)
f0_clean = f0[~np.isnan(f0)]
pitch_std = np.std(f0_clean) if len(f0_clean) > 0 else 0
pitch_jitter = np.mean(np.abs(np.diff(f0_clean))) if len(f0_clean) > 1 else 0
```

Real human speech exhibits natural pitch jitter (micro-variations in fundamental frequency) and shimmer (amplitude perturbations). Deepfake audio generated by Tacotron 2, VALL-E, or ElevenLabs typically shows reduced jitter and shimmer compared to genuine speech.

### Step 5: Spectrogram Visual Inspection

Generate spectrograms for manual forensic review:

```python
import librosa.display
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 2, figsize=(14, 10))
librosa.display.specshow(librosa.power_to_db(librosa.feature.melspectrogram(y=y_norm, sr=sr)),
                         sr=sr, ax=axes[0, 0], x_axis='time', y_axis='mel')
axes[0, 0].set_title('Mel Spectrogram')
librosa.display.specshow(mfccs, sr=sr, ax=axes[0, 1], x_axis='time')
axes[0, 1].set_title('MFCCs')
```

Visual inspection reveals banding artifacts in mel spectrograms, unnatural energy cutoffs above the vocoder's frequency ceiling, and periodic noise patterns in the high-frequency range that are characteristic of neural speech synthesis.

### Step 6: Generate Forensic Report

Compile findings into an actionable report:

```
DEEPFAKE AUDIO ANALYSIS REPORT
================================
File:              suspect_executive_call.wav
Duration:          47.3 seconds
Sample Rate:       16000 Hz
Analysis Date:     2026-03-19

CLASSIFICATION RESULT
Verdict:           LIKELY DEEPFAKE (confidence: 94.2%)
Ensemble Score:    RF=0.91, GBT=0.97, Avg=0.94

FEATURE ANOMALIES DETECTED
- MFCC variance in coefficients 13-20: 62% below genuine baseline
- Spectral contrast (4-8 kHz): 0.23 (genuine avg: 0.41)
- Pitch jitter: 0.8 Hz (genuine avg: 2.4 Hz)
- Zero-crossing rate std: 0.003 (genuine avg: 0.011)

SPECTROGRAM ARTIFACTS
- Energy cutoff above 7.8 kHz (consistent with neural vocoder ceiling)
- Banding pattern at 50ms intervals in mel spectrogram
- Missing formant transitions at 12.4s, 23.1s, 35.7s timestamps

RECOMMENDATION
High confidence of AI-generated audio. Recommend out-of-band
verification with the purported speaker. Preserve original audio
file with chain of custody documentation for potential legal action.
```

## Key Concepts

| Term | Definition |
|------|------------|
| **MFCC** | Mel-Frequency Cepstral Coefficients; representation of the short-term power spectrum on a mel (perceptual) frequency scale |
| **Spectral Centroid** | Weighted mean of frequencies present in the signal; indicates perceived brightness of a sound |
| **Spectral Contrast** | Difference in amplitude between peaks and valleys in the spectrum across frequency sub-bands |
| **Vocoder** | Signal processing component that synthesizes audio waveforms from acoustic features; used in TTS and voice cloning |
| **Pitch Jitter** | Cycle-to-cycle variation in fundamental frequency; natural in human speech, reduced in synthetic speech |
| **Vishing** | Voice phishing; social engineering attack conducted via phone calls, increasingly using AI-cloned voices |
| **Formant** | Resonant frequencies of the vocal tract that define vowel sounds; transitions between formants are difficult for AI to replicate perfectly |

## Tools & Systems

- **librosa**: Python library for audio analysis providing MFCC, spectral feature extraction, and spectrogram generation
- **scikit-learn**: Machine learning library used for Random Forest and Gradient Boosting classification
- **Resemblyzer**: Speaker embedding library for comparing voice identity between known genuine and suspect samples
- **Speechbrain**: Deep learning toolkit for speech processing with pretrained deepfake detection models
- **Praat**: Phonetics software for detailed pitch, jitter, and shimmer analysis of speech samples
- **FFmpeg**: Audio format conversion and preprocessing utility required by librosa

## Common Scenarios

### Scenario: Executive Impersonation Wire Transfer Fraud

**Context**: CFO receives a phone call appearing to be from the CEO requesting an urgent wire transfer of $2.3M. The call came from an unknown number but the voice sounded identical to the CEO. IT security was able to obtain a recording of the call from the phone system.

**Approach**:
1. Extract the audio from the phone system recording and convert to WAV at 16kHz
2. Run MFCC and spectral feature extraction on the suspect audio
3. Compare against known genuine CEO voice samples from recorded meetings
4. Analyze pitch jitter and shimmer against human speech baselines
5. Classify using the trained ensemble model and generate confidence score
6. Produce forensic report with spectrogram evidence for legal/compliance

**Pitfalls**:
- Phone codec compression (G.711, AMR) degrades audio quality and can mask deepfake artifacts
- Short audio clips (under 3 seconds) produce unreliable feature statistics
- Background noise from the call environment can reduce classification accuracy
- Highly sophisticated voice cloning (e.g., fine-tuned VALL-E with 30+ minutes of training data) may evade basic feature analysis
- Genuine speech transmitted through VoIP may exhibit spectral artifacts similar to deepfakes

===== END FILE =====


===== FILE: skills/detecting-deepfake-audio-in-vishing-attacks/LICENSE =====

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to the Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by the Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding any notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. Please do not remove or change
      the license header comment from a contributed file except when
      necessary.

   Copyright 2026 mukul975

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

===== END FILE =====


===== FILE: skills/detecting-deepfake-audio-in-vishing-attacks/references/api-reference.md =====
# API Reference: Deepfake Audio Detection

## librosa - Audio Feature Extraction

### Loading and Preprocessing
```python
import librosa

# Load audio with resampling
y, sr = librosa.load("file.wav", sr=16000, mono=True)

# Trim silence (top_db = threshold in dB below peak)
y_trimmed, index = librosa.effects.trim(y, top_db=25)
```

### MFCC Extraction
```python
# Extract n MFCCs per frame
mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=20, hop_length=512, n_fft=2048)
# Returns: numpy array of shape (n_mfcc, num_frames)

# Delta (first derivative) and delta-delta (second derivative)
mfcc_delta = librosa.feature.delta(mfccs)
mfcc_delta2 = librosa.feature.delta(mfccs, order=2)
```

### Spectral Features
```python
# Spectral centroid - "center of mass" of the spectrum
centroid = librosa.feature.spectral_centroid(y=y, sr=sr)

# Spectral bandwidth - weighted standard deviation of frequencies
bandwidth = librosa.feature.spectral_bandwidth(y=y, sr=sr)

# Spectral contrast - difference between peaks and valleys per sub-band
contrast = librosa.feature.spectral_contrast(y=y, sr=sr)
# Returns: shape (n_bands + 1, num_frames), default 7 bands

# Spectral rolloff - frequency below which 85% of energy is concentrated
rolloff = librosa.feature.spectral_rolloff(y=y, sr=sr)

# Spectral flatness - measure of noisiness vs tonality (0=tonal, 1=noise)
flatness = librosa.feature.spectral_flatness(y=y)

# Zero-crossing rate - rate of sign changes in the signal
zcr = librosa.feature.zero_crossing_rate(y, hop_length=512)
```

### Pitch Estimation (pYIN Algorithm)
```python
# Fundamental frequency estimation using probabilistic YIN
f0, voiced_flag, voiced_probs = librosa.pyin(
    y, fmin=50, fmax=500, sr=sr, hop_length=512
)
# f0: numpy array with NaN for unvoiced frames
# voiced_flag: boolean array
# voiced_probs: probability of voicing per frame
```

### Mel Spectrogram
```python
# Compute mel-scaled spectrogram
mel_spec = librosa.feature.melspectrogram(y=y, sr=sr, n_mels=128)

# Convert to dB scale for visualization
mel_db = librosa.power_to_db(mel_spec, ref=np.max)
```

### Onset Detection
```python
# Onset strength envelope
onset_env = librosa.onset.onset_strength(y=y, sr=sr)

# Tempo estimation
tempo = librosa.feature.tempo(onset_envelope=onset_env, sr=sr)
```

## scikit-learn - ML Classification

### Random Forest Classifier
```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(
    n_estimators=200,    # number of trees
    max_depth=15,        # max tree depth
    random_state=42,
    n_jobs=-1            # use all CPU cores
)
rf.fit(X_train, y_train)
proba = rf.predict_proba(X_test)  # returns [P(genuine), P(deepfake)]
```

### Gradient Boosting Classifier
```python
from sklearn.ensemble import GradientBoostingClassifier

gbt = GradientBoostingClassifier(
    n_estimators=150,
    max_depth=5,
    learning_rate=0.1,
    random_state=42
)
gbt.fit(X_train, y_train)
proba = gbt.predict_proba(X_test)
```

### Feature Scaling
```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

### Cross-Validation
```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
print(f"Accuracy: {scores.mean():.3f} (+/- {scores.std():.3f})")
```

## Datasets for Training

### ASVspoof Challenge
- **ASVspoof 2019 LA**: Logical access partition with TTS and voice conversion attacks
- **ASVspoof 2021**: Extended with telephony and compression conditions
- URL: https://www.asvspoof.org/
- Format: FLAC audio files with protocol files mapping utterance IDs to labels

### FakeAVCeleb
- Multimodal deepfake dataset with audio-visual content
- Contains real and deepfake celebrity audio/video
- URL: https://github.com/DASH-Lab/FakeAVCeleb

### In-the-Wild Dataset
- Real-world deepfake audio collected from social media and news
- URL: https://deepfake-demo.aisec.fraunhofer.de/in_the_wild

## Feature Importance for Deepfake Detection

Based on research from IEEE and Springer publications:

| Feature | Importance | Why |
|---------|-----------|-----|
| MFCC 13-20 variance | High | Neural vocoders smooth high-order cepstral coefficients |
| Pitch jitter | High | TTS systems produce unnaturally stable F0 contours |
| Spectral contrast (4-8kHz) | Medium | Vocoders compress high-frequency spectral detail |
| ZCR standard deviation | Medium | Synthetic speech lacks micro-perturbations |
| Spectral centroid CV | Medium | Deepfakes have more consistent spectral center |
| MFCC delta-delta | Medium | Second-order dynamics are harder for AI to replicate |
| Spectral flatness | Low | Slightly elevated in vocoder artifacts |
| RMS energy variance | Low | Some vocoders produce smoother energy contours |

## CLI Usage Examples

```bash
# Analyze a single audio file
python agent.py analyze suspect_call.wav

# Analyze with trained model
python agent.py analyze suspect_call.wav --model deepfake_model.joblib -o result.json

# Batch analyze a directory
python agent.py batch /path/to/audio/samples/ -o batch_results.json

# Train a model from labeled data
python agent.py train --genuine /data/genuine/ --deepfake /data/deepfake/ -o model.joblib

# Extract features only (for custom analysis)
python agent.py features suspect_call.wav -o features.json
```

===== END FILE =====


===== FILE: skills/detecting-deepfake-audio-in-vishing-attacks/scripts/agent.py =====
#!/usr/bin/env python3
"""Deepfake audio detection agent using spectral analysis, MFCC features, and ML classifiers.

Analyzes audio files to determine whether they contain AI-generated (deepfake) speech,
commonly used in vishing (voice phishing) attacks. Extracts spectral features with librosa,
builds feature vectors, and classifies using ensemble ML models.
"""

import os
import sys
import json
import warnings
import argparse
from pathlib import Path
from datetime import datetime

import numpy as np

try:
    import librosa
    HAS_LIBROSA = True
except ImportError:
    HAS_LIBROSA = False

try:
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.preprocessing import StandardScaler
    from sklearn.model_selection import cross_val_score
    HAS_SKLEARN = True
except ImportError:
    HAS_SKLEARN = False

warnings.filterwarnings("ignore", category=UserWarning)

# Default analysis parameters
DEFAULT_SR = 16000
DEFAULT_N_MFCC = 20
DEFAULT_HOP_LENGTH = 512
DEFAULT_N_FFT = 2048
TRIM_TOP_DB = 25
MIN_DURATION_SEC = 1.0

# Thresholds derived from research on deepfake vs genuine speech characteristics
# Based on findings from IEEE paper "Deepfake Audio Detection via MFCC Features Using ML"
DEEPFAKE_THRESHOLDS = {
    "mfcc_high_order_var_ratio": 0.5,     # deepfakes have <50% variance of genuine in MFCC 13-20
    "spectral_contrast_4_8khz": 0.30,     # genuine speech typically >0.35 in this band
    "pitch_jitter_hz": 1.5,              # genuine speech jitter typically >2.0 Hz
    "zcr_std_threshold": 0.006,          # genuine ZCR std typically >0.008
    "spectral_centroid_cv": 0.15,        # coefficient of variation; deepfakes show less variation
    "spectral_rolloff_std": 200,         # genuine rolloff std typically >300 Hz
}


def load_and_preprocess(audio_path, sr=DEFAULT_SR):
    """Load audio file, resample to target rate, trim silence, and normalize."""
    if not os.path.isfile(audio_path):
        raise FileNotFoundError(f"Audio file not found: {audio_path}")

    y, orig_sr = librosa.load(audio_path, sr=sr, mono=True)

    if len(y) / sr < MIN_DURATION_SEC:
        raise ValueError(f"Audio too short ({len(y)/sr:.1f}s). Minimum {MIN_DURATION_SEC}s required.")

    y_trimmed, trim_indices = librosa.effects.trim(y, top_db=TRIM_TOP_DB)

    if len(y_trimmed) < sr * MIN_DURATION_SEC:
        y_trimmed = y  # fall back to untrimmed if trim removes too much

    max_amp = np.max(np.abs(y_trimmed))
    if max_amp > 0:
        y_norm = y_trimmed / max_amp
    else:
        raise ValueError("Audio file contains only silence.")

    return y_norm, sr


def extract_mfcc_features(y, sr, n_mfcc=DEFAULT_N_MFCC):
    """Extract MFCC, delta, and delta-delta features with statistical aggregation."""
    mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=n_mfcc,
                                  hop_length=DEFAULT_HOP_LENGTH, n_fft=DEFAULT_N_FFT)
    mfcc_delta = librosa.feature.delta(mfccs)
    mfcc_delta2 = librosa.feature.delta(mfccs, order=2)

    features = {}
    for i, coeff_row in enumerate(mfccs):
        prefix = f"mfcc_{i}"
        features[f"{prefix}_mean"] = float(np.mean(coeff_row))
        features[f"{prefix}_std"] = float(np.std(coeff_row))
        features[f"{prefix}_min"] = float(np.min(coeff_row))
        features[f"{prefix}_max"] = float(np.max(coeff_row))
        features[f"{prefix}_skew"] = float(_safe_skew(coeff_row))
        features[f"{prefix}_kurtosis"] = float(_safe_kurtosis(coeff_row))

    for i, row in enumerate(mfcc_delta):
        features[f"mfcc_delta_{i}_mean"] = float(np.mean(row))
        features[f"mfcc_delta_{i}_std"] = float(np.std(row))

    for i, row in enumerate(mfcc_delta2):
        features[f"mfcc_delta2_{i}_mean"] = float(np.mean(row))
        features[f"mfcc_delta2_{i}_std"] = float(np.std(row))

    return features, mfccs


def extract_spectral_features(y, sr):
    """Extract spectral centroid, bandwidth, contrast, rolloff, and ZCR."""
    features = {}

    spectral_centroid = librosa.feature.spectral_centroid(y=y, sr=sr,
                                                          hop_length=DEFAULT_HOP_LENGTH)
    features["spectral_centroid_mean"] = float(np.mean(spectral_centroid))
    features["spectral_centroid_std"] = float(np.std(spectral_centroid))
    centroid_mean = features["spectral_centroid_mean"]
    features["spectral_centroid_cv"] = (
        float(features["spectral_centroid_std"] / centroid_mean) if centroid_mean > 0 else 0.0
    )

    spectral_bw = librosa.feature.spectral_bandwidth(y=y, sr=sr, hop_length=DEFAULT_HOP_LENGTH)
    features["spectral_bandwidth_mean"] = float(np.mean(spectral_bw))
    features["spectral_bandwidth_std"] = float(np.std(spectral_bw))

    spectral_contrast = librosa.feature.spectral_contrast(y=y, sr=sr,
                                                           hop_length=DEFAULT_HOP_LENGTH)
    for i, band in enumerate(spectral_contrast):
        features[f"spectral_contrast_band_{i}_mean"] = float(np.mean(band))
        features[f"spectral_contrast_band_{i}_std"] = float(np.std(band))

    # Aggregate contrast in 4-8 kHz range (bands 4-5 at 16kHz SR)
    high_band_indices = [4, 5] if spectral_contrast.shape[0] > 5 else [spectral_contrast.shape[0] - 1]
    high_contrast_vals = [np.mean(spectral_contrast[i]) for i in high_band_indices]
    features["spectral_contrast_4_8khz"] = float(np.mean(high_contrast_vals))

    spectral_rolloff = librosa.feature.spectral_rolloff(y=y, sr=sr, hop_length=DEFAULT_HOP_LENGTH)
    features["spectral_rolloff_mean"] = float(np.mean(spectral_rolloff))
    features["spectral_rolloff_std"] = float(np.std(spectral_rolloff))

    zcr = librosa.feature.zero_crossing_rate(y, hop_length=DEFAULT_HOP_LENGTH)
    features["zcr_mean"] = float(np.mean(zcr))
    features["zcr_std"] = float(np.std(zcr))

    spectral_flatness = librosa.feature.spectral_flatness(y=y, hop_length=DEFAULT_HOP_LENGTH)
    features["spectral_flatness_mean"] = float(np.mean(spectral_flatness))
    features["spectral_flatness_std"] = float(np.std(spectral_flatness))

    return features


def extract_pitch_features(y, sr):
    """Extract fundamental frequency (F0), jitter, and shimmer-like features."""
    features = {}

    f0, voiced_flag, voiced_probs = librosa.pyin(y, fmin=50, fmax=500, sr=sr,
                                                  hop_length=DEFAULT_HOP_LENGTH)
    f0_clean = f0[~np.isnan(f0)]

    if len(f0_clean) > 1:
        features["pitch_mean"] = float(np.mean(f0_clean))
        features["pitch_std"] = float(np.std(f0_clean))
        features["pitch_range"] = float(np.max(f0_clean) - np.min(f0_clean))

        # Jitter: average absolute difference between consecutive F0 values
        pitch_diffs = np.abs(np.diff(f0_clean))
        features["pitch_jitter_hz"] = float(np.mean(pitch_diffs))
        features["pitch_jitter_relative"] = float(
            np.mean(pitch_diffs) / np.mean(f0_clean) if np.mean(f0_clean) > 0 else 0
        )

        # Shimmer approximation via amplitude envelope variation at pitch periods
        features["voiced_ratio"] = float(np.sum(~np.isnan(f0)) / len(f0))
        features["voiced_prob_mean"] = float(np.mean(voiced_probs[~np.isnan(voiced_probs)]))
    else:
        features["pitch_mean"] = 0.0
        features["pitch_std"] = 0.0
        features["pitch_range"] = 0.0
        features["pitch_jitter_hz"] = 0.0
        features["pitch_jitter_relative"] = 0.0
        features["voiced_ratio"] = 0.0
        features["voiced_prob_mean"] = 0.0

    return features


def extract_temporal_features(y, sr):
    """Extract time-domain features: RMS energy, tempo, onset strength."""
    features = {}

    rms = librosa.feature.rms(y=y, hop_length=DEFAULT_HOP_LENGTH)
    features["rms_mean"] = float(np.mean(rms))
    features["rms_std"] = float(np.std(rms))

    onset_env = librosa.onset.onset_strength(y=y, sr=sr, hop_length=DEFAULT_HOP_LENGTH)
    features["onset_strength_mean"] = float(np.mean(onset_env))
    features["onset_strength_std"] = float(np.std(onset_env))

    tempo = librosa.feature.tempo(onset_envelope=onset_env, sr=sr,
                                   hop_length=DEFAULT_HOP_LENGTH)
    features["tempo"] = float(tempo[0]) if len(tempo) > 0 else 0.0

    return features


def build_full_feature_vector(audio_path, sr=DEFAULT_SR):
    """Load audio and extract the complete feature set as a dict and numpy vector."""
    y, sr = load_and_preprocess(audio_path, sr=sr)

    all_features = {}
    mfcc_feats, raw_mfccs = extract_mfcc_features(y, sr)
    all_features.update(mfcc_feats)

    spectral_feats = extract_spectral_features(y, sr)
    all_features.update(spectral_feats)

    pitch_feats = extract_pitch_features(y, sr)
    all_features.update(pitch_feats)

    temporal_feats = extract_temporal_features(y, sr)
    all_features.update(temporal_feats)

    feature_names = sorted(all_features.keys())
    feature_vector = np.array([all_features[k] for k in feature_names])

    return all_features, feature_vector, feature_names, y, sr


def heuristic_deepfake_score(features):
    """Rule-based deepfake scoring using research-backed thresholds.

    Returns a score between 0.0 (likely genuine) and 1.0 (likely deepfake)
    based on known acoustic differences between real and synthetic speech.
    """
    indicators = []

    # 1. High-order MFCC variance check (coefficients 13-19 have lower variance in deepfakes)
    high_mfcc_stds = [features.get(f"mfcc_{i}_std", 1.0) for i in range(13, 20)]
    low_mfcc_stds = [features.get(f"mfcc_{i}_std", 1.0) for i in range(1, 7)]
    if np.mean(low_mfcc_stds) > 0:
        ratio = np.mean(high_mfcc_stds) / np.mean(low_mfcc_stds)
        indicators.append(1.0 if ratio < DEEPFAKE_THRESHOLDS["mfcc_high_order_var_ratio"] else 0.0)

    # 2. Spectral contrast in 4-8 kHz
    sc_4_8 = features.get("spectral_contrast_4_8khz", 0.5)
    indicators.append(1.0 if sc_4_8 < DEEPFAKE_THRESHOLDS["spectral_contrast_4_8khz"] else 0.0)

    # 3. Pitch jitter (lower in deepfakes)
    jitter = features.get("pitch_jitter_hz", 3.0)
    indicators.append(1.0 if jitter < DEEPFAKE_THRESHOLDS["pitch_jitter_hz"] else 0.0)

    # 4. Zero-crossing rate standard deviation
    zcr_std = features.get("zcr_std", 0.01)
    indicators.append(1.0 if zcr_std < DEEPFAKE_THRESHOLDS["zcr_std_threshold"] else 0.0)

    # 5. Spectral centroid coefficient of variation
    centroid_cv = features.get("spectral_centroid_cv", 0.3)
    indicators.append(1.0 if centroid_cv < DEEPFAKE_THRESHOLDS["spectral_centroid_cv"] else 0.0)

    # 6. Spectral rolloff stability
    rolloff_std = features.get("spectral_rolloff_std", 500)
    indicators.append(1.0 if rolloff_std < DEEPFAKE_THRESHOLDS["spectral_rolloff_std"] else 0.0)

    if not indicators:
        return 0.5

    # Weighted average: MFCC and pitch jitter are stronger signals
    weights = [1.5, 1.0, 1.5, 0.8, 1.0, 0.8]
    weights = weights[:len(indicators)]
    score = np.average(indicators, weights=weights)
    return float(np.clip(score, 0.0, 1.0))


def classify_with_ensemble(feature_vector, model_path=None):
    """Classify audio using pre-trained ensemble models if available.

    Falls back to heuristic scoring if no trained model is found.
    Returns dict with model predictions and confidence.
    """
    if model_path and os.path.isfile(model_path):
        try:
            import joblib
            model_data = joblib.load(model_path)
            scaler = model_data["scaler"]
            rf_model = model_data["random_forest"]
            gbt_model = model_data["gradient_boosting"]

            X_scaled = scaler.transform(feature_vector.reshape(1, -1))
            rf_prob = rf_model.predict_proba(X_scaled)[0][1]
            gbt_prob = gbt_model.predict_proba(X_scaled)[0][1]
            ensemble_prob = (rf_prob + gbt_prob) / 2.0

            return {
                "method": "trained_ensemble",
                "random_forest_score": float(rf_prob),
                "gradient_boosting_score": float(gbt_prob),
                "ensemble_score": float(ensemble_prob),
                "verdict": "LIKELY DEEPFAKE" if ensemble_prob > 0.5 else "LIKELY GENUINE",
            }
        except Exception as e:
            print(f"[WARN] Failed to load model from {model_path}: {e}", file=sys.stderr)

    return None


def train_model(genuine_dir, deepfake_dir, output_path):
    """Train ensemble classifier on directories of genuine and deepfake audio samples.

    Expects two directories containing WAV/MP3/FLAC files:
    - genuine_dir: directory of known real speech samples
    - deepfake_dir: directory of known AI-generated speech samples

    Saves trained model (scaler + RF + GBT) to output_path via joblib.
    """
    if not HAS_SKLEARN:
        print("[ERROR] scikit-learn required for training. Install with: pip install scikit-learn",
              file=sys.stderr)
        return None

    try:
        import joblib
    except ImportError:
        print("[ERROR] joblib required for model serialization. Install with: pip install joblib",
              file=sys.stderr)
        return None

    X, y_labels = [], []
    audio_extensions = {".wav", ".mp3", ".flac", ".ogg", ".m4a"}

    for label, directory in [(0, genuine_dir), (1, deepfake_dir)]:
        if not os.path.isdir(directory):
            print(f"[ERROR] Directory not found: {directory}", file=sys.stderr)
            return None
        for fname in os.listdir(directory):
            if Path(fname).suffix.lower() in audio_extensions:
                fpath = os.path.join(directory, fname)
                try:
                    _, fv, _, _, _ = build_full_feature_vector(fpath)
                    X.append(fv)
                    y_labels.append(label)
                    print(f"  Processed: {fname} (label={'deepfake' if label else 'genuine'})")
                except Exception as e:
                    print(f"  [WARN] Skipping {fname}: {e}", file=sys.stderr)

    if len(X) < 10:
        print(f"[ERROR] Need at least 10 samples, got {len(X)}. Add more audio files.",
              file=sys.stderr)
        return None

    X = np.array(X)
    y_labels = np.array(y_labels)

    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)

    rf = RandomForestClassifier(n_estimators=200, max_depth=15, random_state=42, n_jobs=-1)
    gbt = GradientBoostingClassifier(n_estimators=150, max_depth=5, learning_rate=0.1,
                                      random_state=42)

    print("\n[INFO] Training Random Forest...")
    rf_scores = cross_val_score(rf, X_scaled, y_labels, cv=min(5, len(X) // 2), scoring="accuracy")
    print(f"  RF Cross-val accuracy: {np.mean(rf_scores):.3f} (+/- {np.std(rf_scores):.3f})")

    print("[INFO] Training Gradient Boosting...")
    gbt_scores = cross_val_score(gbt, X_scaled, y_labels, cv=min(5, len(X) // 2), scoring="accuracy")
    print(f"  GBT Cross-val accuracy: {np.mean(gbt_scores):.3f} (+/- {np.std(gbt_scores):.3f})")

    rf.fit(X_scaled, y_labels)
    gbt.fit(X_scaled, y_labels)

    model_data = {
        "scaler": scaler,
        "random_forest": rf,
        "gradient_boosting": gbt,
        "feature_count": X_scaled.shape[1],
        "training_samples": len(X),
        "trained_at": datetime.utcnow().isoformat(),
    }
    joblib.dump(model_data, output_path)
    print(f"\n[OK] Model saved to {output_path}")
    return model_data


def analyze_audio(audio_path, model_path=None, output_json=None):
    """Full analysis pipeline: load, extract features, classify, and report."""
    print(f"\n{'='*60}")
    print(f"DEEPFAKE AUDIO ANALYSIS")
    print(f"{'='*60}")
    print(f"File:           {audio_path}")
    print(f"Analysis Date:  {datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')}")

    features, feature_vector, feature_names, y, sr = build_full_feature_vector(audio_path)
    duration = len(y) / sr
    print(f"Duration:       {duration:.1f} seconds")
    print(f"Sample Rate:    {sr} Hz")
    print(f"Features:       {len(feature_names)} extracted")

    # Try trained model first, fall back to heuristic
    ml_result = classify_with_ensemble(feature_vector, model_path)
    heuristic_score = heuristic_deepfake_score(features)

    if ml_result:
        print(f"\n--- ML Classification (Trained Model) ---")
        print(f"Random Forest:     {ml_result['random_forest_score']:.3f}")
        print(f"Gradient Boosting: {ml_result['gradient_boosting_score']:.3f}")
        print(f"Ensemble Score:    {ml_result['ensemble_score']:.3f}")
        print(f"Verdict:           {ml_result['verdict']}")
        final_score = ml_result["ensemble_score"]
        method = "trained_ensemble"
    else:
        print(f"\n--- Heuristic Classification (No trained model) ---")
        print(f"Heuristic Score:   {heuristic_score:.3f}")
        verdict = "LIKELY DEEPFAKE" if heuristic_score > 0.5 else "LIKELY GENUINE"
        print(f"Verdict:           {verdict}")
        final_score = heuristic_score
        method = "heuristic"

    # Print feature anomalies
    print(f"\n--- Feature Anomaly Report ---")
    anomalies = []

    jitter = features.get("pitch_jitter_hz", 0)
    if jitter < DEEPFAKE_THRESHOLDS["pitch_jitter_hz"]:
        msg = f"Pitch jitter: {jitter:.2f} Hz (below genuine threshold of {DEEPFAKE_THRESHOLDS['pitch_jitter_hz']} Hz)"
        anomalies.append(msg)
        print(f"  [!] {msg}")

    zcr_std = features.get("zcr_std", 0)
    if zcr_std < DEEPFAKE_THRESHOLDS["zcr_std_threshold"]:
        msg = f"ZCR std: {zcr_std:.4f} (below genuine threshold of {DEEPFAKE_THRESHOLDS['zcr_std_threshold']})"
        anomalies.append(msg)
        print(f"  [!] {msg}")

    sc_4_8 = features.get("spectral_contrast_4_8khz", 0)
    if sc_4_8 < DEEPFAKE_THRESHOLDS["spectral_contrast_4_8khz"]:
        msg = f"Spectral contrast (4-8kHz): {sc_4_8:.3f} (below threshold of {DEEPFAKE_THRESHOLDS['spectral_contrast_4_8khz']})"
        anomalies.append(msg)
        print(f"  [!] {msg}")

    centroid_cv = features.get("spectral_centroid_cv", 0)
    if centroid_cv < DEEPFAKE_THRESHOLDS["spectral_centroid_cv"]:
        msg = f"Spectral centroid CV: {centroid_cv:.4f} (below threshold of {DEEPFAKE_THRESHOLDS['spectral_centroid_cv']})"
        anomalies.append(msg)
        print(f"  [!] {msg}")

    if not anomalies:
        print("  No significant anomalies detected.")

    # Build result dict
    result = {
        "file": audio_path,
        "duration_seconds": duration,
        "sample_rate": sr,
        "analysis_timestamp": datetime.utcnow().isoformat(),
        "classification": {
            "method": method,
            "deepfake_score": final_score,
            "verdict": "LIKELY DEEPFAKE" if final_score > 0.5 else "LIKELY GENUINE",
            "confidence_pct": round(max(final_score, 1 - final_score) * 100, 1),
        },
        "anomalies": anomalies,
        "features": {k: round(v, 6) if isinstance(v, float) else v for k, v in features.items()},
    }

    if ml_result:
        result["classification"]["random_forest_score"] = ml_result["random_forest_score"]
        result["classification"]["gradient_boosting_score"] = ml_result["gradient_boosting_score"]

    if output_json:
        with open(output_json, "w") as f:
            json.dump(result, f, indent=2)
        print(f"\n[OK] Full results saved to {output_json}")

    return result


def batch_analyze(audio_dir, model_path=None, output_json=None):
    """Analyze all audio files in a directory."""
    audio_extensions = {".wav", ".mp3", ".flac", ".ogg", ".m4a"}
    results = []

    if not os.path.isdir(audio_dir):
        print(f"[ERROR] Directory not found: {audio_dir}", file=sys.stderr)
        return results

    audio_files = [f for f in os.listdir(audio_dir)
                   if Path(f).suffix.lower() in audio_extensions]

    if not audio_files:
        print(f"[WARN] No audio files found in {audio_dir}", file=sys.stderr)
        return results

    print(f"\n[INFO] Batch analyzing {len(audio_files)} files from {audio_dir}\n")
    for fname in sorted(audio_files):
        fpath = os.path.join(audio_dir, fname)
        try:
            result = analyze_audio(fpath, model_path=model_path)
            results.append(result)
        except Exception as e:
            print(f"\n[ERROR] Failed to analyze {fname}: {e}", file=sys.stderr)
            results.append({"file": fpath, "error": str(e)})

    # Summary
    deepfakes = sum(1 for r in results if r.get("classification", {}).get("verdict") == "LIKELY DEEPFAKE")
    genuine = sum(1 for r in results if r.get("classification", {}).get("verdict") == "LIKELY GENUINE")
    errors = sum(1 for r in results if "error" in r)

    print(f"\n{'='*60}")
    print(f"BATCH ANALYSIS SUMMARY")
    print(f"{'='*60}")
    print(f"Total Files:    {len(results)}")
    print(f"Likely Deepfake: {deepfakes}")
    print(f"Likely Genuine:  {genuine}")
    print(f"Errors:          {errors}")

    if output_json:
        with open(output_json, "w") as f:
            json.dump(results, f, indent=2)
        print(f"\n[OK] Batch results saved to {output_json}")

    return results


def _safe_skew(arr):
    """Compute skewness without scipy dependency."""
    n = len(arr)
    if n < 3:
        return 0.0
    mean = np.mean(arr)
    std = np.std(arr)
    if std == 0:
        return 0.0
    return float(np.mean(((arr - mean) / std) ** 3))


def _safe_kurtosis(arr):
    """Compute excess kurtosis without scipy dependency."""
    n = len(arr)
    if n < 4:
        return 0.0
    mean = np.mean(arr)
    std = np.std(arr)
    if std == 0:
        return 0.0
    return float(np.mean(((arr - mean) / std) ** 4) - 3.0)


def main():
    parser = argparse.ArgumentParser(
        description="Deepfake Audio Detection Agent - Analyzes audio for AI-generated speech"
    )
    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # Analyze single file
    analyze_parser = subparsers.add_parser("analyze", help="Analyze a single audio file")
    analyze_parser.add_argument("audio_path", help="Path to audio file (WAV, MP3, FLAC)")
    analyze_parser.add_argument("--model", help="Path to trained model (.joblib)")
    analyze_parser.add_argument("--output", "-o", help="Save results to JSON file")

    # Batch analyze directory
    batch_parser = subparsers.add_parser("batch", help="Analyze all audio files in a directory")
    batch_parser.add_argument("audio_dir", help="Directory containing audio files")
    batch_parser.add_argument("--model", help="Path to trained model (.joblib)")
    batch_parser.add_argument("--output", "-o", help="Save batch results to JSON file")

    # Train model
    train_parser = subparsers.add_parser("train", help="Train deepfake detection model")
    train_parser.add_argument("--genuine", required=True, help="Directory of genuine audio samples")
    train_parser.add_argument("--deepfake", required=True, help="Directory of deepfake audio samples")
    train_parser.add_argument("--output", "-o", default="deepfake_model.joblib",
                              help="Output model path (default: deepfake_model.joblib)")

    # Extract features only
    features_parser = subparsers.add_parser("features", help="Extract features and print as JSON")
    features_parser.add_argument("audio_path", help="Path to audio file")
    features_parser.add_argument("--output", "-o", help="Save features to JSON file")

    args = parser.parse_args()

    if not HAS_LIBROSA:
        print("[ERROR] librosa is required. Install with: pip install librosa", file=sys.stderr)
        sys.exit(1)

    if args.command == "analyze":
        analyze_audio(args.audio_path, model_path=args.model, output_json=args.output)

    elif args.command == "batch":
        batch_analyze(args.audio_dir, model_path=args.model, output_json=args.output)

    elif args.command == "train":
        if not HAS_SKLEARN:
            print("[ERROR] scikit-learn required. Install with: pip install scikit-learn",
                  file=sys.stderr)
            sys.exit(1)
        train_model(args.genuine, args.deepfake, args.output)

    elif args.command == "features":
        features, fv, names, _, _ = build_full_feature_vector(args.audio_path)
        output = {"file": args.audio_path, "feature_count": len(names), "features": features}
        if args.output:
            with open(args.output, "w") as f:
                json.dump(output, f, indent=2)
            print(f"[OK] Features saved to {args.output}")
        else:
            print(json.dumps(output, indent=2))

    else:
        parser.print_help()


if __name__ == "__main__":
    main()

===== END FILE =====
