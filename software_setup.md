---
layout: default
title: Software Setup & Build
nav_order: 3
---

# 3. Software Environment & Compilation

This section covers how to set up your local development environment and compile the firmware from source. 


## 3.1 Core Software Installation

To compile the code, you need a specific set of tools. Follow the instructions for your operating system.

### 1. Essential Tools
* **Git**: Used for downloading and managing the source code.  
    [Download Git here](https://git-scm.com/downloads)  
    *Installation Tip: Just keep clicking "Next" for all default options.*
* **Visual Studio Code (VS Code)**: The recommended code editor.  
    [Download VS Code here](https://code.visualstudio.com/)

### 2. ARM Toolchain & CMake
This is the "engine" that turns C++ code into a file the Pico can understand.

* **Windows Users (Highly Recommended)**:  
    Download and run the **[Raspberry Pi Pico Windows Installer](https://github.com/raspberrypi/pico-setup-windows/releases/latest)**.  
    This is a "one-click" installer that sets up CMake, Python, and the ARM GCC compiler automatically. It prevents 99% of environment variable errors.
* **Mac Users**:  
    Open your terminal and run:  
    `brew install cmake arm-none-eabi-gcc libnewlib-arm-none-eabi`
* **Linux Users (Ubuntu/Debian)**:  
    `sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential`

### 3. VS Code Extensions
Open VS Code, go to the **Extensions** view (Ctrl+Shift+X), and install:
1.  **C/C++** 
2.  **CMake Tools** 

---

## 3.2 Fetching the Source Code

> [!CAUTION]
> **DO NOT** click the "Download ZIP" button on GitHub.  
> This project uses "Submodules" (like SimpleFOC). A ZIP download will result in empty folders, and the project **will not compile**.

### The Command
Open your terminal (or Git Bash) and run the following command to download the repository and all its dependencies:


```bash
git clone --recursive [https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal.git](https://github.com/Realtime-Gimbal-Team/Realtime-CineFollow-Gimbal.git)

```

## 3.3 VS Code Local Compilation Process

Once the environment is configured and the source code is downloaded, follow these steps to "ignite" the compilation:

### Step 1: Open the Project
Launch VS Code, click **File > Open Folder...**, and select the `Realtime-CineFollow-Gimbal` folder you just downloaded/cloned.

### Step 2: Select the Compiler (Kit)
Observe the status bar at the very bottom of VS Code. Click on the area that says **"No Kit Selected"** (or shows an existing compiler name). In the dropdown list that appears at the top of the screen, select:

> **GCC for arm-none-eabi** (e.g., version 13.2.1).



### Step 3: One-Click Build
1. Click the **Build** button (the small "gear" icon) on the bottom status bar.
2. Monitor the **Output** window at the bottom. When you see `[100%] Built target CineFollow_Gimbal`, the compilation is successful!

### Step 4: Locate the Output File
After a successful build, navigate to the following path in your file explorer:
`build/pico/Pico-firmware/`

You will find a file with the **`.uf2`** extension. This is the "document of the firmware that we will flash into the Pico.

### Step 5: Flash to Pico
1. **Unplug** the USB/power cable from the Pico.
2. Press and **hold** the white **BOOTSEL** button on the Pico.
3. **Connect** the Pico to your computer via USB while holding the button, then release it.
4. Your computer will recognize a new removable drive named **RPI-RP2**.
5. **Drag and drop** the `.uf2` file you found in Step 4 directly into this drive. 

The Pico will reboot automatically, and the gimbal program will start running immediately.
