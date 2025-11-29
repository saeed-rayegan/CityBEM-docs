# Installation

CityBEM V2 is implemented in modern **C++17** for high-performance city-scale simulation.  
The framework is compiled using **Microsoft Visual Studio 2022 (MSVC v143)** and includes optional Python utilities for preprocessing, automation, and visualization.

This guide explains how to install, compile, and run CityBEM V2 from source.

---

## 1. Requirements

### **Core Software**
To build the C++ solver:

  - Microsoft Visual Studio 2022
  - Workload: Desktop development with C++
  - Platform Toolset: MSVC v143
  - Windows SDK Version: 10.0 (latest installed version)  
  - C++ Language Standard: ISO C++17 (/std:c++17)
  - Git Bash  Required for cloning and managing CityBEM V2 source code.

<div style="background-color: #e8f1ff; padding: 14px; border-left: 4px solid #2f81f7; border-radius: 6px; margin-top: 12px;">
  <strong>Download The Latest Version of Visual Studio:</strong><br>
  <a href="https://visualstudio.microsoft.com/" target="_blank" style="font-size: 15px; color: #0a58ca; font-weight: 600; text-decoration: none;">
    https://visualstudio.microsoft.com/
  </a>
</div>

### **Optional: Python Tools**
Python utilities assist with:
- Input pre-processing  
- Batch simulation automation  
- Time-series and spatial post-processing  
- Optional 3D visualization  

Install Python (3.10+) and run:

```bash
pip install numpy pandas matplotlib pyvista
```

---

## 2. Clone the Repository

Download the CityBEM V2 source code using Git:

```bash
git clone https://example.git/CityBEM.git
cd CityBEM
```

<div style="background-color: #fff3cd; padding: 12px; border-left: 4px solid #ffca2c; border-radius: 4px; margin-top: 10px;">
<strong>Important:</strong> The official open-source CityBEM V2 repository is <strong>not yet published</strong>.<br>
This is only a placeholder URL. The public release will be announced once the repository is available.
</div>

---

## 3. Build CityBEM V2 from Source (Visual Studio 2022)

CityBEM V2 ships as a modern C++17 Visual Studio project.  
Your project settings (from your screenshots) confirm correct configuration.

### ✔ Visual Studio Project Settings (Already Configured)

| Setting | Value |
|--------|-------|
| **Platform Toolset** | Visual Studio 2022 (v143) |
| **Windows SDK Version** | 10.0 (Latest Installed) |
| **C++ Language Standard** | ISO C++17 (/std:c++17) |
| **Debug Information Format** | Program Database (/ZI) |
| **Warning Level** | Level 3 (/W3) |
| **SDL Checks** | Enabled (/sdl) |
| **Optimization (Release)** | Maximize Speed (/O2) |

### **Step-by-Step Build Instructions**

#### **Step 1 — Open the Project**
- Launch **Visual Studio 2022**
- Choose **Open a local folder**  
  or open the included `CityBEM.sln` file  
- Visual Studio will automatically configure the project

#### **Step 2 — Select Build Mode**
- Choose **Release x64** for simulation performance  
- Or **Debug x64** for development and testing

#### **Step 3 — Build the Executable**
Press:

```
Ctrl + Shift + B
```

The compiled solver will appear in:

```
/out/build/x64-Release/CityBEM.exe
```

Or the corresponding Debug directory.

---

## 4. Running CityBEM V2

After compiling, you can run the solver via the command line:

```bash
./CityBEM.exe path/to/config.yaml
```

Typical workflow:
1. Prepare your building and city input data  
2. Configure the simulation parameters in a YAML file  
3. Run the solver  
4. Use Python tools for post-processing (optional)

---

## 5. Optional: Python Tools for Automation & Visualization

Install Python utilities:

```bash
pip install numpy pandas matplotlib pyvista
```

You may use Python scripts to:
- Automate multi-scenario runs  
- Analyze output time-series  
- Visualize temperature, fluxes, and system variables  
- Create 3D visualizations with PyVista  

---

## 6. Optional: Build Documentation Locally

If contributing to documentation:

```bash
pip install mkdocs-material
mkdocs serve
```

This launches a live local preview of the CityBEM V2 documentation.

---

CityBEM V2 is designed to remain lightweight, extendable, and developer‑friendly.  
Compiling from source ensures maximum compatibility and performance for large-scale city simulations.