# Certification Alignment — Linux Foundations (Milestone 00)

This guide maps the Linux topics practiced in Sage's initial system and storage inspection to relevant certification skill areas. This is **study alignment, not proof of exam completion**. Verify requirements against the current official objectives before exam preparation. All examples are sanitized.

| Topic practiced | RHCSA EX200 relevance | LFCS relevance |
| --- | --- | --- |
| Shell commands, options, `man`, `less`, `cat` | Essential tools, system documentation | Essential commands and documentation |
| `uname`, `hostnamectl`, `lscpu`, `free`, `dmidecode` | Basic system inspection and troubleshooting supporting system maintenance | Operating system diagnostics and performance investigation |
| `ls -la`, `stat`, ugo permissions and numeric modes | Essential tools: file permissions | File access controls and ownership |
| `lsblk`, `df -hT`, partition and filesystem inspection | Filesystem and storage management | Storage management |
| LVM: `pvs`, `vgs`, `lvs` | Create and extend logical volumes | Configure and manage LVM |
| `/etc/fstab`, UUIDs, swap, `findmnt` | Mount and use filesystems; persistent mounts | Filesystems, storage and automounting |
| NFSv3/v4, `hard`, `nofail`, `_netdev`, systemd automount | Mount NFS and configure autofs | Remote filesystems and automounters |
| `systemctl status`, `is-active`, `is-enabled`, `--no-pager` | System service management | Service configuration and troubleshooting |
| SSH remote access and inspection | Secure remote access | OpenSSH configuration and troubleshooting |

## Primary certifications

**RHCSA EX200 — Red Hat Certified System Administrator.** Strong practical relevance for permissions, LVM, filesystems, NFS, automounts, service management and core shell usage. Our Sage exercises run on Ubuntu, while EX200 assesses Red Hat Enterprise Linux; practice on RHEL is also required. SELinux, package management, firewalls and other EX200 objectives remain to be covered.

**LFCS — Linux Foundation Certified System Administrator.** Strong relevance for Linux commands, storage, filesystem permissions, services, automounters and remote access. Unit 00.1 is only an introductory subset of LFCS preparation.

## Foundational alignment for later milestones

**Red Hat EX188 (Containers):** Linux permissions, storage, processes and services prepare for Podman, images, registries and container lifecycle work. We have not completed container-specific objectives yet.

**RHCE EX294:** Linux foundations support later Ansible automation; this unit is not an EX294 automation exercise.

**CKA / CKAD / CKS and other Kubernetes certifications:** Linux services, mounts and access controls are useful prerequisites, but these tasks do not directly demonstrate Kubernetes exam proficiency.

**AWS Solutions Architect / DevOps Professional:** Operating systems, access controls and storage underpin cloud infrastructure knowledge, but this unit is not AWS-specific exam practice.

## Mapping to the review lab

| Lab task | Primary alignment |
| --- | --- |
| 1. Identify OS, hostname and kernel | Essential tools and system inspection |
| 2. CPU, memory, swap | System diagnostics |
| 3. Disk layout and LVM | Storage and filesystems |
| 4. Permission changes | File permissions and access |
| 5. `stat`, inode, timestamps | File metadata and troubleshooting |
| 6. Interpret `fstab` and NFS options | Persistent and remote mounts |
| 7. Inspect systemd services and pagers | Service management, documentation |
| 8. Troubleshoot `VFree` vs filesystem space and automount state | Storage and service troubleshooting |

## Next practice steps

1. Repeat the Linux foundations exercises in a disposable **RHEL** environment for EX200.
2. Create and extend LVM storage on test disks, rather than only inspecting existing volumes.
3. Practice NFS and automount configuration using fictional test resources, not production shares.
4. Practice safe service-management tasks and troubleshoot with journal logs.
5. Start Podman-specific EX188 practice during the container milestone.

## Official certification sources

- [Red Hat RHCSA EX200 exam](https://www.redhat.com/en/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam)
- [Linux Foundation LFCS](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Red Hat EX188 Containers exam](https://www.redhat.com/en/services/training/ex188-red-hat-certified-specialist-containers-exam)
- [Red Hat RHCE EX294 exam](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam)

Related study documents: [Command reference](linux-foundations-command-reference.md), [Review lab](linux-foundations-review-lab.md), [Permissions guide](linux-permissions-study-guide.md).
