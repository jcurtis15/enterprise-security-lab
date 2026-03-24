# Enterprise Security Lab - Jeff Curtis

## Overview

This repository documents the creation and use of a virtualized enterprise lab environment built in Oracle VirtualBox. This lab simulates a real-world network consisting of Windows and Linux systems and is used to develop hands-on skills in system administration, networking, and cybersecurity.

## Lab Environment

This lab includes the following systems:

- **DC01** - Windows Server 2025 (Domain Controller, DNS, Active Directory)
- **SRV01** - Windows Server 2025 (Management Server, RSAT tools)
- **CLIENT01** - Windows 11 (Domain-joined workstation)
- **KALI01** - Kali Linux (Attacker machine)
- **UBU01** - Ubuntu Server (Linux target)

## Key Features

- Active Directory Domain Services (AD DS)
- DNS configuration and troubleshooting
- User and group management
- Group Policy (GPO)
- File shares and permissions
- Cross-platform networking (Windows + Linux)
- Attack simulation and enumeration using Kali Linux

## Repository Structure

```text
enterprise-security-lab/
	setup/				# System installation and configuration
	troubleshooting/	# Issues encountered and resolutions
	labs/				# Hands-on labs (admin, network, cybersecurity)
	assets/				# Screenshots and diagrams
```

## Labs

Labs will cover:

- System Administration (users, groups, GPO, file shares)
- Network Analysis (Nmap, DNS, service enumeration)
- Cybersecurity (enumeration, credential attacks, lateral movement)
- Blue Team (log analysis, detection, account security)

## Tools & Technologies

- VirtualBox
- Windows Server 2025
- Windows 11
- Kali Linux
- Ubuntu Server
- Active Directory & DNS
- Nmap, enum4linux, CrackMapExec (planned)

## Goals

- Build and manage a realistic enterprise environment
- Develop practical system and network administration skills
- Perform offensive security techniques in a controlled lab
- Understand detection and defensive strategies

## Status

- [x] Lab environment fully built
- [x] Domain configured and validated
- [ ] Enumeration labs in progress
- [ ] Attack simulations
- [ ] Detection & logging labs

## About

This project is part of my journey to develop hands-on cybersecurity skills through real-world lab environments and structured documentation.
