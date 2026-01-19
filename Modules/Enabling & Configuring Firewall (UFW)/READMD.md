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

2. (Recommended) Check what is currently listening before applying firewall rules.

   ```bash
   sudo ss -tulpn
   ```

3. Reset UFW to a clean baseline.

   ```bash
   sudo ufw disable
   sudo ufw reset
   ```

4. Apply Profile 3 defaults (deny inbound by default, allow outbound by default).

   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   ```

5. Allow only the inbound services you actually need.

   **SSH (recommended: allow only from a trusted subnet):**
   Replace `192.168.1.0/24` with your LAN subnet or trusted management subnet.

   ```bash
   sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
   ```

   **Optional: Rate-limit SSH (useful to slow brute force attempts):**
   ```bash
   sudo ufw limit 22/tcp
   ```

   **Example: Allow a web service only from LAN (if applicable):**
   ```bash
   sudo ufw allow from 192.168.1.0/24 to any port 80 proto tcp
   sudo ufw allow from 192.168.1.0/24 to any port 443 proto tcp
   ```

   **Example: Temporary lab port (tight scope to a trusted host):**
   ```bash
   sudo ufw allow from <trusted-ip> to any port 4444 proto tcp
   ```

6. Enable UFW.

   ```bash
   sudo ufw enable
   ```

7. Enable logging (good for validation and portfolio evidence).

   ```bash
   sudo ufw logging medium
   ```

8. Validate the firewall configuration.

   ```bash
   sudo ufw status verbose
   sudo ufw status numbered
   sudo ufw show added
   ```

   (Optional) Review recent events if your system writes to `/var/log/ufw.log`:

   ```bash
   sudo tail -n 50 /var/log/ufw.log
   ```

9. Remove temporary rules after use.

   List rules with numbers:
   ```bash
   sudo ufw status numbered
   ```

   Delete a rule by number:
   ```bash
   sudo ufw delete <rule-number>
   ```

10. (Optional) Drop ping replies (ICMP echo-request) for reduced discoverability.
    Be considerate—this can break troubleshooting and collaboration on managed networks.

    Edit UFW’s `before.rules`:
    ```bash
    sudo nano /etc/ufw/before.rules
    ```

    Find the line similar to:
    ```text
    -A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
    ```

    Comment it out by adding `#`:
    ```text
    # -A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
    ```

    Save and exit: [Ctrl+O] [Enter] [Ctrl+X]

    Reload UFW:
    ```bash
    sudo ufw reload
    ```

---

## Optional: Profile 4 (Strict Egress / Deny Outbound by Default)

**Warning:** This profile can break normal functionality until you explicitly allow required outbound traffic (updates, DNS, browsing, tooling). Only apply this if you’re comfortable maintaining an outbound allowlist.

1. Switch outbound defaults to deny.

   ```bash
   sudo ufw default deny outgoing
   ```

2. Allow essential outbound traffic (common baseline).

   **DNS (UDP/TCP 53):**
   ```bash
   sudo ufw allow out 53/udp
   sudo ufw allow out 53/tcp
   ```

   **NTP time sync (recommended):**
   ```bash
   sudo ufw allow out 123/udp
   ```

   **HTTP/HTTPS (APT updates, browsing, GitHub over HTTPS):**
   ```bash
   sudo ufw allow out 80/tcp
   sudo ufw allow out 443/tcp
   ```

   **SSH outbound (only if you use it):**
   ```bash
   sudo ufw allow out 22/tcp
   ```

3. Validate and troubleshoot.

   ```bash
   sudo ufw status verbose
   ```

   If something breaks, identify what that app needs and allow the minimum ports/protocols required.

---

## Rollback / Recovery

If you need to revert firewall changes:

```bash
sudo ufw disable
sudo ufw reset
```
