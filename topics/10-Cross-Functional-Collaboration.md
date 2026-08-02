# Topic 10: Cross-Functional Collaboration

## Overview
Working effectively with neuroscientists, neurosurgeons, chip designers, and clinical teams as a BCI Neuromorphic Engineer.

---

### Q1: How do you collaborate effectively with neuroscientists who specify BCI decoding requirements, given the potential gap between neuroscientific concepts (neural tuning curves, population coding) and hardware-implementable SNN specifications?

**A:**
**Collaborative translation approach:**
1. **Learn to speak neuroscience fluently enough to engage substantively, not just receive specifications:** A neuromorphic BCI engineer who only receives neuroscience requirements as an external specification — "the decoder needs to achieve R² > 0.7 on 3D cursor velocity" — and proceeds to implement a black-box SNN meeting that spec misses the genuine collaborative value: understanding the neuroscientific underpinning of the decoding task (which neural populations encode the relevant variable, what the expected neural coding properties are, what physiological mechanisms may change over time during chronic use) substantially informs better SNN architecture choices than treating the neuroscience as an opaque requirement source. This requires genuine investment in neuroscience domain literacy (connecting directly to the analogous cross-disciplinary literacy principle emphasized in the Biocomputer Software Engineer repository's Topic 10).

2. **Translate neuroscientific concepts into hardware design constraints collaboratively, not unilaterally:** When a neuroscientist specifies a requirement that has non-obvious hardware implications (e.g., "the decoder must handle the neural population's low-dimensional manifold structure" — which implies specific SNN architecture choices for dimensionality reduction), making the hardware implications explicit and discussing them with neuroscience collaborators (rather than unilaterally translating the spec into a specific hardware implementation and presenting the implementation decision as a fait accompli) enables better joint optimization and avoids hardware choices that inadvertently exclude neuroscientifically-important processing steps.

3. **Provide neuroscientist collaborators with quantitative hardware constraint context for their specification choices:** Neuroscientists specifying performance requirements may not have direct awareness of the hardware cost implications of different specification choices (e.g., "adding STDP-based on-chip learning requires X additional mm² and Y additional µW, which trades off against Z channels of AFE capability at fixed total budget"). Making these hardware costs explicit and quantitative — in terms the neuroscientist can understand and incorporate into their specification trade-off reasoning — enables more informed joint specification choices rather than creating a situation where neuroscience requirements are set without hardware constraint awareness and then have to be painfully renegotiated after hardware design constraints are discovered.

### Q2: Describe how you would collaborate with an analog IC/chip design team on a neuromorphic BCI ASIC development, as a neuromorphic systems/algorithm engineer who doesn't necessarily design individual transistor circuits yourself.

**A:**
**Collaborative practices:**
1. **Provide circuit designers with characterization targets grounded in system-level analysis, not arbitrary numbers:** System-level analysis (power budgeting, Topic 08; noise requirement from signal amplitude and SNR targets, Topic 04; latency budget from closed-loop control requirements, Topic 03) generates specific, justified circuit performance targets (e.g., "the AFE must achieve <5µV_rms IRN in 300Hz-10kHz at <8µW/channel to meet our 256-channel power budget while maintaining adequate spike detection SNR"). Providing circuit designers with these analysis-derived, justified targets — with the underlying system reasoning made explicit — is far more useful than handing down a requirements table without context, enabling circuit designers to make informed trade-offs when meeting all targets simultaneously proves challenging.

2. **Develop and maintain SNN simulation models that interface with accurate circuit behavioral models for system-level verification:** Before silicon is available, verifying that the full system (AFE → ADC → spike detection → SNN decoder) meets performance requirements requires simulation models of each subsystem. The neuromorphic systems engineer should maintain high-level behavioral models of the circuit subsystems (validated against circuit designers' transistor-level simulations) and integrate these into end-to-end system simulations that verify SNN decoder performance under realistic circuit imperfections (noise, mismatch, quantization effects) — providing both the system-level verification and a natural communication artifact for joint debugging when system simulation reveals performance shortfalls.

3. **Participate in architecture reviews and design reviews with constructive, systems-informed feedback:** Architecture review and design review participation is most valuable when the systems engineer can provide feedback informed by understanding the circuit-level implementation (e.g., flagging that a specific circuit architecture choice has implications for the SNN's input noise properties that will degrade decoding accuracy in ways not visible from the circuit-level perspective alone) — which requires sufficient circuit understanding to engage substantively, even without being the circuit designer.

4. **Jointly own tapeout readiness criteria that span circuit and systems performance:** Tapeout readiness (the decision to submit the chip design for manufacturing) typically requires confidence that all design requirements are met — but "requirements" span both circuit-level performance (noise, power, timing) and systems-level performance (SNN decoding accuracy under realistic circuit conditions). Jointly owning and reviewing tapeout readiness criteria that explicitly span both levels — rather than treating circuit-level checks as the chip team's responsibility and systems-level checks as the algorithm team's separate concern — ensures the intersection of these requirements is actually verified, not assumed.

### Q3–Q13: (Representative additional topics)
- Collaborating with neurosurgeons and clinical teams on implantation and post-operative monitoring procedures relevant to system performance
- Working with clinical regulatory affairs on FDA submission documentation contributions
- Communication with BCI end-users (ALS patients, spinal cord injury patients) about system performance expectations and limitations
- Research partnership management with academic neuroscience collaborators providing clinical trial sites
- Communicating SNN algorithm performance to non-technical stakeholders including clinical trial participants, clinicians, and investors
- Cross-functional design of clinical trial protocols incorporating system performance monitoring
- Managing external chip manufacturing (ASIC foundry relationship management, shuttle run participation)
- Knowledge transfer and documentation practices for interdisciplinary BCI development teams
- Open-source community engagement for BCI software infrastructure development
- Mentoring embedded software or neuroscience colleagues in neuromorphic hardware context

---

## Summary
Effective cross-functional collaboration for a BCI Neuromorphic Engineer requires genuine, invested domain literacy in both neuroscience and analog/mixed-signal circuit design — without which the engineer can neither effectively translate between neuroscience requirements and hardware capabilities nor identify and resolve the system-level interactions between circuit imperfections and SNN algorithm performance that only become visible when these domains are actively bridged.
