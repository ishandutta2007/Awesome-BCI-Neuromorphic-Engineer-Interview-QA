# Topic 11: Troubleshooting & Case Studies

## Overview
Diagnosing signal degradation, SNN decoder instability, and power violations in neuromorphic BCI systems — structured problem-solving grounded in this field's unique hardware-software-biology intersection.

---

### Q1: An implanted BCI ASIC that performed well at implant surgery shows progressively degrading neural decoding accuracy over 3 months, with recording quality metrics (spike count, SNR) appearing stable on most channels. Walk through your diagnostic approach.

**A:**
**Key diagnostic insight — decoding accuracy degradation without apparent recording quality loss:**
This specific pattern (stable recording quality but degrading decoding accuracy) is more diagnostically specific than a simple "the system is getting worse" complaint. It suggests the problem is likely at the neural coding level (the relationship between neural activity and intended movement is changing) rather than the signal acquisition level (the hardware is still recording well), directly pointing toward neural plasticity, neuronal remapping, or learning-related coding changes as the primary hypothesis.

**Systematic diagnostic progression:**

1. **Confirm the recording quality characterization is genuinely stable, not just superficially stable:** "Spike count appears stable" may mask more subtle changes — verify whether the spike waveform shapes on each channel have changed (indicating neuronal turnover even if total spike count appears similar), whether the cross-channel correlation structure of neural activity has shifted, and whether individual unit tuning curves (relationship between each recorded neuron's firing rate and movement direction) remain stable or have gradually rotated/shifted. Stable aggregate spike count can mask substantial remapping of the underlying neural code.

2. **If tuning curves have shifted, this is the likely dominant root cause:** Gradual remapping of motor cortex tuning curves during chronic BCI use is a well-documented phenomenon — the cortex's own plasticity causes the neural code for intended movements to evolve over time, rendering a decoder calibrated to the original tuning curves progressively mismatched to the current code. This is a biological (not hardware) phenomenon, and the primary remediation is decoder recalibration (if the on-chip learning system, Topic 07, is not handling this automatically) or adjustment of the on-chip adaptation parameters.

3. **If tuning curves are stable, investigate whether the on-chip SNN decoder itself has developed unexpected behavior:** In an SNN with on-chip STDP adaptation (Topic 07), confirm the adaptation is proceeding as intended and hasn't saturated weights, driven weights to pathological extremes, or introduced unexpected dynamics. Log the on-chip SNN's internal state (if accessible via debug telemetry) and compare against expected behavior from pre-implant simulation.

4. **Check for systematic bias in the decoding error pattern:** If the 3D velocity decoding is degrading but with systematic bias (e.g., one velocity dimension degrades much faster than others, or decoded velocities show systematic offset in a specific direction), this pattern provides additional diagnostic information — dimension-specific degradation may indicate specific electrode channels/neural populations contributing to that dimension have changed, while systematic offset may indicate a slow drift in baseline neural activity levels relative to the decoder's calibration reference.

**Resolution:** If tuning curve remapping is confirmed as the cause, the resolution path is decoder recalibration using recent neural data — either triggering more aggressive on-chip adaptation (Topic 07), or using the raw telemetry snippet data (retained per Topic 03's architecture) to retrain a new decoder offline and transmit the updated weights wirelessly to the implant.

### Q2: Case study — During benchtop testing of a new neuromorphic BCI ASIC, the total system power measurement exceeds the 10mW budget by 40% (measuring 14mW). Characterize your systematic approach to identifying the excess power sources.

**A:**
**Systematic power budget investigation:**

1. **Partition measurement into subsystems using controlled test sequences:** Rather than measuring total system power as a single number, isolate each subsystem's contribution using targeted test sequences that enable individual subsystem power quantification:
   - **AFE power:** Configure all channels to record but hold the SNN inference engine and wireless telemetry disabled; measure power minus known digital quiescent → AFE power
   - **Digital/SNN power:** Configure AFE off (or in lowest-power state), SNN inference enabled, wireless disabled; inject synthetic spike events at known rates via digital loopback; measure power increment per unit SNN activity
   - **Wireless telemetry power:** Enable telemetry with controlled data rate while holding other blocks at characterization state; measure power vs. telemetry data rate
   - **Stimulation (if applicable):** Deliver controlled stimulation sequences at specified parameters; measure stimulation power vs. pulse parameters

2. **Compare measured subsystem power against pre-silicon simulation predictions, block by block:** Compare each measured subsystem power against the simulation estimate from design phase. The block(s) showing largest discrepancy between simulation and measurement are the primary investigation targets.

3. **For the block(s) showing excess power, investigate likely discrepancy sources:**
   - **Analog (AFE/ADC):** Biasing current higher than simulation (threshold voltage lower than modeled, causing gm and quiescent current increase); bandwidth wider than specification causing higher switching noise from ADC; parasitic resistance causing unexpected quiescent current paths; latchup or incorrect operating point
   - **Digital (SNN/DSP):** Higher switching activity than simulation assumed (activity factor underestimated); clock gating not functioning as simulated (more clock distribution switching than expected); unexpected combinational glitching in critical paths adding spurious switching energy
   - **Memory (SRAM):** Higher access rate than simulation assumed; SRAM assist circuits consuming more power than modeled; leakage higher than expected (threshold voltage mismatch from nominal)
   - **Wireless:** PA operating at higher output power than nominal (impedance mismatch causing PA efficiency reduction); oscillator consuming more power than simulated

4. **For highest-impact identified source, determine whether design fix or specification change is appropriate:**
   - If AFE consumes 2× more than budgeted, and this is due to threshold voltage lower than modeled (process variation): potential mitigations include adding biasing trim to adjust to nominal operating point, or accepting the power impact and reducing channel count from 256 to stay within budget
   - If wireless consumes excess due to antenna impedance mismatch in test environment: verify whether production environment (implanted in tissue) provides better match (the test environment mismatch may not represent clinical use) before deciding whether a design fix is needed

### Q3–Q13: (Representative additional topics)
- Diagnosing stimulation artifact contamination in decoded spike trains
- Troubleshooting SNN decoder instability or oscillatory behavior after on-chip STDP adaptation
- Root-causing inter-channel crosstalk in high-density neural recording arrays
- Investigating timing violations in the real-time spike detection pipeline
- Debugging wireless link reliability issues in a tissue environment
- Root-causing SRAM read errors at chip operating voltage
- Diagnosing unexpected SNN latency violations during high-firing-rate neural input conditions
- Troubleshooting hermetic package failure indicators from impedance monitoring data
- Investigating post-implantation electrode impedance increase and its signal quality implications
- Systematic silicon bring-up procedure for a new neuromorphic BCI ASIC

---

## Summary
Troubleshooting neuromorphic BCI systems requires systematic, hypothesis-driven isolation — accurately distinguishing biological (neural plasticity, neuronal remapping), hardware (AFE power excess, antenna mismatch), and algorithmic (SNN adaptation pathology, decoder calibration drift) root causes, using the controlled test sequences and subsystem isolation strategies that provide clean, diagnostic evidence rather than undifferentiated "the system doesn't work as expected" symptom reports.
