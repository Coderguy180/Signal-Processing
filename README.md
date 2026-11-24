# potfolio-
Personal portfolio website showcasing my projects, skills, and experience as a software developer aspiring quant guy whilst pursuing Electronics and telecommunications.
# Earthquake Signal Processing in MATLAB

A digital signal processing (DSP) pipeline built in MATLAB to preprocess, analyze, and isolate teleseismic earthquake waveforms from background and local seismic noise. This project expands upon the seismic analysis workflow introduced in the **MathWorks Signal Processing Onramp**.

![Seismic Signal Analysis](assets/seismic_analysis.png)

## Overview
Seismic monitoring stations continuously record ground vibrations from multiple sources—ranging from nearby traffic and local micro-tremors to massive earthquakes occurring thousands of miles away. Because high-frequency energy attenuates faster over long distances through the Earth's crust, distant (**teleseismic**) earthquakes arrive predominantly as **low-frequency waves (< 2–3 Hz)**.

This repository demonstrates how to separate distant seismic events from local high-frequency interference using time-domain and frequency-domain techniques.

## Signal Processing Pipeline
1. **Preprocessing (`detrend`)**: Removes constant DC offsets and linear baseline drift introduced by seismometer hardware.
2. **Spectral Analysis (`pwelch`)**: Computes the Power Spectral Density (PSD) to identify dominant frequency bands and distinguish low-frequency earthquake energy from high-frequency noise.
3. **Digital Filtering (`lowpass`)**: Applies a zero-phase lowpass filter (`2.5 Hz` cutoff) to attenuate local vibrations while preserving the teleseismic P-wave and S-wave arrival envelope.
4. **Time-Frequency Analysis (`pspectrum`)**: Generates a spectrogram to visualize how signal energy evolves across frequency and time during the event arrival.
5. **Signal Measurements (`rms`, `max`)**: Extracts peak ground amplitude and root-mean-square (RMS) energy metrics from the cleaned waveform.

## Repository Structure
* `earthquake_analysis.m` — Self-contained MATLAB script that loads/generates seismic data, runs the DSP pipeline, prints summary metrics, and exports the visualization.
* `assets/seismic_analysis.png` — Multi-panel figure displaying the time-domain comparison, Welch PSD, and spectrogram.

## Requirements & Usage
* **MATLAB** (R2020b or newer recommended)
* **Signal Processing Toolbox** (`pwelch`, `lowpass`, `pspectrum`)

### Running the Script
1. Clone this repository:
   ```bash
   git clone https://github.com
   ```
2. Open `earthquake_analysis.m` in MATLAB and click **Run**.
3. The script will output signal metrics in the Command Window and save an updated figure to `assets/seismic_analysis.png`.
