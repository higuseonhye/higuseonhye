# Seonhye Gu | Clinical Systems & Physical AI

Bridging clinical data governance with low-latency AI serving, closed-loop control, and a secure execution runtime.

These repositories are synthetic benchmarks and local labs. They are not clinical devices. None of them reads a hospital record, a PACS, or a robot.

---

### End-to-End Architecture

```text
┌────────────────────────────────────────────────────────────────────────┐
│  [Top Layer: Agent Safety Runtime]                                     │
│  • safemed-sandbox                                                     │
│    - Ephemeral session / local mock gatekeeper (median overhead 120 ms)│
│    - Dry-run interception (unsafe actuator and chart calls rejected)   │
│    - Host protection (0 bytes of the repository modified)              │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │  Pre-flight validation
┌───────────────────────────────────▼────────────────────────────────────┐
│  [Middle Layer: Domain Use-Case Engines]                               │
│  • vital-stream-triage                                                 │
│      19-channel ward grid, time-decay MSE 0.723,                       │
│      false alarms -57.6%, 1,000-bed micro-batch P99 6.1 ms             │
│  • dicom-stream-triage                                                 │
│      Threaded DICOM 69.7 fps, click path 24.9 ms,                      │
│      false positives -85.7%, emergency wait -93.7%                     │
│  • surgical-scene-triage                                               │
│      Occlusion rejection 100% (120/120), no-go penetration 0/40,       │
│      async hold P99 1.96 ms                                            │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │  Tensors, alarms, and policies
┌───────────────────────────────────▼────────────────────────────────────┐
│  [Bottom Layer: Serving, Evaluation, and Control]                      │
│  • triton-serving-lab                                                  │
│      Local ONNX queue. Concurrency-8 P99 47.59 ms -> 7.01 ms           │
│  • surgeval-bench                                                      │
│      Phase, tool, grounding, and safety VQA harness                    │
│  • diffusion-recovery-bench                                            │
│      4-step flow matching P99 4.3 ms (10 ms period),                   │
│      6 cm closed-loop recovery 81%                                     │
│  • embodied-adaptivity                                                 │
│      Experiments on capability as the physical world changes           │
└────────────────────────────────────────────────────────────────────────┘
```

Each figure is the recorded CPU run in that repository. `triton-serving-lab` does not start NVIDIA Triton Server. The 4.3 ms figure is the action-head latency on `diffusion-recovery-bench`, not a camera-plus-robot loop.

### Repositories

| Layer | Repository | Recorded result |
| --- | --- | --- |
| Runtime | [safemed-sandbox](https://github.com/higuseonhye/safemed-sandbox) | Ephemeral session in front of the three triage gates. Median overhead 120 ms. Unsafe actuator and chart calls rejected. Host tree unchanged. |
| Clinical stream | [vital-stream-triage](https://github.com/higuseonhye/vital-stream-triage) | Synthetic hourly ward grid. Time-decay standardized MSE 0.723. False alarms down 57.6%. 1,000-bed micro-batch P99 6.1 ms. |
| Imaging | [dicom-stream-triage](https://github.com/higuseonhye/dicom-stream-triage) | Synthetic chest DICOM. Threaded decode 69.7 fps. Sequential click path 24.9 ms. False positives down 85.7%. Emergency wait down 93.7%. |
| Surgical scene | [surgical-scene-triage](https://github.com/higuseonhye/surgical-scene-triage) | Smoke and splash rejection 100% (120/120). Barrier penetration 0/40. Async hold P99 1.96 ms. |
| Policy recovery | [diffusion-recovery-bench](https://github.com/higuseonhye/diffusion-recovery-bench) | Four-step flow matching, action-head P99 4.3 ms inside a 10 ms period. Recovery after a 6 cm jump: 81%. |
| Adaptivity | [embodied-adaptivity](https://github.com/higuseonhye/embodied-adaptivity) | Controlled experiments on keeping capability when the physical world changes. |
| Serving | [triton-serving-lab](https://github.com/higuseonhye/triton-serving-lab) | Local dynamic-batch queue in front of a small ONNX model. Concurrency-8 P99 moved from 47.59 ms to 7.01 ms. |
| Evaluation | [surgeval-bench](https://github.com/higuseonhye/surgeval-bench) | Harness for phase, tool, grounding, and safety questions. It scores predictions. It does not train a model or move a robot. |
