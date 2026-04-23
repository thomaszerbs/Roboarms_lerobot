# ROCm on WSL2 - Full Setup Guide

**Hardware:** AMD Ryzen APU (integrated GPU)  
**OS:** Windows 11 + WSL2 (Ubuntu 24.04)

---

## Before Running — Do These in Windows First

1. **Install AMD Adrenalin driver 26.2.2 or later**  
   https://www.amd.com/en/support/download/drivers.html

2. **Install Windows SDK in Windows (NOT in WSL)**  
   https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/  
   Default install path: `C:\Program Files (x86)\Windows Kits\10\`

3. **Verify the SDK path exists in WSL before continuing:**
   ```bash
   ls "/mnt/c/Program Files (x86)/Windows Kits/10/Include/"
   ```
   Note the version folder shown (e.g. `10.0.26100.0`) and update the `cmake` line in Step 2 if your version differs.

4. **Install WSL2 with Ubuntu 24.04** (run in Windows PowerShell):
   ```
   wsl --install Ubuntu-24.04
   ```

---

## Step 1 — Install ROCm (run these manually in WSL)

```bash
wget https://repo.radeon.com/amdgpu-install/6.3.2/ubuntu/noble/amdgpu-install_6.3.60302-1_all.deb
sudo apt install ./amdgpu-install_6.3.60302-1_all.deb
amdgpu-install -y --usecase=wsl,rocm --no-dkms
```

---

## Step 2 — Build and Configure (run as one block)

Builds and installs `librocdxg` (bridges ROCm to the Windows GPU via the DXG interface),
sets the required environment variable, configures WSL memory allocation, and verifies the setup.

```bash
sudo apt update -qq && sudo apt install -y cmake build-essential git && \
cd ~ && rm -rf librocdxg && git clone https://github.com/ROCm/librocdxg.git && \
cd librocdxg && git checkout develop && \
mkdir -p build && cd build && \
cmake .. -DWIN_SDK="/mnt/c/Program Files (x86)/Windows Kits/10/Include/10.0.26100.0/shared" && \
make -j$(nproc) && sudo make install && \
grep -q "HSA_ENABLE_DXG_DETECTION" ~/.bashrc || echo 'export HSA_ENABLE_DXG_DETECTION=1' >> ~/.bashrc && \
export HSA_ENABLE_DXG_DETECTION=1

# Configure WSL memory allocation (24GB out of 46GB total)
cat > /mnt/c/Users/thomas/.wslconfig << 'EOF'
[wsl2]
memory=24GB
EOF

# Verify setup - GPU should appear in rocminfo output
rocminfo
```
