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

`Begin by opening Hyper-V Manager and launching the wizard to create a new Virtual Machine. Click "Next" on the Before You Begin screen.
Specify a Name and Location for your virtual machine.`

![New VM 1](image_captures/001_new/hyperv_new_001.PNG)

`Select Generation 2. This is required for modern Windows features, UEFI, and Secure Boot.`

![New VM 2](image_captures/001_new/hyperv_new_002.PNG)

`Assign adequate startup memory (e.g., 8192 MB) to ensure a smooth installation process.`

![New VM 3](image_captures/001_new/hyperv_new_003.PNG)

`Configure your networking by connecting the VM to your virtual switch (e.g., Default Switch) so it can access the network or select "Not Connected"`

![New VM 4](image_captures/001_new/hyperv_new_004.PNG)

`Create a new Virtual Hard Disk (VHDX), verifying the name, location, and size.`

![New VM 5](image_captures/001_new/hyperv_new_005.PNG)

`Finally, select the option to install an operating system from a bootable image file and browse to your Windows ISO.`

![New VM 6](image_captures/001_new/hyperv_new_006.PNG)

`Click Finish.`

![New VM 7](image_captures/001_new/hyperv_new_007.PNG)

---

## Step 2: Configuring the Virtual Machine

`Before starting the VM, right-click it and open its Settings. In the "Security" tab, ensure that Secure Boot and the Trusted Platform Module (TPM) are enabled.`

![Config 1](image_captures/002_config/hyperv_cfg_001.PNG)

`Under the "Processor" tab, increase the number of virtual processors (e.g., to 4) to speed up the installation and operations.`

![Config 2](image_captures/002_config/hyperv_cfg_002.PNG)

`In the "Integration Services" section, ensure that "Guest services" is checked to allow better interaction and file sharing between the host and the VM.`

![Config 3](image_captures/002_config/hyperv_cfg_003.PNG)

**⚠️ CRITICAL WARNING:** You **MUST** disable Automatic Checkpoints in the Hyper-V settings for this VM, and you must **NOT** manually create any checkpoints during this entire process. If the VM has checkpoints, you will not be able to attach the resulting VHDX to the Windows Disk Management tool later on for the final image capture. 
Navigate to the **Checkpoints** tab and uncheck "Enable checkpoints".

![Config 4](image_captures/002_config/hyperv_cfg_004.PNG)

---

## Step 3: Starting the Virtual Machine

`Power on the VM and open the console connection. The VM will start to boot.`

![Start 1](image_captures/003_start/hyperv_start_001.PNG)

`When the VM starts, you will see a prompt asking you to "Press any key to boot from CD or DVD". You must click inside the black virtual machine window to focus your mouse, and then press any key quickly.`

![Start 2](image_captures/003_start/hyperv_start_002.PNG)

`Troubleshooting: Missed the boot prompt? If you do not press a key in time, you will reach a "Boot Failure" screen because there is no OS installed yet. `

![Start 3](image_captures/003_start/hyperv_start_003.PNG)

`If you reach this error screen, your mouse will not work to click the "Restart" button. To fix it press the "TAB" key on your keyboard until the "Restart" button is highlighted, then press "ENTER" to restart the VM and take you back to the boot prompt.`

![Start 4](image_captures/003_start/hyperv_start_004.PNG)

---

## Step 4: Installing Windows

`Once booted into the ISO, select your preferred language, time format, and keyboard layout, then click Next.`

![Install 1](image_captures/004_install/hyperv_ins_001.PNG)

`Click the "Install now" button in the center of the screen.`

![Install 2](image_captures/004_install/hyperv_ins_002.PNG)

`You may be prompted to enter a product key. Select "I don't have a product key" at the bottom to continue.`

![Install 3](image_captures/004_install/hyperv_ins_003.PNG)

`Check the box to accept the software license terms and click Next. If rou ISO has more than one edition, select the specific Windows edition you want to install for your Gold Image (e.g., Windows 10 IoT Enterprise) and click Next.`

![Install 4](image_captures/004_install/hyperv_ins_004.PNG)

`Choose the "Custom: Install Windows only (advanced)" option to perform a clean installation.`

![Install 5](image_captures/004_install/hyperv_ins_005.PNG)

`Select the unallocated space on your virtual drive (Drive 0) and click Next.`

![Install 6](image_captures/004_install/hyperv_ins_006.PNG)

`Windows will now copy files and install. The VM will restart automatically during this process.`

![Install 7](image_captures/004_install/hyperv_ins_007.PNG)
![Install 8](image_captures/004_install/hyperv_ins_008.PNG)

---

## Step 5: Audit Mode & System Preparation

**⚠️ CRITICAL WARNING:** After the installation restarts, you will reach the Out-of-Box Experience (OOBE) screen asking for your region `"Let's start with region. Is this right?"`. DO NOT proceed normally. 
At this exact screen, press `[CTRL]` + `[SHIFT]` + `[F3]` on your keyboard. This interrupts the normal user setup and reboots the system into **Audit Mode**.

![Audit 1](image_captures/005_audit/hyperv_audit_001.PNG)

`The system will reboot and display the "Just a moment..." loading screen while preparing the Audit Mode environment.`

![Audit 2](image_captures/005_audit/hyperv_audit_002.PNG)

`You will be logged in automatically using the built-in Administrator account.`

![Audit 3](image_captures/005_audit/hyperv_audit_003.PNG)

`Once the desktop loads, the System Preparation Tool (Sysprep) window will pop up automatically in the center of the screen. Leave it open for now while you configure the system.`

![Audit 4](image_captures/005_audit/hyperv_audit_004.PNG)

### System Cleanup in Audit Mode
While in Audit Mode, AFTER you customize your image (install your software, apply Windows updates, etc.). You must run a command in an elevated command prompt to ensure system integrity.

Open the Start menu, type `cmd`, right-click on Command Prompt and run it as Administrator. Type the following command and press Enter:
```cmd
sfc /scannow
```
`Wait for the verification phase to reach 100% and repair any corrupt files.`

![Audit 5](image_captures/005_audit/hyperv_audit_005.PNG)

`At this time you can go online to download the updates and customize your image selecting the correct network switch on the VM config`

![Audit 6](image_captures/005_audit/hyperv_audit_006.PNG)

`You may be prompted about network discovery on the right side of the screen. You can click "Yes" to allow the PC to be discoverable if you need network resources during customization.`

![Audit 7](image_captures/005_audit/hyperv_audit_007.PNG)

`Open Windows Settings and go to Update & Security. Check for updates and let them download. If updates require a restart, click "Restart now". The system will restart back into Audit Mode automatically.`

![Audit 8](image_captures/005_audit/hyperv_audit_008.PNG)
![Audit 9](image_captures/005_audit/hyperv_audit_009.PNG)

`After restarting and finishing updates, open an elevated Command Prompt again. You need to configure the disk cleanup tool by running:`
```cmd
cleanmgr /sageset:1
```
`The command above opens the Disk Cleanup Settings dialog. Scroll through the list and check all the boxes (including Windows Update Cleanup, Temporary files, etc.) and click OK.`

![Audit 10](image_captures/005_audit/hyperv_audit_010.PNG)

`Back in the command prompt, run the following command to execute the cleanup silently using the settings you just saved:`
```cmd
cleanmgr /sagerun:1
```
`The Disk Cleanup window will appear temporarily showing the progress of deleting the files you selected.`

![Audit 11](image_captures/005_audit/hyperv_audit_011.PNG)

`Once the cleanup is completely finished, return to the Sysprep tool that opened at startup. Set System Cleanup Action to "Enter System Out-of-Box Experience (OOBE)". Check the box for "Generalize" (Critical for removing hardware-specific info). Set Shutdown Options to "Shutdown". Click OK to seal the image. The VM will perform the generalization process and turn off completely.`

![Audit 12](image_captures/005_audit/hyperv_audit_012.PNG)
![Audit 13](image_captures/005_audit/hyperv_audit_013.PNG)

---

## Step 6: Capturing the Image with DISM

`After Sysprep finishes and the VM shuts down, do not turn the VM back on. Instead, you need to capture the generalized OS partition directly from the VM's VHDX file. To do this, you must open the Disk Management tool on your host Windows machine. You can do this by right-clicking the Start button and selecting "Disk Management".`

![DISM 1 Method 1](image_captures/006_dism/hyperv_dism_001_method01.PNG)

`Alternatively, press [Win] + [R], type "diskmgmt.msc", and press Enter.`

![DISM 1 Method 2](image_captures/006_dism/hyperv_dism_001_method02.PNG)

`In Disk Management, click on "Action" in the top menu and select "Attach VHD".`

![DISM 2](image_captures/006_dism/hyperv_dism_002.PNG)

`Browse to the location of your VM's ".vhdx" file and attach it.`

![DISM 3](image_captures/006_dism/hyperv_dism_003.PNG)
![DISM 4](image_captures/006_dism/hyperv_dism_004.PNG)

`Ensure you have checked the "Read-only checkbox"`

![DISM 5](image_captures/006_dism/hyperv_dism_005.PNG)

`Once the disk is attached, look at the bottom section of Disk Management to find the newly attached disk (it will appear with a blue bar if it has partitions). Identify the large main NTFS partition (not the small 100MB EFI partition) and note the drive letter assigned to it (e.g., D: in this example).`

![DISM 6](image_captures/006_dism/hyperv_dism_006.PNG)

`Open Command Prompt as Administrator on your host machine to run the DISM capture process. Before running the capture, ensure the destination folder where you want to save the ".wim" file exists on your host machine (e.g., C:\location). In the Administrator Command Prompt, type the "DISM" command to capture the image. You must specify the image file destination, the capture directory (the drive letter of the mounted VHDX), and a name for the image.`

![DISM 7](image_captures/006_dism/hyperv_dism_007.PNG)

`The tool will process the files and save the generalized image into a ".wim" file. This may take some time depending on the size of your installation. When the process reaches 100% and states "The operation completed successfully", your ".wim" image file is ready. You must now return to Disk Management, right-click the grey area of the attached disk on the left side, and select "Detach VHD".`

![DISM 8](image_captures/006_dism/hyperv_dism_008.PNG)

### DISM Command Syntax
Make sure to adjust the drive letters and paths according to your environment.

```cmd
Dism /Capture-Image /ImageFile:C:\location\install.wim /CaptureDir:W:\ /Name:"CHANGE THIS"
```

**Parameters Explained:**
- `/Capture-Image`: The DISM execution tool flag.
- `/ImageFile:C:\location\install.wim`: Change this directory to where you want the captured `.wim` file saved.
- `/CaptureDir:W:\`: The drive letter of the generalized Windows partition you want to capture (Change `W:\` to the actual drive letter shown in Disk Management).
- `/Name:"CHANGE THIS"`: Change this to the desired name of the OS image.
