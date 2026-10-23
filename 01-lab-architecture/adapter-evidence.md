# Network adapter evidence — VMware Workstation

> **Captured:** 23 October 2026. **Status:** Six screenshot-pair profiles mapped to VM display names by the lab owner in a follow-up message. The mappings are **operator-reported**; the hardware settings are **screenshot-verified**, but each settings dialog itself does not display the VM name. This is not proof of guest IP addresses, live connectivity, or network segmentation.

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

Six VMs are mapped above using the owner's A–F labels; `Caldera-SOC` is additionally confirmed by the separate named Library screenshot.

**Still missing adapter settings:** `Linux Mint` and `Metasploitable3`. Do not infer their connections from the other VMs.

**Attention:** `SOC_DC-1` is reported as dual-homed (VMnet1 and VMnet8). This does not imply it is routing traffic, but it merits a guest-level review of its DNS registration, default route, IP forwarding and exposure. `Kali Linux` is also dual-adapter with generic Host-only and NAT modes; record which VMware networks those modes resolve to in the present configuration.

## Security verification before running attack simulations

- Check whether **Metasploitable3** (a deliberately vulnerable target) has NAT access or only a host-only connection. Restrict exposure to the authorised test network; avoid unrestricted external access if not necessary.
- Check whether guests with both adapters have IP forwarding, network bridging, routing or firewall policies that could unintentionally connect the attack-test network to other networks.
- Review VMnet8 NAT port forwarding and any host firewall allowances.
- Record actual guest IP address, subnet mask, default gateway, DNS and adapter-to-interface mapping later; VMware Hardware screenshots alone are insufficient.

## Next evidence needed

1. Capture **both Network Adapter settings** for `Linux Mint` and `Metasploitable3` and identify which VM each capture belongs to. They may have one adapter rather than two.
2. During guest validation, confirm current interface addressing, routing, DNS, and forwarding on `SOC_DC-1`, `Kali Linux` and the other dual-homed VMs.
3. Cross-check the operator's A–F mapping against named VM summaries during the next review if a fully screenshot-backed audit trail is desired.

**Privacy:** No raw screenshot has been committed; this document is a textual, sanitised transcription only.
