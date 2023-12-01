## Force-enable GPU patching 
> **System Requirments**: macOS 12 or newer<br>
> **OCLP**: 0.6.9 or newer

In some cases, discrete GPUs might not be detected by OpenCore Legacy Patcher because their device-id or IOName is not in the list of supported GPUs used in real Macs. In this case, you need to force-enable GPU patching in the configuration file of the OpenCore Lagacy Patcher Source Code since the GUI of the app does not include settings to enable GPU patching manually (for probably good reasons).

> [!CAUTION]
> 
> If your system does not have any of the GPUs supported by OCLP installed on your system but apply any of the GPU patches in this section anyways, you will brick the installation and have to [revert root patches](https://github.com/5T33Z0/OC-Little-Translated/blob/main/14_OCLP_Wintel/Guides/Reverting_Root_Patches.md) in order to recover macOS!  

### Force NVIDIA Kepler Patching
To force enable patching of **NVIDIA Kepler Cards** (GT(X) 7xx Series) in OCLP, do the following:

- Download the **OCLP** [Source Code](https://github.com/dortania/OpenCore-Legacy-Patcher) and unzip it
- Run Terminal and enter the following commands (line by line):
    ```shell
    cd ~/Downloads/OpenCore-Legacy-Patcher-main
    pip3 install -r requirements.txt
    ```
- Wait unti the download of the pip3 stuff has finished
- In Finder, navigate to "Downloads/OpenCore-Legacy-Patcher-main"
- Double-click on `Build-Binary.command` &rarr; It will open another Terminal window and download `payloads.dmg` (≈46 MB) and `Universal-Bibaries.dmg`(≈522 MB). These are required so patching won't fail.
- Once the download is complete, navigate to `/Downloads/OpenCore-Legacy-Patcher-main/resources/sys_patch/`
- Open `sys_patch_detect.py` with IDLE, TextEdit, Visual Studio Code or Xcode
- Under **"# GPU Patch Detection"**, change the following setting from `False` to `True`:
	- **`self.kepler_gpu = True`**
 	- Save and close the .py file 
- Back in Finder Double-click on `OpenCore-Patcher-GUI.command` to run the Patcher App.
- Click on "Post-Install Root Patch". The option "Graphics: Nvidia Kepler" should now appear in the list of applicable patches:<br>![KEPLER](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/56441ba0-73f9-46e0-a4d2-7ae320f2f551)
- Start Patching. 
- Once it's done, reboot

Enjoy working GPU Acceleration again!

### Force NVIDIA Tesler Patching
&rarr; Work in Progress