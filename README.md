# Microsoft Entra Joined/Cloud-Only Windows-11 Autopilot

## Objective

This project demonstrates zero-touch provisioning of a Windows 11 device as Microsoft Entra joined and cloud-only (no on-prem AD dependency), using the traditional Windows Autopilot user-driven deployment path.
It establishes the baseline provisioning model that the rest of the Windows 11 track builds on top of.

### Skills Learned

- Hardware hash capture and registration in Windows Autopilot
- Deployment profile design for Entra joined (cloud-only), not Hybrid joined, devices
- Dynamic Entra ID group design driven by an Autopilot Group Tag
- Enrollment Status Page (ESP) design to block use until provisioning finishes

### Tools Used

- Microsoft Intune / Endpoint Manager admin center
- Windows Autopilot
- Microsoft Entra ID (dynamic groups)
- PowerShell (`Get-WindowsAutoPilotInfo`)

## Steps

#### 1. Generate and Register the Hardware Hash

```powershell
Install-Script -Name Get-WindowsAutoPilotInfo -Force
Get-WindowsAutoPilotInfo -OutputFile hash.csv
```

Imported `hash.csv` into Devices → Enrollment → Windows Autopilot devices, tagging the device `GT-Autopilot-Pilot`.

<img width="800" height="450" alt="image" src="docs/img/01-hardware-hash.png" />

*Ref 1: Hardware hash registration*

#### 2. Create the Deployment Profile — Entra Joined, Cloud-Only

| Setting | Value |
|---|---|
| Deployment mode | User-driven |
| Join type | Microsoft Entra joined |
| Hybrid Azure AD joined | Not used — cloud-only by design |
| EULA / Privacy settings | Hide |
| User account type | Standard |

<img width="800" height="450" alt="image" src="docs/img/02-deployment-profile.png" />

*Ref 2: Cloud-only deployment profile*

#### 3. Dynamic Entra ID Group — SG-Win11-Autopilot-Pilot

```
(device.devicePhysicalIds -any (_ -eq "[OrderID]:GT-Autopilot-Pilot"))
```

<img width="800" height="450" alt="image" src="docs/img/03-dynamic-group.png" />

*Ref 3: Dynamic device group*

#### 4. Enrollment Status Page (ESP)

| Setting | Value |
|---|---|
| Block device use until required apps/profiles install | Yes |
| Allow users to collect logs on failure | Yes |
| Timeout | 60 minutes |

<img width="800" height="450" alt="image" src="docs/img/04-esp-config.png" />

*Ref 4: ESP configuration*

#### 5. End-to-End Enrollment

Reset the target VM to OOBE (from the [Lab Environment Preparation](../00-Lab-Environment-Preparation) checkpoint) and enrolled live: device registers as Entra joined only, no on-prem identity involved at any point, and lands compliant with no IT hands-on-keyboard step.

No apps are assigned at this stage — that's covered next, in the [Application Deployment](../02-Intune-Application-Deployment) project, before [Autopilot Device Preparation](../03-Windows-Autopilot-Device-Preparation) is introduced as an alternative to this flow.

<img width="800" height="450" alt="image" src="docs/img/05-oobe-complete.png" />

*Ref 5: Completed enrollment*

## About

Zero-touch Windows 11 provisioning as Microsoft Entra joined and cloud-only, built and validated end-to-end in a Microsoft 365 test tenant.
