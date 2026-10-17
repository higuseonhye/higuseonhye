# Seonhye Gu | Clinical Systems & Physical AI Platform Architect

Bridging clinical data governance with low-latency AI serving, autonomous robotics control, and secure execution runtimes.

---

### 🏛️ End-to-End Clinical & Physical AI Architecture

```text
┌────────────────────────────────────────────────────────────────────────┐
│  [Top Layer: Agent Safety Runtime]                                     │
│  • safemed-sandbox                                                     │
│    - Ephemeral Micro-VM / Mock Gatekeeper (120 ms overhead)            │
│    - Dry-run Interception (Unsafe Actuator/EMR Orders Blocked)         │
│    - Zero-Fault Host Protection (0 bytes modified on host)             │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ (Pre-flight Validation)
┌───────────────────────────────────▼────────────────────────────────────┐
│  [Middle Layer: Domain Use-Case Engines]                               │
│  • vital-stream-triage   : 19-vital EMR streaming, Time-decay (0.723), │
│                            False Alarms -57.6%, 1,000 beds P99 6.1 ms  │
│  • dicom-stream-triage   : Zero-copy DICOM (69.7 fps, 24.9 ms),        │
│                            False Positives -85.7%, Critical Wait -93.7%│
│  • surgical-scene-triage : 100% Occlusion Rejection, CBF No-Go Zone    │
│                            0% penetration, Async E-Hold P99 1.95 ms    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ (High-Throughput Tensors & Policies)
┌───────────────────────────────────▼────────────────────────────────────┐
│  [Bottom Layer: Core Serving & Control Engines]                        │
│  • triton-serving-lab / surgeval-bench                                 │
│    - Zero-copy mmap ingestion (29,277 patches/s), P99 < 7 ms           │
│  • diffusion-recovery-bench / embodied-adaptivity                      │
│    - 100 Hz Flow Matching (4.3 ms), 6 cm Closed-loop Recovery (81%)    │
└────────────────────────────────────────────────────────────────────────┘
```

### 📦 Key Repositories

**Runtime**  
[safemed-sandbox](https://github.com/higuseonhye/safemed-sandbox)

**Clinical Vital Streaming**  
[vital-stream-triage](https://github.com/higuseonhye/vital-stream-triage)

**Medical Imaging CADt**  
[dicom-stream-triage](https://github.com/higuseonhye/dicom-stream-triage)

**Surgical Scene Safety**  
[surgical-scene-triage](https://github.com/higuseonhye/surgical-scene-triage)

**Physical Policy Recovery**  
[diffusion-recovery-bench](https://github.com/higuseonhye/diffusion-recovery-bench) · [embodied-adaptivity](https://github.com/higuseonhye/embodied-adaptivity)

**High-Throughput Serving**  
[triton-serving-lab](https://github.com/higuseonhye/triton-serving-lab) · [surgeval-bench](https://github.com/higuseonhye/surgeval-bench)
