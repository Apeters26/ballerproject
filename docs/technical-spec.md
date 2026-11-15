# Autonomous drone simulator · MVP technical specification

**Status** Proposed build contract. The [product specification](product-spec.md) defines the user workflow and the [roadmap](roadmap.md) sequences later capabilities.

## Reference environment and implementation boundary

Build a local Python package named `drone_sim`. Use PyBullet for rigid-body dynamics and contacts, `gym-pybullet-drones` for the CF2X vehicle and motor model, and its `DSLPIDControl` for attitude and position control. The upstream PID example demonstrates `CtrlAviary`, a `(1,4)` RPM action, `computeControlFromState`, and the five-value Gymnasium step return. Use CPU physics in DIRECT mode by default. GUI mode attaches the PyBullet viewer to the same runner.

Target CPython 3.12 on x86-64 Linux and WSL2 on Windows. Evaluate native Windows separately with a clean install and headless smoke run. The current upstream manifest includes Torch and Stable-Baselines3 even though the PID policy requires no trained model. Resolve CPU packages and measure cold-install size before release. Use upstream 2.2.0 at `348b5c18a4bb770a48928b9d044decf77ca33302` as the initial compatibility target. The first implementation PR must pin transitive dependencies and platform markers in `uv.lock`, and verify the CF2X asset loads from a clean checkout. If a PyBullet wheel is unavailable, document and smoke-test the required native build toolchain. Record the upstream SHA, Python patch version and PyBullet version in the release notes. The links at the end describe the upstream interface at design time.

## Course `five-gate-v1`

Use a right-handed world frame with metres, seconds, +z up and forward travel along +x. Initial yaw is zero. The start and finish pads are painted zones on the same solid ground plane at z=0. Each has radius 0.35 m. Place the CF2X at `(0,0)` with zero velocities and its lowest collision surface touching the plane. Derive its resting centre height from the pinned collision geometry. The landing pad centre is `(12,0)`.

| Gate | Centre (x, y, z), metres | Clear opening (width × height) |
| --- | --- | --- |
| G1 | (2, 0, 1.00) | 1.00 × 1.00 m |
| G2 | (4, 0.65, 1.10) | 1.00 × 1.00 m |
| G3 | (6, −0.55, 1.25) | 1.00 × 1.00 m |
| G4 | (8, 0.55, 1.05) | 1.00 × 1.00 m |
| G5 | (10, 0, 1.00) | 1.00 × 1.00 m |

Construct each static frame from four axis-aligned box colliders surrounding the opening, with 0.08 m depth along x and 0.08 m bar thickness. The invisible plane spanning the opening is a mathematical scoring surface. Bound the world centre position to x ∈ [−1,13], y ∈ [−3,3], z ∈ [0,3]. Load one versioned course asset into the viewer, runner and judge. Validate its unique IDs, strictly increasing x, finite dimensions, positive opening and containment before spawning. Include its exact bytes and SHA-256 in the run bundle. V1 has no wind, randomized starts or sensor noise.

## Modules and data flow

| Owner | Input → output | Implementation rule |
| --- | --- | --- |
| `course` | Versioned JSON → immutable world and gate definitions | Convert box dimensions to Bullet half-extents in one place. |
| `sim` | Four RPMs → one physics tick, pose, velocities and contacts | Own the `CtrlAviary` instance, client ID, vehicle body ID and mapping of body IDs to ground or gate. Always close on exit. |
| `control` | Observation, known course and mission phase → four RPMs | Receive values only. The controller has no reference to Bullet or the scoring implementation. |
| `judge` | Two successive states, contact snapshot and prior mission state → progress events and optional result | Use pure logic that can be tested with synthetic segments. |
| `runner` | Settings → ordered ticks, events and terminal summary | Own the integer clock, controller schedule, terminal precedence and output lifecycle. |
| `replay` | Completed bundle → trajectory, ordered events and score | Read recorded data locally. Preserve playback when a controller or simulator dependency changes. |

The adapter normalizes upstream observations to `position_m[3]`, quaternion `xyzw[4]`, world `velocity_mps[3]` and world `angular_velocity_radps[3]`. The baseline PID adapter can take these named fields through `computeControl`, or reconstruct the corresponding fields of the upstream vector for `computeControlFromState`. The public controller interface uses named fields in either case. Check the field order against the selected upstream revision with a known-pose fixture. All numeric values must be finite. A policy output is four RPMs in upstream motor order. Clip finite RPMs to `[0, env.MAX_RPM]` and record applied values. A malformed observation or command aborts as an execution error.

## Fixed-step mission loop

1. Reset the single-drone `CtrlAviary` with `physics=Physics.PYB`, `pyb_freq=240`, `ctrl_freq=240`, upstream obstacles off and debug controls off. After each reset build the gate bodies and pad markings, since reset can recreate the Bullet world. Disable real-time stepping. Record tick 0.
2. On ticks divisible by 5, compute a fresh reference at 48 Hz. Pass `control_timestep=1/48`, observed state, target position, velocity and zero-yaw attitude to `DSLPIDControl.computeControlFromState`. Hold the returned RPMs for five physics ticks. Reset PID integrators between episodes.
3. Call `env.step(rpm.reshape(1,4))` exactly once per tick. Collect the new state and `getContactPoints` for the vehicle on that same client. Evaluate every 1/240 s to catch brief frame contacts and crossings between controller updates. Treat unexpected upstream episode termination as an execution error.
4. Judge and log the resulting interval, then stop immediately upon terminal outcome. Otherwise run through tick 14,400 (60 simulated seconds). Show gate numbers and a judge-driven progress overlay in the live viewer. GUI pacing may sleep after logging. Elapsed simulated seconds always equal the terminal tick divided by 240.

**PID reference path.** Take off vertically to `(0,0,1)`. For each gate use three anchors, 0.45 m before the centre, at the centre, and 0.45 m after it, with the gate's y and z. From G5 exit, fly to the point 0.45 m above the landing pad, then descend vertically. For each segment use smoothstep `s=3u²−2u³` with `u` clamped to [0,1]. Send its analytic velocity as feed-forward. Set duration to at least `1.5 × segment_length / speed_cap`, with speed caps of 0.5 m/s for takeoff, 0.8 m/s in transit, and 0.2 m/s for descent. Hold a completed anchor until position error ≤0.10 m, except the final touchdown anchor where the judge decides completion. Move the final vertical reference to 0.01 m below the vehicle's resting centre height to bring the body into gentle contact rather than hovering over the pad. The ground plane still constrains the actual vehicle. After the first legal finish-pad contact, command zero RPM until the judge declares a result. Tune reference speeds or settling behaviour against the release fixture while retaining the published scoring contract.

## Gate, contact and outcome rules

For each gate let `d=x−gate.x`. A forward crossing occurs when `d_prev < 0 ≤ d_next`. Evaluate y and z at `alpha=−d_prev/(d_next−d_prev)` by linear interpolation. A valid crossing has `abs(y−gate.y) ≤ width/2−0.10` and `abs(z−gate.z) ≤ height/2−0.10`. The 0.10 m inset is a conservative vehicle clearance, independent of collision detection. Count each gate once. Process crossings in ascending alpha. The first forward crossing of an unpassed gate is decisive. First check the gate ID. Crossing a later gate before the expected one fails with `wrong_order`. For the expected gate, crossing outside the valid aperture fails with `missed_gate`. Backward crossings and recrossing a credited plane earn no credit. Integer tick plus alpha locates a crossing within a trajectory sample.

For a hand-checkable G2 fixture, assume G1 is credited. A collision-free segment from `(3.98, 0.65, 1.10)` to `(4.02, 0.65, 1.10)` earns G2 at `alpha=0.5`. Moving both y values to 1.051 m makes the same segment miss by 1 mm beyond the inset. If the physics engine also reports a frame contact during that interval, the terminal reason is `collision`.

Any vehicle contact with a frame is a collision. A pad contains the vehicle when its centre projected onto the ground is within the pad radius, including the edge. Permit initial ground contact inside the start pad. Retire this exemption permanently on leaving the pad or exceeding 0.15 m centre height. Emit `liftoff` on the first airborne tick above that height. Ground contact before G5 or outside the finish pad after G5 fails with `collision`, except for the initial exemption.

At the start of each finish-pad contact episode, the previous tick must have downward speed ≤0.30 m/s, horizontal speed ≤0.20 m/s and roll and pitch magnitudes ≤12°. A harder or tilted touchdown fails with `collision`. After legal contact, credit landing only after 120 consecutive ground-contact ticks inside the pad with total speed ≤0.10 m/s and roll and pitch magnitudes ≤10°. A bounce or excessive settling speed resets the dwell counter. The runner supplies the legal-contact observation to the baseline. Its motor cut takes effect at the next 48 Hz update and stays latched, while the judge continues to inspect every 240 Hz tick.

Within each tick resolve disallowed contact first, then `out_of_bounds`, invalid or valid crossings, landing, and finally `timeout`. A collision interval cannot earn a gate. Success on the deadline is valid. Return the five-gate count only after five credited events, and `success` only after the landing dwell. For ranking, group completed missions first and order those by increasing terminal tick. Exact ties share rank. Show failed runs with their reason and progress without a time ranking.

## Operator interface and files

Proposed commands, to be delivered by the implementation PRs, from the repository root after `uv sync --frozen`. Windows PowerShell and WSL shells can use the same commands. The visual run needs a working graphical display. Test WSLg on the target Windows laptop.

```sh
uv run python -m drone_sim run --course five-gate-v1 --controller pid --out runs/demo
uv run python -m drone_sim run --course five-gate-v1 --controller pid --gui --out runs/visual
uv run python -m drone_sim replay runs/demo
uv run pytest
```

Replay renders recorded 3D positions with gates, start and finish markers, an adjustable time cursor, play/pause and event annotations. A Matplotlib window suffices for the local viewer. `replay --summary` prints the same result on a machine without a display. Refuse an existing nonempty output directory. Write a manifest last and atomically rename it into place. A missing or unsupported manifest produces an actionable error. CLI exit codes are 0 for success or successful replay, 1 for a scored flight failure, and 2 for invalid input, unavailable display, corrupt output or other execution errors.

| Bundle file | Required v1 contents |
| --- | --- |
| `course.json` | Exact course definition including pads, bounds, gate centres, openings and bar geometry. |
| `trajectory.csv` | One row per tick including tick 0 and terminal tick. Columns `tick,x,y,z,qx,qy,qz,qw,vx,vy,vz,wx,wy,wz,rpm0,rpm1,rpm2,rpm3`. RPMs correspond to the interval ending on that row and are zero at reset. Use round-trip float precision. |
| `events.jsonl` | Ordered objects with `tick`, `type`, optional `gate_id`, `alpha`, and `reason`. For a gate event, `tick` is the ending tick and its interpolated time is `(tick - 1 + alpha) / 240` seconds. Types `liftoff`, `gate_pass`, `touchdown` (first qualifying contact in a dwell), `landed` and `failed`. |
| `manifest.json` | `schema_version`, course ID and hash, controller ID and version, live-view setting, application revision, upstream revision, lock hash, engine/Python versions, OS/architecture, physics/control Hz, `outcome`, `reason`, `gates_passed`, `terminal_tick`, `elapsed_sim_s`, and hashes of all three other files. |

Emit one terminal event per scored run. On replay, verify schema and hashes, contiguous ticks, final trajectory tick and terminal event against the manifest. The known course is identical for every run. Repeatability is evaluated under identical locked builds on the same machine. On a different supported platform, rerun the full mission and require the same ordered gate events and a successful landing. Record any differences in timing and trajectory for review. Only compare flight times competitively on the same locked environment.

For example, this synthetic line in `events.jsonl` credits G2 halfway between ticks 959 and 960. Replay should mark that point on the recorded path at 3.9979 simulated seconds. The line illustrates the format and does not report a measured flight.

```json
{"tick":960,"type":"gate_pass","gate_id":"G2","alpha":0.5}
```

## Build sequence and verification

| Increment | Verification before proceeding |
| --- | --- |
| Package and physics adapter | Clean `uv sync --frozen`, direct reset and 240 steps on Linux CPU. Verify finite states, one model, gravity, hover RPM bounds and exactly 1 s of advancement. Smoke-test WSL2 and record laptop model, CPU, OS and install steps. |
| Course and judge | Table-driven tests for centred pass, inset edge and edge +1 mm, two crossings in one synthetic interval, reverse travel, repeated gate, early later gate, ground contact after takeoff, legal initial ground contact, 119 vs 120 dwell ticks, bounce reset, hard touchdown, bounds and deadline precedence. |
| Closed-loop mission | Run reference controller against the actual Bullet world. It must lift off, clear G1–G5 once in order, land, and finish by tick 14,400. Inject one targeted frame strike, one wide miss and one hover timeout. Check distinct reasons and the last credited gate. |
| Replay and release | Replay successful and failed bundles offline, reject truncated/corrupt files, exercise unwritable output. Require identical gate order, reason and terminal tick in GUI and DIRECT on the same machine (zero-tick timing tolerance). Measure wall time for a full headless flight and playback responsiveness on the reference laptop. |

Product requirement coverage maps setup to P01, the closed-loop flight to P02, gate and landing fixtures to P03–P04, terminal precedence to P05, live viewer parity to P06, bundle replay to P07, and repeatability and result ordering to P08.

CI runs contract tests and one complete DIRECT mission for each proposed change. For release, run ten fresh processes with the same locked configuration on the reference machine. Require identical gate sequences, reason and terminal tick, and maximum same-tick trajectory position difference ≤1 mm. Keep a representative run bundle and CI failure artifacts for diagnosis. Record test results, measured resource use, and any verified platform limits before claiming laptop support. If a vehicle can pass through a frame without a reported contact at supported speeds, add a swept-volume collision check and rerun the gate and touchdown fixtures. Changing the published clock requires a new version of the benchmark.

## Upstream references

- Vehicle setup, motor action shape, client access and PID control loop. [Upstream PID example](https://github.com/learnsyslab/gym-pybullet-drones/blob/348b5c18a4bb770a48928b9d044decf77ca33302/gym_pybullet_drones/examples/pid.py)
- Python version and full environment dependencies. [Upstream package manifest](https://github.com/learnsyslab/gym-pybullet-drones/blob/348b5c18a4bb770a48928b9d044decf77ca33302/pyproject.toml)
- Simulation reset, observation and physics stepping interfaces. [BaseAviary](https://github.com/learnsyslab/gym-pybullet-drones/blob/348b5c18a4bb770a48928b9d044decf77ca33302/gym_pybullet_drones/envs/BaseAviary.py)
