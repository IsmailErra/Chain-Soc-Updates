# Chain-Soc-Updates
# ChainSOC -- Decentralized SIEM Prototype

<br>

## Project Overview

ChainSOC is a security monitoring prototype that uses three virtual machines to collect, analyze, and store system logs in a decentralized way.

The Target machine generates logs. The Watcher machine retrieves them via SSH, checks for suspicious entries, and sends everything to the Vault machine for secure storage. The whole pipeline runs automatically every minute using a cron job.

<br>

---

<br>

## Architecture

```
+-------------------+         +-------------------+         +-------------------+
|   Target Machine  |  SSH    |  Watcher Machine  |  SSH    |   Vault Machine   |
|                   | ------> |                   | ------> |                   |
|  - Generates logs |         |  - Retrieves logs |         |  - Stores logs    |
|  - Simulates      |         |  - Detects threats|         |  - vault_logs.txt |
|    suspicious     |         |  - Sends alerts   |         |                   |
|    activity       |         |  - Forwards logs  |         |                   |
+-------------------+         +-------------------+         +-------------------+
```

**Flow:** Target generates logs --> Watcher retrieves and analyzes them --> Watcher detects threats and creates alerts --> Watcher forwards logs to Vault --> Vault stores everything securely.

<br>

---

<br>

## Technologies Used

| Technology         | Purpose                                    |
|--------------------|--------------------------------------------|
| Ubuntu 24.04 LTS   | Target machine OS                          |
| Ubuntu 22.04 LTS   | Watcher and Vault machines OS              |
| VMware Workstation  | Virtualization platform                    |
| SSH (OpenSSH)      | Secure communication between all machines  |
| Bash Scripting     | Log retrieval, detection, and forwarding   |
| Cron Jobs          | Automated scheduling (every minute)        |
| journalctl / syslog| Log generation and querying                |
| React              | Dashboard prototype                        |

<br>

---

<br>

## Implementation Steps

<br>

### Step 1: Target Network Configuration

The Target machine was set up with two network interfaces: `ens33` (DHCP, for internet) and `ens37` (static IP `192.168.233.103/24`, for the internal ChainSOC network). Configuration was done via Netplan.

![Target network configuration with ip a showing both interfaces](screenshots/target_network_config.png)

<br>

### Step 2: SSH Setup and User Creation

SSH was enabled on the Target machine. A dedicated `watcher` user was created so the Watcher machine could connect remotely. The SSH service was verified as active on port 22.

![SSH service running on Target with watcher user created](screenshots/target_ssh_service_setup.png)

<br>

### Step 3: SSH Communication Between Machines

SSH connections were established between all three machines:

- Watcher connects to Target (`192.168.233.103`) to retrieve logs.
- Watcher connects to Vault (`192.168.233.102`) to forward logs.

![Watcher SSH connection to the Vault machine](screenshots/watcher_ssh_to_vault.png)

<br>

### Step 4: Log Generation on Target

Test logs were created on the Target using the `logger` command and verified with `journalctl`.

![Log generated and verified on Target machine](screenshots/target_log_verification.png)

<br>

### Step 5: Watcher Log Retrieval

A Bash script (`check_logs.sh`) was written on the Watcher to automatically SSH into the Target and retrieve recent logs matching `chainsoc_test`.

![check_logs.sh script in nano editor](screenshots/watcher_check_logs_script.png)

![Script executed successfully, logs retrieved from Target](screenshots/watcher_log_retrieval_result.png)

<br>

### Step 6: SSH Key-Based Authentication

SSH keys were generated on the Watcher (`ssh-keygen`) and copied to the Target (`ssh-copy-id`) so that all connections run without requiring a password.

![SSH key generation and passwordless login working](screenshots/watcher_ssh_key_auth_success.png)

<br>

### Step 7: Log Forwarding to Vault

A second script (`send_to_vault.sh`) was created on the Watcher. It retrieves logs from the Target, checks for suspicious entries, and forwards them to `vault_logs.txt` on the Vault machine via SSH.

![send_to_vault.sh script with alert detection logic](screenshots/watcher_alert_detection_script.png)

![Logs successfully received and stored on Vault](screenshots/vault_log_storage_verification.png)

<br>

### Step 8: Alert Detection

When the script finds a suspicious log entry, it prints `"ALERT: Suspicious log detected!"` and saves the entry to `alerts.txt` on the Watcher.

![Alert triggered and recorded in alerts.txt](screenshots/watcher_alert_detection_result.png)

<br>

### Step 9: Cron Automation

The `send_to_vault.sh` script was added to crontab to run every minute (`* * * * *`). This automates the entire pipeline: retrieval, detection, alerting, and forwarding.

![Crontab configuration with the script scheduled every minute](screenshots/cron_automation_setup.png)

![Vault showing accumulated logs from multiple cron cycles](screenshots/vault_cron_logs_accumulated.png)

<br>

---

<br>

## Current Results

Everything listed above is fully working:

- Three VMs networked on `192.168.233.0/24`
- SSH with key-based authentication between all machines
- Automated log retrieval from Target via `check_logs.sh`
- Suspicious log detection with alerts saved to `alerts.txt`
- Log forwarding to Vault via `send_to_vault.sh`
- Cron job running the full pipeline every minute
- Logs accumulating in `vault_logs.txt` on the Vault

<br>

---

<br>

## Future Improvements

- Full blockchain integration for tamper-proof log storage
- IPFS-based decentralized log archiving
- Smart contract logging for on-chain verification
- Complete the React dashboard with live log visualization

<br>

---

<br>

## Conclusion

ChainSOC demonstrates a working decentralized SIEM prototype using three Ubuntu VMs, SSH, Bash scripts, and cron automation. The system successfully collects logs from a target machine, detects suspicious activity, generates alerts, and stores everything securely on a vault machine -- all fully automated.

