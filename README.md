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

#### 2. Dynamic Entra ID Group — DG-Win11-Autopilot-Pilot

```
(device.devicePhysicalIds -any (_ -eq "[OrderID]:GT-Autopilot-Pilot"))
```
<img width="1622" height="565" alt="02-dynamic-group" src="https://github.com/user-attachments/assets/3d1dabe7-4dac-43e7-adcc-84030918f5b0" />

*Ref 3: Dynamic device group*

#### 3. Create the Deployment Profile - Entra Joined, Cloud-Only

| Setting | Value |
|---|---|
| Deployment mode | User-driven |
| Join type | Microsoft Entra joined |
| Hybrid Azure AD joined | Not used — cloud-only by design |
| EULA / Privacy settings | Hide |
| User account type | Standard |

<img width="836" height="771" alt="03-deployment-profile" src="https://github.com/user-attachments/assets/85d00c34-c3eb-4e6a-936e-692c10b7d717" />

*Ref 4: Cloud-only deployment profile*

#### 4. Enrollment Status Page (ESP)

| Setting | Value |
|---|---|
| Block device use until required apps/profiles install | Yes |
| Allow users to collect logs on failure | Yes |
| Timeout | 60 minutes |

<img width="985" height="765" alt="04-esp-config" src="https://github.com/user-attachments/assets/2fbad643-8f47-44b6-a05e-3df8a7275765" />

*Ref 5: ESP configuration*

#### 5. End-to-End Enrollment

Restart the target device to OOBE. Device registers as Entra joined only, no on-prem identity involved at any point, and lands compliant with no IT hands-on-keyboard step.

<img width="1021" height="768" alt="image" src="https://github.com/user-attachments/assets/1097bb09-55b7-482e-a5c1-9625a7d4a240" />

*Ref 6: Completed enrollment*
