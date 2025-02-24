Below is a comprehensive, professional guide for managing your NodeOps Network node on a VPS. This article is organized from top to bottom with clear sections, headings, and code blocks—with plenty of emojis to help guide you!

---

# 🚀 The Ultimate Guide to Configuring & Managing Your NodeOps Network Node on a VPS

_NodeOps Network (formerly Atlas Network) offers decentralized node management solutions. This guide will walk you through ensuring your VPS meets requirements, verifying network settings, updating your system, handling critical swap issues, and removing an old node before deploying a new one._

---

## 🔍 **Section I: VPS Hardware Requirements & Checks**

Before running your NodeOps Network node, **make sure your VPS meets these minimum requirements**:

- **CPUs or vCPUs:** ≥2  
- **RAM:** ≥4GB  
- **Storage:** ≥80GB NVMe SSD  
- **Network Bandwidth:** ≥1Gbps (unlimited)  
- **Uptime:** ≥99%  
- **OS:** Debian (12+) or Ubuntu (22.04+) with Linux Kernel 6.1+, updated with the latest security patches

### **🖥️ Verify Your VPS Specs**

1. **Check CPU Cores:**
   ```bash
   nproc
   ```
   *Output should be 2 or higher.*

2. **Check Total RAM:**
   ```bash
   free -m | awk '/Mem:/ {print $2}'
   ```
   *Output should be ≥4096 MB (4GB).*

3. **Check Storage:**
   ```bash
   df -h / | awk 'NR==2 {print $2}'
   ```
   *Output should show ≥80G.*

4. **Check Network Speed:**
   ```bash
   curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python3 -
   ```
   *Ensure speeds are around 1Gbps or as specified by your plan.*

5. **Check OS & Kernel Version:**
   ```bash
   cat /etc/os-release; uname -r
   ```
   *Confirm you are running Debian 12+ or Ubuntu 22.04+ with Kernel ≥6.1.*

6. **Check for Latest Updates:**
   ```bash
   sudo apt update && apt list --upgradable
   ```
   *No critical updates should be pending.*

---

## 🔐 **Section II: Verifying & Opening Required Ports**

Your node relies on specific ports to communicate. Ensure the following ports are open:

- **UDP 8472** (used for overlay networks like VXLAN)
- **TCP 10250** (commonly for Kubelet API in Kubernetes)
- **UDP 51820 & UDP 51821** (typically used by WireGuard VPN)

### **🔗 Verify Open Ports**

```bash
sudo netstat -tuln | grep -E '8472|10250|51820|51821'
```
*If nothing appears, check that related services (e.g., Cilium, WireGuard) are active.*

### **For UFW Users:**

```bash
sudo ufw allow 8472/udp
sudo ufw allow 10250/tcp
sudo ufw allow 51820/udp
sudo ufw allow 51821/udp
sudo ufw reload
```

### **For iptables Users:**

```bash
sudo iptables -A INPUT -p udp --dport 8472 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 10250 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 51821 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
```

---

## 📅 **Section III: System Update & Security Preparation**

Keep your system secure and up to date before deploying your node.

```bash
sudo apt update -y && sudo apt upgrade -y && \
UBUNTU_VERSION=$(lsb_release -rs | cut -d '.' -f1) && \
if [ "$UBUNTU_VERSION" = "20" ]; then \
  KERNEL_PACKAGE="linux-generic-hwe-20.04"; \
elif [ "$UBUNTU_VERSION" = "22" ]; then \
  KERNEL_PACKAGE="linux-generic-hwe-22.04"; \
elif [ "$UBUNTU_VERSION" = "24" ]; then \
  KERNEL_PACKAGE="linux-generic-hwe-24.04"; \
else \
  echo "Ubuntu Version Not Supported: $UBUNTU_VERSION"; \
  exit 1; \
fi && \
sudo apt-get install --install-recommends $KERNEL_PACKAGE -y && \
sudo apt autoremove --purge -y && \
sudo apt install ufw -y && \
sudo ufw default deny incoming && \
sudo ufw default allow outgoing && \
sudo ufw allow ssh && \
sudo ufw allow 8472/udp && \
sudo ufw allow 10250/tcp && \
sudo ufw allow 51820/udp && \
sudo ufw allow 51821/udp && \
sudo ufw enable && \
sudo ufw reload && \
sudo ufw status verbose
```

*This command updates your system, installs the correct kernel version, and configures your firewall with the necessary ports.*

---

## ⚠️ **Section IV: Critical Swap Issue – Disable Swap**

For proper node operation, swap must be disabled.

### **Temporarily Disable Swap**

```bash
sudo swapoff -a
```
*This command immediately turns off swap.*

### **Verify Swap is Disabled**

```bash
swapon --summary
```
*No output indicates swap is off.*

### **Permanently Disable Swap**

1. Open `/etc/fstab` for editing:
   ```bash
   sudo nano /etc/fstab
   ```
2. Locate the swap entry (e.g., `/swapfile none swap sw 0 0`) and comment it out by adding a `#`:
   ```
   # /swapfile none swap sw 0 0
   ```
3. Save and exit (press **Ctrl+O**, then **Enter**, then **Ctrl+X**).

4. Reboot your system to apply changes:
   ```bash
   sudo reboot
   ```

---

## 🗑️ **Section V: Removing an Existing Node (NodeOps Network Provider Removal Script)**

If your node is stuck (e.g., in an "awaiting-stake" state), you may want to remove it completely before deploying a new one.

### **Node Removal Script**

```bash
#!/bin/bash
# Stop and disable the NodeOps Network provider service 🔴
sudo systemctl stop atlasnetwork-provider.service 2>/dev/null
sudo systemctl disable atlasnetwork-provider.service 2>/dev/null
sudo rm -f /etc/systemd/system/atlasnetwork-provider.service
sudo systemctl daemon-reload

# Kill any related processes 💀
sudo pkill -9 -f "atlasnetwork-provider" 2>/dev/null
sudo pkill -9 -f "containerd-shim" 2>/dev/null

# Remove all NodeOps Network related files 🗑️
sudo rm -f /usr/local/bin/atlas*
sudo rm -rf /var/lib/atlasnetwork
sudo rm -rf ~/.atlas-network
sudo rm -rf /var/log/pods/atlasnetwork-system_net-check-*
sudo rm -f /var/log/containers/net-check-*.log
sudo rm -rf /sys/fs/cgroup/system.slice/atlasnetwork-provider.service
sudo rm -rf /run/calico/cgroup/system.slice/atlasnetwork-provider.service

# Clean up any Kubernetes or containerd leftovers 🧹
sudo systemctl stop containerd 2>/dev/null
sudo rm -rf /var/lib/rancher/k3s
sudo rm -rf /run/k3s
sudo rm -rf /etc/rancher
sudo rm -f /usr/local/bin/k3s*
sudo systemctl start containerd 2>/dev/null

# (Optional) Reboot for a clean slate 🚀
sudo reboot
```

*Run this script to completely remove the old node and its associated files from your VPS.*

---

## 🌟 **Section VI: Deploying a New Node**

After ensuring your VPS is ready (hardware, ports, updated system, and swap disabled), deploy a new node with the proper script:

```bash
curl -L https://get.atlasnetwork.dev | sh -s - YOUR_NEW_NODE_CODE
```

*Replace `YOUR_NEW_NODE_CODE` with the actual code provided for the new node.*

---

## 📈 **Section VII: Monitoring & Maintenance**

After deployment, keep an eye on your node’s performance:

- **Check Service Status:**
  ```bash
  sudo systemctl status atlasnetwork-provider.service
  ```
  *Look for “Active: active (running)” to confirm proper operation.*

- **Monitor Logs:**
  ```bash
  journalctl -u atlasnetwork-provider.service -f
  ```
  *This helps you see real-time logs for any issues.*

- **Resource Monitoring:**
  ```bash
  htop
  ```
  *Ensure your node has sufficient CPU and memory.*

---

# 🚀 **Final Thoughts**

By following these organized steps—from verifying your VPS specs to opening the necessary ports, updating your system, disabling swap, and finally removing and redeploying your node—you can ensure a smooth and efficient operation of your NodeOps Network node.

This guide should serve as a complete resource for troubleshooting and maintaining your node environment. If you have any further questions or need assistance, feel free to reach out to the community or support team.

Happy node deploying! 🚀🔧

---
