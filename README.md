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
Figure X shows the block diagram of the system. The design uses the XMOS XU316 as a digital signal processing and interface device to convert USB audio data into an I²S data stream. This I²S stream is then processed by the ES9039 32-bit digital-to-analog converter (DAC), which produces a high-fidelity analog audio signal. The analog output is subsequently amplified by the TPA6120 headphone amplifier to drive the headphone load. <br>

Supporting power architecture is implemented using multiple low-dropout regulators (LDOs) to supply the required voltage domains. Dedicated LDOs provide regulated 3.3 V, 1.8 V, and 0.9 V rails for the digital and mixed-signal components. The headphone amplifier is powered by a differential supply derived from a 5.5 V rail using the ADP5071 and buffered using LDO's. An initial design utilizing a ±12 V differential supply was evaluated; however, concerns regarding excessive output power and the risk of overdriving headphones led to the selection of a lower-voltage differential supply. <br>

The XMOS XU316 requires multiple clock sources, including an external master clock and an I²S clock, to ensure accurate audio timing and low jitter performance. Additionally, an external boot memory device is required for firmware storage, which is provided by the W25Q32JV QSPI flash memory.

<p align="center">
 <img width="804" height="928" alt="image" src="https://github.com/user-attachments/assets/dde0d51c-e25b-4e6c-8754-f3023f65e35e" /> <br>
 Figure X: Block Diagram of system
</p>

### 4.3 Component-Level Architecture
The system works as follows when viewing the flowchart; USB to a device connects to the board VIA USB-C and communicates to the XMOS through USB. The XMOS converts the audio signal from USB to I2S. This I2S signal comes into the ES9039, where a 32 bit word is turned into an analog voltage, where each bit of the word represents a voltage, and the sum of those voltages gives an aproximation of the sampled audio. This now analog signal is then sent to a 2-pole lowpass filter using the OPA1612 in order to remove any frequencies that are not present in human hearing (>20kHz). This filtered frequency is then sent to the TPS7A3301, where the signal is amplified to output to a headphone through a 3.5mm TTRS jack.

## 5. Circuit Design

### 5.1 Schematic Overview
The schematic design is implemented in KiCad due to its versatility and low barrier to entry. In addition to its accessibility, KiCad provides robust support for hierarchical schematic design, which is critical for enabling a synchronous and collaborative development workflow. To support multiple contributors working on different subsystems simultaneously, the project is managed using Git in conjunction with hierarchical schematics.<br>

Each major functional block of the system is implemented as an individual hierarchical subschematic, allowing for modular development, improved readability, and simplified version control. The functional blocks are organized as follows: 
* USB Input: XMOS, QSPI Flash, and USB-C header
* Audio rails: Differential powersupply resides aswell as supporting LDO's
* General power: 3.3V, 1.8V, and 0.9V LDO's
* DAC: ES9039 and supporting parts
* Headphone AMP stage: OPA1612, TPA6120 headphone AMP, and supporting parts
* Clocking: System and audio clocks
* Debug: XTAG debug connector that will connect to the XMOS

All subschematics are connected through hierarchical labels to a single root schematic, which defines the system-level interconnections. Within each subschematic, components are grouped into clearly defined blocks to reflect functional signal flow. This hierarchical organization improves schematic readability, supports collaborative development, and provides a clear mapping between system-level architecture and circuit-level implementation.

### 5.2 Component Selection

### 5.3 Simulation Results
Simulation was performed to evaluate the performance and stability of the analog low-pass filter used in the output stage. Initial designs focused on achieving a sharp cutoff frequency near the upper limit of the audible band using a Chebyshev Filter. This filter utilised a single opamp while simultaniously being a 3rd order system. The following is what the simulated filter looked like.<br>

<p align="center">
 <img width="1790" height="829" alt="image" src="https://github.com/user-attachments/assets/55925889-c1a2-4a23-a91f-f3752e15d5ab" />
 <br>
 Figure X: LTspice Simulation of filter
</p>


While these higher-order filter topologies were successful in simulation, concerns arose regarding stability, particularly due to the use of multiple operational amplifiers and large resistor values within the feedback network.<br>

Additionally, a review of manufacturer reference designs revealed that most datasheets implementing headphone amplifier stages employ a single operational amplifier with a simple feedback network, typically including a 2200pF feedback capacitor in parallel with an 800 ohm resistor. This approach minimizes loop complexity and reduces the risk of instability while maintaining acceptable frequency response characteristics. The following is the prefilters from the TPA6120 datasheet. <br>

<p align="center">
 <img width="430" height="346" alt="image" src="https://github.com/user-attachments/assets/5b2e7f82-de49-4716-ba8b-3272ecf1a990" />
 <br>
 Figure X: LTspice Simulation of filter
</p>

Based on these considerations, a single-op-amp topology with a 2200pF feedback capacitor in parallel with an 800 ohm resistor was selected. Simulation results confirm that this configuration provides a cutoff frequency of approximately 20 kHz, which is sufficient to attenuate out-of-band noise while preserving the full audible frequency range. The selected design offers a balance between frequency selectivity, stability, and implementation robustness.

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



