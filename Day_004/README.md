<div align="left">
  <img src="../images/0.png" width="120" height="120" alt="KodeKloud icon" align="right"/>
  <h1>Day_004 | Permissions and Special Permissions</h1>
</div>

## Outlines
- [Task Objective](#the-objective-of-this-lab-is-to)
- [Core Concepts (Theoretical Background)](#core-concepts-theoretical-background)
  - [Binary Executables vs. Interpreted Scripts](#binary-executables-vs-interpreted-scripts)
- [Lab Solution & Commands](#lab-solution--commands)
- [Key Learnings & Notes](#key-learnings--notes)
- [Bonus: Special Permissions (Sticky Bit & SUID)](#bonus-special-permissions-sticky-bit--suid)

---

## **The objective of this lab is to:**

<div align="center">
    <h1>"Grant executable permissions to the /tmp/xfusioncorp.sh script"</h1> <h3>on App Server 2, ensuring that all users have the capability to execute it.</h3>
</div>

---

## Core Concepts (Theoretical Background)

### Before applying `chmod`, it's important to understand that the `x` bit does not mean the same thing for every type of file.

### Binary Executables vs. Interpreted Scripts

* **Binary Executable** (e.g., a compiled C program): This is raw machine code. The OS and CPU read and execute it directly, so it only requires the `x` (execute) permission to run.
* **Interpreted Script** (e.g., Bash or Python): This is just a plain text file. When you run `./script.sh`, the OS doesn't execute the file itself — it hands it off to an **interpreter** (`/bin/bash`). That interpreter has to **open the file and read its instructions line by line** before it can run anything.

> **Note:** Because an interpreter must *read* the script's contents first, any user running it needs both `r` (read) and `x` (execute) permissions. Having `x` alone is not enough for scripts — it only works for binaries.

---

## Lab Solution & Commands

### Checking Initial Permissions
Logged into App Server 2 and checked the script's current permissions:
```bash
ls -l /tmp/xfusioncorp.sh
```
**Output:**
```text
---------- 1 root root 40 Aug 5 03:13 /tmp/xfusioncorp.sh
```
No permissions were set at all.

### The Intuitive (But Incomplete) Approach
The obvious first move was to just add execute permission for everyone:
```bash
sudo chmod +x /tmp/xfusioncorp.sh
```
Submitting the lab at this point returned **Failed**.

### Investigating the Failure
Checked the permissions again:
```bash
ls -l /tmp/xfusioncorp.sh
```
**Output:**
```text
--x--x--x 1 root root 40 Aug 5 03:13 /tmp/xfusioncorp.sh
```
Every user class had `x`, so the script *looked* runnable — but it still wasn't accepted as correct.

### Why It Failed
`xfusioncorp.sh` is a Bash script, not a binary. When a user (or the grading check) tries to run it, `/bin/bash` needs to open the file and read its contents first. With `x` but no `r`, the OS allows the file to be *executed* as a process, but the interpreter itself is denied when it tries to *read* the instructions inside — so the script fails silently despite having "execute" permission.

### The Correct Approach
Both `read` and `execute` needed to be granted together:
```bash
sudo chmod +rx /tmp/xfusioncorp.sh
```
**Resulting permissions:**
```text
-r-xr-xr-x 1 root root 40 Aug 5 03:13 /tmp/xfusioncorp.sh
```

> ### Verify the Lab Solution: `ls -l /tmp/xfusioncorp.sh` should show `-r-xr-xr-x`, and any user should be able to execute the script successfully.

---

## Key Learnings & Notes

* **`x` without `r` is meaningless for scripts.** The permission bit only controls whether the OS will *launch* a process from that file — it says nothing about whether the interpreter reading the file's contents is allowed to do so. For interpreted scripts, `r` is a hard requirement.
* **Binaries are the exception, not the rule.** Compiled machine code is mapped and executed directly by the kernel without a separate "read the instructions" step from an interpreter, which is why `x`-only binaries can work while `x`-only scripts cannot.

---

## Bonus: Special Permissions (Sticky Bit & SUID)

While working with `/tmp` and `chmod` in this lab, it's worth understanding two related "special" permission bits that go beyond the standard `rwx` model.

### The Sticky Bit — `/tmp`'s Secret
Running `ls -ld /tmp` shows:
```text
drwxrwxrwt 10 root root 4096 Aug 5 03:13 /tmp
```
Notice the `t` at the end instead of the usual `x`. This is the **Sticky Bit**.

Normally, if a directory has write permission, any user can delete or rename *any* file inside it — including files they don't own. The Sticky Bit changes that rule: it tells the system "anyone can write new files here, but only the file's owner (or root) can delete or rename it." This is exactly why `/tmp` can be world-writable without users being able to wipe out each other's files.

### SUID — How `passwd` Works
Running `ls -l /usr/bin/passwd` shows:
```text
-rwsr-xr-x 1 root root 54256 Aug 5 03:13 /usr/bin/passwd
```
The `s` in place of the owner's `x` is the **SUID (Set User ID)** bit. It tells the kernel: "run this program with the permissions of its **owner** (root), not the permissions of the user who launched it."

This is how a regular, non-root user can run `passwd` and successfully modify `/etc/shadow` — a file only root can normally write to. While `passwd` is running, it temporarily executes with root's privileges, updates the hash, and exits cleanly.

> **Security Warning:** SUID is powerful and dangerous if misapplied. If the bit is mistakenly set on a general-purpose binary (`vim`, `find`, `bash`, etc.), an attacker with access to a low-privilege account can search for it with:
> ```bash
> find / -perm -4000
> ```
> and use it to spawn a root shell, fully compromising the system. Look up **GTFOBins** to see how many common binaries can be abused this way when SUID is set incorrectly.

Together with **SGID** (the group-level equivalent of SUID), these special permissions are essential to understanding Linux security beyond basic `rwx`.

---

## Deep Dive References

*   [**Managing Ownership**](https://github.com/k-fathi/learn-DevOps-tools/blob/main/01_Learn-Linux/Admin_1/16_Managing_Ownership.md): Deepen your understanding of standard Linux file ownership, permissions, and fundamental access controls.
*   [**Special Permissions**](https://github.com/k-fathi/learn-DevOps-tools/blob/main/01_Learn-Linux/Admin_1/17_Special_Permissions.md): Master the concepts of SUID, SGID, and the Sticky Bit to handle complex security requirements beyond the standard read, write, and execute bits.

---

### **KodeKloud 100 Days of DevOps:** [**Here**](https://engineer.kodekloud.com/practice)

### **Main Learning Repository:** [**learn-DevOps-tools**](https://github.com/k-fathi/learn-DevOps-tools)