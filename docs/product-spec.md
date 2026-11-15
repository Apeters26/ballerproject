# Autonomous drone simulator MVP — product specification

**Status** Proposed for implementation. **Audience** The two people building and testing the simulator.

## Purpose and first flight

Give two Python developers a common course on which they can build and evaluate autonomous flight controllers. Their first shared result is a flight they can both reproduce and inspect. One simulated quadcopter takes off, passes five gates in order and in the designated direction, and lands on the marked pad. The pilot supplies no flight commands after starting the run.

The MVP ships one fixed course and one baseline controller. The controller knows the course layout and receives the drone's simulated state. A waypoint navigator and a conventional PID flight controller make autonomous flight possible on an ordinary CPU laptop. Model training, camera-based navigation, simultaneous racing and physical aircraft integration are future work. See the [enhancement roadmap](roadmap.md).

## People and workflows

| User | Job to be done | Evidence the product delivered |
| --- | --- | --- |
| Developer running the simulator for the first time | Install it from a fresh checkout and watch an autonomous flight locally. | The quickstart names the supported environment and commands. The developer can identify the launch point, five gates, landing pad and final result. |
| Developer changing the navigator or controller | Run the known course repeatedly without a display and see where a change improved or broke the flight. | Two runs can be compared using completion, gates cleared, terminal reason and simulated time. Their saved records identify the course and controller versions. |
| Collaborator reviewing a run | Understand a success or failure from the recording, including when the course judge awarded a gate. | The developer can open the saved run offline, inspect the path and identify the last accepted gate without reproducing the flight first. |

## Operator experience

1. **Start.** The quickstart offers a visual demonstration and a headless run using the same default course and baseline controller. Starting headless requires neither a window nor an NVIDIA GPU. The run identifies the selected course, controller and destination for its record.
2. **Follow the flight.** In the visual run the operator can distinguish the drone, course order and landing pad. Gate progress comes from the course judge. An operator can tell the difference between approaching gate 3 and receiving credit for it.
3. **Read the result.** At termination the operator sees success or failure, gates cleared out of five, elapsed simulated time and where to find the recording. If a flight fails, the report names the reason and shows how far the drone got.
4. **Review and repeat.** Open the saved record to replay the path against the same gate geometry, then run the baseline again from the same inputs. A later run preserves the earlier evidence so the two developers can compare results.

## Requirements and acceptance evidence

| ID | Product requirement | Acceptance evidence |
| --- | --- | --- |
| P01 | Provide a local, repeatable setup with a CPU-only headless path and an optional live view. | On a documented supported laptop environment, a developer follows the quickstart from a clean checkout, runs headless and opens the live demonstration. The quickstart records any display prerequisites. |
| P02 | Fly the full mission without operator steering. | From the specified start the baseline takes off, clears gates 1–5, lands and finishes within the course time limit. The result shows 5/5 and success. |
| P03 | Award each gate once, in order, for a legal crossing through the opening in the prescribed direction. | A fixture crossing the next gate legally increments the count once. Approaching without crossing, crossing in reverse, crossing outside the opening, repeating a gate or skipping ahead earns no credit for that crossing. |
| P04 | Require a valid landing after completing the gates. | A stable touchdown inside the specified landing area after gate 5 can complete the mission. Flying through gate 5, skimming the pad or touching down outside the landing area cannot produce success. |
| P05 | Bound every flight and explain its outcome. | Tests exercise a collision and the time limit. The saved result names the reason, last accepted gate and terminal simulation time. An unusable input or startup error produces an actionable error distinct from a scored flight failure. |
| P06 | Use identical flight and judging rules in headless and visual modes. | On the same supported environment and settings both modes award the same gates in the same order, report the same terminal reason and agree on elapsed simulation time within the technical tolerance. |
| P07 | Preserve a run record sufficient for independent review. | A collaborator receives the saved output, reads its course, controller and run settings, and opens an offline path replay showing gate geometry, trajectory and gate events through termination. Failed flights can be inspected the same way. |
| P08 | Make baseline results repeatable and comparable. | Identical pinned inputs on the same supported environment yield matching gate order and outcome, and timing within the technical tolerance. A successful run ranks ahead of every unsuccessful run. Among successful runs on the same course and settings, lower elapsed simulated time is better. |

For P03–P06 the [technical specification](technical-spec.md) sets the geometry, crossing and touchdown tolerances, timing convention and terminal-event precedence. Elapsed time is measured by the simulator so a slower laptop does not receive a worse score. Partial gate counts are diagnostic and can never imply a successful finish.

### How two collaborators would read their results

Suppose one controller lands after 44 simulated seconds. A change brings that down to 39 seconds on the same course. If both runs cleared all five gates and landed, the second run is faster. If the second run hit the ground outside the pad at 39 seconds, the 44-second run wins. Even a 5/5 gate count is compatible with a failed landing. The result and replay must make that distinction visible. These times illustrate scoring and do not set a performance target.

## Release decision

The first developer records the default headless success and shares the recording and exact reproduction command. The second follows the quickstart from a clean checkout, watches a visual flight and reviews the shared recording offline. The implementation PR includes the tested OS, Python and dependency versions, the automated acceptance results, and a failure recording from a controlled test case. Release requires evidence for P01–P08. Successful flight on the single known course establishes the baseline for later challenges. Performance on unfamiliar courses is a separate roadmap milestone.
