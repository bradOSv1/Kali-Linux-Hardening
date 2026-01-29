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

   **If you're accessing your system remotely via SSH, allow SSH  (22/tcp) BEFORE enabling UFW or you can lock yourself out.**

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

   **Ports and Protocols**

   A list of commonly used ports and protocols.
   
   - AFP - 548/tcp
   - DHCP - 67/upd (server) 68/upd (client)
   - DNS - 53/upd (for most queries) tcp (for zone transfers/large responses)
   - FTP - 21/tcp
   - HTTP - 80/tcp
   - HTTPs - 443/tcp
   - IMAP - 143/tcp
   - LDAP - 389/tcp
   - LDAPS - 636/tcp
   - NetBIOS/NetBT - 137/upd (Name Service) 138/upd (Datagram Service) 139/tcp (Session Service)
   - POP3 - 110/tcp
   - RDP - 3389/tcp & upd
   - SLP - 427/tcp & upd
   - SMB/CIFS - 445/tcp
   - SMTP - 25/tcp
   - SNMP - 161/upd (queries) 162/upd (traps)
   - SSH - 22/tcp
   - Telnet - 23/tcp
  
6. Enable UFW

   ```bash
   sudo ufw enable
   ```
7. Enable logging

   ```bash
   sudo ufw logging medium
   ```
8. Verify the firewall configuration

   ```bash
   sudo ufw status verbose
   sudo ufw status numbered
   sudo ufw show added
   ```

   

