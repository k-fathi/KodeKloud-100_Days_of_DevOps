<div align="left">
  <img src="../images/0.png" width="120" height="120" alt="KodeKloud icon" align="right" style="border-radius: 50%;" />
  <h1>Day_003 | Disable Direct SSH Root Login</h1>
</div>

## Outlines
- [Task Objective](#the-objective-of-this-lab-is-to)
- [Core Concepts (Theoretical Background)](#core-concepts-theoretical-background)
- [Execution Steps](#execution-steps)
- [Bonus: Simplify Connections with SSH Config](#bonus-simplify-connections-with-ssh-config)

---

## **The objective of this lab is to:**

<div align="center">
    <h1>"Disable direct SSH root login on all app servers"</h1> <h3>within the Stratos Datacenter following new security audit protocols.<h3>
</div>

---

## Core Concepts (Theoretical Background)

### Before editing configurations, we must understand why disabling direct root access is a critical security standard.

If a regular user has `sudo` privileges, they technically have the same power as `root`. So why block direct `root` access?

1. **Brute-Force Predictability:** The username `root` exists on almost every Linux server in the world. It is the very first target for automated bots. An attacker already knows half of the credential pair (the username). Using a custom username with `sudo` adds an extra layer of obscurity.
2. **Accountability & Audit Logs:** If all administrators log in directly as `root`, the system logs (`/var/log/secure` or `/var/log/auth.log`) will only show that "root logged in." You won't know *which* human actually performed the action. Forcing users to log in with their personal accounts first, then escalating privileges via `sudo`, ensures clear accountability in the audit logs.

### PermitRootLogin Options Explained
The `PermitRootLogin` directive in the SSH daemon configuration (`sshd_config`) accepts several values:
* **`yes`**: The root user can log in using any authentication method.
* **`no`**: Direct root login is completely disabled.
* **`prohibit-password`**: The root user cannot log in using a password, but SSH Key authentication is allowed.
* **`forced-commands-only`**: Root can only log in using an SSH Key to execute one specific, pre-configured command. No interactive shell is granted.

---

## Execution Steps

For this lab, the requirement was to apply this policy across **3 Application Servers** in the datacenter.

*(Note: Since they were only 3 servers, I SSH'ed into each one manually. If there were more, I would be "The Automation Guy" and use tools like Ansible).*

1. Log into the app server and open the SSH server configuration file:
    ```bash
    sudo vi /etc/ssh/sshd_config
    ```

2. Locate the `PermitRootLogin` directive and change its value to `no`:
    ```text
    PermitRootLogin no
    ```

3. Restart the SSH daemon to apply the changes:
    ```bash
    sudo systemctl restart sshd
    ```

*(Repeated the same process for the remaining 2 application servers).*


> ### Verify the Lab Solution: Try to SSH directly into any of the servers as `root` (e.g., `ssh root@app01`). It should immediately reject the connection.

---

## Bonus: Simplify Connections with SSH Config

Instead of typing long SSH commands with multiple flags for every server, you can automate the parameters client-side.

Normally, connecting to a remote server requires specifying the user, host IP, port, and identity file. The SSH client has a built-in feature that allows you to store these parameters in `~/.ssh/config`. You can also use Wildcards (`*`) to apply common configurations to multiple servers at once.

### Configuration Example:
```bash
vim ~/.ssh/config
```

**Inside the file:**
```ini
Host argo
    HostName 192.168.1.50
    User karim
    IdentityFile ~/.ssh/monitoring_key
    Port 2222

Host *.prod.com
    User admin
    Port 2244

Host *
    ServerAliveInterval 60
```

Now, to connect to the `argo` server, I simply type `ssh argo`.

**The "First Match Wins" Rule (CRITICAL):** 
The SSH client parses the `~/.ssh/config` file sequentially from **Top to Bottom**. Once it finds a parameter (like a specific Port), it locks it in and will **NEVER** overwrite it. Because of this architecture, global or wildcard blocks like `Host *` must **ALWAYS be placed at the very bottom** of the configuration file.

---

### **KodeKloud 100 Days of DevOps:** [**Here**](https://engineer.kodekloud.com/practice)
