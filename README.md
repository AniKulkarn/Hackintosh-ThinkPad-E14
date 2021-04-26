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
- Graphics are too poor. iGPU is most probably not getting detected. In System Info, it's showing "Graphics 7MB" which surely is a problem. 
- Touchpad left-click is not working. Also, touch-clicks are not working. Only press-clicks are working. Gestures seem to be working fine. 
- Audio (both, speakers and headphone jack) is not working. 
- Wi-Fi is not working. 
- Bluetooth is not working. 
- Battery management not showing up. 
- Too much time for booting up (almost 3-4 minutes). Verbose gets irritating after a while :P

## Working things after first successful bootup
- USB mouse is working. USB drives are getting detected. 
- Keyboard is working. 
- iServices working. 
- Ethernet is working. 

## Successful Wi-Fi Patch
- Debugging: AirportItlwm.kext was in a subfolder which ProperTree apparently could not get in the snapshot. As a result, the config.plist file did not have AirportItlwm.kext in the Kernels. 
- Solution: After moving the AirportItlwm.kext to the "EFI/OC/Kexts" folder, updated the config.plist by a clean snapshot. 
- Solution worked. Wi-Fi access points were visible. Could connect to the internet through a 5GHz network. 

## Successful iGPU and Backlight Patch
- Debugging for iGPU not detecting: iGPU Intel (U)HD 620 needs device-id faking in "DeviceProperties" section of config.plist. Apart from this, config.plist lacked "PciRoot(0x0)/Pci(0x2,0x0)" child under "DeviceProperties/Add" as it wasn't there in Sample.plist. 
- The device-id and other information was filled according to this section of the OpenCore install guide - https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#deviceproperties. "PciRoot(0x0)/Pci(0x2,0x0)" was added as a child to "DeviceProperties/Add".
- Debugging for backlight problem: The order of SSDT was incorrect. 
- Solution: Order of SSDT .aml files was edited in config.plist:
	1. SSDT-XOSI.aml
	2. SSDT-AWAC.aml
	3. SSDT-EC-USBX-LAPTOP.aml
	4. SSDT-PLUG-DRTNIA.aml
	5. SSDT-PNLF-CFL.aml
