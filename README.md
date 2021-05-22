# NVIDIA OPTIMUS LINUX

[![19-15-27-22-05-2021-nvidiadingvoorhandleiding.png](https://i.postimg.cc/rmD5nbP8/19-15-27-22-05-2021-nvidiadingvoorhandleiding.png)](https://postimg.cc/qhdtgmrY)


This is a repository is mainly focused about the (OLDER) NVIDIA OPTIMUS laptops, that utilizes a discrete NVIDIA video/graphics card like the NVIDIA 740M.
And is made for people who have issues with their laptops running Linux distributions like; Fedora, Ubuntu, Pop!_OS and others.

Targeted at people to assist and aid in the issues that they are experiencing with their particular laptop and isn't meant as a one-size-fits-all "solution".

This all done by Guillermo and made for myself as future reference.

Credits don't go to me, but to the respectively user(s), group, website, company and anyone who is involved in this.


## issues

The issue(s) consists of:

- "The screen freezes after a resume."
- "Blank screen after resume."
- "NVIDIA HDMI audio errors after resume: snd_hda_intel: spurious response last cmd" (https://forums.developer.nvidia.com/t/nvidia-hdmi-audio-errors-after-resume-snd-hda-intel-spurious-response-last-cmd/149003/21)
- Kernel 5.6: system freeze when resuming from suspend or hibernate (https://forums.developer.nvidia.com/t/kernel-5-6-system-freeze-when-resuming-from-suspend-or-hibernate/121630) ~ thesourcehim on the NVIDIA forum.
- Blackscreen after Fedora 32, 33, 34 etc installation ~ guillermo

---

#### Work-around Kernel 5.6: system freeze when resuming from suspend or hibernate (https://forums.developer.nvidia.com/t/kernel-5-6-system-freeze-when-resuming-from-suspend-or-hibernate/121630) ~ thesourcehim on the NVIDIA forum.

```
Thank you, @generix, for pointing me in the right direction! I added the simple udev rule to remove problematic device completely:

cat /etc/udev/rules.d/10-remove-nvidia-audio.rules
ACTION==“add”, KERNEL==“0000:01:00.1”, SUBSYSTEM==“pci”, RUN+="/bin/sh -c ‘echo 1 > /sys/bus/pci/devices/0000:01:00.1/remove’"

Now lspci doesn’t list the device. And the problem is gone! Laptop wakes up perfectly!

```
https://forums.developer.nvidia.com/t/kernel-5-6-system-freeze-when-resuming-from-suspend-or-hibernate/121630/21

```
ince the log contains hdaudioC1D0 I tried to disable the Audio device GM204
HDA (device ID: 0x0fbb, vendor ID: 0x10de). First I tried to blacklist the modules
snd_hda_codec_realtek and snd_hda_codec_generic with the following kernel parameters:
modprobe.blacklist=snd_hda_codec_realtek modprobe.blacklist=snd_hda_codec_generic

This did not work, since the device was still detected as snd_hda_codec_generic.

To disable the device I used `lspci | grep Audio` to determine the PCI device address.
With the command `echo 1 > /sys/bus/pci/devices/0000:01:00.1/remove` I removed the PCI device.
The first time this worked and I could power off without issues.
The second time, the ZSH shell got frozen and was not interuptable. The remove file was gone.
I tried to shutdown, but the system got a similar error log, but with ZSH.

The disable the device permanently I created a udev rule /etc/udev/rules.d/01-disable-nivida-hda.rules:
```
ACTION=="add", KERNEL=="0000:01:00.1", SUBSYSTEM=="pci", RUN+="/bin/sh -c 'echo 1 > /sys/bus/pci/devices/0000:01:00.1/remove'"
```

I tried 3 shutdowns and all shutdowns finished properly.
The only drawback of this workaround is, that the Nvidia HDA Audio device is removed.
Since my built in speaker is wired to the Intel audio device, I have no issues with playing audio.
```

https://bugs.archlinux.org/task/63697


https://forum.cockos.com/archive/index.php/t-214537.html


##### udev rules in Linux

https://linuxconfig.org/tutorial-on-how-to-write-basic-udev-rules-in-linux

---

## Check Video encoding NVIDIA

https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new


#### Blackscreen after Fedora 32, 33 and 34 installation ~ guillermo

```
# Log in - open a terminal and execute : sudo vi /etc/default/grub
# Add nouveau.modeset=0 to the line GRUB_CMDLINE_LINUX="rhgb quiet",
# so that it reads : GRUB_CMDLINE_LINUX="rhgb quiet nouveau.modeset=0"
# sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg (Fedora installation in EFI mode)
# sudo grub2-mkconfig -o /boot/grub2/grub.cfg (Fedora installation in legacy BIOS mode)
# https://unix.stackexchange.com/questions/286150/fedora-installation-nouveau-issue

```


## Troubleshooting and aid

#### If you have a problem, PLEASE read this first

If you have a problem, and would like assistance, please follow these steps:

Please always include a detailed description of the problem (both when starting a new thread and when posting to existing threads), as distinct problems can have nearly identical symptoms (e.g. black screens, hung X servers, …). Reliable reproduction steps are especially useful. If you post a reply reading “I have the same problem” in response to another user’s bug report without the information requested in this post, you may make it more difficult for NVIDIA to track and diagnose the problem.

Please always include a copy of an `nvidia-bug-report.log.gz` file, which can be generated with the `nvidia-bug-report.sh` script shipped with the NVIDIA Linux/FreeBSD graphics drivers and installed in your PATH; the log file will be placed in the current working directory. Attach the log file by clicking the Upload upload button in the post composition window.

To make sure this log file includes as much relevant information as possible, please start the X server with `startx -- -logverbose 6` and run `nvidia-bug-report.sh` after the problem has occurred. If X can not be started or the machine appears to have crashed, please check if you can log into it remotely (e.g. via ssh) and run `nvidia-bug-report.sh` in the remote shell, if possible.

If the machine can not be logged into remotely, please run the bug report script with X running (this will ensure that data points only available when the device is initialized, such as the VBIOS revision, are captured in the log file). All requests for assistance from NVIDIA must include the bug report, per the instructions above.

If `nvidia-bug-report.sh` appears to hang, it may still have collected enough useful information about your problem. In this case, please provide the partial log file generated, if any.

If you have collected additional information, but are not sure if it is relevant / appropriate, please attach it anyway.

When creating a new thread, please make the thread subject as descriptive as possible. For example,


    Good title: "X server 1.2 crashes when Kopete opens a popup notification on GeForce 8800 GTX"
    Bad title: "The X server keeps crashing"
    Really bad title: "Help me!"

#### Journalctl

https://www.loggly.com/ultimate-guide/using-journalctl/

#### Find out information about your GPU

`hwinfo --gfxcard --short`

`sudo lshw -C display`

#### Verify your NVIDIA GPU

`nvidia-smi`

#### Troubleshooting power management

https://wiki.archlinux.org/title/Power_management#Suspend/resume_service_files

#### How to Choose the Proper ACPI Kernel Argument from the Linux Manjaro team: Linesma https://forum.manjaro.org/t/how-to-choose-the-proper-acpi-kernel-argument/1405

