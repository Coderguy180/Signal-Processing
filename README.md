# Earthquake Signal Processing in MATLAB

A digital signal processing (DSP) pipeline built in MATLAB to preprocess, analyze, and isolate teleseismic earthquake waveforms from background and local seismic noise. This project expands upon the seismic analysis workflow introduced in the **MathWorks Signal Processing Onramp**.

## Signal Visualization
![Seismic Signal Analysis](assets/image_b55312f7.jpg)

## Overview
Seismic monitoring stations continuously record ground vibrations from multiple sources—ranging from nearby traffic and local micro-tremors to massive earthquakes occurring thousands of miles away. Because high-frequency energy attenuates faster over long distances through the Earth's crust, distant (**teleseismic**) earthquakes arrive predominantly as **low-frequency waves (< 2–3 Hz)**.

This repository demonstrates how to separate distant seismic events from local high-frequency interference using time-domain and frequency-domain techniques.

## Signal Processing Pipeline
1. **Preprocessing (`detrend`)**: Removes constant DC offsets and linear baseline drift introduced by seismometer hardware.
2. **Spectral Analysis (`pwelch`)**: Computes the Power Spectral Density (PSD) to identify dominant frequency bands and distinguish low-frequency earthquake energy from high-frequency noise.
3. **Digital Filtering (`lowpass`)**: Applies a zero-phase lowpass filter (`2.5 Hz` cutoff) to attenuate local vibrations while preserving the teleseismic P-wave and S-wave arrival envelope.
4. **Time-Frequency Analysis (`pspectrum`)**: Generates a spectrogram to visualize how signal energy evolves across frequency and time during the event arrival.
5. **Signal Measurements (`rms`, `max`)**: Extracts peak ground amplitude and root-mean-square (RMS) energy metrics from the cleaned waveform.

## MATLAB Implementation (`earthquake_analysis.m`)

```matlab
%% Earthquake Signal Processing & Spectral Analysis
% Inspired by MathWorks Signal Processing Onramp
% Objective: Preprocess seismic data, analyze frequency content, and isolate
% distant (teleseismic) earthquake waves from high-frequency local noise.

clear; clc; close all;

%% 1. Load or Synthesize Seismic Station Data
% Attempt to load built-in MATLAB quake dataset; fallback to synthetic multi-station data
if exist('quake.mat', 'file')
    load('quake.mat'); % Loads e, n, v components
    fs = 200;          % Sampling frequency (Hz)
    rawSignal = v - mean(v); % Remove DC offset (detrend)
    t = (0:length(rawSignal)-1)' / fs;
else
    fs = 50;           % 50 Hz seismic station sampling rate
    t = (0:1/fs:120)'; % 120-second recording window
    rng(42);
    
    % Distant teleseismic earthquake (low frequency: 0.5 - 1.5 Hz arriving at t = 40s)
    quakeEnvelope = exp(-0.08 * max(0, t - 40)) .* (t >= 40);
    teleseismic = 1.8 * quakeEnvelope .* sin(2*pi*0.9*t + 0.4*sin(2*pi*0.2*t));
    
    % Local tremor / cultural noise (high frequency: 8 - 18 Hz) + sensor drift
    localNoise = 0.6 * sin(2*pi*12*t) + 0.35 * randn(size(t));
    dcOffset = 1.25;
    
    rawSignal = teleseismic + localNoise + dcOffset;
end

%% 2. Preprocessing: Detrending
% Remove constant offset and linear sensor drift
detrendedSignal = detrend(rawSignal);

%% 3. Spectral Analysis (Power Spectral Density)
% Estimate power spectrum using Welch's method to identify dominant frequencies
[pxxRaw, fRaw] = pwelch(detrendedSignal, [], [], [], fs);

%% 4. Filtering: Isolate Distant Earthquake Signal
% Distant earthquakes exhibit low-frequency energy (< 2 Hz), while local
% disturbances occupy higher frequencies. Apply a lowpass filter at 2.5 Hz.
cutoffFreq = 2.5; % Hz
filteredSignal = lowpass(detrendedSignal, cutoffFreq, fs);

[pxxFilt, fFilt] = pwelch(filteredSignal, [], [], [], fs);

%% 5. Signal Measurements
peakAmplitude = max(abs(filteredSignal));
rmsValue = rms(filteredSignal);
fprintf('--- Seismic Event Summary ---\n');
fprintf('Peak Filtered Amplitude : %.4f\n', peakAmplitude);
fprintf('Filtered Signal RMS      : %.4f\n', rmsValue);

%% 6. Visualization & Export for GitHub README
fig = figure('Color', 'w', 'Position',);

% Subplot 1: Time-Domain Comparison
subplot(3,1,1);
plot(t, detrendedSignal, 'Color', [0.2 0.45 0.8 0.55], 'LineWidth', 1); hold on;
plot(t, filteredSignal, 'Color', [0.85 0.15 0.15], 'LineWidth', 1.6);
grid on;
title('Raw vs. Lowpass-Filtered Seismic Waveform');
xlabel('Time (s)');
ylabel('Amplitude');
legend('Detrended Raw Signal', sprintf('Lowpass Filtered (< %.1f Hz)', cutoffFreq), 'Location', 'northeast');

% Subplot 2: Power Spectral Density
subplot(3,1,2);
plot(fRaw, 10*log10(pxxRaw), 'Color', [0.2 0.45 0.8], 'LineWidth', 1.2); hold on;
plot(fFilt, 10*log10(pxxFilt), 'Color', [0.85 0.15 0.15], 'LineWidth', 1.5);
grid on;
xlim([0 min(25, fs/2)]);
title('Power Spectral Density (Welch Estimate)');
xlabel('Frequency (Hz)');
ylabel('Power/Frequency (dB/Hz)');
legend('Raw Spectrum', 'Filtered Spectrum', 'Location', 'northeast');

% Subplot 3: Time-Frequency Spectrogram
subplot(3,1,3);
pspectrum(filteredSignal, fs, 'spectrogram', 'FrequencyLimits', [0 min(15, fs/2)]);
title('Time-Frequency Spectrogram of Isolated Earthquake Wave');

% Auto-save figure if assets folder exists
if ~exist('assets', 'dir')
    mkdir('assets');
end
exportgraphics(fig, 'assets/image_b55312f7.jpg', 'Resolution', 300);
```
