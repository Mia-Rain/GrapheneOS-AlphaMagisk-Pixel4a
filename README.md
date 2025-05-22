# GrapheneOS-AlphaMagisk-Pixel4a (GrapheneOS++)

## Description

GrapheneOS-AlphaMagisk-Pixel4a is a `shell` script designed to patch GrapheneOS OTA (Over The Air) images with custom modules, providing additional features. This tool relies heavily on upstream projects for its functionality.

## Features

- [BCR](https://github.com/chenxiaolong/BCR)
- [Custota](https://github.com/chenxiaolong/Custota)
- [Magisk](https://github.com/pixincreate/Magisk)
- [MSD](https://github.com/chenxiaolong/MSD)
- [OEMUnlockOnBoot](https://github.com/chenxiaolong/OEMUnlockOnBoot)
- [AlterInstaller](https://github.com/chenxiaolong/AlterInstalle)

> [!NOTE]
>
> 1. This project is not affiliated with GrapheneOS or any of the mentioned projects. It is a personal project for personal use.
> 2. Currently, the project only supports Linux due to compatibility issues with other operating systems (`libsepol` is highly Linux-specific).

## Requirements

To use this project, you need the following (most dependencies will be handled by the script, except for `git` and `python`):

- A Linux machine is recommended (Needed for running a statically-linked Android executable). `WSL` (Windows Subsystem for Linux) or a `VM` (Virtual Machine) can also be used instead
- Tools (must be in the path):
  - `afsr` (>= version 1.0.2)
  - `avbroot` (>= version 3.12.0)
  - `my_avbroot_setup` (>= commit `16636c`)
  - `custota-tool` (>= version 5.2)
  - `git`
  - `Magisk` (>= version 27006 -- optional)
  - `python`
- Modules:
  - `AlterInstaller` (>= version 2.0)
  - `BCR` (>= version 1.65)
  - `Charge Limit`
  - `Custota` (>= version 5.2)
  - `MSD` (>= version 1.8)
  - `OEMUnlockOnBoot` (>= version 1.1)
- Dependencies:
  - `e2fsprogs`
  - `pkg-config`
  - `tomlkit` (Python dependency)
  - `pydantic` (Python dependency)

## Patch
The patched OTA is released and available in the [releases section](https://github.com/Mia-Rain/GrapheneOS-AlphaMagisk-Pixel4a/releases).

## Usage

### Getting Started

Reading the [AVBRoot docs](https://github.com/chenxiaolong/AVBRoot) is essential before proceeding with GrapheneOS-AlphaMagisk-Pixel4a.

1. Ensure the device has an unpatched version of GrapheneOS installed. The version must match the one from GrapheneOS-AlphaMagisk-Pixel4a. It is important to make sure that the version installed matches the version on GrapheneOS-AlphaMagisk-Pixel4a
2. Start with a version before the latest to ensure OTA functionality.

> [!IMPORTANT] > `Factory image` and `OTA image` are different. AVBRoot is meant to deal with **OTA images**. So does GrapheneOS-AlphaMagisk-Pixel4a.

### Detailed Instructions

#### Web Install

It is easier to use the web installer to flash GrapheneOS. However, it is recommended to use the manual method since it makes it possible to install an older version of GrapheneOS unlike the web installer which always installs the latest version.

- Use the [web installer](https://grapheneos.org/install/web) to install GrapheneOS
- Once installed, **do not** re-lock the bootloader by clicking `Lock bootloader` under the `Locking the bootloader` section
- Proceed to the [patching section](#patching)

#### Manual Install

1. Ensure Fastboot version is `34` or newer. `35` or above is recommended as older versions are known to have bugs that prevent commands like `fastboot flashall` from running.

   ```shell
   fastboot --version
   ```

2. Reboot into `fastboot` mode and unlock the bootloader if not already. **This will trigger a data wipe.** Ensure data is backed up.

   ```shell
   fastboot flashing unlock
   ```

3. When setting PixeneOS up for the first time, the device must already be running the correct OS. Flash the original unpatched OTA or factory image if needed.

   ```shell
   bsdtar xvf DEVICE_NAME-factory-VERSION.zip # tar on Windows and macOS
   ./flash-all.sh # or .bat on Windows
   ```

4. Proceed to the [patching section](#patching-grapheneos-cooking-pixeneos)

#### Patching

1. Download the [OTA from the releases](https://github.com/pixincreate/PixeneOS/releases). Ensure the version matches the installed version.

   Extract the partition images from the patched OTA that are different from the original.

   ```shell
   avbroot ota extract \
       --input /path/to/ota.zip.patched \
       --directory extracted \
       --fastboot
   ```

   To extract and flash all OS partitions, pass `--all`.

2. Set the `ANDROID_PRODUCT_OUT` environment variable to the directory containing the extracted files.

   For `sh`/`bash`/`zsh` (Linux, macOS, WSL):

   ```shell
   export ANDROID_PRODUCT_OUT=extracted
   ```

   For PowerShell (Windows):

   ```powershell
   $env:ANDROID_PRODUCT_OUT = "extracted"
   ```

   For cmd (Windows):

   ```bat
   set ANDROID_PRODUCT_OUT=extracted
   ```

3. Flash the partition images.

   ```shell
   fastboot flashall --skip-reboot
   ```

   Note: This only flashes the OS partitions. The bootloader and modem/radio partitions are left untouched due to fastboot limitations. If they are not already up to date or if unsure, after fastboot completes, follow the steps in the [updates section](#updates) to sideload the patched OTA once. Sideloading OTAs always ensures that all partitions are up to date.

   Alternatively, for Pixel devices, running `flash-base.sh` from the factory image will also update the bootloader and modem.

4. Set up the custom AVB public key in the bootloader after rebooting from fastbootd to bootloader.

   ```shell
   fastboot reboot-bootloader
   fastboot erase avb_custom_key
   fastboot flash avb_custom_key /path/to/avb_pkmd.bin
   ```

5. **[Optional]** Before locking the bootloader, reboot into Android to confirm proper signing.

   Install the Magisk or KernelSU app and run:

   ```shell
   adb shell su -c 'dmesg | grep libfs_avb'
   ```

   If AVB is working, you should see:

   ```shell
   init: [libfs_avb]Returning avb_handle with status: Success
   ```

6. Reboot into fastboot and lock the bootloader. This will trigger a data wipe.

   ```shell
   fastboot flashing lock
   ```

   Confirm by pressing volume down and then power. Then reboot.

   > [!CAUTION] > **Do not uncheck `OEM unlocking`!**

7. For future updates, see the [updates section](#updates).

### Using Root

Rooting, from security point of view is **not** recommended. But that should not stop a power user from using it.

The version of [Magisk](https://github.com/topjohnwu/Magisk) provided by Topjohnwu does not hold good with GrapheneOS as the developers of Magisk are hostile with GrapheneOS developers and its users. See [7606](https://github.com/topjohnwu/Magisk/pull/7606).

Other forks of Magisk like [pixincreate/Magisk](https://github.com/pixincreate/Magisk), [1q23lyc45/KitsuneMagisk](https://github.com/1q23lyc45/KitsuneMagisk) & [Magisk Alpha](https://t.me/magiskalpha) do exist however, each with different issues.

This fork of PixeneOS uses AlphaMagisk as zygisk with the pixincreate magisk fork has issues with the denylist.
Kitsune Magisk is also a good option but MagiskHide has been made basically useless if you read the AlphaMagisk telegram channel linked above.

> [!NOTE]
> For Magisk preinit, see [Magisk preinit](#magisk-preinit)

### Magisk Preinit

Magisk versions 25211 and newer require a writable partition for storing custom SELinux rules that need to be accessed during early boot stages. This can only be determined on a real device, so avbroot requires the partition to be explicitly specified via `--magisk-preinit-device <name>`. To find the partition name:

1. Extract the boot image from the original/unpatched OTA:

   ```shell
   avbroot ota extract \
       --input /path/to/ota.zip \
       --directory . \
       --boot-only
   ```

2. Patch the boot image via the Magisk app on the target device.

   The Magisk app will print out a line like:

   ```shell
   Pre-init storage partition device ID: <name>
   ```

   Alternatively, run:

   ```shell
   avbroot boot magisk-info \
       --image magisk_patched-*.img
   ```

   The partition name will be shown as `PREINITDEVICE=<name>`.

   Now that the partition name is known, it can be passed to avbroot when patching via `--magisk-preinit-device <name>`. The partition name should be saved somewhere for future reference since it's unlikely to change across Magisk updates.

   If the device is unbootable, patch and flash the OTA once using `--ignore-magisk-warnings`, then repatch and reflash the OTA with `--magisk-preinit-device <name>`.


## Reverting Back to Stock

To revert to stock GrapheneOS or firmware:

1. Reboot into fastboot mode and unlock the bootloader. **This will trigger a data wipe**. Ensure data is backed up.

2. Erase the custom AVB public key:

   ```shell
   fastboot erase avb_custom_key
   ```

3. Flash the stock firmware.

## More information

To know more about the projects used in this repository, refer to the following links:

- [AFSR](https://github.com/chenxiaolong/AFSR)
- [AlterInstaller](https://github.com/chenxiaolong/AlterInstaller)
- [AVBRoot](https://github.com/chenxiaolong/AVBRoot)
- [BCR](https://github.com/chenxiaolong/BCR)
- [ChargeLimit](https://github.com/chenxiaolong/ChargeLimit)
- [Custota](https://github.com/chenxiaolong/Custota)
- [GrapheneOS](https://grapheneos.org)
- [Magisk](https://github.com/pixincreate/Magisk)
- [MSD](https://github.com/chenxiaolong/MSD)
- [OEMUnlockOnBoot](https://github.com/chenxiaolong/OEMUnlockOnBoot)
- [Rooted Graphene](https://github.com/schnatterer/rooted-graphene)

## License

This project is licensed under the `MIT`. For more information, please refer to the [LICENSE](LICENSE) file.
Dependencies are downloaded from their respective repositories and are licensed under their respective licenses. Refer to the respective repositories for more information.

## Credits

- [GrapheneOS](https://grapheneos.org) -- for the OS
- [Chenxiaolong](https://github.com/chenxiaolong) -- for additional features and tools
  - [afsr](https://github.com/chenxiaolong/afsr)
  - [avbroot](https://github.com/chenxiaolong/avbroot)
  - [BCR](https://github.com/chenxiaolong/BCR)
  - [Custota](https://github.com/chenxiaolong/Custota)
  - [MSD](https://github.com/chenxiaolong/MSD)
  - [my-avbroot-setup](https://github.com/chenxiaolong/my-avbroot-setup)
  - [OEMUnlockOnBoot](https://github.com/chenxiaolong/OEMUnlockOnBoot)
- [Rooted-Graphene](https://github.com/schnatterer/rooted-graphene) -- for motivation and inspiration
- [PixeneOS](https://github.com/pixincreate/PixeneOS) -- original project
- [AlphaMagisk](https://t.me/magiskalpha) -- AlphaMagisk

## Disclaimer

THIS SOFTWARE IS PROVIDED "AS IS" AND ANY EXPRESSED OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
