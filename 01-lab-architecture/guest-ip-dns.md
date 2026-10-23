# Guest IP, gateway and DNS evidence

> **Scope:** Live guest-configuration observations from operator-supplied screenshots; each observation is time-bound. A screenshot does **not** establish future configuration, service health, complete routing policy or successful DNS resolution.

## SOC_DC-1 — Windows Server / domain controller

**Capture date:** 23 October 2026 (VMware screenshot with the VM named `SOC_DC-1` selected and `ipconfig /all` open in elevated Windows PowerShell). Source was supplied as a one-page PDF containing a screenshot. **Raw PDF and screenshots were not uploaded to this repository**.

### Windows identity (screen-verified)

| Property | Observation |
| --- | --- |
| VMware VM display name | `SOC_DC-1` |
| Windows hostname | `SOC_DC01` |
| Primary DNS suffix | `soc.home.lab` |
| DNS suffix search list | `soc.home.lab`, `localdomain` |
| IP Routing Enabled | **No** |
| WINS Proxy Enabled | No |

**Important correction:** Earlier informal lab history mentioned a domain `lab.local`. The **current primary DNS suffix is `soc.home.lab`**, not `lab.local`. The screenshot alone does not prove the AD forest/domain name; verify it from Active Directory before describing `soc.home.lab` as the authoritative domain.

### Adapter 1 — internal SOC network

| Property | Screenshot reading |
| --- | --- |
| Windows interface alias | `Soc nw` |
| VMware association | Adapter 1 — Custom `VMnet1` (verified earlier from VM settings) |
| IPv4 address | `10.10.10.10` |
| IPv4 subnet mask | `255.255.255.0` (`/24`) |
| DHCP enabled | No (static address) |
| Default gateway | **Blank / none shown** |
| DNS server | **`0.0.0.0`** (as displayed by `ipconfig /all`) |
| NetBIOS over TCP/IP | Enabled |

### Adapter 2 — VMware NAT network

| Property | Screenshot reading |
| --- | --- |
| Windows interface alias | `Ethernet1` |
| Connection-specific DNS suffix | `localdomain` |
| VMware association | Adapter 2 — Custom `VMnet8` (verified earlier from VM settings) |
| IPv4 address | `192.168.111.130` |
| IPv4 subnet mask | `255.255.255.0` (`/24`) |
| DHCP enabled | Yes |
| Default gateway | `192.168.111.2` |
| DHCP server | `192.168.111.254` |
| DNS server | `192.168.111.2` |
| Primary WINS server | `192.168.111.2` |
| NetBIOS over TCP/IP | Enabled |

The VMware Virtual Network Editor separately confirmed `VMnet1` as Host-only (`10.10.10.0/24`, DHCP disabled) and `VMnet8` as NAT (`192.168.111.0/24`, DHCP enabled).

### Assessment and follow-up

**Confirmed by screenshot:** The internal address is correctly within the designated `VMnet1` subnet, with static IPv4 and no internal default gateway. The external/NAT adapter received a DHCP address and uses the VMnet8 subnet. Windows reports `IP Routing Enabled: No`.

**DNS issue requiring verification:** The internal adapter reports DNS server `0.0.0.0`; the NAT adapter reports DNS server `192.168.111.2`. For a Windows Server domain controller that runs AD-integrated DNS, the resolver arrangement should be checked. Usually the DC should resolve its AD domain through an appropriate internal AD DNS server. Do not assume `0.0.0.0` is correct, and **do not make disruptive DNS changes until the current DNS service and domain are verified**. A NAT DHCP-provided DNS server generally should not be the authoritative resolver for AD records.

**Segmentation note:** Windows reporting `IP Routing Enabled: No` is useful evidence but not a conclusive isolation test. Later check guest forwarding, network routes, firewall and bridge policies, especially on dual-homed systems.

### DNS client verification — 23 October 2026

A second VMware screenshot, with `SOC_DC-1` selected, shows the successful execution of:

```powershell
Get-DnsClientServerAddress -AddressFamily IPv4 | Format-Table InterfaceAlias, ServerAddresses -AutoSize
```

| Windows interface alias | IPv4 DNS client server addresses | Evidence |
| --- | --- | --- |
| `Soc nw` | `{0.0.0.0}` | Read-only PowerShell output |
| `Ethernet1` | `{192.168.111.2}` | Read-only PowerShell output |
| `Loopback Pseudo-Interface 1` | `{}` (none listed) | Read-only PowerShell output |

The screenshot also shows an earlier failed command due to mistyping `Get-DnsClientServicesAddress`; the later `Get-DnsClientServerAddress` command succeeded. There is no indication the failure changed any settings.

**Interpretation:** Two separate read-only commands now show `0.0.0.0` for the internal adapter and `192.168.111.2` for the NAT adapter. This confirms the Windows DNS client configuration, **not that the DNS server service is active or resolving Active Directory records**. No DNS changes have been made or claimed.

**Next read-only verification command (PowerShell, Domain Controller):**

```powershell
Get-Service -Name DNS
```

Check that the Windows DNS Server service is installed and its current status is `Running`. Then inspect the actual Active Directory domain and DNS zones before considering resolver changes.

### Data handling

The submitted screenshot contained MAC addresses, IPv6 link-local addresses and Windows guest UI information. To reduce unnecessary disclosure, these identifier values and the raw PDF were **not** committed; only the operationally relevant IPv4, DNS and adapter settings are transcribed.

## Remaining guest verification

- [x] `SOC_DC-1`: Capture IP, gateway, DHCP and DNS values from `ipconfig /all` (23 October 2026)
- [x] `SOC_DC-1`: Confirm DNS client settings with `Get-DnsClientServerAddress` (23 October 2026)
- [ ] `SOC_DC-1`: Verify DNS Server service (`Get-Service -Name DNS`), AD domain, DNS zones and DNS registration on the NAT adapter
- [ ] `W.10.Pro_Soc-Lab Client`: Verify guest IP, gateway and DNS
- [ ] `Ubuntu Wazuh`: Verify guest IP, gateway and DNS
- [ ] `Splunk-SIEM`: Verify guest IP, gateway and DNS
- [ ] `Caldera-SOC`: Verify guest IP, gateway and DNS
- [ ] `Kali Linux`: Verify guest IP, gateway and DNS
- [ ] `Linux Mint`: Verify guest IP, gateway and DNS
- [ ] `Metasploitable3`: Verify guest IP on VMnet1 and confirm NAT Adapter 2 remains disconnected when powered on
