# ADR-001 — Separate Ubuntu host services from Windows VM workloads

## Status

Accepted

## Context

The infrastructure lab needs to support both Linux-native Data/ML workloads and Windows-only tooling. The first version placed too much tooling inside the Windows VM, including data-science and container-related components.

This created instability and resource pressure. Docker Desktop inside the Windows VM was especially problematic because the VM environment did not provide the same assumptions as a physical Windows workstation.

## Decision

Keep the architecture split:

- **Ubuntu host**: Linux administration, Docker, Jupyter, Python, future PostgreSQL/DataOps services.
- **Windows VM**: Windows-only tools, vendor software, backtesting/market-data tools, office utilities.

## Consequences

### Positive

- Cleaner separation between host services and Windows workload tools.
- Better alignment with production Data/MLOps runtime patterns.
- Lower risk of VM instability affecting Linux-native services.
- Easier documentation of service responsibility.

### Negative

- More explicit network and file-sharing design is required.
- Some workflows require crossing host/VM boundaries.
- The operator must maintain both host-level and VM-level checklists.

## Alternatives considered

| Alternative | Reason rejected |
|---|---|
| Put everything in the Windows VM | Too fragile for Docker/Jupyter and not representative of Linux production runtimes |
| Use only Ubuntu, no Windows VM | Does not support Windows-only workload requirements |
| Use a public cloud VM only | Less useful for practicing host-level virtualization and recovery |

## Portfolio signal

This decision demonstrates that infrastructure design is not only about installing tools. It requires runtime isolation, failure-domain thinking and operational trade-offs.
