# Aspire V5-471G Linux Issue Fixes
This repository contains explanation of problems I countered when using several Linux distros on Acer Aspire V5-471G, including Ubuntu, Linux Mint, LMDE, Zorin OS, Debian, antiX, MXLinux, elementary OS, Lubuntu, Xubuntu, and Q4OS. The problems still exist even on LTS kernel. The problems doesn't exist on Windows. This repository also contains explanation why these problems exist, how to fix it (workaround), and side effects of the fixes.

## Acer Aspire V5-471G Specification
before going to the main topic, here is the detailed specification of the device:
- CPU	: Intel(R) Core(TM) i7-3517U 1.90GHz
- iGPU	: Intel(R) HD Graphics 4000
- dGPU: Nvidia GeForce GT620M
- System BIOS Version	: V1.22 Phoenix (updated from 1.14)
- VGA BIOS Version	: 70.08.B1.00.07
- Ports: 2 × USB 2.0, USB 3.0, SD Card Reader, HDMI, Audio Jack, DC Jack

## Problems
There are two problems:
- Random reboot

  The system will reboot randomly whenever it is, right after the login screen appear, in the middle of work, or right before       shutdown. The Reboot happen suddenly, no warning, no logs, no kernel panics, just boom, black screen, then the Acer logo appear.
- Reboot instead shutdown

  The system will boot again after shutdown. No matter what the method is, `sudo systemctl poweroff`, `sudo shutdown -h now`, or    simply through GUI, every methods does the same.
## Causes
For random reboot, it's:
- CPU C-State

  When the CPU enter this state, the system become unstable. Then, if CPU want to leave this state, the system don't do it          properly. Resulting some electric spike or drop (i'm not sure). That's why there is no warning, no logs, no panics. When the      system reboot without logs, there must be something faster than the system, this thing makes the system reboot before the         system itself can write some logs, it's electricity.

For reboot instead shutdown, it's:
- USB 3.0 Port

  I relieze that when something is connected to this port, the system can shutdown perfectly. it could be USB stick, wireless       mouse dongle, Wi-Fi dongle, or phone through USB cable.
## Fixes
The fixes are simple, it's just kernel parameter:
- Random reboot

  Add `intel_idle.max_cstate=1` and `processor.max_cstate=1` to your grub.
- Reboot instead shutdown

  Add `usbcore.autosuspend=-1` to your grub. No need to plug anything to the USB 3.0 port.

So, it will be something like this:
- `GRUB_CMDLINE_LINUX_DEFAULT="intel_idle.max_cstate=1 processor.max_cstate=1 usbcore.autosuspend=-1"`
## Side Effects
The side effect of two kernel parameters before are:
- Higher power consumption

  The `intel_idle.max_cstate=1` and `processor.max_cstate=1` parameters force the CPU to enter the C-State only to the first        level. There are 10 levels of C-State (if i'm not mistaken), level 10 is the slowest C-State the CPU can enter. The               `usbcore.autosuspend=-1` parameter force the USB ports to stay awake, no suspend, so these ports only turned off when the         system is also off.
