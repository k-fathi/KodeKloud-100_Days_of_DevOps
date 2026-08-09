<div align="left">
  <img src="../images/0.png" width="120" height="120" alt="KodeKloud icon" align="right" />
  <h1>Day_006 | Cron Jobs</h1>
</div>

## Outlines
- [Task Objective](#the-objective-of-this-lab-is-to)
- [Lab Solution & Commands](#lab-solution--commands)
- [Key Learnings & Notes](#key-learnings--notes)
  - [Cron Environment & PATH Variables](#cron-environment--path-variables)
  - [The Shell Built-in Exception](#the-shell-built-in-exception)
  - [The Sudo Trap in Cron](#the-sudo-trap-in-cron)

---

## **The objective of this lab is to:**

<div align="center">
    <h1>"Install the cronie package and configure a Cron Job"</h1> <h3>for the root user to print 'hello' into a file every 5 minutes.</h3>
</div>

---

## Lab Solution & Commands

To accomplish the lab, we first ensure the `cronie` package is installed on the servers, and then we add the requested Cron job to the root user's crontab.

```bash
*/5 * * * * echo hello > /tmp/cron_text
```
*(Tip: If you ever get confused by Cron syntax like `*/5 * * * *`, [crontab.guru](https://crontab.guru) is a great tool to translate it into human-readable schedules).*

---

## Key Learnings & Notes

### Cron Environment & PATH Variables
If you write a Cron job using a standard command like `docker ps` or `systemctl`, it will likely fail with a `command not found` error. 
This happens because Cron jobs execute inside a **Non-interactive Shell**. The `$PATH` environment variable in this shell is extremely minimal and does not load your user's full profile (like `~/.bashrc` or `~/.profile`). If you don't provide the absolute path (e.g., `/usr/bin/docker`), Cron simply won't find the command.

### The Shell Built-in Exception
So, if Cron needs absolute paths, why did the command `echo hello` work perfectly without specifying its path?
The `echo` command is an exception because it is a **Shell Built-in**. It doesn't exist as a separate executable binary on the disk that the shell needs to look up in the `$PATH` — it is a part of the shell itself. Therefore, it requires no path resolution to run.

### The Sudo Trap in Cron
A common pitfall when writing automated scripts that require root permissions is placing the `sudo` command directly inside the script, and then scheduling it via a standard user's crontab.

* **The Problem:** Because Cron runs in a **Non-interactive Shell**, there is no terminal attached and no user present to type the password when `sudo` prompts for it. 
* **The Result:** The script gets stuck waiting for an input that will never come, and the command silently fails without throwing obvious errors. This could lead to critical tasks (like automated backups) failing for weeks without you noticing.
* **The Solution:** Remove `sudo` from the script entirely. Instead, add the job directly to the root user's crontab:
  ```bash
  sudo crontab -e
  ```
  Any script scheduled here automatically executes with `root` privileges directly, bypassing the need for a password prompt entirely.

---

### **KodeKloud 100 Days of DevOps:** [**Here**](https://engineer.kodekloud.com/practice)

### **Main Learning Repository:** [**learn-DevOps-tools**](https://github.com/k-fathi/learn-DevOps-tools)