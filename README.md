# HACC GPU Computing Environment Guide  
**AMD MI100 + ROCm + Apptainer**

This guide explains how to:

1. SSH into the HACC cluster  
2. Allocate a GPU node with SLURM  
3. Verify AMD Instinct MI100 GPUs  
4. Enter the official course Apptainer environment  
5. Test PyTorch ROCm GPU access  
6. Build your own C++/Python project inside the container  

---

## 1. SSH into HACC

Use your NUS account:

```bash
ssh yourname@xacchead.ddns.comp.nus.edu.sg
```

---

## 2. Request an MI100 GPU node (interactive session)

Use **srun** to request a GPU node and 8 CPU cores:

```bash
srun -p 8xMI100 -c 8 --pty bash
```

---

## 3. Check which GPUs are currently in use

Before entering the Apptainer environment, inspect the MI100 GPUs on the node:

```bash
rocm-smi
```

This shows all 8 MI100 GPUs and their usage status. Example:

```
GPU[0]   Busy   PID: 3112 (alice: python)
GPU[1]   Idle
GPU[2]   Busy   PID: 4221 (bob: train.py)
GPU[3]   Idle
GPU[4]   Idle
GPU[5]   Busy   PID: 8123 (charlie: notebook)
GPU[6]   Idle
GPU[7]   Idle
```

### You can see:

- Which GPUs are **Idle**
- Which GPUs are **Busy**
- Which user/process is using each GPU

### Choosing a GPU

If you want to use a specific free GPU (for example GPU 3), export its ID:

```bash
export HIP_VISIBLE_DEVICES=3
```

Then enter the Apptainer environment:

```bash
apptainer exec --rocm /opt/containers/hacc_rocm63.sif bash
```

Inside the container:

```bash
python3 - << 'EOF'
import torch
print("GPU available:", torch.cuda.is_available())
print("GPU:", torch.cuda.get_device_name(0) if torch.cuda.is_available() else None)
EOF
```

Expected result:

```
GPU available: True
GPU: AMD Instinct MI100
```

This means your process is now using **GPU 3** (or whichever GPU you selected).

---

## 4. Enter the course Apptainer environment

Start an interactive shell inside the container:

```bash
apptainer exec --rocm /opt/containers/hacc_rocm63.sif bash
```

Your prompt will change to:

```
Apptainer>
```

You are now inside the custom teaching environment with Python, PyTorch, GCC, CMake, etc.

---

## 5. Test GPU access inside Apptainer

```bash
python3 - << 'EOF'
import torch
print("GPU available:", torch.cuda.is_available())
print("GPU:", torch.cuda.get_device_name(0) if torch.cuda.is_available() else None)
EOF
```

Expected:

```
GPU available: True
GPU: AMD Instinct MI100
```

---

## 6. Build your own project (NeuroSim or any C++/Python project)

Inside the Apptainer environment:

### Clone your project

```bash
cd ~
git clone <your_project_repo_url>
cd <your_project_repo_dir>
```

### Build a C/C++ project

```bash
make
```

or CMake:

```bash
mkdir build
cd build
cmake ..
make
```

### Install Python dependencies

```bash
pip install --user -r requirements.txt
```

All files will be stored in your home directory, not inside the container.

---

## 7. Installing additional Python packages (students)

You should **not modify the shared container image**.  
Instead, install extra Python packages into your **own home directory**:

```bash
pip install --user <package-name>
```

Examples:

```bash
pip install --user transformers
pip install --user networkx
pip install --user matplotlib
```

Packages are installed to:

```
~/.local/lib/python3.x/site-packages/
```

If Python cannot find the package, add:

```bash
export PYTHONPATH=$HOME/.local/lib/python3.10/site-packages:$PYTHONPATH
```

---

## Summary of Common Commands

| Task | Command |
|------|---------|
| SSH into cluster | `ssh yourname@xacchad.ddns.comp.nus.edu.sg` |
| Check GPU usage | `rocm-smi` |
| Choose GPU 3 | `export HIP_VISIBLE_DEVICES=3` |
| Enter container | `apptainer exec --rocm /opt/containers/hacc_rocm63.sif bash` |
| Test PyTorch GPU | *(Python script above)* |
| Install Python package | `pip install --user <package>` |
| Exit container | `exit` |

---
