---
layout: default
usemathjax: true
permalink: /network/ch12
---

# Digital Transmission

**Analog** signals: continuous with infinitely many levels

**Digital** signals: discrete with finite levels

![Analog vs Digital](/notes-blog/assets/img/network/analog_v_digital.png)

Encoding data in binary as **voltages** that can be transmitted over wires.

## Encoding methods

1. Non-return to zero (NRZ)
2. Return to zero (RZ)
3. Manchester

| Encoding Method | Illustration |
| -------------- | ------------- |
| NRZ | ![](/notes-blog/assets/img/network/nrz.png) |
| RZ | ![](/notes-blog/assets/img/network/rz.png) |
| Manchester | ![](/notes-blog/assets/img/network/manchester.png) |

# Analog Transmission

Represented as wave, e.g. sine wave

$$
A\sin(2\pi ft + \phi)
$$

- $A$ for amplitude
- $f$ for frequency
- $\phi$ for phase (like a shift or offset for the signal from the reference time.)

**Bandwidth**: The difference between the highest and lowest frequency that can pass through a channel

**Signal to Noise Ratio**: The strength of signal over noise.

![Receiver Comparator](/notes-blog/assets/img/network/comparator_circuit.png)

At the signal receiver side, the signal is sent to a sort of comparator circuit as shown above.

**Shannon Channel Capacity**: The theoretical maximum bit rate of a noisy channel

$$
C = B \log (1 + \text{SNR})
$$

**Modem**: Modulator/Demodulator

## Encoding methods

1. Amplitude Shift Keying
2. Frequency Shift Keying
3. Phase Shift Keying