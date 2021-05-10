# Hackintosh-ThinkPad-E14
Files required for prepping a Hackintosh on ThinkPad E14. 

## Method Used
Dortania's OpenCore Install Guide - https://dortania.github.io/OpenCore-Install-Guide/

## First attempt 
- EFI bootable USB created. Followed steps from above guide thoroughly. 
	
		### SETUP PROBLEM 1: Laptop won't boot from USB. 	
		Solution 
		- Turn off Fast Start-up from Control Panel\Hardware and Sound\Power Options\System Settings.
		- While booting, Lenovo logo will pop up. Hit F12 for boot options. 
		- In boot options, click on USB boot. 

- Booted into macOS recovery. Format hard disk partition and install macOS Big Sur from the internet (Note: only Ethernet connection works)

		### SETUP PROBLEM 2: Mistakenly formatted macOS installation partition on HDD to macOS Journaled (Extended). Problem while booting from hard drive. 
		Solution
		- Only Sierra and High Sierra require macOS Journaled (Extended) formatting. All other macOS versions require APFS. 
		- Reformatted the hard drive partition to APFS and reinstalled macOS Big Sur on HDD. 

- Restarted after installation. Booted from USB again and selected "Install macOS Big Sur on HDD" from OpenCore boot options. No problems at this stage. 
- While installation, computer restarted several times. When restart occurs, note that you have to hit F12 and boot from USB every time. I later changed my boot order to boot from USB as the first option.  
- After installation complete, macOS setup initiated. Setup was completed successfully with ethernet connection. Apple ID was signed in successfully. 

## Problems after first successful bootup
- [x] Graphics are too poor. iGPU is most probably not getting detected. In System Info, it's showing "Graphics 7MB" which surely is a problem. 
- [x] Backlight control not working.
- [x] Touchpad left-click is not working: instead, when left-click is hit, OS thinks it's a force touch (it's super annoying). Also, touch-clicks are not working. Only press-clicks are working. Gestures seem to be working fine. (This got fixed mysteriously after battery patch)
- [x] Audio (both, speakers and headphone jack) is not working. 
- [x] Wi-Fi is not working. 
- [x] Bluetooth is not working. 
- [x] Battery management not showing up. 
- [x] Webcam is not getting detected. 
- [ ] Sleep/Wake problem. When I press the power button, nothing happens. When I click on the Sleep option from the Menu Bar, the desktop goes off, the power button blinks once, Bluetooth disconnects and connects again, nothing happens then (the power button light stays on). Now if I press the power button, the system wakes up but my touchpad stops working. Even when I close the lid, only Music playback stops but the power button doesn't start blinking. Only after 15 minutes, if I leave the computer idle, it goes to sleep - the power button starts blinking. When I press the power button, the system wakes up and starts working normally.
- [ ] HDMI is not working. Nothing happens when HDMI is plugged in.

Other problems: Too much time for booting up (almost 3-4 minutes). 

P. S. Checklist is added to denote live debugging process. Ticked items are successfully debugged items. Details about debugging is mentioned in this README file itself. 

## Working things after first successful bootup
- USB mouse is working. USB drives are getting detected. 
- Keyboard is working. 
- iServices working. 
- Ethernet is working. 

## Successful Wi-Fi Patch
- **Debugging:** AirportItlwm.kext was in a subfolder which ProperTree apparently could not get in the snapshot. As a result, the config.plist file did not have AirportItlwm.kext in the Kernels. 
- **Solution:** After moving the AirportItlwm.kext to the "EFI/OC/Kexts" folder, updated the config.plist by a clean snapshot. Solution worked. Wi-Fi access points were visible. Could connect to the internet through a 5GHz network. However, internet gets disconnected after connected for about half an hour or so. Turning off Wi-Fi and turning it on works rarely. Changing Wi-Fi network temporarily to another network and switching it back to the first one worked.  

## Successful iGPU and Backlight Patch
- **Debugging for iGPU not detecting:** iGPU Intel (U)HD 620 needs device-id faking in "DeviceProperties" section of config.plist. Apart from this, config.plist lacked "PciRoot(0x0)/Pci(0x2,0x0)" child under "DeviceProperties/Add" as it wasn't there in Sample.plist. 
- **Solution:** The device-id and other information was filled according to this section of the OpenCore install guide - https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#deviceproperties. "PciRoot(0x0)/Pci(0x2,0x0)" was added as a child to "DeviceProperties/Add".
- **Debugging for backlight problem:** The order of SSDT was incorrect. 
- **Solution:** Solving the DeviceProperties bug solved this problem too. 

## Successful sound patch
- **Debugging for sound problem:** Sound card details were not identified. Hence, sound card ID and layout-id was not known. 
- **Solution:** 
	1. Testing layout-id:
		1. Ran Live Ubunto distro on computer through USB. 
		2. In terminal, ran `cat /proc/asound/card0/codec#0`. 
		3. Noted down the `Codec: Conexant CX8070`. 
		4. Went to https://github.com/acidanthera/AppleALC/wiki/Supported-codecs and noted down the layout-id for CX8070 vendor. 
		5. Then, updated `alcid=15` under "NVRAM" section of config.plist. 
		6. Reboot system and check sound. 
	2. Permanently fixing audio:
		1. Follow instructions mentioned here - https://dortania.github.io/OpenCore-Post-Install/universal/audio.html#making-layout-id-more-permanent.
		2. While removing `boot-arg` in config.plist, keep `-v debug=0x100 keepsyms=1`. Remove `alcid=xxx`.

## Successful Bluetooth and Webcam patch
- **Debugging for BT and Webcam problem:** All USB ports were not being recognised for some reason. Saw that using https://github.com/corpnewt/USBMap.
- **Solution:** Refered https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html. Thanks to https://github.com/seven-of-eleven/Lenovo-ThinkPad-L13-Yoga-Hackintosh#post-install. I was able to relate to the problem and solve it. 
	1. Added https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-RHUB.aml to ACPI folder and updated config.plist using OC-Screenshot. Rebooted the system and found out all USB ports were getting detected on USBMap tool. 
	2. Created USBMap.kext using USBMap tool as instructed on the Dortania guide. Added this to `EFI/OC/Kexts` and deleted USBInjectAll.kext. Updated the config.plist using OC-Screenshot. Rebooted. Problem solved. 

## Successful battery patch
- **Debugging:** Battery monitoring requires DSDT patching.
- **Solution:** Refered this Reddit thread - https://www.reddit.com/r/hackintosh/comments/m2sz5b/thinkpad_e15_battery_indicator_not_working_after/?utm_source=share&utm_medium=web2x&context=3
	1. Downloaded SSDT-BAT.aml and config.plist from the https://drive.google.com/drive/folders/1Oc5RFBLt3o3n8GT9-EAgDL_VXyqRiWqj?usp=sharing. 
	2. Pasted SSDT-BAT.aml in `EFI/OC/ACPI` of the USB Flash Drive. Copied DSDT patches from downloaded config.plist and pasted them in my config.plist file located on the USB Flash Drive using Text Editor. 
	3. Rebooted. Problem was fixed. Mysteriously, even TrackPad gestures problem was solved. 

## Cosmetic changes
- Followed instructions as per https://dortania.github.io/OpenCore-Post-Install/cosmetic/verbose.html#macos-decluttering.
