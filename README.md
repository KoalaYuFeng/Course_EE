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
ssh yourname@xacchad.ddns.comp.nus.edu.sg
```

---

## 2. Request an MI100 GPU node (interactive session)

Use **srun** to request a GPU node with 1 MI100 GPU and 8 CPU cores:

```bash
srun -p 8xMI100 -c 8 --gres=gpu:1 --pty bash
```

You will be placed on a GPU node such as `hacc-gpu1`, `hacc-gpu5`, etc.

---

## 3. Verify the GPUs on the node

```bash
rocm-smi
```

You should see something similar to:

```
========================ROCm System Management Interface========================
GPU  Temp   Power   Memory   Usage  ...
0    45°C   30W     0%       0%     AMD Instinct MI100
1    44°C   29W     0%       0%     AMD Instinct MI100
...
```

This confirms you are on a node with AMD MI100 GPUs.

---

## 4. Enter the course Apptainer environment

The course environment is packaged as:

```
/opt/containers/hacc_rocm63.sif
```

Start an interactive shell inside it:

```bash
apptainer exec --rocm /opt/containers/hacc_rocm63.sif bash
```

Your prompt will change to:

```
Apptainer>
```

You are now inside the custom teaching environment with Python, PyTorch, GCC, CMake, etc.

---

## 5. Test GPU access inside the container

Run the following Python test:

```bash
python3 - << 'EOF'
import torch
print("GPU available:", torch.cuda.is_available())
print("GPU:", torch.cuda.get_device_name(0) if torch.cuda.is_available() else None)
EOF
```

Expected output:

```
GPU available: True
GPU: AMD Instinct MI100
```

This means PyTorch has successfully detected the GPU via ROCm.

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

or using CMake:

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

## 7. Exit the container and the node

Exit Apptainer:

```bash
exit
```

Exit the GPU node:

```bash
exit
```

Exit the login node:

```bash
exit
```

---

## Summary of Common Commands

| Task | Command |
|------|---------|
| SSH into cluster | `ssh yourname@xacchad.ddns.comp.nus.edu.sg` |
| Request MI100 GPU node | `srun -p 8xMI100 -c 8 --gres=gpu:1 --pty bash` |
| Check GPU info | `rocm-smi` |
| Enter Apptainer container | `apptainer exec --rocm /opt/containers/hacc_rocm63.sif bash` |
| Test PyTorch GPU | *(Python script above)* |
| Exit container | `exit` |

---
