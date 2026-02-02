# **Chameleon Cloud Setup Runbook**

## **Overview**
Chameleon Cloud is a research testbed for cloud computing experiments. This document summarizes how to:
1. Create a Chameleon account and join a project
2. Set up SSH keys
3. Launch and connect to VM instances
4. Set up Jupyter Notebook on remote VMs
5. Connect from local IDE (Cursor/VS Code)

---

## **1. ACCOUNT SETUP**

### **1.1 Create Account**
1. Go to [https://chameleoncloud.org/](https://chameleoncloud.org/)
2. Click "Log In" → "Sign up now" → "Sign in via federated login"
3. Select your university and login with university credentials
4. Accept terms and conditions

### **1.2 Join Project**
1. Get project join link from instructor OR
2. Ask instructor to add your email (`ng3483@nyu.edu`) to project `CHI-251409`

---

## **2. SSH KEY SETUP**

### **2.1 Generate SSH Key on Local Mac**
```bash
ssh-keygen -t rsa -f ~/.ssh/id_rsa_chameleon
```
- Press Enter for default location
- Optional: Add passphrase for extra security
- This creates two files:
  - `~/.ssh/id_rsa_chameleon` (PRIVATE KEY - keep secret!)
  - `~/.ssh/id_rsa_chameleon.pub` (PUBLIC KEY - upload to Chameleon)

### **2.2 Upload Public Key to Chameleon**
1. Log into Chameleon dashboard
2. Go to: Experiment → KVM@TACC → Key Pairs → Import Public Key
3. Name: `mac-key`
4. Paste contents of `~/.ssh/id_rsa_chameleon.pub`
5. Repeat for CHI@TACC and CHI@UC if needed

### **2.3 Set Proper Permissions on Mac**
```bash
chmod 600 ~/.ssh/id_rsa_chameleon
chmod 644 ~/.ssh/id_rsa_chameleon.pub
```

---

## **3. VM INSTANCE MANAGEMENT**

### **3.1 Launch New Instance via Web UI**
1. Go to: Experiment → KVM@TACC → Instances → Launch Instance
2. Configure:
   - Instance Name: `your-instance-name`
   - Flavor: `m1.small` (1 CPU, 2GB RAM, 20GB disk)
   - Image: `CC-Ubuntu24.04`
   - Key Pair: Select your uploaded key
   - Security Groups: `default`
3. Click "Launch Instance"

### **3.2 Assign Floating IP**
1. In Instance list, click instance name
2. Click "Associate Floating IP"
3. Allocate new IP or use existing
4. Note the Floating IP (e.g., `129.114.27.244`)

### **3.3 Connect via SSH**
```bash
ssh -i ~/.ssh/id_rsa_chameleon cc@YOUR_FLOATING_IP
```
- Username: `cc` (Chameleon default)
- First connection: Type `yes` to accept host fingerprint

### **3.4 Delete Instance When Done**
1. Web UI: Instances → Select instance → Delete Instances
2. Also delete Floating IP if no longer needed

---

## **4. JUPYTER NOTEBOOK SETUP**

### **4.1 Basic Jupyter Setup on VM**
```bash
# Update system
sudo apt update

# Install Python and pip
sudo apt install python3 python3-pip -y

# Install Jupyter
pip3 install notebook

# Create password (optional)
python3 -c "from notebook.auth import passwd; print(passwd('yourpassword'))"
# Copy the output hash

# Start Jupyter server
jupyter notebook --no-browser --port=8889 --ip=0.0.0.0 --NotebookApp.password='PASTE_HASH_HERE'
```

### **4.2 Create SSH Tunnel from Local Mac**
```bash
# In a NEW terminal on your Mac
ssh -i ~/.ssh/id_rsa_chameleon -N -L 8889:localhost:8889 cc@YOUR_FLOATING_IP
```
- Keep this terminal running
- Port `8889` on VM → `localhost:8889` on your Mac

### **4.3 Access Jupyter**
1. Open browser: `http://localhost:8889`
2. Enter password (or token from Jupyter logs)

### **4.4 Using Virtual Environment (Recommended)**
```bash
# On VM
python3 -m venv jupyter_env
source jupyter_env/bin/activate
pip install notebook pandas numpy matplotlib
# Then start Jupyter as above
```

---

## **5. IDE INTEGRATION**

### **5.1 Cursor IDE Connection**
1. Ensure Jupyter is running on VM and tunnel is active
2. In Cursor, create Python file with:
   ```python
   # %%
   print("Test cell")
   ```
3. Click "Run Cell" that appears above `# %%`
4. If prompted for kernel, select existing Jupyter server at `http://localhost:8889`

### **5.2 VS Code Connection**
1. Install "Jupyter" and "Python" extensions
2. `Cmd+Shift+P` → "Jupyter: Specify local or remote Jupyter server"
3. Choose "Existing" → Enter `http://localhost:8889`

### **5.3 File Transfer with SCP**
```bash
# Copy FROM VM to Mac
scp -i ~/.ssh/id_rsa_chameleon cc@YOUR_FLOATING_IP:/remote/path/file.txt /local/path/

# Copy TO VM from Mac
scp -i ~/.ssh/id_rsa_chameleon /local/path/file.txt cc@YOUR_FLOATING_IP:/remote/path/
```

---

## **6. TROUBLESHOOTING**

### **6.1 Common Errors**

**"Permission denied (publickey)"**
```bash
# Fix key permissions
chmod 600 ~/.ssh/id_rsa_chameleon
```

**"No allocation available"**
- Check project membership is approved
- Try different availability zone
- Use smaller flavor (m1.tiny)

**Jupyter connection refused**
```bash
# Check if Jupyter is running on VM
ps aux | grep jupyter

# Kill existing Jupyter
pkill -f jupyter

# Try different port
jupyter notebook --no-browser --port=8890 --ip=0.0.0.0
```

**SSH tunnel breaks**
```bash
# Check tunnel is running
lsof -i :8889

# Recreate tunnel
ssh -i ~/.ssh/id_rsa_chameleon -N -L 8889:localhost:8889 cc@YOUR_FLOATING_IP
```

### **6.2 Useful VM Commands**
```bash
# Check system info
hostname
ip addr show
df -h  # disk usage
free -h  # memory usage

# Install common packages
sudo apt install htop tmux screen git curl wget

# Monitor processes
htop
```

---

## **7. SECURITY BEST PRACTICES**

1. **Always use SSH keys** - never passwords
2. **Set strong passphrase** on SSH key
3. **Use Jupyter password/token** - never run without authentication
4. **Delete instances** when not in use
5. **Regularly update** VM packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
6. **Use firewalld** on bare metal instances
7. **Backup important data** - VM storage is ephemeral

---

## **8. QUICK REFERENCE COMMANDS**

```bash
# SSH to VM
ssh -i ~/.ssh/id_rsa_chameleon cc@129.114.27.244

# Start Jupyter on VM
jupyter notebook --no-browser --port=8889 --ip=0.0.0.0 --NotebookApp.token='1234'

# SSH tunnel from Mac
ssh -i ~/.ssh/id_rsa_chameleon -N -L 8889:localhost:8889 cc@129.114.27.244

# Test connection
curl http://localhost:8889

# Kill Jupyter on VM
pkill -f jupyter
```

---

## **9. RESOURCE MANAGEMENT**

### **Monitoring Usage**
1. Web Dashboard: Shows active instances, leases, usage quotas
2. Command Line: `openstack` CLI commands
3. Set usage alerts in project settings

### **Cost Control**
- Chameleon is free for research/education
- But resources are limited - be mindful
- Delete unused instances
- Use smaller flavors when possible

---

## **10. SUPPORT**

1. **Chameleon Help Desk**: https://www.chameleoncloud.org/user/help/
2. **Documentation**: https://chameleoncloud.readthedocs.io/
3. **Community Forum**: Ask questions and share experiments

---

## **NOTES FOR YOUR SPECIFIC SETUP**

- **Project ID**: `CHI-251409`
- **Email**: `ng3483@nyu.edu`
- **Current VM**: `nidhish-mac` at `129.114.27.244`
- **SSH Key**: `~/.ssh/id_rsa_chameleon`
- **Jupyter Port**: `8889` (tunneled to `localhost:8889`)

---

**Last Updated**: February 2, 2026  
**Status**: Working connection established to Jupyter on VM

---

# **Quick Start Cheat Sheet**

```bash
# 1. SSH to VM
ssh -i ~/.ssh/id_rsa_chameleon cc@129.114.27.244

# 2. On VM: Start Jupyter
jupyter notebook --no-browser --port=8889 --ip=0.0.0.0 --NotebookApp.token='1234'

# 3. On Mac (new terminal): Create tunnel
ssh -i ~/.ssh/id_rsa_chameleon -N -L 8889:localhost:8889 cc@129.114.27.244

# 4. Open browser: http://localhost:8889?token=1234

# 5. In Cursor: Create Python file with # %% cells and run
```

---

**Remember**: Always delete instances when finished to free resources for others!
