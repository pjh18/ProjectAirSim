# Project AirSim Installation Instructions

Tested on a fresh install of Ubuntu Gnome 22.04.5 LTS with all available updates installed. This documentation will install Unreal Engine and Project AirSim into the home directory of your user account (/home/[user], aka ~).

---
## Installing System Prerequisites

1. For Nvidia GPU drivers, you can confirm that they are working correctly by running:

```
nvidia-smi
```

2. For Ubuntu, the Vulkan libraries also need to be installed by running:

```
sudo apt update
sudo apt install libvulkan1 vulkan-tools
sudo reboot
```

*Note: If official/proprietary GPU drivers from Nvidia (or AMD) are not installed, OR if you want to use open source GPU drivers (like Nouveau on Linux), OR use an integrated GPU (like Intel), you may need to install the Mesa Vulkan driver package by running:*

```
sudo apt install mesa-vulkan-drivers
```

*To confirm that the Vulkan libraries are working correctly, you can run:*

```
vulkaninfo
```

3. Install the tools required to perform the setup:

```
sudo apt install build-essential git
```

---

## Developer Initial Setup for Linux

1. Clone the Project AirSim repo into your home directory:

```
git clone https://github.com/iamaisim/ProjectAirSim.git
```

2. From the `~/projectairsim/` folder, install the prerequisites:

```
./setup_linux_dev_tools.sh
```

This will install:
   - **make** - used by build scripts to drive CMake commands
   - **cmake** - used to build sim lib components
   - **clang 13, libc++ 13** - used to build sim lib components and match the bundled toolchain of Unreal Engine 5.2
   - **ninja** - used as the preferred CMake generator/build tool
   - **vulkan loader library** - for UE rendering on Linux (OpenGL has been deprecated)
   - **vulkan utils** - utilities like `vulkaninfo` for checking for graphics support
   
You may also need to install the latest drivers for your GPU to support the Vulkan interface.

3. Download and install **[VisualStudio Code](https://code.visualstudio.com/docs/?dv=linux64_deb)** and the following extensions:

   `sudo dpkg -i code_[version]_amd64.deb`

    *Note: If not manually installed, these extensions will be recommended for automatic install on opening the Project AirSim project workspace*

    - **[CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)**
    
        `code --install-extension ms-vscode.cmake-tools`
    - **[C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)** (includes clang-format functionality, use **Alt-Shift-F** to auto-format the current code file using the repo's .clang-format settings)
    
        `code --install-extension ms-vscode.cpptools`
    - **[CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)**
   
       `code --install-extension vadimcn.vscode-lldb`
    - **[C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)**
   
       `code --install-extension ms-dotnettools.csharp`
    - **[Mono Debug](https://marketplace.visualstudio.com/items?itemName=ms-vscode.mono-debug)**

      `code --install-extension ms-vscode.mono-debug`
    - **[C++ TestMate](https://marketplace.visualstudio.com/items?itemName=matepek.vscode-catch2-test-adapter)** (also automatically installs **[Test Explorer UI](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-test-explorer)**)
   
       `code --install-extension matepek.vscode-catch2-test-adapter`
    - **[Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)** and **[Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)**
    
        `code --install-extension ms-python.python`

4. Customize VS Code User settings

    - Open Visual Studio Code (from the applications menu), and access the Preferences search by pressing **Ctrl+Shift+P**. Search for *Preferences: Open User Settings (JSON)*, click to open and add the following code:
```
    {
        "editor.tabSize": 2,
        "editor.showFoldingControls": "always",
        "editor.minimap.showSlider": "always",
        "editor.rulers": [
            80
        ],
        "files.trimTrailingWhitespace": true,
        "files.exclude": {
            "**/Intermediate": false,
            "**/Binaries": true,
            "**/Saved": true,
            "**/Content": true,
            "**/.pytest_cache": true,
            "**/__pycache__": true,
            "**/Plugins/ProjectAirSim/SimLibs": true
        },
        "debug.onTaskErrors": "abort",
        "debug.toolBarLocation": "docked",
        "debug.showBreakpointsInOverviewRuler": true,
        "terminal.integrated.scrollback": 1000000,
    "cmake.configureOnOpen": false,
    "cmake.autoSelectActiveFolder": false,
    "C_Cpp.intelliSenseEngineFallback": "Enabled",
    "C_Cpp.vcpkg.enabled": false,
    "[cpp]": {
        "editor.defaultFormatter": "ms-vscode.cpptools"
    },
    "testMate.cpp.test.executables": "build/**/Debug/**/*test*",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": false,
    "python.linting.flake8Enabled": true,
    "python.linting.flake8CategorySeverity.E": "Information",
    "python.linting.flake8Args": [
        "--max-line-length=88"
    ],
    "[python]": {
        "editor.defaultFormatter": "ms-python.python",
        "editor.rulers": [
            88
        ],
    },
    "python.formatting.provider": "black",
    "python.formatting.blackArgs": [
        "--line-length",
        "88"
    ]
}
```

5. Install Unreal Engine 5.2.1

    a) If you don't have one yet, sign up for an Epic Games account at **[UnrealEngine.com](https://www.unrealengine.com)** (press **Sign In** in the top right hand corner) and register your GitHub ID with Epic using **[these instructions](https://www.unrealengine.com/ue4-on-github)**.

    b) Once registered, get the Unreal Engine 5.2.1 source from **[Unreal Engine's private GitHub repo](https://github.com/EpicGames/UnrealEngine)**. The filename should be `UnrealEngine-5.2.1-release.tar.gz`. Extract it into your home directory.

    c) Make an installed build of Unreal Engine, using these steps:

        
        cd ~/UnrealEngine-5.2.1-release

        ./Setup.sh

        rm -rf Engine/Platforms/*

        ./Engine/Build/BatchFiles/RunUAT.sh BuildGraph \
            -target="Make Installed Build Linux" \
            -script=Engine/Build/InstalledEngineBuild.xml \
            -set:HostPlatformOnly=true \
            -set:WithLinuxAArch64=false \
            -set:WithFullDebugInfo=false \
            -set:WithDDC=true \
            -set:GameConfigurations="DebugGame;Development"
        

    *Note: Making an installed build will take a **lot of disk space (~200 GB)** and a **lot of time (~4+ hours)** because it will build the engine and precompile the engine's **[DDC](https://dev.epicgames.com/documentation/en-us/unreal-engine/derived-data-cache?application_version=5.2)** content. However, using the installed build for Project AirSim development can save a lot of developer iteration time since the engine will never recompile and the bulk of the initial shader compilation will already be complete. This is similar to using a Windows binary engine downloaded from Epic's Launcher. Most of the original source package and build files can also be deleted to just keep the installed build version which is ~40 GB.*

    d) After the installed build completes, the binary version of the engine will be located in `~/UnrealEngine-5.2.1-release/LocalBuilds/Engine/Linux/` and the contents of this folder can be moved to wherever you plan to use the engine from. The other contents of the `UnrealEngine-5.2.1-release` directory can be deleted since the installed build engine will not need to rebuild again.

   ```
   mv ./LocalBuilds/Engine/Linux ~/UE_installed_build
   cd ~/UE_installed_build
   rm -rf ~/UnrealEngine-5.2.1-release
   ```

6. Set a `UE_ROOT` environment variable to wherever you put the built Unreal Engine (you can add this to the bottom of `~/.bashrc` to make it permanent)

    ```
    export UE_ROOT=/home/[user]/UE_installed_build
    ```

## Project AirSim Client Setup

An up-to-date Ubuntu 22.04.5 LTS installation comes with Python 3.10.12. Project AirSim requires a Python virtual environment to be configured.

1. Install Python 3 virtual environment support to your system:  

        sudo apt install python3-dev python3-venv

2. Activate the Python environment to use with Project AirSim.

    a) Create a new `virtualenv` environment (here named `airsim-venv` but you may choose any convenient name):

        python -m venv ~/airsim-venv

    b) Activate your environment:

        source ~/airsim-venv/bin/activate

    c) Install basic tools for setting up other pip packages in the activated environment:

        python -m pip install --upgrade pip
        python -m pip install setuptools wheel

    Developers must also install the cmake package:

       python -m pip install cmake

3. Install the Project AirSim Python client library:

        cd ~/projectairsim
        python -m pip install -e client/python/projectairsim
    
    If you get this error:

       Error: Could not find a version that satisfies the requirement open3d

    most likely your virtual environment is using a 32-bit build or an unsupported version of Python. In either case, delete and rebuild the virtual environment (see step 2A) using a supported 64-bit version of Python.

---

## Build Project AirSim From Source as a Developer

From the ~/projectairsim folder, run the `build.sh` shell script.

### build.sh Options

`build.sh {target from below}`

```
all = Clean + Build + Test + Package everything
clean = Clean sim libs + Blocks build files

simlibs_debug = Build + Package sim libs for Debug
simlibs_release = Build + Package sim libs for Release
test_simlibs_debug = Test sim libs for Debug
test_simlibs_release = Test sim libs for Release

blocks_debuggame = Build Plugin + Blocks for DebugGame (uses Debug sim libs)
blocks_development = Build Plugin + Blocks for Development (uses Release sim libs)

package_plugin = Package Project AirSim UE Plugin for Debug + Release
package_blocks_debuggame = Package stand-alone Blocks environment executable for DebugGame
package_blocks_development = Package stand-alone Blocks environment executable for Development
```
### Building using the Unreal Engine Installed Build

Running any of the `build.sh` options apart from `build.sh all` and and `build.sh package_plugin` is possible using the Unreal Engine Installed Build completed in the Initial Setup steps. This is because the Unreal Engine build was compiled using the flag `-set:GameConfigurations="DebugGame;Development"`. Simply run the options that are required from the command line.

#### Performing `build.sh all` and `build.sh package_plugin` *(not required, for reference only)*

This step is not required to be run, but is here for information purposes only. `build.sh all` and `build.sh package_plugin` will perform a Shipping level build as part of its script and requires the pre-built source downloaded from the Unreal Engine site. Building it using the compiled source code will result in an error on compilation:

`Targets cannot be built in the Shipping configuration with this engine distribution.`

To perform a full Project AirSim build:

1. Download the `Linux_Unreal_Engine_5.2.1.zip` file from **[Unreal Engine](https://www.unrealengine.com/en-US/linux)** and extract it into your home directory.
2. Set a `UE_ROOT` environment variable to where you extracted the built Unreal Engine (you can add this to the bottom of `~/.bashrc` to make it permanent)

    ```
    export UE_ROOT=/home/[user]/Linux_Unreal_Engine_5.2.1
    ```

### Project AirSim Component Locations

The sim lib components and unit test executables are built in the `projectairsim/build/` folder using CMake, and are automatically copied to the Unreal Blocks environment folder to be ready for building the plugin.

The Plugin components are built in the `projectairsim/unreal/Blocks/` folder using the Unreal Engine's build system.

The packaged outputs (sim lib components, UE Plugin, Blocks stand-alone executable) are copied to the `projectairsim/packages/` folder, where they can be picked up for external uses.

The Python client files are in the `projectairsim/client/python/` folder, with the Project AirSim Client Library in the `projectairsim/client/python/projectairsim` sub-folder.

### Checking Client Operation

After installation, you can check if the client works by launching Project AirSim and running one of the client demo scripts such as `hello_drone.py` with the Project AirSim Python environment activated:

```
<from activated Python environment with Project AirSim running>

python hello_drone.py
```

In your own custom client scripts, just import the client for your robot type (ex. Drone) from the Project AirSim Python client library:

``` python
from projectairsim import ProjectAirSimClient, Drone, World
```

For more details about using the client to connect, send/receive signals, etc, see the `hello_drone.py` example script contents or the specific **[API documentation](https://github.com/iamaisim/ProjectAirSim/blob/main/docs/api.md)** for your simulation scenario.

---
## Working with Project Files

### Opening the Blocks Project in Unreal Editor

Open Blocks in the Unreal Editor with this command:

`~/UE_installed_build/Engine/Binaries/Linux/UnrealEditor ~/ProjectAirSim/unreal/Blocks/Blocks.uproject`

### Generating the Visual Studio Code Workspace File

Run the following script:

`bash ~/ProjectAirSim/unreal/Blocks/blocks_genprojfiles_vscode.sh`

Since some parts of the sim libs haven't been built yet, there may be some warning outputs in yellow text, but it is ok to ignore them.

After it completes, use VS Code to open the `~/ProjectAirSim/unreal/Blocks/Blocks.code-workspace` file that was generated.

---

Sourced from official Project AirSim doco located [here](https://github.com/iamaisim/ProjectAirSim/blob/main/docs/development/dev_setup_linux.md).
