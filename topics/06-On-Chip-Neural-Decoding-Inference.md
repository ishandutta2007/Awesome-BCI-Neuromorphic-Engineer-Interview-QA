# Topic 06: On-Chip Neural Decoding & Inference

## Overview
SNN inference engine architectures, latency-accuracy-power trade-offs for on-chip neural decoding, and the hardware implementations enabling real-time BCI decode within implantable constraints.

---

### Q1: Compare approaches to implementing a neural decoder on a neuromorphic chip — SNN-based inference versus quantized ANN inference on a conventional digital core — discussing the power, latency, and accuracy implications of each.

**A:**
**SNN-based inference on neuromorphic hardware:**
The "native" approach for a neuromorphic chip — the SNN inference engine processes spike input events from the neural recording channels (Topic 05) using the chip's silicon neuron network, producing spike outputs encoding the decoded state.

- **Power:** Fundamentally event-driven and sparse — computation only occurs when spikes arrive; quiet periods consume minimal power (only leakage currents). For typical neural input sparsity (neurons firing at 1-50 Hz on average, producing infrequent spike events), effective dynamic power is much lower than for continuously-operating digital inference. Representative power: 0.1-1mW for SNN inference of a 256-channel motor decoder.
- **Latency:** Can be extremely low — spike events propagate through the SNN on an event-driven basis; decoded output is available within a few milliseconds of input spikes arriving, potentially sub-millisecond if few SNN layers are required. No batch processing or fixed-cycle clocking imposes minimum latency floor.
- **Accuracy:** Currently achievable SNN decoder accuracy (for motor decoding from neural populations) is competitive with but sometimes marginally lower than equivalent ANN decoders, particularly if the SNN inference time window is constrained by latency requirements. For typical clinical motor BCI targets, achieved accuracy is generally sufficient but may require architectural optimization.
- **Key design challenge:** Training and deploying an SNN that performs acceptably for the specific decoding task while exploiting the neuromorphic hardware's efficiency (Topic 02's SNN training challenges).

**Quantized ANN inference on an embedded digital core:**
Implementing a conventional ANN (e.g., LSTM or Transformer for temporal sequence decoding, linear/MLM for simpler cases) with weight quantization (8-bit, 4-bit, or lower) on a small embedded processor or custom digital accelerator co-located with the analog front-end on the same implantable chip.

- **Power:** Digital CMOS power scales with switching activity (α × C × V² × f) — even with quantization, continuous matrix-multiplication operations at each timestep require significantly more power than event-driven SNN processing for sparse inputs. Typically 2-10× higher dynamic power than SNN inference for equivalent architectural scale, though modern ultra-low-power MCU technology has narrowed this gap.
- **Latency:** Fixed computational latency per inference step, determined by matrix operation throughput — typically predictable but with a fixed floor determined by the minimum clock cycles required for the matrix operations, regardless of input sparsity.
- **Accuracy:** Typically higher than equivalent SNN for the same model size/depth, benefiting from the mature ANN training ecosystem and the absence of surrogate gradient approximation errors. For quantized models, 8-bit quantization typically incurs <1% accuracy loss versus full-precision ANN; 4-bit may incur 1-3% depending on the model.

**Recommendation and hybrid approaches:** For implantable BCIs with the most stringent power constraints, SNN-based inference is generally preferred when achievable SNN accuracy is sufficient for the decoding task — the power efficiency advantage is significant. For applications where accuracy is paramount and power budget is somewhat more flexible (e.g., high-channel-count, high-precision prosthetic), a quantized ANN inference approach or hybrid SNN+ANN architecture may be more appropriate. Many production designs will use hybrid approaches — SNN for initial spike-pattern detection/feature extraction (where sparsity makes SNN most efficient) followed by a compact quantized linear decoder for final regression (where the final mapping from neural features to decoded output is most naturally represented by matrix multiplication).

### Q2: Design an SNN inference engine for decoding continuous 3D cursor velocity from 256 sorted neural units on a neuromorphic chip with a 2mW power budget and 10ms latency target. Specify the network architecture and hardware implementation approach.

**A:**
**Task requirements:**
- Input: 256 sorted neural units, spike events in continuous time
- Output: 3D velocity vector (vx, vy, vz) updated at ~100Hz (10ms update period)
- Power budget: 2mW on-chip for inference
- Latency: ≤10ms from spike input to decoded output

**SNN architecture design:**
```
Input layer: 256 input neurons (one per sorted unit)
  - Spike input from translation layer (Topic 05)
  
Hidden layer 1: 512 LIF neurons
  - Recurrent connections within layer (for temporal integration)
  - Fan-in: each neuron receives from ~64 input units (sparse connectivity)
  - STDP-capable synapses for online adaptation (Topic 07)

Hidden layer 2: 128 LIF neurons
  - Reduced dimensionality: extracts compact trajectory-encoding representation
  
Output layer: 3 linear "readout" neurons (one per velocity dimension)
  - Conventional weighted sum of H2 spike rates (not spiking themselves)
  - Output sampled every 10ms to produce velocity vector
```

**Hardware implementation:**
```
Memory: Spike event buffer (circular, ~16KB for 10ms event window at max firing rates)
Synaptic weight storage: SRAM, 8-bit quantized weights
  - 256×512 = 131,072 weights (H1 input-hidden) → 128KB
  - 512×128 + 128×3 = 65,920 weights (H1H2 + H2 readout) → 64KB
  - Total: ~192KB SRAM
  
Neuron update engine: Event-driven, triggered only on incoming spikes
  - Each spike event from input triggers membrane potential update
    for all postsynaptic neurons of the spiking neuron
  - Digital fixed-point arithmetic: 16-bit membrane potential
  - Threshold comparison: single clock cycle per neuron update
  
Power estimation:
  - Synaptic operations per second: 256 input units × avg 20Hz × 64 fan-out
    = ~328K SOPS/second; at 10pJ/SOP (target) → 3.3µW (far below budget)
  - SRAM access power (dominant): ~1-1.5mW for 192KB SRAM at typical
    neural firing activity levels with SRAM access per spike
  - Logic/arithmetic: ~0.3-0.5mW
  - Total estimated: ~1.8-2.0mW — fits within 2mW budget
```

**Latency analysis:**
10ms window for rate estimation; output computed at end of each 10ms window from accumulated H2 spike rates. Spike-to-output propagation latency (event processing within the SNN) << 1ms, so the dominant latency is the 10ms rate-estimation window. This satisfies the ≤10ms latency target (though represents an end-to-end 10ms latency that may be suboptimal for very tight closed-loop control; reducing to 5ms would halve the rate-estimation window, slightly degrading velocity estimate smoothness).

### Q3–Q15: (Representative additional topics)
- Continuous vs. discrete-time SNN inference for real-time BCI decoding
- Recurrent SNN architectures for sequential/temporal neural decoding tasks
- Readout layer design options: rate-coded readout, population vector, linear regression on SNN activity
- Hardware resource utilization and scheduling for time-multiplexed neuromorphic inference
- Benchmarking SNN decoders against linear (Kalman filter, Wiener filter) and ANN BCI decoders
- Uncertainty quantification and confidence estimation for neural decoders
- Multi-task SNN decoders (simultaneously decoding multiple BCI control signals)
- Model compression for deploying large pre-trained SNN decoders on area-constrained chips
- Deployment pipeline: simulation → software SNN → neuromorphic hardware deployment verification
- Performance monitoring and anomaly detection for deployed on-chip decoders

---

## Summary
On-chip neural decoding requires choosing between SNN-native inference (best power efficiency via event-driven sparsity exploitation) and quantized ANN inference (potentially higher accuracy at higher power cost), with hybrid approaches increasingly common — the 2mW inference engine example illustrates that modern neuromorphic architectures can achieve competitive motor decoding within implantable power budgets when SRAM access power and synaptic operation efficiency are jointly optimized.
