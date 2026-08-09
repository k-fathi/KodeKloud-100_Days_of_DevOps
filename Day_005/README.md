<div align="left">
  <img src="../images/0.png" width="120" height="120" alt="KodeKloud icon" align="right" />
  <h1>Day_005 | SELinux Fundamentals: DAC vs MAC</h1>
</div>

## Outlines
- [Task Objective](#the-objective-of-this-lab-is-to)
- [Core Concepts (Theoretical Background)](#core-concepts-theoretical-background)
  - [DAC vs MAC](#dac-vs-mac)
  - [SELinux States vs Modes](#selinux-states-vs-modes)
- [Lab Solution & Commands](#lab-solution--commands)
- [Key Learnings & Notes](#key-learnings--notes)
- [Bonus: SELinux Context and File Operations (mv vs cp)](#bonus-selinux-context-and-file-operations-mv-vs-cp)

---

## **The objective of this lab is to:**

<div align="center">
    <h1>"Install the required SELinux packages and permanently disable SELinux"</h1> <h3>on App Server 1, with a maintenance reboot already scheduled for tonight to apply the change.</h3>
</div>

---

## Core Concepts (Theoretical Background)

### Before touching any config file, it's worth understanding what SELinux actually protects against that standard permissions don't.

### DAC vs MAC

Standard Linux runs on **DAC (Discretionary Access Control)** — access is decided purely by the classic `rwx` permission bits.

The problem: a process like `httpd` runs as a regular, unprivileged user. If it tries to reach a system file like `/etc/passwd` (permissions `644`), the kernel sees that this process is neither the file's owner nor in its group, so it's automatically treated as "Others" — and Others have read access. `httpd` can read `/etc/passwd` with zero resistance, even though it has no legitimate reason to touch anything outside the web root. If an attacker compromises `httpd`, DAC hands them a door straight into system files.

**SELinux** solves this with **MAC (Mandatory Access Control)**: every process and every file gets a **Label** (Security Context). `httpd` is labeled `httpd_t`, and a file like `/etc/passwd` is labeled `passwd_file_t`. Even if the file's permissions are `777`, SELinux ignores `rwx` entirely — it only asks: *does the Policy explicitly allow `httpd_t` to access `passwd_file_t`?* If not, it's an immediate **Access Denied**.

### SELinux States vs Modes

SELinux has two independent layers:

* **State** — `Enabled` or `Disabled`. Switching between them **requires a reboot**, because SELinux is embedded in the kernel itself; it has to start from scratch to apply or remove labels across the whole system.
* **Mode** (only relevant when `Enabled`):
  * **Enforcing** — any access attempt that violates the Policy is rejected immediately and logged to `/var/log/audit/audit.log`.
  * **Permissive** — the same violation is logged, but *not* blocked. The process runs as if SELinux weren't there, while every denial-worthy event is still recorded. This is invaluable for troubleshooting: you can tell whether SELinux is the culprit behind a failure without breaking anything live.

Switching between `Enforcing` and `Permissive` is instant, no reboot needed:
```bash
setenforce 0   # Permissive
setenforce 1   # Enforcing
```

---

## Lab Solution & Commands

The requirement was to disable SELinux **permanently**, with the state showing `Disabled` only after tonight's already-scheduled reboot — not immediately.

`setenforce` was not the right tool here, since it only toggles the **Mode** (Enforcing/Permissive), not the **State** (Enabled/Disabled).

Instead, the SELinux configuration file itself was edited:
```bash
sudo vi /etc/selinux/config
```
Changed the directive to:
```text
SELINUX=disabled
```
No manual reboot was triggered — the scheduled maintenance reboot tonight will pick up the change and bring the system up with SELinux fully disabled.

> ### Verify the Lab Solution: After the scheduled reboot, `sestatus` (or `getenforce`) should report `Disabled`.

---

## Key Learnings & Notes

* **State vs Mode are not the same knob.** `setenforce` only ever affects Enforcing/Permissive; it can never turn SELinux fully off. Permanently disabling it always goes through `/etc/selinux/config` and a reboot.
* **All denials, in either Enforcing or Permissive mode, land in the same place:** `/var/log/audit/audit.log`.

---

## Bonus: SELinux Context and File Operations (mv vs cp)

After the lab, I ran into a confusing case: a file with fully correct DAC permissions and the right owner, yet the web server still returned `403 Forbidden` when trying to serve it.

The cause wasn't permissions at all — it was **how the file got there**.

### What Happens with `mv`
When moving a file within the same partition, Linux doesn't recreate the data — it just repoints the file to the same **inode**. Since the inode itself never changes, the file keeps its old metadata, including its old **SELinux Context**.

A file originally in `/home/karim` carries the label `user_home_t`. Moving it into `/var/www/html/` with `mv` doesn't change that — it's still `user_home_t`. When `httpd_t` tries to read it, SELinux denies access because the labels don't match, regardless of how correct the `rwx` bits look.

### What Happens with `cp`
Copying creates a **brand-new file** with a new inode at the destination. Because it's a genuinely new file born inside `/var/www/html/`, it inherits that directory's **default SELinux context** — automatically becoming `httpd_sys_content_t`, which `httpd_t` is allowed to read.

### The Fix: `restorecon`
No need to delete and re-copy the file. `restorecon` compares a file against its current directory and relabels it to match what the Policy expects for that location:
```bash
sudo restorecon -v /var/www/html/index.html
```
The file's context is corrected in place, and the `403` disappears immediately.

> **Discussion:** For a deployment script that runs `mv` on many files daily, how do you guarantee `restorecon` always runs afterward automatically, instead of relying on remembering to do it manually every time?

---

## Deep Dive References

*   [**SELinux Security and Concepts**](https://github.com/k-fathi/learn-DevOps-tools/blob/main/01_Learn-Linux/Admin_2/SELinux.md): Explore comprehensive guides on configuring, troubleshooting, and understanding SELinux policies and contexts.

---

### **KodeKloud 100 Days of DevOps:** [**Here**](https://engineer.kodekloud.com/practice)

### **Main Learning Repository:** [**learn-DevOps-tools**](https://github.com/k-fathi/learn-DevOps-tools)