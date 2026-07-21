# Linux Misconfiguration Analysis: Privilege Escalation via Weak Permissions

## 1. Executive Summary
This technical research scenario explores the risks associated with system misconfigurations in Linux environments. Specifically, it analyzes how insecure file permissions (e.g., `chmod 777`) on system automation scripts and exposed credentials can be exploited by an authenticated low-privileged insider to achieve unauthorized privilege escalation or lateral movement.

> ⚖️ **Legal & Ethical Disclaimer:** This simulation was performed entirely inside an isolated local laboratory environment for defensive research and educational purposes. No unauthorized infrastructure was targeted.

---

## 2. Laboratory Setup & Vulnerability Simulation
To simulate a real-world enterprise administrative oversight, two critical vulnerabilities were introduced into the isolated Linux environment:

1. **Exposed Credentials:** A sensitive SSH Private Key (`id_rsa`) was left in a hidden directory.
2. **Weak Script Permissions:** An automated script (`backup_script.sh`) was given full write permissions for all local users.

### Execution Commands (Simulation Phase):
```bash
# Simulating exposed configurations
mkdir -p /tmp/.hidden_backup/
echo "---BEGIN OPENSSH PRIVATE KEY---" > /tmp/.hidden_backup/id_rsa

# Simulating an automated system task with weak permissions
echo -e '#!/bin/bash\necho "System Backup Running..."' > /tmp/backup_script.sh
chmod 777 /tmp/backup_script.sh
```

---

## 3. Enumeration & Discovery Phase
Assuming the role of a low-privileged auditor or an insider threat, the system was enumerated using core Linux binaries to discover misconfigured assets without raising traditional security alerts.

### 3.1. Discovering Exposed SSH Keys
The file system was scanned for leaked or forgotten authentication keys while suppressing permission denial errors:
```bash
find / -type f -name "*id_rsa*" -not -path "/usr/*" 2>/dev/null
```
* **Finding:** The system exposed an isolated private key at `/tmp/.hidden_backup/id_rsa`.

### 3.2. Identifying Insecure Automated Scripts
The `/tmp` infrastructure was analyzed to isolate files that permit global write access (`-perm -o+w`), which could indicate potential cronjob manipulation vectors:
```bash
find /tmp -type f -perm -o+w 2>/dev/null
```
* **Finding:** The system revealed that `/tmp/backup_script.sh` could be modified by any local user account.

---

## 4. Privilege Escalation Implementation
Since the identified script (`backup_script.sh`) is designed to be executed automatically by high-privileged system services (such as root-owned Cron tasks), a payload injection was tested. 

By appending a harmless verification command to the executable, we simulate how a malicious actor could intercept execution flow to run arbitrary code under higher privileges:

```bash
echo "whoami >> /tmp/analiz_sonuc.txt" >> /tmp/backup_script.sh
```

### Verification (Code Inspection):
Executing `cat /tmp/backup_script.sh` confirms that the payload has been successfully integrated into the system process routine:
```bash
#!/bin/bash
echo "System Backup Running..."
whoami >> /tmp/analiz_sonuc.txt
```

---

## 5. Defensive Remediation & Security Hardening
To mitigate these critical infrastructure risks, enterprise system administrators must enforce strict Principle of Least Privilege (PoLP) models:

1. **Enforce Strict Permissions:** System scripts should never be world-writable. Automated tasks must use minimal necessary permissions (e.g., `chmod 700` or `750` owned strictly by the service account).
2. **Credential Auditing:** SSH keys and configuration strings must never reside in temporary directories (`/tmp`). Automated scanning tools should be deployed to detect exposed secrets.
3. **Integrity Monitoring:** Implement File Integrity Monitoring (FIM) systems (like AIDE or Tripwire) to alert administrators instantly if system automation utilities are modified.

