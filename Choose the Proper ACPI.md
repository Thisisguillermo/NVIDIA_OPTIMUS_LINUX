# From the Linux Manjaro team: Linesma https://forum.manjaro.org/t/how-to-choose-the-proper-acpi-kernel-argument/1405

The introduction of Optimus laptops have created new challenges for Linux users. One of the biggest issues is having the proprietary graphics card work when the system boots. Many times the advice given is to add an Advanced Configuration and Power Interface (acpi ) kernel parameter to grub. While a specific acpi kernel parameter may be given, it is sometimes not appropriate to the hardware it is being applied to. This can cause system instability and some functions of the laptop to not work properly. Once the user does find a parameter that works, they are told to add it to their grub config file so it will be automatically load at boot. This can cause its own problems. If the user does not use the proper syntax when adding this parameter, it will be ignored and not work. Let’s take a look at how to choose the proper acpi kernel parameter and then how to properly add it to the grub config file.

Before you decide which acpi kernel parameter you need, you have to ask yourself one question.

*Do I even need an acpi kernel parameter?*

Short of digging through logs and looking for acpi errors, there are several ways to see if an acpi kernel parameter is needed.

1. This is probably the easiest way to tell if you need an acpi kernel parameter. This problem can be seen whether you are running Linux from a Live USB or it is installed to a local hard drive. When you shut down your computer and it hangs before shutting down, or it hangs and you have to push the power button to shut it down, you may need an acpi kernel parameter.
2. You are using an Optimus laptop and you have installed the nVidia proprietary driver. You boot your laptop and it hangs on the following stepsa. “Started TLP system startup/shutdown”
b. “reached target graphical interface”
c. An error message about configuring “Backlight”
d. Your laptop just boots to a black screen and the fans start to run constantly.

If any of these behaviors are noticed, you may need to add the acpi kernel parameter.

3. This requires a little more effort on the user. Press “E” on the grub screen and remove the “quiet” kernel parameter before you boot your computer. As your computer boots, you will be presented with a list of what is being loaded. Before the kernel starts loading, it does a quick hardware check. If it has a problem with powering up any hardware, it will list an acpi error. If you see an acpi error, you may need an acpi kernel parameter.

*Choosing an acpi kernel parameter.*

The bios or UEFI in use by your laptop looks for certain “identifiers” in how to handle your hardware based on the Operating System (OS) it was designed to operate. It does this through acpi “calls” from the operating system to the bios. When you see problems such as listed above, that means the bios does not understand the calls being sent to it by the OS. The good thing is, since acpi used by both Windows and Linux follow the UEFI specifications ¹, they both use the same OSI strings to identify what OS is used. Unlike the Windows kernel, the Linux kernel can determine what “power calls” are sent to the bios by the use of these OSI Strings in the acpi kernel parameter. By adding these parameters, you can basically tell Linux to “mimic” the acpi calls sent by another OS.

When you purchased your laptop, it probably came with Windows pre-installed. The version of Windows that was installed is your first clue in figuring out which OSI String your acpi kernel parameter needs. From Windows 2000 to Windows 8.1, you can use the Windows release name to find the OSI String you should use. Windows 10, however, since it uses a “rolling release” model, finding the OSI String you need to use requires a little more work. To find your Windows install’s version number, open a command prompt in Windows and type, winver . This will give your Windows version in the following manner, Windows 10 version 1607 . That is the information you need to determine the OSI String your kernel parameter needs.

Once you have your version of Windows, use the below chart to find how your version of Windows needs to be represented in the kernel parameter.

| Windows| Description |
| -------------|------------------|
| Windows 2000 |	Windows 2000                        |
| Windows 2001 |	Windows XP  |
| Windows 2001 | SP1	Windows XP SP1  |
| Windows 2001 |	Windows Server 2003 |
| Windows 2001 | SP2	Windows XP SP2  |
| Windows 2001 | SP1	Windows Server 2003 SP1 |
| Windows 2006 |	Windows Vista |
| Windows 2006 | SP1	Windows Vista SP1 |
| Windows 2006 |	Windows Server 2008 |
| Windows 2009 |	Windows 7, Win Server 2008 R2 |
| Windows 2012 |	Windows 8, Win Server 2012  |
| Windows 2013 |	Windows 8.1 |
| Windows 2015 |	Windows 10  |
| Windows 2016 |	Windows 10, version 1607  |
| Windows 2017 |	Windows 10, version 1703  |
| Windows 2017 |	Windows 10, version 1709  |
| Windows 2018 |	Windows 10, version 1803  |
| Windows 2018 |	Windows 10, version 1809  |
| Windows 2019 |	Windows 10, version 1903  |
| Windows 2020 |	Windows 10, version 2004  |

Once you have your OSI string, you now have all the information needed for your kernel parameter.

To have Linux mimic your version of Windows, you need to add the following kernel parameter,
`acpi_osi='OSI String'`

### Examples

Windows 7 - `acpi_osi='Windows 2009'`

Windows 10 ver. 1709 – `acpi_osi='Windows 2017'`

You will notice that Windows 10 version 1709 has an OSI String of Windows 2017.2 and I used Windows 2017 instead. In my testing, I found that Windows 2017.2 was not recognized as a valid argument.

Note on acpi_osi=!

This argument disables all vendor strings that maybe present. It should only be used if one of the above OSI strings does not work on its own. If you use it when it is not needed, you maybe able to boot without any acpi errors, but your touchpad or wifi will not work. It also must be used in combination one of the above OSI strings.

**Example**

`acpi_osi=! acpi_osi='Windows 2012'`

This will disable all the vendor strings and then tell the kernel to “mimic” Windows 8 when it talks to the bios.

*For Asus laptops by guillermo*

http://manpages.ubuntu.com/manpages/trusty/man4/acpi_asus.4freebsd.html

```
The acpi_asus driver provides support for the extra ACPI-controlled gadgets, such as hotkeys
     and leds, found on recent Asus (and Medion) laptops.  It allows one to use the sysctl(8)
     interface to manipulate the brightness of the LCD panel and the display output state.
     Hotkey events are passed to devd(8) for easy handling in userspace with the default
     configuration in /etc/devd/asus.conf.

 ```

*Note for Dell Laptops*

Sometimes the above kernel parameters will not work properly on some Dell laptops. If that is the case, you can try the following: acpi_rev_override=# Replace the “#” with a number between 1 to 5. In order to have this kernel parameter applied properly, cold booting (shutting your system down completely before restarting) your laptop twice may be required.

*Adding the acpi kernel to grub*

Once you have found the kernel argument that works the best for your hardware, you need to add it to grub in order to have it applied every time you boot your system. This is easier than it sounds. The problem most users have is syntax or how to type it on the appropriate line. They key to having the argument recognized is to only use “single quotes”, or as we call them in the United States, the apostrophe.

To add the argument to grub, open a terminal, and type the following:

1. `sudo nano /etc/default/grub`
2. Add the kernel argument to the following line: GRUB_CMDLINE_LINUX_DEFAULT .

**Example** from

```
GRUB_CMDLINE_LINUX_DEFAULT="acpi_osi='Windows 2018' rd.udev.log-priority=3 bootsplash.bootfile=bootsplash-themes/manjaro/bootsplash nvme_core.default_ps_max_latency_us=5500"
```

3. Ctrl+X to exit and Y to save.
4. `sudo update-grub`

Now your kernel argument has been added to grub and will be loaded every time you boot your laptop.

This is by no means an exhaustive guide in the use of acpi kernel parameters. That would require a much longer document. Instead it gives some basic instructions that one can follow to get their system up and running. If you are still having acpi issues, and need help solving it, there are a couple of options open to you. First off, search the forum to see if your problem has been solved before. If that does not solve your issue, create a new support thread on the forum discussing your issue.

**Additional Notes**

I will use this section for minor updates and additional information.

The kernel parameter apci_osi=Linux can be used in newer (late 2019 and later) Asus laptops to fix an issue where the external HDMI port is not working. Use this one instead of a Windows related parameter.

---

Sources:

¹ https://uefi.org/specifications 10

² https://docs.microsoft.com/en-us/windows-hardware/drivers/acpi/winacpi-osi 18

³ https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt 25

Originally created by @linesma on archived forum