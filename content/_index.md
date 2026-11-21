+++
title = "Tensorflight"
description = "An eBPF and libbpf-based GPU/CUDA ioctl syscall monitoring and forensics tool written in C on Linux."
+++

Tensorflight is an open-source GPU runtime observability and digital forensics research project built in pure C using eBPF and [libbpf](https://docs.kernel.org/bpf/libbpf/libbpf_overview.html).

It captures and analyzes GPU-related kernel interactions by observing `ioctl` system calls and their associated device and process context. Operating at the kernel boundary, Tensorflight provides low-overhead telemetry of GPU driver interactions without requiring instrumentation, recompilation, or modification of target applications.

The project aims to establish a foundation for kernel-level GPU runtime observability, event recording, and forensic analysis.

Tensorflight is still in **very early** stage, if you want to contribute the repository is in [GitHub](https://github.com/ezrantn/tensorflight).

## Core Capabilities

- **Kernel-Level Observation:** Monitor GPU-related `ioctl` activity through eBPF.
- **Command Decoding:** Extract direction, payload size, type, and command number from `ioctl` command identifiers.
- **Process and Device Attribution:** Associate observed events with process metadata and device context.
- **Bounded Argument Inspection:** Perform verifier-safe inspection of accessible syscall argument data.
- **Structured Event Streaming:** Transfer kernel-generated telemetry to userspace through BPF ring buffers.

## Overview

Modern GPU and CUDA workloads rely heavily on `ioctl` system calls to communicate with kernel drivers and device nodes (such as `nvidia-uvm` and related character devices). While high-level profilers and application logs show *what* an application reports doing, tracing `ioctl` traffic directly at the syscall layer provides a ground-truth view of control and data-plane interactions without instrumenting target binaries.

Tensorflight approaches this problem through small, verifiable layers:

1. **Command Decoding (`cmd`):** Unpacking 32-bit `ioctl` command numbers in kernel space into direction (`_IOC_DIR`), payload size (`_IOC_SIZE`), type/magic byte (`_IOC_TYPE`), and command number (`_IOC_NR`).
2. **Device & Process Attribution:** Correlating events with process metadata (`PID`, `COMM`) and target device descriptors so telemetry stays focused and legible.
3. **Bounded Payload Inspection:** Safely reading and analyzing argument pointers (`arg`) under strict BPF verifier constraints and driver-specific `ioctl` conventions.
4. **Forensic Telemetry & Event Streaming:** Shipping structured events from kernel space to user space via ring buffers for inspection, auditing, and anomaly/forensics analysis.

## Goals

- **Zero-Target Instrumentation:** Observe GPU/CUDA `ioctl` activity from the kernel without modifying, recompiling, or injecting libraries into user-space workloads.
- **Verifier-Safe Kernel Extraction:** Extract `ioctl` metadata and device context reliably while satisfying strict eBPF verifier bounds on loops, memory reads, and stack usage.
- **Forensic Visibility:** Capture clear, structured call records (`PID`, command name, device, direction, magic, command number, and payload size) to support low-level debugging and GPU/driver forensic analysis.
- **Modular Architecture:** Keep decoding, device resolution, and argument/payload inspection decoupled so each stage can be tested, reasoned about, and extended independently.

## How to Run

Before CLI flags are added, the binary runs with default hardcoded behavior (streaming NVIDIA `ioctl` events to `stdout` until SIGINT):

```bash
make
make test
sudo ./.output/tensorflight
```

In a second terminal, trigger GPU activity (for example `nvidia-smi` or a CUDA workload), then press Ctrl+C in the collector terminal to exit cleanly.