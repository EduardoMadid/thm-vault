---
title: Troubleshooting Hardware - CompTIA A+ 220-1201 - 5.1
tags:
  - comptiaA+
  - core1
---
## POST - Power On Self Test
- Test majos systems components before booting the OS
	- Main systems
	- Video
	- Memory
- Failures are usually noted with beeps and/or codes
	- Bios versions can differ, check your documentation
- Don't bother memorizing the beep codes
	- They are all different between manufacturers
	- Know what to do when you hear them


# POST and boot
- Blank screen on boot
	- Listen for beeps 
	- Bad video, bad RAM, bad CPU
	- BIOS configuration issue
- BIOS time and setting
	- Maintained with the motherboard battery
	- Replace the battery
- Attempts to boot to incorrect device
	- Set boot order in BIOS configuration
	- Confirm that the startup device has a valid OS
	- Check for media in a startup device


## Crash screens
- Windows Stop Error
- Blue Screen of Death
	- You don't want this
- Contains importart information
	- Also written to event log


## Bluescreens and shutdowns
- Startup and shutdown BSOD
	- Bad hardware, bad drivers, bad application
- Use Last Known Good, System Restore, or Rollback Driver
	- Try Safe Mode
- Reseat or remove the hardware
	- If possible
- Run hardware diagnostics
	- Provided by the manufacturer
	- BIOS may have hardware diagnostics

## Proprietary crash screens
- Every application has their own notifications
	- Some are very informational
	- Some are quite bad
- The quality of information contained in a help desk ticket can vary
	- Get as many detail as possible
- A detailed transcript of the error would be valuable
	- Screenshots are even better
	- Ask for the screenshot in the ticket instructions 


## Blank screen
- Is the monitor connected?
	- We wouldn't ask if it wasn't a common solution
	- Check both power and signal cable
- Input selection on monitor
	- HDMI, DVI, VGA, etc 
- Image is dim
	- Check britghtness controls
- Swap the monitor
	- Try the monitor on another computer
- No video after Windows loads
	- Use VGA (F8) 


## No power
- No power at source
- No power ftom the power supply
- Get out your multimeter
- Fans spin - no power to other devices
	- Where is your fan connected?
	- No POST - bad motherboard?
	- Case fans have lower voltage requirements
	- Check the power supply output


## Sluggish perfomance
- Task Manager
	- Check for high CPU utilization and I/O
- Windows Update
	- Latest patches and drivers
- Disk space
	- Check for available space and defrag
- Laptops may be using power-saving mode
	- Throttles the CPU
- Anti-virus and anti-malware
	- Scan for bad guys


## Overheating
- Heat generation
	- CPUs, video adapters, memory
- Cooling systems
	- Fans and airflow
	- Heat sinks
	- Clean and clear
- Verify with monitoring software
	- Built into the BIOS
	- Try HWMonitor (WINDOWS)

## Smoke and burning smell
- Electrical problems
	- The smoke makes everything work
- Always disconnect power
	- There should never be a burned color
- Locate bad components
	- Even after the system has cooled down
	- Replace all damaged components


## Random shutdown
- No warning, black screen
	- May have some details in your event viewer
- Heat-related issue
	- High CPU or graphics, gaming
	- Check all fans and heat sinks
	- BIOS may show fan status and temperatures
- Failing hardware
	- Has anything changed?
	- Check Device Manager, run diagnostics
- Could be anything
	- Eliminate what's working


## Application crashes
- Application stops working
	- May provide an error message
	- May just dissapear
- Check the Event Viewer or logs
	- Often includes useful reconnaissance
- Check the Reliability Monitor
	- A history of application problems
	- Checks for resolutions
- Reinstall the application
	- Contact application support


## Unusual noises
- Computers should hum
	- Not grind
- Rattling
	- Loose components
- Scraping
	- Hard drive issues
- Clicking
	- Fan problems
- Pop
	- Blown capacitor


## Inaccurate system date/time
- Bad motherboard battery
	- Often a button style battery
- A bad battery will require a BIOS configuration or date/time configuration by removing the battery
	- Newer computers use a jumper


[Troubleshooting Hardware - CompTIA A+ 220-1201 - 5.1](https://www.youtube.com/watch?v=HgeURynWn_w&list=PLG49S3nxzAnnes8ZGI-OBlKEukHCX46N8&index=58)
