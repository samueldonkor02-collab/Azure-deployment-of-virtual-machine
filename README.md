# azure virtual machine deployment 
Deployed a Windows Server VM in Azure, connected via RDP, and installed IIS to get it serving a web page over its public IP.

# Azure Windows Server Deployment (Provisioning a VM, RDP Access, and IIS Web Server Setup)

`Microsoft Azure` `Windows Server 2025` `IIS` `RDP` `Virtual Machines` `Cloud Infrastructure`

## Overview
For this lab I stood up a Windows Server virtual machine in Microsoft Azure from scratch, connected to it remotely, and configured it as a basic web server using Internet Information Services (IIS). I wanted to get hands on with the full lifecycle of a cloud VM, not just click through the creation wizard and call it done. That meant provisioning it through the Azure portal, securing initial access, getting through first boot setup, installing a server role, and then actually checking that the thing worked from the outside.

## Objective
Get comfortable with the Azure VM creation workflow from start to finish, RDP into a freshly provisioned Windows Server instance, install the Web Server (IIS) role through Server Manager, and confirm the server was actually serving traffic by browsing to its public IP.

## Environment
- **Cloud platform:** Microsoft Azure (Free Trial subscription)
- **VM name:** `web-server1zx`
- **Region:** East US
- **Image:** Windows Server 2025 Datacenter, x64 Gen2
- **VM architecture:** x64 (Arm64 isn't supported by this image)
- **Security type:** Trusted launch virtual machine
- **Availability:** No infrastructure redundancy required (single instance)
- **Public IP:** `172.184.133.42`
- **Client device:** ASUS Vivobook (used to RDP into the VM)

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Azure Portal** | Web based management console | Provisioned the VM, configured networking, and set the administrator account |
| **Remote Desktop Protocol (RDP)** | Remote access protocol | Connected to the VM's desktop to complete first boot setup and manage it |
| **Server Manager** | Windows Server management console | Used to add the Web Server (IIS) role |
| **IIS (Internet Information Services)** | Windows web server role | Installed as the target web server for this lab |
| **Azure Pricing Calculator** | Cost estimation tool | Kept an eye on projected cost alongside VM creation |

## What I Did

### Provisioning the VM
I started the "Create a virtual machine" wizard in the Azure portal under Compute infrastructure. On the Instance details tab I named the VM `web-server1zx`, set the region to East US, left availability at "No infrastructure redundancy required," kept Trusted launch virtual machines as the security type, and picked the Windows Server 2025 Datacenter (x64, Gen2) image.

Under Administrator account, I set a username (`azureuser1`) and password for local admin access. Then under Inbound port rules, I set Public inbound ports to "Allow selected ports" and chose HTTP (80) and RDP (3389), since I needed RDP to manage the box and HTTP to serve a page once IIS was up. Azure warned that this opens those ports to all IP addresses, which is fine for a short lived lab VM but not something I'd leave wide open in production.

I reviewed everything and created the VM, which came back with the public IP `172.184.133.42`.

### Connecting via RDP and Getting Through First Boot
I connected to the VM's public IP over RDP from my laptop. On first login, Windows Server walked me through the standard out of box setup, including a "Send diagnostic data to Microsoft" privacy prompt where I could choose Required only or Include Optional diagnostics before actually reaching the desktop. I picked my diagnostic settings and finished signing in.

### Installing the Web Server (IIS) Role
Once I was on the desktop, I opened Server Manager and launched the Add Roles and Features Wizard. I went through Before You Begin, Installation Type, Server Selection, Server Roles, and Features, and selected Web Server (IIS).

The role services list included IIS Management Console under Management Tools, and Common HTTP Features under Web Server (Default Document, Directory Browsing, HTTP Errors, Static Content), along with Health and Diagnostics features like HTTP Logging. I confirmed those and kicked off the install, then watched it complete on the Installation progress screen against the destination server `web-server1zx`.

### Verifying the Web Server
Last step was just opening a browser and hitting the VM's public IP (`172.184.133.42`) over HTTP. The default IIS Windows Server welcome page loaded right away, which told me the role installed correctly and that the inbound port 80 rule I set during VM creation was actually doing its job.

## What's in This Repo

```
azure-iis-web-server-lab/
├── README.md                                # This file
└── screenshots/
    ├── 01-vm-create-admin-account-ports.png       # Administrator account + inbound port rules
    ├── 02-vm-create-instance-details.png          # VM name, region, image, security type
    ├── 03-iis-default-page-verification.png       # IIS welcome page over public IP
    ├── 04-rdp-session-connecting.png              # RDP session to public IP
    ├── 05-first-boot-diagnostic-data-prompt.png   # Windows Server first boot privacy prompt
    └── 06-server-manager-iis-install-progress.png # Add Roles and Features Wizard progress
```

## Skills I Picked Up
- **Provisioning a VM with intent instead of just accepting defaults.** I deliberately chose which inbound ports to expose (HTTP and RDP only) rather than taking whatever the wizard defaulted to, and actually read Azure's warning about opening ports to all source IPs instead of clicking past it.
- **Working through a full VM lifecycle.** Creation, first boot OOBE, role installation, and external verification, rather than stopping the moment the VM showed as "Running."
- **Using Server Manager to add server roles** and understanding what each IIS sub component (Management Tools, Common HTTP Features, Health and Diagnostics) actually gives you.
- **Verifying success from the outside in.** I confirmed the deployment worked by hitting the public IP from a browser instead of just trusting that the installation wizard finished without errors.

## How This Applies in the Real World
Standing up a web facing VM this way is a common first step for hosting internal tools, test environments, or small production workloads in the cloud. The part of this lab that actually matters for security is the networking decision made at creation time: only opening the ports that were actually needed (80 and 3389) instead of leaving the VM's entire port range exposed. In a real deployment, the next moves from here would be restricting RDP access to specific IP ranges (or swapping it out for Azure Bastion), enabling HTTPS instead of plain HTTP, and taking a closer look at the Network Security Group rules Azure generated instead of leaving that "allow all IPs" default in place.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in healthcare. I'm currently studying for CompTIA Security+ and building labs like this one to get real hands on reps with the infrastructure that a lot of security work sits on top of, since understanding how a server actually gets stood up and exposed feels like a prerequisite for understanding how to secure it.

## What I Want to Learn Next
- Restricting the RDP inbound rule to a specific source IP range instead of "any," and comparing that against using Azure Bastion for management access
- Adding an HTTPS binding with a certificate instead of leaving the site on plain HTTP
- Reviewing the Network Security Group rules this VM generated to understand exactly what Azure built behind the scenes
- Hardening the IIS install itself (removing default content, reviewing default document settings, disabling directory browsing)

## Limitations & What I'd Do Differently in Production
- **RDP and HTTP were both left open to all IP addresses**, which Azure flagged as testing only. A production deployment would restrict this to known IP ranges or use a bastion or jump host instead.
- **No HTTPS was configured.** The site is only reachable over unencrypted HTTP, which wouldn't fly for anything handling real traffic.
- **Default IIS install only.** No custom site content, hardening, or logging review was done beyond confirming the default welcome page loaded.
- **Single VM, no redundancy.** This was built without infrastructure redundancy on purpose since it was a one off lab, not a resilient deployment.

## References
- [Microsoft Azure Virtual Machines Documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [IIS Documentation](https://learn.microsoft.com/en-us/iis/)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
