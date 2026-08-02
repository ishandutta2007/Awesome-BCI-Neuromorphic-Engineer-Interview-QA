# Topic 03: BCI System Architecture

## Overview
Implantable versus wearable system topologies, data path partitioning between on-chip and external processing, and the system-level design decisions that define a neuromorphic BCI's capability envelope.

---

### Q1: How does the architectural distinction between fully implanted BCIs (no transcutaneous wires), semi-implanted (percutaneous connector), and wearable non-invasive BCIs shape neuromorphic hardware design requirements?

**A:**
**Fully implanted BCIs (e.g., Neuralink N1, Synchron Stentrode in its wireless version):**
All electronics — electrode array, analog front-end, digitization, processing, wireless telemetry — are sealed inside the body (in the skull, subdural space, or intravascular). This creates the most stringent power, size, and reliability requirements:
- **Power:** No wired power delivery; must be powered wirelessly (inductive power transfer, typically limited to 10-50mW through tissue at practical coupling distances and link efficiencies) or from an implanted battery (limited capacity, must last months to years, may require inductive recharging through skin). Every milliwatt matters.
- **Heat dissipation:** Power dissipated as heat must remain within safe tissue-heating limits (FDA guidance typically limits implant-induced temperature rise to <2°C for chronic implants) — a direct power constraint
- **Size and packaging:** Must fit within available intracranial space (extremely limited); hermetic packaging required for device longevity (moisture ingress causes corrosion/failure); all materials must be biocompatible (Topic 09)
- **Neuromorphic implication:** Ultra-low-power on-chip processing is not a performance optimization but a fundamental enabling requirement — without neuromorphic-class power efficiency (processing at sub-mW to few-mW levels), the required on-chip processing simply cannot be implemented within implantable constraints

**Semi-implanted / percutaneous connector systems (most current clinical BCIs, e.g., BrainGate research systems):**
Implanted electrode array connected via a percutaneous connector (exiting through the skull/skin) to external electronics. Removes the power/size constraint from the implanted portion (power/processing lives in the external unit), but introduces: infection risk at the percutaneous site, mechanical reliability concerns, and patient mobility limitations.
- **Neuromorphic implication:** Less severe on-chip power constraint (or on-chip neuromorphic processing is less critical), but real-time performance and latency requirements remain; external neuromorphic processing can be applied to spike-encoded neural signals for low-latency, low-power external decode

**Wearable non-invasive BCIs (EEG, fNIRS, OPM-MEG-based):**
Electrodes on skin (EEG) or sensors close to the skull — no surgical implantation. Much more accessible but fundamentally limited signal bandwidth/resolution compared to intracortical recording (skin/skull dramatically attenuate and spatially smear high-frequency neural signals).
- **Neuromorphic implication:** Different signal processing challenges (lower SNR, lower spatial resolution, different frequency bands of interest — primarily EEG frequency bands 1-100Hz vs. spike-rate signals at 0.3-10 kHz for intracortical); neuromorphic processing for wearable BCIs often targets efficient EEG temporal pattern classification rather than spike-based decoding; power budgets less severe than implantable but still relevant for wearable form factor (battery life, thermal comfort)

### Q2: Design the data path architecture for a fully implanted 1024-channel cortical BCI, partitioning processing between on-chip neural compute, implanted wireless, and external receiver. Justify each partitioning decision.

**A:**
**Design problem scope:** 1024 channels × 30 kHz/channel × 16 bits = ~491 MB/s raw data — completely impractical to wirelessly transmit from an implant (current implantable wireless links operate at 1-100 Mb/s; transmitting raw data would require 4-40 Gb/s, orders of magnitude beyond feasibility). On-chip data reduction is mandatory.

**Proposed data path:**
```
1024 electrode array
  ↓ (analog signals)
On-chip AFE: 1024 parallel amplifier channels (Topic 04)
  - Bandpass filtering: dual-path (spike band 300-10kHz; LFP band 1-300Hz)
  - Low-power continuous-time filtering before ADC
  ↓ (still analog)
On-chip ADC: Time-division multiplexed or per-channel SAR ADC
  - Spike band: 12-bit @ 30 kHz/channel ≈ 368 Mb/s digital
  - LFP band: 12-bit @ 2 kHz/channel ≈ 24.6 Mb/s digital
  ↓ (digital)
On-chip spike detection (real-time, per-channel threshold crossing):
  - Output: spike timestamps + waveform snippets (~50 samples each)
  - Data rate: ~100 kb/s - 10 Mb/s (depending on firing rates)
  ↓
On-chip neuromorphic decoding (SNN inference engine, Topic 06):
  - Spike events from 1024 channels → population-based decode
  - Output: decoded intent/state vector (e.g., 3D velocity, 10 decoded parameters)
  - Data rate after decode: ~1-10 kb/s
  ↓
On-chip wireless telemetry:
  - Transmit: decoded state + downsampled raw snippets (for offline analysis/retraining)
  - Receive: parameter updates, stimulation commands
  - Link: ~1-10 Mb/s (MICS band or ultra-wideband)
  ↓ (through tissue)
External receiver unit
  - Decoded state → real-time BCI application
  - Snippets → offline analysis, SNN model update, model download back to implant
```

**Key partitioning justifications:**
1. **Spike detection on-chip is mandatory:** 368 Mb/s digital raw spike-band data cannot be transmitted wirelessly; spike detection compresses this to ~1-10 Mb/s of sparse events — required compression, not optional optimization
2. **SNN decoding on-chip reduces telemetry to decoded state:** Transmitting decoded state (kb/s) rather than spike events (Mb/s) further reduces wireless link requirements, saving power and improving link reliability; this is the core architectural argument for on-chip neuromorphic decoding
3. **Retain raw snippet telemetry for offline analysis and model update:** Small bandwidth allocation for raw spike waveform snippets (sampled fraction of events, not all events) enables offline drift monitoring and SNN model retraining/update without discarding all raw information — essential for long-term system maintenance (Topic 07)
4. **Bidirectional link for parameter updates:** Closed-loop stimulation (delivering feedback stimulation based on decoded state) and SNN model updates (downloading retrained models) require an uplink from external unit to implant, making bidirectional wireless architecture necessary from the outset

### Q3–Q16: (Representative additional topics)
- Neural recording electrode array architectures (Utah array, Michigan probe, high-density CMOS-integrated probes)
- Hermetic packaging technology for implantable electronics
- Inductive power transfer design for implantable BCIs (coil design, tissue absorption, thermal limits)
- Wireless telemetry protocol selection for implantable BCIs (MICS band, ultra-wideband, Bluetooth low energy)
- ECoG (electrocorticography) versus intracortical recording trade-offs
- Closed-loop stimulation integration in the BCI data path
- External processing unit design requirements and co-design with implant
- Multi-chip implantable BCI architectures for highest channel counts
- Software-hardware co-design considerations for the on-chip/off-chip partition
- Clinical workflow and intraoperative system testing considerations

---

## Summary
BCI system architecture for neuromorphic implantable systems is fundamentally shaped by the wireless bandwidth and power constraints of the implanted environment — mandatory on-chip data reduction (spike detection, neuromorphic decoding) is not optional optimization but the core enabling architecture without which fully-implanted high-channel-count BCIs are infeasible, with every partitioning decision driven by the arithmetic of achievable wireless link rates versus raw neural data volume.
