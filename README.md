# Windows Gold Image Creation Guide

This repository contains a comprehensive step-by-step guide on how to create a "Gold Image" (reference image) for Windows deployments. The process uses Microsoft Hyper-V, Windows Audit Mode (Sysprep), and the Deployment Image Servicing and Management (DISM) tool.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Step 1: Creating a New Virtual Machine](#step-1-creating-a-new-virtual-machine)
3. [Step 2: Configuring the Virtual Machine](#step-2-configuring-the-virtual-machine)
4. [Step 3: Starting the Virtual Machine](#step-3-starting-the-virtual-machine)
5. [Step 4: Installing Windows](#step-4-installing-windows)
6. [Step 5: Audit Mode & System Preparation](#step-5-audit-mode--system-preparation)
7. [Step 6: Capturing the Image with DISM](#step-6-capturing-the-image-with-dism)

---

## Prerequisites
- Windows OS with Hyper-V enabled.
- A Windows installation ISO.

---

## Step 1: Creating a New Virtual Machine
Begin by creating a new Virtual Machine in Hyper-V Manager.

![New VM 1](image_captures/001_new/hyperv_new_001.PNG)
![New VM 2](image_captures/001_new/hyperv_new_002.PNG)
![New VM 3](image_captures/001_new/hyperv_new_003.PNG)
![New VM 4](image_captures/001_new/hyperv_new_004.PNG)
![New VM 5](image_captures/001_new/hyperv_new_005.PNG)
![New VM 6](image_captures/001_new/hyperv_new_006.PNG)
![New VM 7](image_captures/001_new/hyperv_new_007.PNG)

---

## Step 2: Configuring the Virtual Machine
Configure the VM settings, ensuring you attach the Windows ISO for installation.

**⚠️ CRITICAL WARNING:** You **MUST** disable Automatic Checkpoints in the Hyper-V settings for this VM, and you must **NOT** manually create any checkpoints during this entire process. If the VM has checkpoints, you will not be able to attach the resulting VHDX to the Windows Disk Management tool later on for the final image capture.

![Config 1](image_captures/002_config/hyperv_cfg_001.PNG)
![Config 2](image_captures/002_config/hyperv_cfg_002.PNG)
![Config 3](image_captures/002_config/hyperv_cfg_003.PNG)
![Config 4](image_captures/002_config/hyperv_cfg_004.PNG)

---

## Step 3: Starting the Virtual Machine
Start the VM and boot into the Windows Setup environment.

![Start 1](image_captures/003_start/hyperv_start_001.PNG)
![Start 2](image_captures/003_start/hyperv_start_002.PNG)
![Start 3](image_captures/003_start/hyperv_start_003.PNG)
![Start 4](image_captures/003_start/hyperv_start_004.PNG)

---

## Step 4: Installing Windows
Proceed with the standard Windows installation process.

![Install 1](image_captures/004_install/hyperv_ins_001.PNG)
![Install 2](image_captures/004_install/hyperv_ins_002.PNG)
![Install 3](image_captures/004_install/hyperv_ins_003.PNG)
![Install 4](image_captures/004_install/hyperv_ins_004.PNG)
![Install 5](image_captures/004_install/hyperv_ins_005.PNG)
![Install 6](image_captures/004_install/hyperv_ins_006.PNG)
![Install 7](image_captures/004_install/hyperv_ins_007.PNG)
![Install 8](image_captures/004_install/hyperv_ins_008.PNG)

---

## Step 5: Audit Mode & System Preparation
**CRITICAL:** When you reach the Out-Of-Box Experience (OOBE) screen (where it asks for your region/language), **DO NOT** proceed normally.

Press `CTRL` + `SHIFT` + `F3` to enter **Audit Mode**. The system will reboot and log you in automatically as the built-in Administrator.

![Audit 1](image_captures/005_audit/hyperv_audit_001.PNG)
*(Images 002 through 013 show the process inside Audit Mode)*  
![Audit 2](image_captures/005_audit/hyperv_audit_002.PNG)
![Audit 3](image_captures/005_audit/hyperv_audit_003.PNG)
![Audit 4](image_captures/005_audit/hyperv_audit_004.PNG)
![Audit 5](image_captures/005_audit/hyperv_audit_005.PNG)
![Audit 6](image_captures/005_audit/hyperv_audit_006.PNG)
![Audit 7](image_captures/005_audit/hyperv_audit_007.PNG)
![Audit 8](image_captures/005_audit/hyperv_audit_008.PNG)
![Audit 9](image_captures/005_audit/hyperv_audit_009.PNG)
![Audit 10](image_captures/005_audit/hyperv_audit_010.PNG)
![Audit 11](image_captures/005_audit/hyperv_audit_011.PNG)
![Audit 12](image_captures/005_audit/hyperv_audit_012.PNG)
![Audit 13](image_captures/005_audit/hyperv_audit_013.PNG)

### Audit Commands (Cleanup)
While in Audit mode, customize your image (install software, updates, etc.). Before generalizing, you can run cleanup commands in an elevated command prompt:

```cmd
sfc /scannow
cleanmgr /sageset:1
cleanmgr /sagerun:1
```

Once ready, use the Sysprep tool (which usually opens automatically in Audit Mode) to **Generalize** the system and choose **Shutdown**.

---

## Step 6: Capturing the Image with DISM
After the VM has shut down from Sysprep, you need to capture the generalized OS partition into a `.wim` file. Boot the VM into Windows PE or attach the VHDx to another machine to perform the capture.

![DISM Method 1](image_captures/006_dism/hyperv_dism_001_method01.PNG)
![DISM Method 2](image_captures/006_dism/hyperv_dism_001_method02.PNG)
![DISM 2](image_captures/006_dism/hyperv_dism_002.PNG)
![DISM 3](image_captures/006_dism/hyperv_dism_003.PNG)
![DISM 4](image_captures/006_dism/hyperv_dism_004.PNG)
![DISM 5](image_captures/006_dism/hyperv_dism_005.PNG)
![DISM 6](image_captures/006_dism/hyperv_dism_006.PNG)
![DISM 7](image_captures/006_dism/hyperv_dism_007.PNG)
![DISM 8](image_captures/006_dism/hyperv_dism_008.PNG)

### DISM Command Syntax
Use the following `DISM` command to capture the image. Make sure to adjust the drive letters and paths according to your environment.

```cmd
Dism /Capture-Image /ImageFile:C:\ISO\install.wim /CaptureDir:D:\ /Name:"name_of_the_so"
```

**Parameters Explained:**
- `/Capture-Image`: The DISM execution tool flag.
- `/ImageFile:C:\ISO\install.wim`: Change this directory to where you want the captured `.wim` file saved.
- `/CaptureDir:D:`: The drive letter of the generalized Windows partition you want to capture (Change `D:` to your actual drive letter).
- `/Name:"name_of_the_so"`: Change this to the desired name of the OS image.
