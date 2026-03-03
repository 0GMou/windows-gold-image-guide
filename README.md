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

Begin by opening Hyper-V Manager and launching the wizard to create a new Virtual Machine.
![New VM 1](image_captures/001_new/hyperv_new_001.PNG)

Follow the initial prompts and assign a recognizable name to your VM.
![New VM 2](image_captures/001_new/hyperv_new_002.PNG)

Select **Generation 2**. This is required for modern Windows features, UEFI, and Secure Boot.
![New VM 3](image_captures/001_new/hyperv_new_003.PNG)

Assign adequate startup memory (e.g., 4096 MB or 8192 MB) to ensure a smooth installation process.
![New VM 4](image_captures/001_new/hyperv_new_004.PNG)

Configure your networking by connecting the VM to your virtual switch so it can access the network and download any updates needed later on.
![New VM 5](image_captures/001_new/hyperv_new_005.PNG)

Create a new Virtual Hard Disk (VHDX) with sufficient space to hold the OS and your custom software.
![New VM 6](image_captures/001_new/hyperv_new_006.PNG)

Finally, select the option to install an operating system from a bootable image file and browse to your Windows ISO.
![New VM 7](image_captures/001_new/hyperv_new_007.PNG)

---

## Step 2: Configuring the Virtual Machine

Before starting the VM, right-click it and open its Settings. In the **Security** tab, ensure that Secure Boot and the Trusted Platform Module (TPM) are enabled. This is particularly important for modern Windows versions like Windows 11.
![Config 1](image_captures/002_config/hyperv_cfg_001.PNG)

Under the **Processor** tab, increase the number of virtual processors (e.g., to 4, depending on your host PC's capabilities) to speed up the installation and image preparation.
![Config 2](image_captures/002_config/hyperv_cfg_002.PNG)

In the **Integration Services** section, ensure that "Guest services" is checked to allow better interaction between the host and the virtual machine.
![Config 3](image_captures/002_config/hyperv_cfg_003.PNG)

**⚠️ CRITICAL WARNING:** You **MUST** disable Automatic Checkpoints in the Hyper-V settings for this VM, and you must **NOT** manually create any checkpoints during this entire process. If the VM has checkpoints, you will not be able to attach the resulting VHDX to the Windows Disk Management tool later on for the final image capture. Navigate to the **Checkpoints** tab and uncheck "Enable checkpoints".
![Config 4](image_captures/002_config/hyperv_cfg_004.PNG)

---

## Step 3: Starting the Virtual Machine

Power on the VM and open the console connection.
![Start 1](image_captures/003_start/hyperv_start_001.PNG)

**Troubleshooting: Missed the boot prompt?**
When the VM starts, you will see a prompt asking you to "Press any key to boot from CD or DVD". You must click inside the black virtual machine window to focus your mouse, and then press any key quickly.
![Start 2](image_captures/003_start/hyperv_start_002.PNG)

If you do not press a key in time, you will reach a "Boot Failure" screen. 
![Start 3](image_captures/003_start/hyperv_start_003.PNG)

If you reach this error screen, your mouse will not work to click the "Restart" button. **Fix:** Press the `TAB` key on your keyboard until the "Restart" button is highlighted, then press `ENTER` to restart the VM and take you back to the boot prompt.
![Start 4](image_captures/003_start/hyperv_start_004.PNG)

---

## Step 4: Installing Windows

Once booted into the ISO, select your preferred language, time format, and keyboard layout.
![Install 1](image_captures/004_install/hyperv_ins_001.PNG)

Click "Install now" to start the Windows Setup.
![Install 2](image_captures/004_install/hyperv_ins_002.PNG)

You may be prompted to enter a product key or select "I don't have a product key" to continue.
![Install 3](image_captures/004_install/hyperv_ins_003.PNG)

Select the Windows edition you want to install.
![Install 4](image_captures/004_install/hyperv_ins_004.PNG)

Accept the software license terms.
![Install 5](image_captures/004_install/hyperv_ins_005.PNG)

Choose "Custom: Install Windows only (advanced)" to perform a clean installation.
![Install 6](image_captures/004_install/hyperv_ins_006.PNG)

Select the unallocated space on your virtual drive.
![Install 7](image_captures/004_install/hyperv_ins_007.PNG)

Windows will now copy files and install. The VM will restart automatically during this process.
![Install 8](image_captures/004_install/hyperv_ins_008.PNG)

---

## Step 5: Audit Mode & System Preparation

**CRITICAL:** After the installation restarts, you will reach the Out-Of-Box Experience (OOBE) screen asking for your region. **DO NOT** proceed normally. At this screen, press `CTRL` + `SHIFT` + `F3` on your keyboard. This interrupts the setup and reboots the system into **Audit Mode**.
![Audit 1](image_captures/005_audit/hyperv_audit_001.PNG)

The system will display a loading screen while preparing the environment.
![Audit 2](image_captures/005_audit/hyperv_audit_002.PNG)

You will be logged in automatically using the built-in Administrator account.
![Audit 3](image_captures/005_audit/hyperv_audit_003.PNG)

Once the desktop loads, the System Preparation (Sysprep) tool will pop up automatically. Leave it open for now, or minimize it while you configure the system.
![Audit 4](image_captures/005_audit/hyperv_audit_004.PNG)

### Audit Commands (Cleanup)
While in Audit mode, customize your image (install software, apply Windows updates, etc.). Before generalizing the system, it is highly recommended to run cleanup commands in an elevated command prompt to ensure system integrity and reduce the final image size.

Open Command Prompt as Administrator and run the System File Checker:
```cmd
sfc /scannow
```
![Audit 5](image_captures/005_audit/hyperv_audit_005.PNG)
![Audit 6](image_captures/005_audit/hyperv_audit_006.PNG)
![Audit 7](image_captures/005_audit/hyperv_audit_007.PNG)
![Audit 8](image_captures/005_audit/hyperv_audit_008.PNG)

Once the scan is complete, open the Run dialog (`Win + R`) and launch the extended Disk Cleanup tool:
```cmd
cleanmgr /sageset:1
```
![Audit 9](image_captures/005_audit/hyperv_audit_009.PNG)

This command opens a dialog where you can select all the categories of temporary files you want to clean up. Check the boxes and click OK.
![Audit 10](image_captures/005_audit/hyperv_audit_010.PNG)

After configuring the cleanup settings, run the following command to execute the cleanup silently using those settings:
```cmd
cleanmgr /sagerun:1
```
![Audit 11](image_captures/005_audit/hyperv_audit_011.PNG)
![Audit 12](image_captures/005_audit/hyperv_audit_012.PNG)

Once you have finished customizing and cleaning up, return to the Sysprep tool. You must check the **"Generalize"** option and set the Shutdown Options to **"Shutdown"**. Click OK to seal the image. The VM will perform the generalization process and turn off completely.
![Audit 13](image_captures/005_audit/hyperv_audit_013.PNG)

---

## Step 6: Capturing the Image with DISM

After Sysprep finishes and the VM shuts down, **do not turn the VM back on**. Instead, you need to capture the generalized OS partition directly from the VHDX file.

To do this, you must open the Disk Management tool on your host Windows machine. You can do this in two ways:
**Method 1:** Right-click the Start button and select "Disk Management".
![DISM Method 1](image_captures/006_dism/hyperv_dism_001_method01.PNG)
**Method 2:** Press `Win + R`, type `diskmgmt.msc`, and press Enter.
![DISM Method 2](image_captures/006_dism/hyperv_dism_001_method02.PNG)

In Disk Management, click on "Action" in the top menu and select **"Attach VHD"**. Browse to the location of your VM's `.vhdx` file and attach it. Once attached, take note of the drive letter assigned to the main Windows partition (e.g., `D:\`).
![DISM 2](image_captures/006_dism/hyperv_dism_002.PNG)

Open Command Prompt as Administrator on your host machine to run the DISM capture process.
![DISM 3](image_captures/006_dism/hyperv_dism_003.PNG)

Prepare the folder where you want to save the captured image (for example, `C:\ISO`).
![DISM 4](image_captures/006_dism/hyperv_dism_004.PNG)

Use the `DISM` command to capture the image from the mounted VHDX partition.
![DISM 5](image_captures/006_dism/hyperv_dism_005.PNG)

The tool will process and save the generalized image into a `.wim` file. This may take some time depending on the size of your installation.
![DISM 6](image_captures/006_dism/hyperv_dism_006.PNG)
![DISM 7](image_captures/006_dism/hyperv_dism_007.PNG)

When the process completes successfully, your `.wim` image file is ready. You can now detach the VHDX from Disk Management.
![DISM 8](image_captures/006_dism/hyperv_dism_008.PNG)

### DISM Command Syntax
Make sure to adjust the drive letters and paths according to your environment.

```cmd
Dism /Capture-Image /ImageFile:C:\ISO\install.wim /CaptureDir:D:\ /Name:"name_of_the_so"
```

**Parameters Explained:**
- `/Capture-Image`: The DISM execution tool flag.
- `/ImageFile:C:\ISO\install.wim`: Change this directory to where you want the captured `.wim` file saved.
- `/CaptureDir:D:\`: The drive letter of the generalized Windows partition you want to capture (Change `D:\` to the actual drive letter shown in Disk Management).
- `/Name:"name_of_the_so"`: Change this to the desired name of the OS image.
