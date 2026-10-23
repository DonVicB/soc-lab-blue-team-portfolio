# Network adapter evidence — VMware Workstation

> **Captured:** 23 October 2026. **Status:** All eight VM network-adapter configurations documented. Six Hardware screenshot profiles A–F were attributed to VMs by the lab owner; two further VMware Library screenshots directly identify Linux Mint and Metasploitable3 alongside their network adapter summaries. No guest IP addresses, live connectivity or network isolation are verified yet.

## Source and identification basis

Twelve screenshots show VMware Workstation's **Virtual Machine Settings → Hardware** tab. Each pair captures Adapter 1 and Adapter 2 alongside the same memory, processor count and virtual disk size. These windows do **not** show the selected VM name.

The lab owner subsequently supplied the mapping: **A=Caldera, B=Slunk, C=W.10.Pro, D=wazuh, E=Kali, F=SOC_DC_1**. The abbreviations are normalised to the display names established by the earlier VMware Library screenshot:

- A → `Caldera-SOC`
- B → `Splunk-SIEM` (written as “Slunk” in the mapping)
- C → `W.10.Pro_Soc-Lab Client` (written as “W.10.Pro”)
- D → `Ubuntu Wazuh` (written as “wazuh”)
- E → `Kali Linux` (written as “Kali”)
- F → `SOC_DC-1` (written as “SOC_DC_1”)

**Provenance:** Configuration values are read directly from screenshots; VM identities are supplied by the lab owner, except `Caldera-SOC`, whose adapter configuration was also independently identified in the VMware Library screenshot. The mapping should not be described as independently screenshot-verified for B–F.

## Transcribed adapter profiles

Order A–F follows the order in which the screenshots were presented (most recent capture times first). Times below are from screenshot filenames, **not evidence of system uptime or network activity**.

| Evidence pair | Screenshot times (23 Oct 2026) | RAM | Processors | Virtual disk | Adapter 1 | Adapter 2 | VM identity (operator-confirmed) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 22:45:35, 22:45:40 | 8 GB | 4 | 60 GB SCSI | Custom: VMnet8 | Custom: VMnet1 | `Caldera-SOC` |
| B | 22:45:22, 22:45:25 | 8 GB | 4 | 100 GB SCSI | Custom: VMnet1 | Custom: VMnet8 | `Splunk-SIEM` |
| C | 22:45:11, 22:45:14 | 8 GB | 4 | 80 GB NVMe | Custom: VMnet8 | Custom: VMnet1 | `W.10.Pro_Soc-Lab Client` |
| D | 22:45:00, 22:45:02 | 8 GB | 4 | 80 GB SCSI | Custom: VMnet1 | NAT (automatic) | `Ubuntu Wazuh` |
| E | 22:44:42, 22:44:45 | 8 GB | 4 | 127 GB SCSI | Host-only (automatic) | NAT (automatic) | `Kali Linux` |
| F | 22:44:14, 22:44:17 | 8 GB | 4 | 127 GB SCSI | Custom: VMnet1 | Custom: VMnet8 | `SOC_DC-1` |

**Shared visible adapter state:** Both adapters in every screenshot pair have **Connect at power on** enabled. The separate **Connected** box appears unchecked at capture time. That checkbox does not establish the eventual connection state of a running guest.

**Network-mode distinction:** VMware's `NAT` and `Host-only` radio options are recorded verbatim above. They are not labelled as `Custom (VMnet8)` or `Custom (VMnet1)` respectively in those screenshots. VMware's default NAT and host-only networks are commonly VMnet8 and VMnet1, but exact mode is still important for reproducibility.

**Interpretation:** This is an **operator-provided association of six distinct VM names to six screenshot profiles**, not direct VM-name evidence in the Hardware dialogs. It does not establish guest filesystem capacity, disk utilisation, runtime link state, guest IPs, routes, firewall policy, or whether the vulnerable target has NAT access.

## Named-VM mapping status

All **eight** VMs now have network adapter assignments documented: six are mapped by the owner's A–F labels; `Caldera-SOC` also has separate direct Library screenshot evidence; `Linux Mint` and `Metasploitable3` are **directly identified by two new VMware Workstation Library screenshots**.

### Directly verified named Library summaries (23 October 2026)

| VM | RAM | Processors | Virtual disk | Network Adapter 1 | Network Adapter 2 | Capture-time state |
| --- | --- | --- | --- | --- | --- | --- |
| `Linux Mint` | 8 GB | 4 | 127 GB SCSI | **Host-only** (generic VMware mode) | **NAT** (generic VMware mode) | Powered off |
| `Metasploitable3` | 6.1 GB | 4 | 39.1 GB SCSI | **Custom (VMnet1)** | **Custom (VMnet8)** | Powered off |

The screenshots directly show the VM names and summaries, so **no operator inference** is needed for these two. They do not show the detailed `Connect at power on` toggles, currently assigned guest IP addresses or guest routes. The VMware host filesystem paths visible in the screenshots have deliberately not been copied into this repository.

### Finding: vulnerable target configured with a NAT-connected adapter

`Metasploitable3` is a deliberately vulnerable test target. It has **Custom (VMnet8)** on Network Adapter 2, and the Virtual Network Editor independently confirms **VMnet8 is a NAT network**. This is a **configuration-level exposure concern**, not proof that the VM is currently online or reachable: it was powered off at capture time.

Before booting the target for future attack simulation, review whether it truly needs outside network access. **Recommended:** remove or disconnect Adapter 2 for ordinary isolated testing and retain VMnet1 for authorised attack traffic. If internet access is essential for a specific maintenance task, enable it temporarily with appropriate controls. Also review NAT port forwarding, host firewall rules, guest forwarding/bridging and potential exposure to the host/network.

### NAT adapter isolation change — 23 October 2026 (operator-confirmed saved and off)

A later VMware Virtual Machine Settings screenshot matches `Metasploitable3` (6.1 GB RAM, 4 processors, 39.1 GB SCSI virtual disk) and shows **Network Adapter 2** still assigned to **Custom (VMnet8)** with both **Connected** and **Connect at power on** **unchecked**.

- **Observed in settings dialog:** Both connection checkboxes unchecked for the NAT adapter.
- **What this controls:** With the setting saved, the second adapter should not connect automatically when the VM powers on; this does **not** remove the adapter or its Custom VMnet8 assignment.
- **Save status — owner confirmation:** After the screenshot was reviewed, the lab owner replied **“Done its off”**, confirming the change was saved and Adapter 2 is off. This is **operator-reported confirmation**, not a second screenshot or powered-on network test.
- **Remaining:** Confirm VMnet1 Adapter 1 still has `Connect at power on` enabled. If necessary, inspect host NAT port-forwarding and guest network routes. A disconnected NAT adapter reduces exposure but does not by itself verify full isolation.

### Guest validation still pending

The screenshots prove virtual adapter *assignments*, not guest link status, interface IP, DNS, gateway or IP forwarding. Track those separately during guest-level verification.

**Attention:** `SOC_DC-1` is reported as dual-homed (VMnet1 and VMnet8). This does not imply it is routing traffic, but it merits a guest-level review of its DNS registration, default route, IP forwarding and exposure. `Kali Linux` is also dual-adapter with generic Host-only and NAT modes; record which VMware networks those modes resolve to in the present configuration.

## Security verification before running attack simulations

- **Finding confirmed:** `Metasploitable3` has VMnet1 (host-only) **and** VMnet8 (NAT) assigned. Review disconnecting its NAT adapter before booting it for normal isolated testing; do not assume external exposure from the adapter setting alone.
- Check whether guests with both adapters have IP forwarding, network bridging, routing or firewall policies that could unintentionally connect the attack-test network to other networks.
- Review VMnet8 NAT port forwarding and any host firewall allowances.
- Record actual guest IP address, subnet mask, default gateway, DNS and adapter-to-interface mapping later; VMware Hardware screenshots alone are insufficient.

## Next evidence needed

1. **Done:** Named Library summaries for `Linux Mint` and `Metasploitable3` now show both adapters and hardware allocations.
2. **Done (owner-confirmed):** Metasploitable3 Adapter 2 configured not to connect at power-on and saved. **Next:** ensure Adapter 1 VMnet1 remains enabled before boot; validate guest IPv4 addresses, default gateways, DNS and forwarding in Step 4.4.
3. Cross-check operator-provided A–F associations against the named VMware Library VM summaries if a wholly screenshot-backed audit trail is desired.

**Privacy:** No raw screenshot has been committed; this document is a textual, sanitised transcription only.
