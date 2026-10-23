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

**Important correction:** Earlier informal lab history mentioned `lab.local`, but the lab owner has now confirmed the actual Active Directory domain as `soc.home.lab` using `Get-ADDomain | Select-Object DNSRoot, NetBIOSName` on 23 October 2026. The NetBIOS domain name is `SOC`. The PowerShell values were **operator-reported**, not screenshot-verified; the server's primary DNS suffix was independently visible in the preceding screenshot. AD forest name and DNS zone health have not yet been checked.

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

### DNS Server service status — operator confirmation (23 October 2026)

The lab owner reported that `Get-Service -Name DNS` returned **Running** on `SOC_DC01`; a later screenshot **independently confirms Running**. It confirms the Windows DNS Server service is running at the time of the check, **not** that the correct DNS zones exist, AD DNS registrations are healthy, or that DNS client resolution works.

### Active Directory domain — operator confirmation (23 October 2026)

The lab owner ran the following command, with results later confirmed in a PowerShell screenshot:

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName
```

Its output:

| Field | Value | Provenance |
| --- | --- | --- |
| `DNSRoot` | `soc.home.lab` | Operator-reported, subsequently screenshot-verified |
| `NetBIOSName` | `SOC` | Operator-reported, subsequently screenshot-verified |

This confirms that the AD domain's DNS name matches the previously observed primary DNS suffix. It corrects the historical `lab.local` note. No Active Directory or DNS configuration was changed.

### DNS Manager forward lookup zone evidence — 23 October 2026

A screenshot of the Windows **DNS Manager** console with `soc.home.lab` selected confirms the following **visible forward lookup zones** under the server tree:

- `soc.home.lab`
- `_msdcs.soc.home.lab`

The selected `soc.home.lab` zone contains `_msdcs`, `_sites`, `_tcp`, `_udp`, `DomainDnsZones` and `ForestDnsZones` folder entries. The zone's Start of Authority (SOA) and Name Server (NS) records are visible; its NS entry names `soc_dc01.soc.home.lab.`. **Presence of these entries does not by itself prove that AD-integrated zone replication, SRV registrations or name resolution are healthy.**

#### Screenshot-visible A (IPv4 address) records

| Record name in `soc.home.lab` | Address shown | Assessment |
| --- | --- | --- |
| Zone apex — "(same as parent folder)" | `192.168.111.130` | DC NAT IPv4 address; registration may be undesirable for internal AD clients |
| Zone apex — "(same as parent folder)" | `192.168.254.10` | Outside both verified VMware subnets; **possible stale record**, not yet validated |
| `DonW10` | `10.10.10.50` | Differs from historical notes (`10.10.10.15`); check client live configuration |
| `soc_dc01` | `192.168.111.130` | DC NAT IPv4 address |
| `soc_dc01` | `10.10.10.10` | Expected VMnet1 IPv4 address from current `ipconfig /all` |

**Security / reliability finding:** Both the domain apex and DC hostname have NAT-related A records. This can cause domain clients to attempt a less appropriate network address. Do **not** delete records yet: first verify their current consumers, DNS registration settings, and how clients resolve the DC. The `192.168.254.10` record may be obsolete but needs confirmation.

**Evidence limits:** This screenshot shows records in the DNS Manager interface, not the `ZoneType` or `IsDsIntegrated` values. It also does not show successful or failed DNS queries. No changes were made to DNS.

### DNS zone enumeration — PowerShell screenshot (23 October 2026)

A later screenshot of the selected `SOC_DC-1` VM includes `Get-DnsServerZone | Select-Object ...` output:

| ZoneName | ZoneType | Active Directory integration |
| --- | --- | --- |
| `_msdcs.soc.home.lab` | Primary | **Not determined by this command output** |
| `0.in-addr.arpa` | Primary | Not determined |
| `127.in-addr.arpa` | Primary | Not determined |
| `255.in-addr.arpa` | Primary | Not determined |
| `soc.home.lab` | Primary | **Not determined by this command output** |
| `TrustAnchors` | Primary | Not determined |

**Reason the integration column is blank:** The screenshot shows the property spelled `isdsintergrated` (extra `r`), which is **not** the actual Windows DNS zone property `IsDsIntegrated`. PowerShell `Select-Object` emits an empty column for the unknown property. The missing values are **not** evidence of `False` or a failure of AD integration. The genuine `IsDsIntegrated` Boolean property indicates whether a DNS zone is AD-integrated.

The same screenshot now directly displays the DNS service as **Running** and `Get-ADDomain` output `soc.home.lab` / `SOC`. These were previously operator-reported and are **now screenshot-verified**.

### Corrected zone integration verification — 23 October 2026

The operator supplied a subsequent PowerShell screenshot showing the corrected `IsDsIntegrated` property. These results are **screenshot-verified**:

| ZoneName | ZoneType | IsDsIntegrated |
| --- | --- | --- |
| `_msdcs.soc.home.lab` | Primary | **True** |
| `0.in-addr.arpa` | Primary | False |
| `127.in-addr.arpa` | Primary | False |
| `255.in-addr.arpa` | Primary | False |
| `soc.home.lab` | Primary | **True** |
| `TrustAnchors` | Primary | True |

**Assessment:** Both `soc.home.lab` and `_msdcs.soc.home.lab` are verified as **Active Directory-integrated primary zones**. The three `in-addr.arpa` entries returning False does not by itself indicate a problem; they are special-purpose reverse lookup zones. `TrustAnchors` also reports True.

**Limits:** These results show zone configuration, **not** AD replication health, availability of required SRV records, or successful DNS lookups. The earlier `isdsintergrated` typo and its blank values remain documented above as part of the diagnostic trail. No DNS configuration changes have been made.

### Direct DNS A-record resolution — screenshot verified 23 October 2026

On `SOC_DC-1`, the lab owner ran the read-only command:

```powershell
Resolve-DnsName -Name soc_dc01.soc.home.lab -Type A -Server 10.10.10.10
```

The internal DNS server `10.10.10.10` returned **two A records for `SOC_DC01.soc.home.lab`**:

| Returned address | TTL (seconds) | Interpretation |
| --- | ---: | --- |
| `192.168.111.130` | 1200 | Domain Controller's VMware NAT address (VMnet8), undesired in ordinary internal client discovery |
| `10.10.10.10` | 1200 | Intended internal Domain Controller address (VMnet1) |

**Finding:** Direct DNS resolution works, but the authoritative server returns both the internal and NAT addresses for the DC's hostname. Some domain clients may attempt the NAT address when resolving the DC, potentially causing intermittent connection or authentication problems depending on their routes and filtering. These results **do not demonstrate that either connection attempt succeeded or failed**, only that both addresses are advertised.

**TTL:** 1200 seconds = 20 minutes. Changing the records later would not immediately clear any previously cached responses. No records have been deleted or changed.

### Per-interface DNS dynamic registration — screenshot verified 23 October 2026

The operator ran the read-only command:

```powershell
Get-DnsClient | Select-Object InterfaceAlias, RegisterThisConnectionsAddress, UseSuffixWhenRegistering
```

The initial attempt contained a typo, `SelectObject` rather than `Select-Object`, and failed without changing settings. The corrected command succeeded:

| Interface alias | RegisterThisConnectionsAddress | UseSuffixWhenRegistering | Role |
| --- | --- | --- | --- |
| `Soc nw` | **True** | False | Internal VMnet1 adapter, `10.10.10.10` |
| `Ethernet1` | **True** | False | VMnet8 NAT adapter, `192.168.111.130` |
| `Loopback Pseudo-Interface 1` | **True** | False | System loopback interface |

**Finding:** The NAT adapter currently permits dynamic registration of its address in DNS. That is a plausible mechanism for the unwanted `192.168.111.130` A records previously retrieved from the AD DNS server. `UseSuffixWhenRegistering=False` does **not** disable registration of the host's primary DNS name.

**Remediation plan (NOT YET EXECUTED):** Retain address registration on `Soc nw` (internal VMnet1); turn off `RegisterThisConnectionsAddress` for `Ethernet1` (NAT VMnet8) after recording the other DNS publishing mechanisms and preparing a rollback. This change would prevent DNS-client registration from the NAT adapter; it would **not necessarily** remove already-published A records or stop registration by the DNS Server / Netlogon services.

Microsoft documents three potential sources of unwanted multihomed DC records: DNS client, DNS Server listener interfaces and Netlogon; review the DNS Server interface bindings and the zone's Name Servers tab as part of remediation. Source: [Microsoft Learn — Avoid registering unwanted NICs in DNS on a multihomed domain controller](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/unwanted-nic-registered-dns-mulithomed-dc).

**Next read-only review:** In DNS Manager, right-click `SOC_DC01` (server node) → **Properties → Interfaces**. Record whether DNS listens on all IP addresses or a selected list, and whether `192.168.111.130` appears. **Do not change or apply settings yet.**

**Outstanding:** Check the `soc.home.lab` zone's Name Servers tab; verify the client's live network and use of AD DNS; examine potentially stale `192.168.254.10` record; plan backup and change/rollback validation before editing the registration controls.

**Other pending checks:** Determine whether `192.168.254.10` is stale, verify current Windows 10 guest address and whether it uses internal AD DNS, review DNS Server interface binding and NAT adapter registration, and create a rollback checkpoint before changing production-like AD lab settings.

### Data handling

The submitted screenshot contained MAC addresses, IPv6 link-local addresses and Windows guest UI information. To reduce unnecessary disclosure, these identifier values and the raw PDF were **not** committed; only the operationally relevant IPv4, DNS and adapter settings are transcribed.

## Remaining guest verification

- [x] `SOC_DC-1`: Capture IP, gateway, DHCP and DNS values from `ipconfig /all` (23 October 2026)
- [x] `SOC_DC-1`: Confirm DNS client settings with `Get-DnsClientServerAddress` (23 October 2026)
- [x] `SOC_DC-1`: DNS Server service reported `Running` (operator confirmation, 23 October 2026)
- [x] `SOC_DC-1`: Confirm AD domain as `soc.home.lab` and NetBIOS domain as `SOC` (`Get-ADDomain`; operator report, 23 October 2026)
- [x] `SOC_DC-1`: Confirm expected DNS zone names visible in DNS Manager (23 October 2026)
- [x] `SOC_DC-1`: Enumerate six zones as `Primary` (23 October PowerShell screenshot; AD integration column left blank by misspelled property)
- [x] `SOC_DC-1`: Verify both AD zones as Primary / `IsDsIntegrated=True` (23 October PowerShell screenshot)
- [x] `SOC_DC-1`: Resolve `SOC_DC01.soc.home.lab` against `10.10.10.10` — internal `10.10.10.10` and NAT `192.168.111.130` returned, TTL 1200 (23 October screenshot)
- [x] `SOC_DC-1`: Verify adapter DNS registration — VMnet1 and NAT interfaces both `RegisterThisConnectionsAddress=True` (23 October screenshot)
- [ ] `SOC_DC-1`: Review DNS Server listener interfaces and Name Servers tab; plan a safe change to NAT registration, including rollback and cleanup of stale/NAT A records
- [ ] `W.10.Pro_Soc-Lab Client`: Verify guest IP, gateway and DNS
- [ ] `Ubuntu Wazuh`: Verify guest IP, gateway and DNS
- [ ] `Splunk-SIEM`: Verify guest IP, gateway and DNS
- [ ] `Caldera-SOC`: Verify guest IP, gateway and DNS
- [ ] `Kali Linux`: Verify guest IP, gateway and DNS
- [ ] `Linux Mint`: Verify guest IP, gateway and DNS
- [ ] `Metasploitable3`: Verify guest IP on VMnet1 and confirm NAT Adapter 2 remains disconnected when powered on
