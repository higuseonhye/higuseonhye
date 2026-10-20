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
│  • retro-vital-sync                                                    │
│      Late pre-dose chart. Decay-only RED 100/100.                      │
│      Dose-gated RED 0/100, MSE vs replay 0.0006.                       │
│  • dicom-stream-triage                                                 │
│      Threaded DICOM 69.7 fps, click path 24.9 ms,                      │
│      false positives -85.7%, emergency wait -93.7%                     │
│  • surgical-scene-triage                                               │
│      Occlusion rejection 100% (120/120), no-go penetration 0/40,       │
│      async hold P99 1.96 ms                                            │
│  • holoscan-multimodal-triage                                          │
│      120 s ring. Saline false RED 0/100. Early AMBER 100/100.          │
│      Beta-blocker relative SI +47.5%. Evaluate P99 0.224 ms.           │
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
| Delayed chart | [retro-vital-sync](https://github.com/higuseonhye/retro-vital-sync) | One Gaussian on five z-scores. A pre-dose chart arrives after norepinephrine. Decay-only RED 100/100. Dose-gated RED 0/100, MSE vs event-order replay 0.0006. A 10-minute lag issues at minute 62. |
| Imaging | [dicom-stream-triage](https://github.com/higuseonhye/dicom-stream-triage) | Synthetic chest DICOM. Threaded decode 69.7 fps. Sequential click path 24.9 ms. False positives down 85.7%. Emergency wait down 93.7%. |
| Surgical scene | [surgical-scene-triage](https://github.com/higuseonhye/surgical-scene-triage) | Smoke and splash rejection 100% (120/120). Barrier penetration 0/40. Async hold P99 1.96 ms. |
| Operating-room stream | [holoscan-multimodal-triage](https://github.com/higuseonhye/holoscan-multimodal-triage) | 120 s vital ring with tool gating. Saline false RED 0/100. Early-bleed AMBER 100/100. Missed shock 0/100. Evaluate P99 0.224 ms on a 10 s buffer and 0.305 ms on a full ring. Beta-blocker hemorrhage at instantaneous SI 0.75 reaches relative SI +47.5% and stress 0.824. Light anesthesia stress 0.066. |
| Moving sphere | [deformable-cbf-bench](https://github.com/higuseonhye/deformable-cbf-bench) | Prescribed sphere, point tool. Static hover clearance -0.250 mm. Time-varying hover +0.083 mm. On a 0.3 Hz bob both filters hold the margin, 8/8. |
| Policy recovery | [diffusion-recovery-bench](https://github.com/higuseonhye/diffusion-recovery-bench) | Four-step flow matching, action-head P99 4.3 ms inside a 10 ms period. Recovery after a 6 cm jump: 81%. |
| Adaptivity | [embodied-adaptivity](https://github.com/higuseonhye/embodied-adaptivity) | Controlled experiments on keeping capability when the physical world changes. |
| Serving | [triton-serving-lab](https://github.com/higuseonhye/triton-serving-lab) | Local dynamic-batch queue in front of a small ONNX model. Concurrency-8 P99 moved from 47.59 ms to 7.01 ms. |
| Evaluation | [surgeval-bench](https://github.com/higuseonhye/surgeval-bench) | Harness for phase, tool, grounding, and safety questions. It scores predictions. It does not train a model or move a robot. |


### Operating-room stream

`holoscan-multimodal-triage` keeps a 120 s, 100 Hz vital ring and a tool gate for irrigation and suction. On the recorded CPU run, saline false RED alarms are 0/100, early bleeds route to AMBER on 100/100, and unsafe GREEN dismissals are 0/100. Frame evaluation P99 is 0.224 ms with 10 s stored and 0.305 ms with the ring full. A beta-blocker pattern whose instantaneous shock index is 0.75, below a fixed cut of 0.8, reaches a relative change of +47.5% and stress 0.824. Light anesthesia, with pulse pressure rising, keeps stress at 0.066. The inputs are synthetic.

### Two ward benches

`vital-stream-triage` fills an hourly grid. Labs drawn once or twice a day are mostly missing, so the scored question is how to carry the last value forward in time, and whether a deterioration score above 0.7 should stay silent when the class distribution is flat. The recorded cuts are time-decay standardized MSE 0.723, 57.6% fewer false alarms, and a 1,000-bed micro-batch P99 of 6.1 ms.

`retro-vital-sync` keeps a single five-dimensional Gaussian. The scored question is a chart whose measurement time is earlier than its log time, with a dose between them. Decay-only conditioning pages that recovery on 100 of 100 draws. Gating the cross-covariance by the dose pages it on 0 of 100, with MSE 0.0006 against an event-order replay. The alarm still waits for the chart: a 10-minute lag issues at minute 62, ten minutes after the on-time crossing. A 60-minute late chart measured after the dose still moves the current posterior, because the filter does not store the newer charts. Replaying those charts costs more as the stay grows (mean 96.6 ms at 480 charts, against 0.33 ms for the gated update).

### Two surgical benches

`surgical-scene-triage` keeps a static disk in a camera plane. The barrier is a signed distance, applied by projection, and the scored failure is an attractor that crosses the disk: penetration 40/40 unconstrained, 0/40 with the barrier, worst clearance 1.189 mm. It also refuses a ruined frame and times the hold.

`deformable-cbf-bench` asks what changes when the obstacle moves. The plant is a point and a prescribed sphere. The barrier is a squared ellipsoid and an active-set quadratic program. On the hover, dropping the sphere velocity reaches -0.250 mm. Keeping that velocity in the half-space reaches +0.083 mm. The 0.3 Hz bob is slow enough that the two filters tie at 8/8. A separate metre-unit layer extrapolates a 30 Hz sample across a dropout and blends a phase change. The millimetre episodes do not call that layer. Neither repository starts a simulator or reads a recording.
