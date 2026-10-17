# Linux Foundations — Scenario Review Lab

**Practice style:** Task-based, hands-on and optionally timed, inspired by certification terminal labs (not affiliated with or copied from Killer Shell). **All names, addresses and scenarios are fictional.** Suitable for a disposable Ubuntu VM or throwaway container where the stated commands exist. RHCSA EX200 / LFCS topic overlap, not a claim of official exam coverage.

## Rules

- **Do not use a production host, NAS share, real SSH configuration, or actual `/etc/fstab` for modifications.**
- Do read-only discovery on a disposable machine; do all changes in `~/linux-foundations-lab`.
- No external network, root access or real NFS server is required to complete the core lab.
- Time target: **35 minutes**; suggested passing score: **80/100**. Use `man`/`--help` as in a practical exam.

## Setup (on a disposable Linux environment)

```bash
mkdir -p ~/linux-foundations-lab/{reports,practice}
printf 'alpha\nbeta\ngamma\n' > ~/linux-foundations-lab/practice/notes.txt
printf '192.0.2.10:/export/training /mnt/training nfs4 rw,vers=4.1,hard,_netdev,nofail,x-systemd.automount 0 0\n' > ~/linux-foundations-lab/practice/fstab.sample
chmod 750 ~/linux-foundations-lab/practice
chmod 644 ~/linux-foundations-lab/practice/notes.txt
cd ~/linux-foundations-lab
```

These commands create only local practice files. `192.0.2.10` is a documentation-only IP address, not a live NAS target.

## Tasks (100 points)

**Task 1 — System identification (15 points).** Without changing the host, write your distribution name, kernel release, machine architecture and hostname in `reports/system.txt`. Include the **commands** used and explain what `uname -r`, `uname -m`, and `uname -a` each return.

**Task 2 — CPU and RAM (10 points).** In `reports/resources.txt`, record logical CPU count, cores per socket, threads per core, total memory, available memory and swap. Explain why `available` differs from `free`.

**Task 3 — Block devices versus filesystems (15 points).** Using inspection commands only, record the number of block disks visible, one mount point, its filesystem type and available space in `reports/storage.txt`. Explain the difference between `lsblk` and `df`, and between PV, VG and LV. If LVM is absent, explain how you verified that without creating LVM.

**Task 4 — Permissions (20 points).** Set `practice/notes.txt` to `640`; set `practice` to `750`. Verify using `stat`. In `reports/permissions.txt`, translate `rwxr-x---`, `rw-r-----` and `rwx------` to octal, and state what directory execute allows.

**Task 5 — File metadata (10 points).** Record the inode, hard-link count, owner, group and `mtime` for `practice/notes.txt`. Explain why `ctime` is not necessarily creation time. Show a command that inspects directory metadata rather than its children.

**Task 6 — Read an fstab example (15 points).** Interpret all six fields of `practice/fstab.sample`. Explain `hard`, `_netdev`, `nofail`, `x-systemd.automount` and `vers=4.1`. **Do not mount this sample or edit `/etc/fstab`.** Explain why `active (waiting)` alone does not prove remote access.

**Task 7 — Services and pagers (10 points).** Inspect an installed service with `systemctl status ... --no-pager` (or, if systemd is unavailable, explain this limitation). Record the difference between `is-active` and `is-enabled`; explain `less` and `q`.

**Task 8 — Troubleshooting scenario (5 points).** A fictional engineer reports: “Our VG has `VFree=0`, so the application filesystem must be full. Also, an automount is active, so the NAS is definitely reachable.” Explain **both** mistakes.

## Self-assessment / evaluator checklist

| Task | Full-credit evidence | Points |
| --- | --- | ---: |
| 1 | Four accurate facts; `uname` flags explained | 15 |
| 2 | CPU topology + memory/swap and free vs available explained | 10 |
| 3 | `lsblk`/`df` distinction; correct mount and PV/VG/LV explanation | 15 |
| 4 | Correct permissions and `stat` proof + translations | 20 |
| 5 | Correct `stat` metadata and timestamp interpretation | 10 |
| 6 | Six columns and mount options correctly interpreted, no live modifications | 15 |
| 7 | Service status, enablement and pager behavior explained | 10 |
| 8 | VG unallocated vs filesystem free; automount vs server access | 5 |
| **Total** | | **100** |

**Answers for conceptual checks** (look after attempting): `750`, `640`, `700`; a directory's `x` permits traversal; `ctime` tracks inode/status change; `VFree=0` only means no unallocated extents in the VG; automount waiting means a trigger is ready, not proof of a successful NFS connection.

## Optional challenge — Mock mount inspection

Inspect the sample line with `cat practice/fstab.sample`. Sketch the relationship:

```text
fictional Linux node
   ├─ local disk → partition → (optional) PV → VG → LV → filesystem
   └─ systemd automount → NFS request → fictional remote export
```

Label which parts can be verified locally without connecting to any server. No mount or service changes are needed.

## Retake checklist

- [ ] Complete in 35 minutes without notes
- [ ] Reach at least 80/100
- [ ] Explain every flag used
- [ ] Repeat missed tasks on another disposable environment
- [ ] Write one paragraph on what changed, what was verified, and what you learned

## Cleanup (only inside this dedicated practice directory)

After reviewing your reports: `rm -r ~/linux-foundations-lab` **only after confirming the exact path**. Never substitute production paths.

Related: [Command reference](linux-foundations-command-reference.md) · [Permissions study guide](linux-permissions-study-guide.md).
