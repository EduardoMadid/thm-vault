---
title: Troubleshooting Networks - CompTIA A+ 220-1201 - 5.5
tags:
  - comptiaA+
  - core1
---

## No network connectivity
- Do you have a link light?
	- Is it plugged in?
- Ping loopback (127.0.0.1)
	- Is the protocol stack working?
	- Availability and intermittent connectivity
- Ping local IP address
	- Checks local configuration, adapter, and link signal
- Ping default gateway
	- Connectivity on the local network
- Ping devices on router's other side
	- 8.8.8.8 or 9.9.9.9


## Intermittent wireless connectivity
- Interference
	- Something else is using our frequency
- Signal strength
	- Transmitting signal, transmitting antenna, receiving antenna, etc
- Incorrect channel
	- Usually automatic, look for manual tuning
- Bounce and latency
	- Multipath interference, flat surfaces
- Incorrect access point placement
	- Locate close to users


## Slow network speeds
- The network is slow!
	- It's probably not the network
- Confirm end-to-end connectivity
	- Ping or application login
	- Validate with a speed test
- Evaluate connectivity on each network hop
	- Utilization, errors, total throughput, filtering/ACLs, etc
- May require a packet capture
	- The ultimate verification


## Limited or no connectivity
- Windows alert in the system tray
	- Limited or No connectivity
	- No Internet Access
- Check the local IP address
	- An APIPA address will only have local connectivity
- If DHCP address is obtained, perform the ping tests
	- Local gateway, remote IP address


## Jitter
- Most real-time media is sensitive to delay
	- Data should arrive at regular intervals
	- Voice communication, live video
- If you miss a packet, there's no retransmission
- Jitter is the time between frames
	- Excessive jitter can cause you to miss information, "choppy" voice calls


## Poor VoIP quality
- High speed and low latency
	- Real-time applications are demanding
- Check the Internet connection
	- A speed test can identify slow links
- Verify local networking equipment
	- An old router can cause significant problems
- View the network performance
	- A packet capture would be useful


## Port flapping
- Network interface goes up and down
	- Over and over again
- Verify the cable
	- Check the wiring
- Move between switch interfaces
	- Is the flapping associated with the switch interface or the device?
- Replace bad hardware or cables
	- May require additional purchases


## High latency
- A delay between the request and the response
	- Waiting time
- Some latency is expected and normal
	- Laws of physics apply
- Examine the response times at every step along the way
	- This may require multiple measurement tools
- Packet captures can provide detailed analysis
	- Microsecond granularity
	- Get captures from both sides


## External interference
- Predictable
	- Fluorescent lights
	- Microwave ovens
	- Cordless telephones
	- High-power sources
- Unpredictable
	- Multi-tenant building
- Measurements
	- Signal to noise ratio (SNR)
	- Performance Monitor


## Signal to noise ratio (SNR)
- Signal
	- What you want
- Noise
	- What you don't want
	- Interference from other networks and devices
- You want a very large ratio
	- The same amount of signal to noise (1:1) would be bad


## Authentication issues
- Access a resource
	- Requires the proper credentials
	- Username, password, other factors
- Verify an active session
	- May need to be refreshed
	- Logout and back in
- May be difficult to see
	- Part of a service or background process
- Perform a packet capture
	- Verify connectivity and look for errors


## Intermittent Internet connectivity
- Determine the scope of the outage
	- Ongoing pings
	- Traceroute to a known location
	- Speed tests occasionally
- External issues require external support
	- Work directly with the ISP
	- Have your contact and account information available
- Check your SLA
	- Service level agreement