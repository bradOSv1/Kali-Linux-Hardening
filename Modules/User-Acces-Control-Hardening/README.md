# User & Access Control Hardening

## Objective

The following steps will demonstrate how to harden user access and use of administrative privileges on your Kali system. This is done to prevent threat actors from changing system configurations through privilege escalation. Altogether we’ll restrict elevation paths, implement safer sudo behavior, limit who can become root, and enable kernel-level auditing controls to track privileged actions.

## Walkthrough

** **It’s important to note, these actions are performed with the assumption that at least one fallback admin user already exists.** **

1. Before any changes are made to privilege controls, back up the current sudo configuration. Run:

   ```bash
   sudo cp -a /etc/sudoers /root/sudoers-backup-$(date +%F)
   sudo cp -a /etc/sudoers.d /root/sudoers.d-backup-$(date +%F)
   ```

   In the event an error occurs during configuration, recover the backup using:

   ```bash
   sudo visudo -f /etc/sudoers
   ```

2. Open the sudoers file using visudo. Run:

   ```bash
   sudo visudo
   ```

   Insert the following lines under the existing `Defaults` entries (keep them grouped with the other `Defaults` lines):

   ```text
   Defaults logfile="/var/log/sudo.log"
   Defaults log_input,log_output
   Defaults use_pty
   Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
   Defaults passwd_tries=3
   Defaults passwd_timeout=1
   Defaults timestamp_timeout=0
   ```

   Then save and exit. [Ctrl+O] [Enter] [Ctrl+X]

   Here is an explanation of each configuration:

   logfile — centralizes sudo logs for auditing  
   log_input/log_output — captures commands & I/O for forensic review  
   use_pty — prevents TTY hijacking vulnerabilities  
   secure_path — prevents PATH hijacking by enforcing safe binary paths  
   passwd_tries=3 — limits password attempts during sudo elevation  
   passwd_timeout=1 — limits how long sudo waits for a password  
   timestamp_timeout=0 — requires password every time sudo is used  

3. Install and enable auditd to provide session logging for privileged activity. Run the following:

   ```bash
   sudo apt update
   sudo apt install auditd audispd-plugins
   sudo systemctl enable auditd --now
   ```

4. Using execve create audit rules to log any command executed with effective UID 0 where the real UID is not root. Run:

   ```bash
   sudo nano /etc/audit/rules.d/69-privileged-session.rules
   ```

   Then insert the following:

   ```text
   # Log any executed command where effective UID is root (0) but real UID is not root.
   -a always,exit -F arch=b64 -S execve -C uid!=euid -F euid=0 -k privileged_exec
   -a always,exit -F arch=b32 -S execve -C uid!=euid -F euid=0 -k privileged_exec
   ```

   Then save and exit. [Ctrl+O] [Enter] [Ctrl+X]

   Load the new rules, run:

   ```bash
   sudo augenrules --load
   ```

5. Review the current sudo users, then determine which are necessary users and which can be removed. Run:

   ```bash
   getent group sudo
   ```

   To remove unnecessary users (with at least one backup admin) run:

   ```bash
   sudo gpasswd -d <username> sudo
   ```

6. To restrict root switching to the wheel group, start by creating the wheel group if it doesn’t already exist. Run:

   ```bash
   sudo groupadd wheel 2>/dev/null || true
   ```

   Then add your admin user. Run:

   ```bash
   sudo usermod -aG wheel <admin-username>
   ```

   **Note:** For the `wheel` group change to take effect, log out and back in. You can verify after re-login with:

   ```bash
   groups <admin-username>
   ```

   Next, open the PAM configuration for su and enable the restrictions. Run:

   ```bash
   sudo nano /etc/pam.d/su
   ```

   Insert the following:

   ```text
   auth required pam_wheel.so use_uid
   ```

   Then save and exit. [Ctrl+O] [Enter] [Ctrl+X]

7. Disabling root SSH logins and local TTY root access is an important security protocol that falls under this category. A separate README.md will be dedicated to it soon, and found within the `Modules` folder.

8. Next is filtering environment variables to limit hijack execution flow, manipulating library loading, or bypassing controls. Start by reopening visudo. Run:

   ```bash
   sudo visudo
   ```

   Insert the following:

   ```text
   Defaults env_reset
   Defaults env_keep += "LANG LANGUAGE LC_*"
   Defaults env_check += "LD_PRELOAD"
   Defaults env_check += "LD_LIBRARY_PATH"
   ```

   Then save and exit. [Ctrl+O] [Enter] [Ctrl+X]

   Here is an explanation for each configuration:

   env_reset — clears the user’s environment when running sudo  
   env_keep += "LANG LANGUAGE LC_*" — preserves safe localization variables  
   env_check += "LD_PRELOAD" — checks and blocks unsafe LD_PRELOAD values  
   env_check += "LD_LIBRARY_PATH" — checks and blocks unsafe library path overrides  

9. Create an audit rule that logs any changes to sudo configuration files. Run:

   ```bash
   sudo nano /etc/audit/rules.d/priv_actions.rules
   ```

   Insert the following:

   ```text
   -w /etc/sudoers -p wa -k sudoers_changes
   -w /etc/sudoers.d -p wa -k sudoers_d_changes
   ```

   Then save and exit. [Ctrl+O] [Enter] [Ctrl+X]

   Finally, reload the audit rules. Run:

   ```bash
   sudo augenrules --load
   ```

Thank you for reading. The intention for this walk-through is to showcase my learning and application of user and access control hardening on the Linux platform.
