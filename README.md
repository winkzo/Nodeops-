<a href="https://ibb.co/vCGJRf9P"><img src="https://i.ibb.co/TDSHdJy8/New-Project-6.jpg" alt="New-Project-6" border="0"></a> 

# 🌌 **The Ultimate Guide to Setting Up and Troubleshooting NodeOps Network (Formerly Atlas Node)** 🌌

Welcome to the definitive guide for running a node on **NodeOps Network**, the rebranded evolution of Atlas Node! Whether you’re a beginner setting up your first node or an expert troubleshooting pesky issues like "Pending" status or leftover processes, this article is your one-stop resource. Packed with detailed steps, vibrant code blocks, and emoji-driven clarity, we’ll ensure your node runs smoothly on a VPS. Let’s get started! 🚀

**Current Date:** February 24, 2025  
**Focus:** NodeOps Network (formerly Atlas Node)  

---

## 📋 **Table of Contents**
1. [🔍 Step 1: Verify Your VPS Meets NodeOps Requirements](#step-1-verify-vps)  
2. [🔓 Step 2: Open Required Network Ports](#step-2-open-ports)  
3. [🛠️ Step 3: Update Your System](#step-3-update-system)  
4. [📴 Step 4: Disable Swap (The Critical Fix)](#step-4-disable-swap)  
5. [🗑️ Step 5: Remove an Old Node (If Needed)](#step-5-remove-old-node)  
6. [✅ Step 6: Install and Run Your NodeOps Node](#step-6-install-node)  
7. [🔧 Step 7: Troubleshoot Common Issues](#step-7-troubleshoot)  
8. [💡 Step 8: Tips for Long-Term Success](#step-8-tips)  

---

## 💰 **Step 0: Staking with  Arbitrum Sepolia ETH**  
To fully activate your NodeOps node, you’ll need to stake using **Arbitrum Sepolia ETH** and a you need small amount of real **0.001 ETH in Arbitrum one mainnet ** to get "*Arbitrum Sepolia ETH**"  . Here’s how to handle this step smoothly!  

### **Process**  
1. **Get Arbitrum Sepolia ETH:**  
   - Create a wallet (e.g., MetaMask 🦊).  
   - Visit the faucet: [Alchemy Arbitrum Sepolia Faucet](https://www.alchemy.com/faucets/arbitrum-sepolia) 💧.  
   - Claim some testnet ETH for staking you need to have in your wallet a real **0.001 ETH in Arbitrum one mainnet ** to get "*Arbitrum Sepolia ETH**""  .  

2. **Connect to NodeOps Testnet:**  
   - Head to `testnet-providers.nodeops.network` 🌐.  
   - Connect your wallet 🔗—it’s super easy!  

3. **Stake Your Tokens:**  
   - Initial staking uses **Arbitrum Sepolia ETH**.  
   - If you hit a snag (e.g., page freezes), hit `F5` to reload 🔄.  
   - After paying with Sepolia ETH, you’ll be prompted for **0.005 Arbitrum Sepolia ETH **.  
   - Reload again with `F5` 🔄, then enter your username and email ✍️.  

4. **Run Your Machine:**  
   - Once staked, your node will launch—boom, you’re live! 😎  

### **Troubleshooting**  
- **Staking Fails?** Reload the page (`F5`) and retry.  
- **Need More Sepolia ETH?** Revisit the faucet ⬆️.  
- **Check Logs:**  

--- 

## 🔍 **Step 1: Verify Your VPS Meets NodeOps Requirements**  
Before diving into setup, confirm your VPS can handle NodeOps Network. Below are the minimum specs and commands to check them!  

### **Minimum Requirements**  
- **CPUs or vCPUs:** ≥2  
- **RAM:** ≥4GB  
- **Storage:** ≥80GB NVMe SSD  
- **Bandwidth:** ≥1Gbps unlimited  
- **Uptime:** ≥99%  
- **OS:** Debian 12+ or Ubuntu 22.04+ with Linux kernel 6.1+  

### **Verification Commands**  
#### 1. **Check CPU Cores**  
```bash
nproc
```
- **Expected:** ≥`2` ✅  
- **Why:** Ensures enough processing power.

#### 2. **Check RAM**  
```bash
free -m | awk '/Mem:/ {print $2}'
```
- **Expected:** ≥`4096` MB (4GB) ✅  
- **Why:** Confirms sufficient memory.

#### 3. **Check Storage**  
```bash
df -h / | awk 'NR==2 {print $2}'
```
- **Expected:** ≥`80G` ✅  
- **Why:** Verifies enough disk space.

#### 4. **Check Network Speed**  
```bash
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python3 -
```
- **Expected:** ≥`100MB` or `1Gbps` ✅  
- **Why:** Ensures fast, unlimited bandwidth.

#### 5. **Check OS and Kernel**  
```bash
cat /etc/os-release; uname -r
```
- **Expected:** Debian 12+ or Ubuntu 22.04+, kernel ≥`6.1` ✅  
- **Why:** Ensures OS compatibility.

#### 6. **Check Security Updates**  
```bash
sudo apt update && apt list --upgradable
```
- **Expected:** No critical updates pending ✅  
- **Why:** Keeps your system secure.

💡 **Tip:** If your VPS falls short, upgrade with providers like Contabo, Vultr, Hetzner, or DigitalOcean!

---

## 🔓 **Step 2: Open Required Network Ports**  
NodeOps needs specific ports open to function. Let’s configure your firewall based on your setup.

### **Required Ports**  
- **UDP:** 8472, 51820, 51821  
- **TCP:** 10250  

### **Port Opening Commands**  
#### 1. **Check Active Services**  
```bash
sudo netstat -tuln | grep -E '8472|10250|51820|51821'
```
- **Expected:** Ports listed if services are running ✅  
- **Why:** Confirms Cilium, Kubelet, and WireGuard are active.

#### 2. **For UFW Users**  
```bash
sudo ufw allow 8472/udp
sudo ufw allow 10250/tcp
sudo ufw allow 51820/udp
sudo ufw allow 51821/udp
sudo ufw reload
```
- **Why:** Opens ports and reloads the firewall.

#### 3. **For iptables Users**  
```bash
sudo iptables -A INPUT -p udp --dport 8472 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 10250 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 51821 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
```
- **Why:** Adds rules and saves them persistently.

---

## 🛠️ **Step 3: Update Your System**  
A fresh system prevents compatibility issues. Here’s how to update and secure your VPS.

### **Update Commands**  
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
sudo apt autoremove --purge -y
```
- **Why:** Updates packages, installs the latest kernel, and removes leftovers.

---

## 📴 **Step 4: Disable Swap (The Critical Fix)**  
NodeOps requires swap to be disabled—here’s the most important step to fix "Pending" or "auto-restart" issues.

### **Commands to Disable Swap**  
#### 1. **Turn Off Swap Temporarily**  
```bash
sudo swapoff -a
```
- **Why:** Disables swap immediately.

#### 2. **Verify Swap is Off**  
```bash
swapon --summary
```
- **Expected:** No output ✅  
- **Why:** Confirms swap is disabled.

#### 3. **Disable Swap Permanently**  
- Edit `/etc/fstab`:  
```bash
sudo nano /etc/fstab
```
- Find the swap line (e.g., `/swapfile none swap sw 0 0`) and comment it out:  
```
# /swapfile none swap sw 0 0
```
- Save (`Ctrl+O`, Enter) and exit (`Ctrl+X`).

#### 4. **Reboot**  
```bash
sudo reboot
```
- **Why:** Applies changes permanently.

💡 **Tip:** After reboot, recheck with `free -h`—swap should show `0` everywhere!

---

## 🗑️ **Step 5: Remove an Old Node (If Needed)**  
Stuck with an old Atlas Node? Wipe it clean with this script.

### **Removal Script**  
```bash
#!/bin/bash
systemctl stop atlasnetwork-provider.service 2>/dev/null
systemctl disable atlasnetwork-provider.service 2>/dev/null
rm -f /etc/systemd/system/atlasnetwork-provider.service
systemctl daemon-reload
pkill -9 -f "atlas" 2>/dev/null
pkill -9 -f "containerd-shim" 2>/dev/null
rm -f /usr/local/bin/atlas*
rm -rf /.atlas-network
rm -rf /var/log/pods/atlasnetwork-system_net-check-*
rm -f /var/log/containers/net-check-nqm2d_atlasnetwork-system_net-check-*.log
rm -rf /sys/fs/cgroup/system.slice/atlasnetwork-provider.service
rm -rf /run/calico/cgroup/system.slice/atlasnetwork-provider.service
systemctl stop containerd 2>/dev/null
rm -rf /var/lib/rancher/k3s
rm -rf /run/k3s
rm -rf /etc/rancher
rm -f /usr/local/bin/k3s*
systemctl start containerd 2>/dev/null
sed -i '/vm.panic_on_oom/d' /etc/sysctl.d/90-kubelet.conf 2>/dev/null
sed -i '/vm.overcommit_memory/d' /etc/sysctl.d/90-kubelet.conf 2>/dev/null
sed -i '/kernel.panic/d' /etc/sysctl.d/90-kubelet.conf 2>/dev/null
sed -i '/kernel.panic_on_oops/d' /etc/sysctl.d/90-kubelet.conf 2>/dev/null
sysctl -p /etc/sysctl.d/90-kubelet.conf 2>/dev/null
swapon -a 2>/dev/null
reboot
```
- **Why:** Completely removes old node files and reboots for a fresh start.

---

## ✅ **Step 6: Install and Run Your NodeOps Node**  
Now, let’s get your new node up and running!

### **Installation Command**  
```bash
curl -L https://get.atlasnetwork.dev | sh -s - <YOUR_KEY>
```
- Replace `<YOUR_KEY>` with your unique key from `testnet-providers.nodeops.network`.

### **Check Status**  
```bash
sudo systemctl status atlasnetwork-provider.service
```
- **Expected:** `Active: active (running)` ✅

---

## 🔧 **Step 7: Troubleshoot Common Issues**  
Encountered a snag? Here’s how to fix common NodeOps problems.

### **Issue 1: "Pending" Status**  
- **Cause:** Swap enabled or insufficient resources.  
- **Fix:**  
  - Disable swap (Step 4).  
  - Verify VPS specs (Step 1).  
  - Restart service:  
```bash
sudo systemctl restart atlasnetwork-provider.service
```

### **Issue 2: "Left-over Processes" Error**  
- **Cause:** Unclean termination of previous runs.  
- **Fix:**  
  - Kill processes:  
```bash
sudo pkill -f atlasnetwork-provider
```
  - Restart service:  
```bash
sudo systemctl restart atlasnetwork-provider.service
```

### **Issue 3: Node Stops After SSH Disconnect**  
- **Fix:** Use `tmux` or `nohup`:  
  - **tmux:**  
```bash
tmux new -s atlas
<run your node command>
```
  - Detach: `Ctrl+B`, then `D`.  
  - Reattach: `tmux attach -t atlas`.  
  - **nohup:**  
```bash
nohup <your-node-command> &
```

### **Check Logs for Errors**  
```bash
journalctl -u atlasnetwork-provider.service -f
```

---

## 💡 **Step 8: Tips for Long-Term Success**  
Keep your node humming with these pro tips!

- **Monitor Status:**  
```bash
sudo systemctl status atlasnetwork-provider.service
```
- **Check Resources:**  
```bash
free -h
```
or  
```bash
top
```
- **Stay Updated:** Regularly run system updates (Step 3).  
- **One Node Per VPS:** Avoid conflicts by running only one node per server.

---

:fire::rocket: CRITICAL NODE RESET & STATUS CHECK GUIDE :rocket::fire:  (If Needed) 


---

**:arrows_counterclockwise: Restart Node Command:**
```bash
sudo systemctl restart atlasnetwork-provider.service
```
 **:mobile_phone_off: Disable SWAP Temporarily**  
   ```bash
   sudo swapoff -a
   ``` 
**:mag: Check Node Status:**
```bash
sudo systemctl status atlasnetwork-provider.service
```

**:scroll: (Optional) View Latest Logs:**
```bash
journalctl -u atlasnetwork-provider.service --no-pager --lines=50
```

---

 
## 🌟 **Final Words**  
Congratulations! You’ve now mastered setting up and troubleshooting a **NodeOps Network** node. From verifying your VPS to disabling swap and fixing issues, you’re ready to contribute to the network like a pro. If you hit any roadblocks, revisit the steps or check the logs—success is just a command away! 🚀  

LSG Nodeops team ! 😎
