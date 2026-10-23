# Network adapter evidence — VMware Workstation

> **Captured:** 23 October 2026. **Status:** Six distinct screenshot-pair configurations transcribed; **VM identity not established for each pair**. This is hardware-settings evidence, not proof of guest IP addresses, live connectivity, or network segmentation.

## Source and identification limitation

Twelve user-supplied screenshots show VMware Workstation's **Virtual Machine Settings → Hardware** tab. Each pair captures the first and second network adapters with the same visible memory, processor count and disk information. **The window title is only "Virtual Machine Settings" and does not show which VMware Library VM is selected.**

**Do not assign these configurations to a named VM without corroborating evidence** (for example, a VMware Library screenshot taken with that VM selected, or an annotated list from the operator). Pair A is consistent with the previously screenshot-verified configuration of `Caldera-SOC`, but that visual similarity alone is not proof that Pair A came from that VM. The earlier Caldera-SOC Library screenshot **already established**: Adapter 1 `Custom (VMnet8)`, Adapter 2 `Custom (VMnet1)`.

## Transcribed adapter profiles

Order A–F follows the order in which the screenshots were presented (most recent capture times first). Times below are from screenshot filenames, **not evidence of system uptime or network activity**.

| Evidence pair | Screenshot times (23 Oct 2026) | RAM | Processors | Virtual disk | Adapter 1 | Adapter 2 | VM display name |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 22:45:35, 22:45:40 | 8 GB | 4 | 60 GB SCSI | Custom: VMnet8 | Custom: VMnet1 | **Not shown**; resembles Caldera-SOC prior summary |
| B | 22:45:22, 22:45:25 | 8 GB | 4 | 100 GB SCSI | Custom: VMnet1 | Custom: VMnet8 | **Not shown** |
| C | 22:45:11, 22:45:14 | 8 GB | 4 | 80 GB NVMe | Custom: VMnet8 | Custom: VMnet1 | **Not shown** |
| D | 22:45:00, 22:45:02 | 8 GB | 4 | 80 GB SCSI | Custom: VMnet1 | NAT (automatic) | **Not shown** |
| E | 22:44:42, 22:44:45 | 8 GB | 4 | 127 GB SCSI | Host-only (automatic) | NAT (automatic) | **Not shown** |
| F | 22:44:14, 22:44:17 | 8 GB | 4 | 127 GB SCSI | Custom: VMnet1 | Custom: VMnet8 | **Not shown** |

**Shared visible adapter state:** Both adapters in every screenshot pair have **Connect at power on** enabled. The separate **Connected** box appears unchecked at capture time. That checkbox does not establish the eventual connection state of a running guest.

**Network-mode distinction:** VMware's `NAT` and `Host-only` radio options are recorded verbatim above. They are not labelled as `Custom (VMnet8)` or `Custom (VMnet1)` respectively in those screenshots. VMware's default NAT and host-only networks are commonly VMnet8 and VMnet1, but exact mode is still important for reproducibility.

**Disciplined interpretation:** These are six captured configurations, **not an independently established mapping of six unique VMs**. They do not show the full guest filesystem capacity, disk utilisation, powered-on state, static versus dynamic guest IPs, routing configuration, or whether a vulnerable target is kept off NAT.

## Confirmed named-VM adapter evidence

From the **separate VMware Workstation Library screenshot** captured earlier on 23 October 2026:

| VM display name | Adapter 1 | Adapter 2 | Evidence status |
| --- | --- | --- | --- |
| `Caldera-SOC` | Custom (VMnet8) | Custom (VMnet1) | **Identified directly by selected Library VM** |

The other seven named VMs remain unassigned until screenshots or operator labels explicitly associate an evidence pair with the VM name.

## Security verification before running attack simulations

- Check whether **Metasploitable3** (a deliberately vulnerable target) has NAT access or only a host-only connection. Restrict exposure to the authorised test network; avoid unrestricted external access if not necessary.
- Check whether guests with both adapters have IP forwarding, network bridging, routing or firewall policies that could unintentionally connect the attack-test network to other networks.
- Review VMnet8 NAT port forwarding and any host firewall allowances.
- Record actual guest IP address, subnet mask, default gateway, DNS and adapter-to-interface mapping later; VMware Hardware screenshots alone are insufficient.

## Next evidence needed — label the named VMs

For each of the following, capture a screenshot with the VM **name visible** next to the virtual hardware summary, or give an explicit mapping from evidence pair A–F to the VM name:

`Kali Linux`, `Linux Mint`, `Metasploitable3`, `SOC_DC-1`, `Ubuntu Wazuh`, `W.10.Pro_Soc-Lab Client`, `Splunk-SIEM`.

`Caldera-SOC` is already confirmed from the earlier VMware Library capture.

**Privacy:** No raw screenshot has been committed; this document is a textual, sanitised transcription only.
