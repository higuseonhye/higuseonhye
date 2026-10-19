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
│  • deformable-cbf-bench                                                │
│      Moving sphere. Static hover -0.250 mm.                            │
│      Time-varying hover +0.083 mm. Crossing filters tie, 8/8.          │
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
| Moving sphere | [deformable-cbf-bench](https://github.com/higuseonhye/deformable-cbf-bench) | Prescribed sphere, point tool. Static hover clearance -0.250 mm. Time-varying hover +0.083 mm. On a 0.3 Hz bob both filters hold the margin, 8/8. |
| Policy recovery | [diffusion-recovery-bench](https://github.com/higuseonhye/diffusion-recovery-bench) | Four-step flow matching, action-head P99 4.3 ms inside a 10 ms period. Recovery after a 6 cm jump: 81%. |
| Adaptivity | [embodied-adaptivity](https://github.com/higuseonhye/embodied-adaptivity) | Controlled experiments on keeping capability when the physical world changes. |
| Serving | [triton-serving-lab](https://github.com/higuseonhye/triton-serving-lab) | Local dynamic-batch queue in front of a small ONNX model. Concurrency-8 P99 moved from 47.59 ms to 7.01 ms. |
| Evaluation | [surgeval-bench](https://github.com/higuseonhye/surgeval-bench) | Harness for phase, tool, grounding, and safety questions. It scores predictions. It does not train a model or move a robot. |

### Two surgical benches

`surgical-scene-triage` keeps a static disk in a camera plane. The barrier is a signed distance, applied by projection, and the scored failure is an attractor that crosses the disk: penetration 40/40 unconstrained, 0/40 with the barrier, worst clearance 1.189 mm. It also refuses a ruined frame and times the hold.

`deformable-cbf-bench` asks what changes when the obstacle moves. The plant is a point and a prescribed sphere. The barrier is a squared ellipsoid and an active-set quadratic program. On the hover, dropping the sphere velocity reaches -0.250 mm. Keeping that velocity in the half-space reaches +0.083 mm. The 0.3 Hz bob is slow enough that the two filters tie at 8/8. A separate metre-unit layer extrapolates a 30 Hz sample across a dropout and blends a phase change. The millimetre episodes do not call that layer. Neither repository starts a simulator or reads a recording.
