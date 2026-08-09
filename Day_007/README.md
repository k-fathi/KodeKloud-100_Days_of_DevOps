<div align="left">
  <img src="../images/0.png" width="120" height="120" alt="KodeKloud icon" align="right" />
  <h1>Day_007 | Password-less SSH and Bastion Hosts</h1>
</div>

## Outlines
- [Task Objective](#the-objective-of-this-lab-is-to)
- [Lab Solution & Commands](#lab-solution--commands)
- [Key Learnings & Notes](#key-learnings--notes)
  - [The Jump Host (Bastion Host) Architecture](#the-jump-host-bastion-host-architecture)
  - [Restricting Compromised Keys (Forced Commands)](#restricting-compromised-keys-forced-commands)

---

## **The objective of this lab is to:**

<div align="center">
    <h1>"Set up Password-less SSH Authentication"</h1> <h3>from a user on the jump host to all app servers.</h3>
</div>

---

## Lab Solution & Commands

To establish password-less SSH access from the Jump Host to the App Servers, we use SSH public key authentication.

1. **Generate an SSH key pair** on the Jump Host:
```bash
ssh-keygen -t ed25519
```

2. **Copy the generated public key** to each of the target App Servers (you will be prompted for the password one last time):
```bash
ssh-copy-id user@app-server-ip
```

*(After this step, SSH access to the App Servers will no longer require a password).*

---

## Key Learnings & Notes

### The Jump Host (Bastion Host) Architecture
A Jump Host (or Bastion Host in AWS) is a hardened server acting as the single point of entry into a private network. 
Why use it if it can still be hacked?

1. **Reducing the Attack Surface:** Instead of exposing all 100 App Servers to the public internet, you place them in a private network and expose only one door (the Jump Host).
2. **Granular Security:** App Servers need various open ports to function. The Jump Host only needs SSH (Port 22) open. You can lock it down with strict Firewall rules (allowing only corporate IPs) and enforce MFA without disrupting application traffic.
3. **Centralized Monitoring:** All traffic must pass through this single bottleneck, making it much easier to monitor, log, and detect suspicious login attempts.

*The Jump Host doesn’t make hacking impossible; it makes it significantly harder, slower, and easier to detect early.*

### Restricting Compromised Keys (Forced Commands)
If the automated scripts run from within the Jump Host, the Private Key *must* reside there. But what happens if an attacker compromises the Jump Host and steals the Private Key? Can they open a full shell on any App Server?

Not if you restrict the key's capabilities. Using the `authorized_keys` file on the App Servers, you can prepend specific options to the Public Key to severely limit what it can do:

```text
command="/opt/scripts/backup.sh",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-ed25519 AAAA...
```

* `command="..."`: Forces the server to *only* execute this specific script, regardless of what the user requests.
* `no-pty`: Prevents the allocation of an interactive shell (TTY).
* `no-port-forwarding` & `no-agent-forwarding`: Blocks the attacker from using the SSH connection as a tunnel to reach other internal resources.

**Conclusion:** If the key is stolen, it is virtually useless for anything other than running the intended `backup.sh` script. Always use separate, restricted keys for separate tasks to contain the blast radius if a compromise occurs.

---

## Deep Dive References

*   [**Configuring and Securing SSH**](https://github.com/k-fathi/learn-DevOps-tools/blob/main/01_Learn-Linux/Admin_1/24_Configuring_and_Securing_SSH.md): Explore advanced SSH configuration techniques, hardening strategies, and managing secure remote connections effectively.

---

### **KodeKloud 100 Days of DevOps:** [**Here**](https://engineer.kodekloud.com/practice)

### **Main Learning Repository:** [**learn-DevOps-tools**](https://github.com/k-fathi/learn-DevOps-tools)