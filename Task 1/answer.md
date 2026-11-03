# Task 1 – Evaluate the Requirements of Each Computing Environment (4 Marks)

**Name:** K Avinash  **Register No:** 192411044  **Course:** CSA04 – Operating Systems, Assignment 1 (CO1)

## Environments

| Code | Environment | Key demand from the scenario |
|---|---|---|
| E1 | Bedside patient monitor | Alarm within 10 ms of an abnormal reading; runs for years without reboot |
| E2 | Nurse handheld tablet | Lasts a full shift on battery; touch UI; secure access to records |
| E3 | Doctor desktop workstation | Many applications, multitasking, rich user interface |
| E4 | Central database server | Must never stop, even during updates; many concurrent users |
| E5 | MRI imaging server | Heavy parallel computation over large images using many processors |
| E6 | Partner-hospital link | Share records with two partner hospitals securely over a network |

Device growth: 1 ward (40 monitors, 10 tablets) -> 1 hospital (500 monitors, 120 tablets, 80 workstations) -> 3 hospitals sharing records.

## 1.1 Requirements ranked per environment (1 = most important)

R = response time, L = reliability, T = throughput, P = battery/power, S = security.

| Env. | Rank 1 | Rank 2 | Rank 3 | Rank 4 | Rank 5 |
|---|---|---|---|---|---|
| E1 Monitor | R – deterministic response | L – reliability | S – security | P – power | T – throughput |
| E2 Tablet | P – battery | S – security | L – reliability | R – response (soft) | T – throughput |
| E3 Workstation | S – security | T – throughput (multitasking) | L – reliability | R – response (soft) | P – power |
| E4 DB server | L – never stops | S – security | T – transactions/s | R – response (soft) | P – power |
| E5 Imaging server | T – throughput | L – reliability | S – security | R – response (soft) | P – power |
| E6 Partner link | S – security | L – reliability | T – throughput | R – response (soft) | P – power |

### Justification of each ranking

| Env. | Justification |
|---|---|
| E1 Monitor | A late alarm can be fatal, so a **bounded** (hard) response comes first; the 10 ms deadline is a correctness requirement, not a speed preference. A crashed monitor is blind, so reliability is second. It holds identifiable data and has a network port, so security follows. It is mains-powered, so power is minor; it handles few small data items, so throughput is last. |
| E2 Tablet | The clinical concern is that the battery will not last a shift, so power is first. The tablet carries records outside the ward and can be lost, so security (encryption, remote wipe, app sandbox) is second. A short delay in showing a chart is tolerable (soft), so response is fourth. |
| E3 Workstation | Doctors open many programs and view confidential data, so protection between programs/users and multitasking throughput dominate. Mains-powered; a delay of tens of ms is acceptable. |
| E4 DB server | The requirement is "must never stop"; downtime or lost records affect every ward, so reliability/availability is first. It holds all records, so security is second. Throughput (transactions per second) is third; a 100 ms look-up is acceptable. |
| E5 Imaging server | Value comes from finishing large MRI reconstructions quickly with many processors, so throughput is first. A failed job can be restarted, so reliability is second. It is not time-critical for the patient at the bed. |
| E6 Partner link | Records leave the hospital, so confidentiality, authentication and integrity are first; a dropped link is recoverable by retry, so reliability is second. |

## 1.2 Conflicting requirements

- **Fast, bounded response vs many users/programs on one machine (E1/E3):** time-sharing gives fairness and average speed, not a guarantee; a monitor must not share its CPU with unrelated programs.
- **Throughput vs response (E5):** batch-style scheduling with long time slices maximises throughput but makes interactive response poor.
- **Battery vs performance (E2):** a fast CPU, bright screen and constant radio drain the battery; the OS must sleep aggressively, which adds wake-up latency.
- **Security vs speed:** encryption and sandboxing cost CPU time (acceptable on E4/E6, but they add jitter on E1 unless done in the gateway).
- **Availability vs ease of updating (E4):** patching normally needs a restart; fail-over and rolling updates are needed so "never stop" and "keep patched" can both hold.
- **Rich features vs small, verifiable kernel (E1):** a full OS has millions of lines of code, more bugs and unpredictable timing.

## 1.3 Requirements critical to patient safety

- E1: the 10 ms deadline and continuous operation (a missed alarm is a direct clinical hazard).
- E4: availability and data integrity (wrong or unavailable records lead to wrong treatment).
- E2/E3/E4/E6: protection – one patient's data must not be readable or modifiable by another user or program.
- E5: correctness of results is safety-relevant (misleading images), but a delay is not.

## 1.4 Lessons from the history of operating systems

| Generation | Idea | Where it applies in the hospital |
|---|---|---|
| Batch (1950s–60s) | Jobs queued and run to completion without interaction; goal is CPU utilisation | E5: MRI reconstructions submitted as jobs in a queue; also nightly backups and reports on E4 |
| Time-sharing (1960s–70s) | CPU switched rapidly between users; interactive; needs protection and fair scheduling | E4 and E3: many users/queries share one machine |
| Personal (1980s–) | One user per machine, good GUI, responsiveness over utilisation | E3: doctors' desktops |
| Mobile / handheld (2000s–) | Touch UI, power management, app sandbox, wireless | E2: tablets suspend apps and radios to save battery |
| Real-time / embedded | Predictable timing is correctness; minimal footprint | E1: bedside monitors |

## Alternative Concept and Complexity Analysis – one general-purpose OS on every device

**Solution if used:** every device (monitors, tablets, workstations, servers) runs the same general-purpose OS (e.g. desktop Linux or Windows): one image, one set of tools and drivers.

| Criterion | Type-per-environment (proposed) | Single general-purpose OS |
|---|---|---|
| Worst-case alarm response (E1) | RTOS with priority pre-emption: worst case = interrupt latency + context switch + task run time, about tens of µs to under 1 ms, **bounded and independent of load**: O(1) | Time-sharing scheduler, page faults, disk I/O, drivers and updates: delay grows with the number n of runnable tasks and background work (about n x time slice in the worst case, often tens to hundreds of ms): **unbounded**, cannot guarantee 10 ms |
| Memory per device (space) | E1: tens of KB to a few MB. E2: about 1–3 GB. Servers: as required | Every device carries the full kernel, services, GUI and libraries: typically 0.5–2+ GB RAM and several GB storage even on a monitor: O(size of full OS) per device |
| Administration effort | k different OS types (here 5): k patch processes and skill sets: O(k) types x D devices | One type: O(1) types, still O(D) devices to patch, but patches are large and frequent and every patch risks the monitors (reboots, regressions) |

*Figures such as memory sizes and delays are typical values, not measurements made in this assignment.*

**Evaluation:** the single-OS plan lowers administration (k = 1) but fails the safety-critical requirement of E1 and wastes memory and power on E1 and E2. The per-environment plan is chosen: the extra administration (5 OS images) is manageable; a missed alarm is not.
