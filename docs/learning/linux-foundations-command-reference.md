# Linux Foundations — Command and Configuration Reference

**Track:** Milestone 00 / Unit 00.1 · Linux administration practice (RHCSA EX200 and LFCS overlap; verify current official exam objectives). All examples are generic; no operational hostnames, IP addresses, user identities, or credentials.

## Reading syntax and getting help

In `command -x --long-option argument`, options change command behavior; **the same letter does not mean the same thing for every command**. For example, `uname -r` means kernel release; `cp -r` means recursive copy; `ls -r` means reverse sort.

| Pattern | Meaning |
| --- | --- |
| `man COMMAND` | Open manual in a pager; `q` exits |
| `COMMAND --help` | Display command's help (where supported) |
| `less FILE` | Read with scrolling; `/text` searches; `n` repeats search; `q` exits |
| `more FILE` | Simpler pager |
| `cat FILE` | Print file contents directly |
| `--no-pager` | For supported commands such as `systemctl`, print directly rather than opening `less`; does **not** remove other output limits |
| `sudo COMMAND` | Run with elevated privileges according to sudo policy; only use when needed |

## OS, kernel, host, CPU, RAM

| Command | What it tells you | Option explained |
| --- | --- | --- |
| `cat /etc/os-release` | Distribution ID, release and codename | `cat` needs no flag |
| `uname` or `uname -s` | Kernel name, e.g. Linux | `-s` = kernel name |
| `uname -r` | Running kernel release | `-r` = release |
| `uname -m` | Machine architecture | `-m` = machine |
| `uname -a` | All available kernel/system identification | `-a` = all |
| `hostname` | Current hostname | No flag |
| `hostnamectl` | Hostname, OS, kernel, architecture, firmware info when available | systemd hostname management |
| `lscpu` | CPU model, cores, logical CPUs, threads, virtualization, caches | No flag |
| `free -h` | RAM, swap, available memory | `-h` = human-readable |
| `sudo dmidecode -t memory` | Firmware-reported DIMM capacities, slots, speeds | `-t` = DMI type; `memory` selects memory entries |

**Interpreting RAM:** `free` is unused RAM; `available` estimates capacity available for new applications (including reclaimable cache). Memory module rated speed and configured running speed can differ; do not modify BIOS profiles as part of an inventory exercise. Linux often reports GiB, while marketing labels use GB.

## Block storage and mounted filesystems

| Command | Purpose | Flags |
| --- | --- | --- |
| `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS` | Disks → partitions → logical volumes and where mounted | `-o` = choose output columns |
| `df -hT` | Filesystem used/free space and filesystem types | `-h` = human-readable; `-T` = filesystem type |
| `sudo pvs` | LVM physical volumes (PVs) | `PFree` = unallocated capacity in PV |
| `sudo vgs` | LVM volume groups (VGs) | `VFree` = space not assigned to LVs |
| `sudo lvs` | LVM logical volumes (LVs) | `LSize` = allocated LV capacity |
| `findmnt /example/mount` | Show what is mounted at a path | Can show both automount `autofs` and underlying NFS |
| `ls -la PATH` | Detailed directory listing incl. hidden entries | `-l` = long; `-a` = all |
| `stat PATH` | Inode, owner, permission mode, timestamps and link count | No flag |
| `ls -ld DIRECTORY` | Details *about directory itself* | `-d` = directory entry, not its contents |

**LVM:** physical volume (PV) → pooled volume group (VG) → allocated logical volumes (LVs) → filesystems → mount points. A VG with `VFree=0` can still have lots of free space **inside** its mounted filesystems. LVM on one physical drive does not add redundancy.

**Mounts:** `/` is the root filesystem; `/boot` carries kernel and boot assets; `/boot/efi` carries UEFI boot assets; `/srv/application` is an example of a dedicated service-data mount. A `tmpfs` is memory-backed, and its reported *size* is a limit, not necessarily used RAM.

## /etc/fstab: mount configuration

Inspect without changing anything: `cat /etc/fstab`.

Six fields: `<source> <target> <fstype> <options> <dump> <pass>`.

| Field | Meaning |
| --- | --- |
| source | Filesystem UUID, device path, or network export |
| target | Directory where mounted |
| fstype | e.g. `ext4`, `vfat`, `nfs4`, `swap` |
| options | Comma-separated rules for mounting |
| dump | Legacy dump-backup indicator; commonly `0` |
| pass | Boot filesystem check priority: `0` skip, typically `1` root, `2` other local filesystems |

A UUID is a stable filesystem identifier that is generally safer than relying on a disk name like `/dev/sda` which can change.

**Sanitized NFS example (illustration only; do not paste into a real fstab):**

```fstab
192.0.2.10:/export/training /mnt/training nfs4 rw,vers=4.1,hard,_netdev,nofail,x-systemd.automount 0 0
```

| Option | Meaning |
| --- | --- |
| `rw` | Request read/write mounting; server permissions still apply |
| `vers=4.1` / `vers=3` | Select NFS protocol version |
| `proto=tcp` | Use TCP transport |
| `hard` | Retry failed NFS requests, potentially blocking accessing processes |
| `_netdev` | Network-dependent mount |
| `nofail` | Don't make boot fail just because the mount fails |
| `x-systemd.automount` | Create a systemd automount that activates on access |

An active automount unit with `active (waiting)` is ready to trigger, but **does not prove the remote share is reachable**. Accessing the path can trigger a real NFS mount; `findmnt` may show two entries: `autofs` and the actual `nfs4` filesystem.

## systemd and service management

```bash
systemctl status ssh --no-pager
systemctl is-active ssh
systemctl is-enabled ssh
systemctl status mnt-training.automount --no-pager
```

- `status`: diagnostic detail; `is-active`: current runtime status; `is-enabled`: whether enabled at boot. Enabled and active answer different questions.
- `--no-pager` exits directly instead of opening a pager.
- systemd generated units may represent fstab mounts and automounts. Paths containing hyphens can require escaped unit names; use `systemd-escape -p --suffix=automount /mnt/example-path` to get the correct unit name.
- Inspecting service status is read-only; avoid restarting SSH on a remote session without a recovery path.

## Permissions and metadata

- Owner/group/others each receive `r=4`, `w=2`, `x=1`.
- `drwxr-x---` is a directory with mode `750`; the first `d` identifies the file type.
- In `ls -l`, the number after the permissions is the **hard-link count**, not a permission level. NFS may report directory link counts differently from ext4.
- `stat`: `Access` (atime) means last recorded access, `Modify` (mtime) content/entry modification, `Change` (ctime) inode metadata or content change. `Birth` may be unavailable. `Change` is **not** creation.
- Directory `x` means traverse/search. A server-side NFS export policy or ACL can still restrict access beyond visible mode bits.

See [Linux permissions review cards](linux-permissions-study-guide.md) and the [hands-on review lab](linux-foundations-review-lab.md).

## Documentation routine

Record command, purpose, key flags, expected behavior, **observed** outcome, issues, and decisions. Keep real addresses, usernames, internal topology, screenshots with identifiers, access methods and secrets out of public study notes. Never copy raw `fstab` or config dumps from sensitive environments into public repos.
