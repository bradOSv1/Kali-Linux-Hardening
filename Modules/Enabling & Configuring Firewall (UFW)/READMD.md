# Enabling & Configuring Firewall (UFW)

## Objective 

The purpose for implementing an Uncomplicated Firewall (UFW) is to enforce a host-based network policy that filters and limits traffic entering and leaving the system. This helps to control service exposure on public and private networks. Based on your preferred firewall rule set, UFW reduces the attack surface, logs blocked (and allowed) traffic for review, and may enforce controls for outbound traffic.

## Walk-through

** **It's important to note these steps assume you have local access. Please have at least one fallback admin user. The stricter a firewall's rule set is, the less functionality a system will have with connected networks. This is simply the trade-off between security and convenience.** **

1. Install UFW.

   ```bash
   sudo apt update
   sudo apt install ufw -y
   ```

2. Review ports that are currently open before applying firewall rules.

   ```bash
   sudo ss -tulpn
   ```

3. Set UFW to it's baseline rule set.

   ```bash
   sudo ufw disable
   sudo ufw reset
   ```

4. Choose your default security posture.

   **If you're accessing your system remotely via SSH, allow SSH  (22/tcp) BEFORE enabling UFW or you can be yourself out.**

   You may start by setting a strict security posture (recommended for advanced users), then opening ports as needed. This may reduce network functionality until rules are added.
   ```bash
   sudo ufw default deny incoming
   sudo ufw default deny outgoing
   ```
   Conversely, you may start with a lenient posture and close ports as needed. System will work as normal with preferred restrictions.
   ```bash
   sudo ufw default allow incoming
   sudo ufw default allow outgoing
   ```

   A clean security posture preference.
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   ```

5. Apply your preferred/necessary rule set.

   **Allowing a port (general syntax)**

   Allow a port from anywhere
   ```bash
   sudo ufw allow <port>/<proto>
   ```
   >Example: sudo ufw allow 22/tcp
   
   Allow a port only from a trusted IP
   ```bash
   sudo ufw allow from <trusted-ip> to any port <port> proto <proto>
   ```
   >Example: sudo ufw allow from <trusted-ip> to any port 22 proto tcp
   
   Allow a port only from a trusted subnet
   ```bash
   sudo ufw allow from <trusted-subnet>/<cidr> to any port <port> proto <proto>
   ```
   >Example: sudo ufw allow from <trusted-subnet>/<cidr> to any port 22 proto tcp

   **Denying a port (general syntax)**

   Deny a port from anywhere
   ```bash
   sudo ufw deny <port>/<proto>
   ```
   >Example: sudo ufw deny 445/tcp

   Deny a port only from a specific IP
   ```bash
   sudo ufw deny from <source-ip> to any port <port> proto <proto>
   ```
   >Example: sudo ufw deny from <source-ip> to any port 445 proto tcp

   Deny a port only from a specific subnet
   ```bash
   sudo ufw deny from <source-subnet>/<cidr> to any port <port> proto <proto>
   ```
   >Example: sudo ufw deny from <source-subnet>/<cidr> to any port 445 proto tcp

   
