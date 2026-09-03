---
title: # Virtualization Concepts - CompTIA A+ 220-1201 - 4.1
tags:
  - comptiaA+
  - core1
---

# Virtualization

## Virtualization Basics

- One physical computer running many operating systems
- Each OS is fully separate, with its own allocation of CPU, memory, storage and network
- Host-based virtualization
    - Your normal desktop OS running additional operating systems on top of it
- Standalone virtualization server
    - Dedicated hardware built only to host virtual machines
    - Enterprise-level deployments
- Not a new technology
    - IBM mainframe virtualization dates back to 1967

> [!info] Two hypervisor models A hosted (Type 2) hypervisor runs as an application inside an existing OS. A bare-metal (Type 1) hypervisor runs directly on the hardware with no OS underneath.

## Sandboxing

- Isolated testing environment
    - No connection to production systems or the outside world
    - A technological safe space
- Virtualizes the development process
    - Try some code, break some code, nobody gets hurt
- Additional features that come with virtualization
    - Roll back to a previous snapshot
    - Spin up extra systems on demand

> [!note] Snapshots capture the full VM state. Reverting undoes anything that happened after the snapshot was taken, including malware execution or a failed patch.

## Building the Application

### Develop

- Secure environment for writing code
- Developers test inside their own individual sandboxes

### Test

- Separate virtual environment used only for testing
- Still part of the development stage
- All of the pieces are put together
- Answers the question: does the whole thing actually work?

> [!info] Staging and production come after these two stages, each in its own isolated environment.

## Legacy Software and Operating Systems

- Need to run different application versions on the same system
    - Run each application instance in a separate VM
- Application only runs on a previous OS version
    - Create a VM with the older OS

> [!note] A legacy VM usually means an unpatched OS. Keep its network access restricted.

## Cross-Platform Virtualization

- Windows, macOS and Linux each have their own strengths and weaknesses
- Run different operating systems at the same time
    - Move between each OS seamlessly
    - No rebooting required
- Saves time and resources
    - Everything runs on one physical computer
