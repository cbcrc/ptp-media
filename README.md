<!--
SPDX-FileCopyrightText: 2026 CBC/Radio-Canada
SPDX-License-Identifier: CC-BY-4.0
-->

# ptp-media

> **Coming soon.** The first release is in final testing on production-like
> hardware. Watch this repository to be notified when it is published.

Helm charts for **SMPTE ST 2059-2 PTP on Kubernetes media nodes** with
NVIDIA/Mellanox ConnectX NICs and redundant red/blue PTP networks.

Each node's system clock is disciplined from the NIC's hardware clock. Every
node continuously proves that its time is right. Pods get the PTP hardware
clock as a schedulable device.

## What is coming

Two charts, installed in order:

| Chart | What it does |
|---|---|
| `ptp-operator` | The upstream [k8snetworkplumbingwg PTP Operator](https://github.com/k8snetworkplumbingwg/ptp-operator), packaged as a Helm chart for Kubernetes without OLM or cert-manager, pinned to one upstream commit. |
| `ptp-media` | The site layer: per-node `ptp4l` and `phc2sys`, a read-only clock monitor, NIC and firmware checks, the card clock as a device for pods, and Prometheus alerts. |

Highlights:

- **ST 2059-2 defaults.** Domain 127, UDP/IPv4, E2E delay, slave-only
  ordinary clock, with red/blue failover handled by `ptp4l` on a shared card
  clock.
- **A lock is not proof of time.** The clock monitor goes NotReady when the
  system clock does not follow the card clock, when no fabric is locked, or
  when the grandmaster is not traceable. A pod that stays `Running` while
  the time is wrong is the failure it is built to catch.
- **Correct UTC.** `phc2sys -w` takes the UTC offset from `ptp4l`, never a
  hard-coded `-O`.
- **The card clock for media apps.** The NIC's PTP hardware clock is offered to
  pods as an extended resource, so media apps can run unprivileged. A
  read-only variant is available, and the monitor detects anything else
  adjusting the clock.
- **Checks on your hardware.** At runtime it verifies the NIC model, the clock
  layout and the `REAL_TIME_CLOCK_ENABLE` firmware setting against your
  values.
- **A read-only audit tool** that reports NTP and PTP health for every node of
  a cluster.
- **Design notes** explaining why each check exists.

## Planned requirements

- Kubernetes with Helm 3.17 or later. Tested on k3s.
- NVIDIA/Mellanox ConnectX NICs with hardware timestamping. Tested on
  ConnectX-6 Dx and ConnectX-7.
- A grandmaster reachable on each PTP network, and an IPv4 address on every
  PTP interface, owned by the host's network configuration.

## License

Copyright 2026 CBC/Radio-Canada. Code will be released under the Apache
License, Version 2.0, and documentation under CC BY 4.0.
