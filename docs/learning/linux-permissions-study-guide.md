# Linux Permissions — RHCSA / LFCS Study Guide

Part of the Homelab/Sage **Milestone 00: Linux Foundations** study notes.

> Learning reference, not an official exam-objectives checklist. Confirm coverage against the current RHCSA EX200 and LFCS objectives.

## Memorize the three values

| Permission | Symbol | Value | File meaning | Directory meaning |
| --- | --- | ---: | --- | --- |
| Read | `r` | **4** | Read file contents | List directory entries |
| Write | `w` | **2** | Modify file contents | Create, remove, or rename entries (with execute) |
| Execute | `x` | **1** | Execute program/script | Traverse/search directory |
| None | `-` | **0** | No corresponding permission | No corresponding permission |

The permission triplets are **user (owner), group, others**. Add 4 + 2 + 1 within each triplet.

- `rwxr-x---` = **750** (owner 7, group 5, others 0).
- `rw-r--r--` = **644** (owner 6, group 4, others 4).
- `rwxr-xr-x` = **755**.
- `rwx------` = **700**.
- `rw-------` = **600**.
- `rwxrwxrwx` = **777**; avoid broad access unless deliberately required.

Example `ls -l` output: `drwxr-x--- 1 owner group 120 Oct 17 project`.

The first character `d` is the **type** (directory), not part of `750`. Other type examples include `-` for a regular file and `l` for a symlink. The number after the permissions is the **hard-link count**, not a permission.

## Inspect safely

```bash
ls -ld /path/to/directory
stat /path/to/directory
namei -l /path/to/file
```

`ls -ld` shows directory metadata itself, rather than listing its contents. `stat` shows numeric permissions and ownership. `namei -l` helps diagnose missing traversal permission along a path.

## Make changes (on disposable practice files only)

```bash
mkdir -p ~/permissions-lab
touch ~/permissions-lab/example.txt
chmod 600 ~/permissions-lab/example.txt
chmod u+rw,g-rwx,o-rwx ~/permissions-lab/example.txt
stat -c '%a %A %n' ~/permissions-lab/example.txt
```

`chmod` changes permission bits. `u` means owner, `g` group, `o` others; `+` adds permissions, `-` removes them, and `=` assigns them. A file's owner/group may be changed with `chown`/`chgrp` when appropriate; do not change production storage permissions as a practice exercise.

**Caveats:** ACLs, the file owner, directory traversal, mount/export settings (including NFS), and special permission bits may affect effective access. `777` does not override all of these controls. On NFS, the server may report link counts and permission behavior differently than local ext4.

## Ten review cards

Read each question, answer without looking, then reveal the answer column.

| # | Front | Back |
| ---: | --- | --- |
| 1 | Numeric value of read (`r`)? | **4** |
| 2 | Numeric value of write (`w`)? | **2** |
| 3 | Numeric value of execute (`x`)? | **1** |
| 4 | What does `7` represent? | `rwx` = 4 + 2 + 1 |
| 5 | What does `5` represent? | `r-x` = 4 + 1 |
| 6 | What does `750` represent? | `rwxr-x---` |
| 7 | What does `644` represent? | `rw-r--r--` |
| 8 | Typical private SSH key permission? | `600` (`rw-------`) |
| 9 | Directory execute permission means? | Permission to traverse/search it |
| 10 | What does the first `d` in `drwxr-x---` mean? | Directory file type |

## Self-check exercises

1. Convert `rwxr-----` to octal. **Answer:** `740`.
2. Convert `640` to symbolic permissions. **Answer:** `rw-r-----`.
3. What does `chmod 700 example-dir` allow? **Answer:** Owner full access; group and others no mode-bit permissions.
4. Why can a directory with `r--` but no `x` be difficult to use? **Answer:** You may see names but cannot traverse/search it normally.
5. Why should we avoid `chmod -R 777` on an NFS share? **Answer:** It grants excessively broad mode-bit access and can expose or modify shared data.

## Review checklist

- [ ] Recall `r=4`, `w=2`, `x=1` without looking.
- [ ] Translate `755`, `750`, `700`, `644`, and `600` in both directions.
- [ ] Explain file versus directory execute permissions.
- [ ] Inspect a disposable file with `ls -l` and `stat`.
- [ ] Explain why filesystem permissions are only one layer of NFS security.

## Reference

- `man chmod`, `man stat`, `man ls`, `man chown`, `man acl`
- GNU Coreutils: https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html

**Milestone record:** Sage Milestone 00, Linux system and storage inspection; permission review prompted by directory listings on network-mounted storage. No server permissions changed for this study guide.
