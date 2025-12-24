# Headphone DAC/AMP Overview
The following is a general system overview of an XMOS DAC & TPA6120 AMP
## Abstract

## 1. Introduction

###  1.1 Background & Motivation
High-fidelity audio has become increasingly difficult to access for the average user, as most integrated digital-to-analog converters (DACs) in phones and laptops are limited to 16-bit resolution and a 44.1 kHz sampling rate, which is only marginally sufficient for modern high-resolution audio standards. At the other end of the spectrum, systems capable of 24-bit resolution and sampling rates above 98 kHz are often prohibitively expensive and rely on external power sources, making them bulky and inconvenient for portable use or integration into an already crowded home audio setup.

The goal of this project is to demonstrate that both high resolution and high sampling rates can be achieved in a compact, affordable system. While a moderate amount of theoretical information explaining how a DAC-amplifier system functions is available online, practical guidance regarding component selection, layout considerations, IC integration, and signal-path design remains limited. This project therefore also aims to serve as a technical resource for future designers interested in high-fidelity audio systems, reducing the barrier to entry and simplifying the development process. 

###  1.2 Project Goals

The primary goal of this project is to design, implement, and test a compact, high-fidelity DAC-amplifier that is cost-effective relative to comparable commercial products. The system will convert digital audio data into a clean, low-noise analog output and will support a wide range of headphones and IEM's with impedances between 10 Ω and 300 Ω. All operation will be powered exclusively through a USB-C 2.0 interface, eliminating the need for any external power supply, and output to a 3.5mm balanced jack. <br> <br>

The following requirements should be met:
* 24-bit resolution at a 192kHz sampling rate
  * reasonable spec for "Hi-Fi". Higher is better.
* USB-C Input
* 3.5mm balanced output
* 10Ω to 300Ω capable headphone interfacing
* Adjustable gain for a variety of headphone / IEM models

###  1.3 Scope of Work

This project encompasses the design considerations, system implementation, component selection and placement, 
assembly, and performance testing of a compact, USB-powered DAC-amplifier. Design considerations include evaluating suitable DAC 
and amplifier architectures, power constraints and noise mitigation techniques. Implementation involves translating the schematic into a functional 
PCB layout, with careful attention to grounding, signal routing, and isolation between digital and analog sections. Component placement is
optimized to minimize interference and ensure signal integrity, while still maintaining a compact form factor. Assembly includes PCB fabrication,
component population, and hardware integration. Finally, performance testing is conducted to evaluate signal-to-noise ratio, total harmonic distortion, 
frequency response, and overall system functionality under real-world operating conditions. 

###  1.4 Report Organization

This report will be structured in order of operations; from sources to testing. Each point will have related subpoints that further section off specific concepts to keep the paper concise. 

## 2. Literature Review

### 2.1 DAC/AMP Design Principles & Solutions

#### Ladder DAC
Given that the initial goal of the project was creating a custom solution, the first step was to look at custom DAC topologies. These are often relatively difficult to implement, but one of the simplest versions of this is the Resistor Ladder DAC topology. A Ladder DAC uses a series of resistors arranged in a network to convert digital signals into analog voltages. Each bit of the digital input controls a switch that connects either a reference voltage or ground to the resistor network. The combined effect of the resistor values and switch positions produces a stepped analog output proportional to the digital input. The limitation of these Ladder DACs is that they are heavily dependent on the accuracy across all of the resistors within the network. Any variance can induce distortion into the output, which is not acceptable for the needed application.



### 2.2 USB Audio Standards

### 2.3 Related Designs

## 3. Theory

### 3.1 CT Vs DT Audio
The purpose of a DAC is to convert a binary string into an analog signal. When a time component is added into the scope, the audio has to be sampled at a certain rate and turned into a continuous-time signal rather than a discrete-time signal. According to the Nyquist sample ling theorem, the signal should be sampled at twice the maximum sampled frequencies, as shown in equation one. This minimum sampling frequency, although is enough to recreate the signals, is not enough to recreate the quality of the original continuous-time signal. Even outside of the Nyquist sampling frequency, 40kHz, Aliasing can still occur if there are frequencies that appear outside of the bandwidth of human hearing, so for a more accurate representation of a signal, we sample higher than the minimum sampling frequency.

<p align="center">
 $$F_s > 2F_{max}$$ <br>
 Equation one: Nyquist Sampling Theorem 
</p>

### 3.2 DSP Components
In order to get a signal that the DAC can read, there needs to be a supporting chip that can talk to the host device and has a single master-out line for the DAC to decode. A DSP (digital signal processing) device is needed in order to do this. First, a stable clock must be a part of the communication protocol in order to avoid jitter, the device needs to be able to set the aforementioned sampling frequency with the host device, and optionally, the DSP device needs to be able to change the dB of specific signals in order to introduce a digital equalizer. 

### 3.4 Grounding Strategies

When designing PCBs, a common practice is to often design around the idea of splitting up the grounds used for analog and digital. This is done in very low noise or extremely high frequency applications. The theory behind the separation of these ground connections is that you are controlling the path of the signal's control current, preventing the return current of a high frequency signal from inducing EMI. Many ICs actually have separate pins for analog and digital grounds. Any design utilizing these separated grounds still requires that the grounds be connected at a single point, preferably near the power input of the circuit. This is an old and common practice in these designs, but based upon our research, this is actually an outdated practice.

Theory and practicality require each other to work in many cases. In this case, while in theory an analog and digital ground separation may reduce conducted emissions, in practice the split of these grounds causes inconsistent, long, and winding return paths. The added length or deviations of these return paths actually increase the conducted emissions across the circuit, worsening any noise or signal integrity issues. in lower frequency designs (below about 20kHz), these ground separations make more sense as the current return path is wider, therefore risk of interference is increased. But in higher frequency designs, the analog and digital grounds should not be separated but instead combined, and alternative grounding methods be used.

The following interview is a discussion with Rick Hartley, a prominent figure in the RF PCB Design world. In this interview, Rick discusses the split of a ground plane, and whether it should be implemented. While during the discussion Rick mentions that there are use cases for both, he also states that his first fix in nearly every situation involving noise or conducted emissions is removing the split planes. Rick then goes on to show a presentation slide with a joke he likes that I think sums up the stance he gives on the technique of splitting grounds. "What do we call an engineer who splits their grounds? A customer!" For reference, this joke comes from a conducted emissions debugging specialty company. 
https://youtu.be/vALt6Sd9vlY

Instead of Splitting the grounds in our design, we have chosen to combine them and focus on maintaining ground everywhere throughout the board. The primary thing we chose to do was implement a Ground plane on layer 2 of the PCB.
## 4. System Architecture

### 4.1 Design Requirements

### 4.2 Block Diagram

### 4.3 Component-Level Architecture

## 5. Circuit Design

### 5.1 Schematic Overview

### 5.2 Component Selection

### 5.3 Simulation Results

### 5.4 PCB Layout Considerations

## 6. PCB Fabrication & Assembly

### 6.1 Fabricaton parameters

### 6.2 Assembly process

### 6.3 Photographs

## 7. Testing and Verification

### 7.1 Test Setup

### 7.2 USB Recognition Testing

### 7.3 Functional Testing

### 7.4 Measurements

### 7.5 Comparison with Expectations

## 8. Discussion

### 8.1 Strength

### 8.2 Challenges and Limitations

### 8.3 Lessons Learned

## 9. Conclusion

## 10. Future Work

## References




## Appendicies



