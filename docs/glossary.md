# Glossary: BCI Neuromorphic Engineering Terminology

## Terms A–M

**AFE (Analog Front-End)** – The amplifier chain, filtering, and ADC circuitry that converts electrode voltages to digital neural signals.

**ANN-to-SNN conversion** – Method for producing an SNN by training a conventional ANN, then substituting spiking neurons to approximate the same input-output mapping via rate coding.

**Charge density limit** – Maximum safe electrical charge per unit electrode area per stimulation phase; violations risk electrode dissolution and tissue damage.

**Charge-balanced stimulation** – Stimulation waveform design requiring net-zero charge injection per pulse (equal cathodic and anodic phases) to prevent DC-driven Faradaic damage.

**CMOS (Complementary Metal-Oxide-Semiconductor)** – The dominant transistor process technology for neural recording and neuromorphic ASICs.

**ECoG (Electrocorticography)** – Recording from electrodes placed on the brain surface (subdural or epidural) — less invasive than intracortical but lower spatial resolution.

**Foreign body response** – Tissue inflammation and encapsulation around an implanted electrode, causing impedance increase and recording degradation over weeks-to-months.

**Hermetic packaging** – Sealed implantable device enclosure preventing moisture ingress that would cause corrosion/failure; typically titanium or ceramic for chronic implants.

**IEC 62304** – International standard governing medical device software lifecycle processes; Class C (death risk) applies to most implantable BCI control software.

**LFP (Local Field Potential)** – Slow extracellular voltage fluctuations reflecting summed synaptic activity of a neural population.

**LIF (Leaky Integrate-and-Fire)** – The most common simplified silicon/software neuron model: membrane potential integrates input charge and leaks exponentially, fires (resets) when threshold is crossed.

**Loihi 2** – Intel's second-generation digital neuromorphic research chip; programmable SNN with on-chip learning support.

**LTP / LTD (Long-Term Potentiation / Depression)** – Synaptic weight increases (LTP) and decreases (LTD); the biological substrate of Hebbian learning and STDP.

---

## Terms N–Z

**Neuromorphic computing** – Computing architectures using silicon circuits structurally analogous to biological neural circuits; event-driven, spike-based, co-located memory-compute.

**NoC (Network-on-Chip)** – On-chip spike routing interconnect in a neuromorphic chip, analogous to the axonal projection system.

**Population coding** – Information encoding distributed across joint activity patterns of many neurons; the dominant biological neural coding scheme.

**SAR ADC (Successive Approximation Register ADC)** – The most power-efficient ADC architecture for neural recording resolution/bandwidth requirements; uses binary search via capacitive DAC.

**Shannon limit (stimulation)** – Empirical safety boundary for neural stimulation: log(Q/A) ≤ k (typically k ≈ 1.5-1.7 µC/cm²) where Q is charge/phase, A is electrode area.

**SNN (Spiking Neural Network)** – Neural network using discrete spike events for information transfer rather than continuous activations.

**STDP (Spike-Timing Dependent Plasticity)** – Biologically-inspired local learning rule: LTP if pre-spike precedes post-spike within ~10-20ms; LTD for reverse timing.

**Surrogate gradient** – Smooth differentiable function substituted for the Heaviside spike activation's true (undefined) gradient during SNN backpropagation training.

**TrueNorth** – IBM's neuromorphic chip; 4096 cores, ~1M neurons, ~70mW; fixed LIF model, no on-chip learning; highly energy-efficient inference.

**Utah Array** – Widely-used intracortical electrode array; 10×10 grid of penetrating silicon electrodes; standard in BrainGate and Blackrock systems.

**V_min (SRAM minimum operating voltage)** – Lowest supply voltage at which SRAM maintains reliable read/write margins; typically the binding constraint on digital voltage scaling.

---

## Abbreviations Reference

| Abbr | Full Form |
|------|-----------|
| AFE | Analog Front-End |
| ASIC | Application-Specific Integrated Circuit |
| BCI | Brain-Computer Interface |
| BPTT | Backpropagation Through Time |
| CDAC | Capacitive Digital-to-Analog Converter |
| CMRR | Common-Mode Rejection Ratio |
| DRC/LVS | Design Rule Check / Layout vs. Schematic |
| ECoG | Electrocorticography |
| FMEA | Failure Mode and Effects Analysis |
| IRN | Input-Referred Noise |
| IRB | Institutional Review Board |
| MUA | Multi-Unit Activity |
| NoC | Network-on-Chip |
| PMA | Premarket Approval (FDA) |
| SAR | Successive Approximation Register |
| SNN | Spiking Neural Network |
| SOPS | Synaptic Operations Per Second |
| STDP | Spike-Timing Dependent Plasticity |

---

**Note:** This glossary is not exhaustive. Refer to topic files and primary literature for authoritative definitions.
