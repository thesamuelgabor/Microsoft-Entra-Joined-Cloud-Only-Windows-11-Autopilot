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

> **Note:** There are several ways to get a device's hardware hash into Autopilot — OEM/reseller pre-registration at the point of purchase, a Configuration Manager task sequence, a provisioning package built in Windows Configuration Designer, or running the export script directly on the device. For convenience in this lab, we used the PowerShell `-Online` method below, which registers the hash directly against the tenant with no manual CSV import step.

#### 1. Generate and Register the Hardware Hash — Direct Online Upload

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned

Install-Script -Name Get-WindowsAutopilotInfo -Force

Get-WindowsAutopilotInfo -Online
```

The `-Online` switch skips the CSV export entirely: it signs in interactively as an admin (a browser window opens for the Microsoft Entra ID credential prompt) and registers the device's hardware hash directly against the tenant over Microsoft Graph in the same run.

<img width="1026" height="774" alt="01-hardware-hash" src="https://github.com/user-attachments/assets/1c6bfcba-687e-4558-971c-bb1a7895df7b" />

*Ref 1: Admin sign-in prompt and direct hash registration*

Once the script completes, confirmed the device appeared under Devices → Enrollment → Windows Autopilot devices with no separate import step. Change the Group tag to `GT-Autopilot-Pilot` in the portal.

<img width="1614" height="462" alt="01b-hash-upload" src="https://github.com/user-attachments/assets/e384ac09-2237-490d-acb6-e42d6faa0a60" />

*Ref 2: Device registered in the Intune admin center*

#### 2. Dynamic Entra ID Group — SG-Win11-Autopilot-Pilot

```
(device.devicePhysicalIds -any (_ -eq "[OrderID]:GT-Autopilot-Pilot"))
```

<img width="1622" height="533" alt="02-dynamic-group png" src="https://github.com/user-attachments/assets/1182e8fd-19d2-452c-817f-750e3824bded" />

*Ref 3: Dynamic device group*

#### 3. Create the Deployment Profile - Entra Joined, Cloud-Only

| Setting | Value |
|---|---|
| Deployment mode | User-driven |
| Join type | Microsoft Entra joined |
| Hybrid Azure AD joined | Not used — cloud-only by design |
| EULA / Privacy settings | Hide |
| User account type | Standard |

<img width="827" height="766" alt="03-deployment-profile" src="https://github.com/user-attachments/assets/c9a37274-fc6e-4f5d-b787-02ba94c00df0" />

*Ref 4: Cloud-only deployment profile*

#### 4. Enrollment Status Page (ESP)

| Setting | Value |
|---|---|
| Block device use until required apps/profiles install | Yes |
| Allow users to collect logs on failure | Yes |
| Timeout | 60 minutes |

<img width="800" height="450" alt="image" src="docs/img/04-esp-config.png" />

*Ref 5: ESP configuration*

#### 5. End-to-End Enrollment

Reset the target VM to OOBE (from the [Lab Environment Preparation](../00-Lab-Environment-Preparation) checkpoint) and enrolled live: device registers as Entra joined only, no on-prem identity involved at any point, and lands compliant with no IT hands-on-keyboard step.

No apps are assigned at this stage — that's covered next, in the [Application Deployment](../02-Intune-Application-Deployment) project, before [Autopilot Device Preparation](../03-Windows-Autopilot-Device-Preparation) is introduced as an alternative to this flow.

<img width="800" height="450" alt="image" src="docs/img/05-oobe-complete.png" />

*Ref 6: Completed enrollment*

## About

Zero-touch Windows 11 provisioning as Microsoft Entra joined and cloud-only, built and validated end-to-end in a Microsoft 365 test tenant.
