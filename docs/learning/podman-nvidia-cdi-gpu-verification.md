# Podman and NVIDIA CDI — GPU Access Verification

**Track:** Linux foundations / container preparation (Red Hat EX188 concepts, with RHCSA EX200 and LFCS foundational commands). This document uses a **generic, fictional GPU host**. Do not publish real GPU UUIDs, LAN addresses, exact internal topology, authentication configuration or raw operational logs.

## Scenario and findings

A Linux host with two NVIDIA GPUs uses rootless Podman and the `crun` OCI runtime. Following resolution of a Podman/AppArmor executable-profile mismatch, CDI device discovery and two separate, lightweight container inspections succeeded.

| Check | Observed result |
| --- | --- |
| `podman info --format '{{.Host.Security.Rootless}}'` | `true` |
| `podman info --format '{{.Host.OCIRuntime.Name}}'` | `crun` (local executable path) |
| `nvidia-ctk cdi list` | Individual GPU devices by index and UUID, plus `nvidia.com/gpu=all` |
| Container test A, device `nvidia.com/gpu=1` | Only the selected NVIDIA GPU visible via `nvidia-smi` |
| Container test B, device `nvidia.com/gpu=0` | Only the other selected GPU visible via `nvidia-smi` |

**Verification scope:** These checks establish basic per-device container visibility. They do **not** prove GPU computation, model inference, GPU performance, long-running reliability, or that every containerized application will function. Both tests were idle `nvidia-smi` queries, not stress tests.

## Device mapping

Host GPU indexes may differ from GPU indexes *inside* a container. If only one physical GPU is exposed through CDI, the tool inside the container can report it as `GPU 0` even if it is `GPU 1` on the host. Favor GPU UUIDs over host numeric indexes for durable per-service assignments; store actual UUID mappings in private inventory.

```text
Fictional Linux host
  ├── GPU A ── CDI individual device ── auxiliary container
  └── GPU B ── CDI individual device ── primary inference container
                    |
              Podman (rootless)
                    |
                  crun
                    |
               Linux kernel
```

## Commands and flags

```bash
podman --version
podman info --format '{{.Host.Security.Rootless}}'
podman info --format '{{.Host.OCIRuntime.Name}}'
nvidia-smi
lspci | grep -i nvidia
nvidia-smi --query-gpu=index,name,uuid,memory.total --format=csv
nvidia-ctk cdi list
podman images
```

| Command or option | Explanation |
| --- | --- |
| `lspci` | PCI-connected hardware information |
| `|` | Pipeline; send standard output to next command |
| `grep -i` | Case-insensitive text match |
| `nvidia-smi` | NVIDIA driver/device information (read-only inspection) |
| `--query-gpu=` | Select GPU properties |
| `--format=csv` | Print comma-separated output |
| `nvidia-ctk cdi list` | List available CDI devices (discovery, not a container test) |
| `podman images` | List images in the current user's container storage |
| `--format '{{...}}'` | Podman Go-template field selection |
| `--device nvidia.com/gpu=...` | Request a CDI GPU inside the container |
| `--pull=never` | Do not download an image; require it locally |
| `--rm` | Remove the temporary container on exit |

**Example (use only on an isolated test host with an image already cached):**

```bash
podman run --rm \
  --pull=never \
  --device nvidia.com/gpu=0 \
  docker.io/nvidia/cuda:13.0.2-base-ubuntu24.04 \
  nvidia-smi
```

The image tag is a tested **example**, not an assurance of ongoing freshness. Do not assume CUDA code can run merely because `nvidia-smi` reports driver-supported CUDA version. Image compatibility and required libraries must be tested separately.

## Scenario-based review lab (30 minutes; 100 points)

**Safety:** Do not modify real GPU routing or application containers for this exercise. Use a disposable GPU-capable lab machine if available. Otherwise perform the investigation/interpretation portions on paper or a CPU-only VM. Avoid simultaneous GPU stress tests.

1. **Inventory (15):** Explain which command shows PCI devices and which checks NVIDIA driver communication. Explain the `.0` GPU and `.1` audio PCI functions.
2. **Runtime inspection (15):** Determine Podman version, rootless status, configured OCI runtime, and list of existing images. Explain the template fields in `podman info --format`.
3. **CDI discovery (15):** Explain why a two-GPU machine can show five CDI selectors (two indexes, two UUIDs, `all`). Identify which form is better for durable configurations.
4. **Container command comprehension (20):** Explain each flag in the sample `podman run` command. Predict what happens if the image is absent and `--pull=never` is used.
5. **GPU mapping (15):** A fictional host lists devices 0 and 1. A container exposes host GPU 1 but its `nvidia-smi` lists GPU 0. Explain how this can be correct.
6. **Troubleshooting (10):** `podman --version` works but `podman info` returns permission denied. Describe two read-only checks and explain why broad `chmod` changes are inappropriate.
7. **Verification boundaries (10):** Differentiate CDI registration, GPU visible inside a container, and successful CUDA computation. State what still must be tested before serving AI models.

**Scoring:** 80/100 target. Record commands, flags, expected outcome, observed outcome, issues and recovery. Evaluate conceptual tasks without requiring a physical GPU.

**Answer guidance:** A missing local image with `--pull=never` should fail instead of downloading. GPU indexes can be remapped inside containers. CDI list alone does not establish successful container access. AppArmor denials may require examining `journalctl -k`, `aa-status`, and executable paths without disabling global security.

## Certification alignment

- **Red Hat EX188:** Relevant conceptual and hands-on exposure to Podman commands, rootless mode, images, container lifecycle, and device handling. NVIDIA CDI itself is supplementary; this test alone does not fulfill all EX188 objectives. Practice on the environment specified by the official exam.
- **RHCSA EX200 / LFCS:** Supporting Linux device discovery, shell pipelines, process and service troubleshooting, and security/log inspection.
- **CKA/CKS and MLOps:** Later prerequisite knowledge; no Kubernetes objective completed by these tests.

Related: [Linux command reference](linux-foundations-command-reference.md) · [AppArmor troubleshooting](rootless-podman-apparmor-troubleshooting.md) · [Certification alignment](linux-foundations-certification-alignment.md).
