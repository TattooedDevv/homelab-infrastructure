# Rootless Podman: AppArmor executable-path mismatch

**Track:** Linux Foundations / container preparation (RHCSA EX200, LFCS, Red Hat EX188 supporting knowledge). This is a **sanitized, reproducible case study** using generic local file paths, not a copy of a live server configuration.

## Symptom

A manually installed Podman binary displayed its version but rootless initialization failed:

```text
$ podman info --format '{{.Host.Security.Rootless}}'
failed to reexec: Permission denied
```

## Diagnostic sequence and what each command tells us

| Inspection command | What it tests |
| --- | --- |
| `podman --version` | Binary launches and reports installed version |
| `command -v podman` | Which executable the shell selects from `PATH` |
| `ls -l "$(command -v podman)"` | File ownership and execute permission |
| `findmnt -T /usr/local/bin/podman -o TARGET,FSTYPE,OPTIONS` | Filesystem mount options, including possible `noexec` |
| `file /usr/local/bin/podman` | Binary type and architecture |
| `sysctl kernel.unprivileged_userns_clone` | Unprivileged user-namespace feature switch |
| `sysctl kernel.apparmor_restrict_unprivileged_userns` | Ubuntu AppArmor user-namespace restriction switch |
| `sudo journalctl -k --since "30 minutes ago" --no-pager \| grep -iE 'apparmor\|denied\|podman'` | Kernel AppArmor audit denials |
| `sudo aa-status` | Loaded profiles and their enforcement modes |
| `/usr/bin/podman info --format '{{.Host.Security.Rootless}}'` | Compare distro-supplied executable behavior |
| `aa-exec -p podman -- /usr/local/bin/podman info --format '{{.Host.Security.Rootless}}'` | Controlled test under existing AppArmor profile; note permissive profile trade-off |

**Relevant evidence:** audit logged an `exec` denial for `/proc/self/exe` with `profile="unprivileged_userns"`. The packaged binary under `/usr/bin` could initialize rootless, while a newer manually installed `/usr/local/bin` binary could not. Running the newer binary under the already available unconfined `podman` profile succeeded.

## Explanation

On certain Ubuntu AppArmor configurations, unprivileged user namespaces are restricted unless an application has an appropriate AppArmor rule. An existing profile may match only `/usr/bin/podman`, leaving a manually installed executable at `/usr/local/bin/podman` unmatched. The unmatched process can transition into the restricted `unprivileged_userns` profile and fail when Podman re-executes itself.

This pattern is a **path-to-profile mismatch**, not a general Unix `chmod` issue. Actual audit logs must be checked before attributing unrelated failures to AppArmor.

## Example remediation observed

A separate profile was created for the manually installed executable, using the same *permissive* policy design as the distro's provided Podman profile:

```apparmor
abi <abi/4.0>,
include <tunables/global>

profile podman-local /usr/local/bin/podman flags=(unconfined) {
  userns,
}
```

This is a historical example of the tested configuration, **not a general production hardening recommendation**. The `flags=(unconfined)` setting largely avoids application confinement while explicitly permitting `userns`. It does **not** mean containers themselves are unconfined, and is not a substitute for reviewing runtime/container security.

Validation and test sequence:

```bash
sudo apparmor_parser -Q /etc/apparmor.d/usr.local.bin.podman   # parse without loading
echo $?                                                         # 0 means success (immediately after previous command)
sudo apparmor_parser -a /etc/apparmor.d/usr.local.bin.podman   # add loaded profile
sudo aa-status | grep podman
podman info --format '{{.Host.Security.Rootless}}'             # expect true for rootless user
```

The resulting test produced `true`. This shows the runtime can initialize, **not yet** that image pulls, container starts, CDI GPU access, or boot-persistent operation have been verified.

## Operational cautions and rollback

- **Never globally disable AppArmor** or turn off namespace restrictions as a reflexive workaround.
- Preserve an existing working packaged runtime until the newer version has been tested.
- Back up local policy files before subsequent changes, and ensure a recovery shell remains available.
- If reverting this *example* profile, stop using the newer executable, back up its profile file, then remove the loaded profile with `sudo apparmor_parser -R /etc/apparmor.d/usr.local.bin.podman`; remove the local example file only if it is confirmed to be the one you created. Re-test packaged Podman afterward. Avoid making security-policy changes on systems you cannot recover.
- Exact hostnames, UUIDs, PCI addresses, IPs, runtime artifacts and private audit output belong in private operational records, not this public guide.

## Mini review questions

1. What does `command -v` reveal that `podman --version` does not?
2. Why can a file with mode `755` still fail during Podman startup?
3. What is the difference between the `-Q` and `-a` options of `apparmor_parser`?
4. Why is `aa-exec -p podman` a meaningful diagnostic but not evidence of restrictive confinement?
5. Which further tests are required before declaring a GPU container environment production-ready?

**Certification alignment:** RHCSA EX200/LFCS: permissions, paths, services, logs and security diagnosis. EX188: preparation for rootless Podman, containers and runtime operations. Ubuntu AppArmor syntax is not a replacement for RHEL SELinux exam preparation.
