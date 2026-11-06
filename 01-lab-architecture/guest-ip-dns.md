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

### DNS Server listening interfaces — screenshot verified 24 October 2026

A screenshot of **DNS Manager → SOC_DC01 Properties → Interfaces** shows **Listen on: All IP addresses** selected. The pane lists (with checkmarks) the following server interfaces:

- Internal IPv4: `10.10.10.10` (VMnet1)
- NAT IPv4: `192.168.111.130` (VMnet8)
- Two IPv6 link-local interface addresses (values deliberately omitted from the portfolio)

The `Only the following IP addresses` radio control is **not** selected. Therefore, DNS Server is configured to listen on all bound addresses, **including the NAT adapter**. The checkboxes shown in the inactive selection list are not evidence of an active restricted listener policy. This is a **DNS service listener** setting and is distinct from the previously verified `RegisterThisConnectionsAddress=True` DNS client configuration.

**Security finding:** A multihomed AD Domain Controller is both publishing a NAT-related A record and listening for DNS queries on the NAT-facing interface. This can widen the set of lab systems able to reach the DNS service; the screenshot does **not** prove which systems can actually reach it through VMware routing/firewall controls. Restricting DNS Server listeners would **not by itself** delete existing NAT A records.

A DNS/Active Directory initial-synchronisation warning is visible in Server Manager behind the properties window. Its current/historical status is **unknown**; do not conclude that directory replication is presently broken. Verify separately if still active.

**No changes were applied** during this observation. An interface-listening change and NAT address-registration cleanup should be planned together only after validation and a usable rollback point.

### Zone Name Servers tab — screenshot verified 24 October 2026

The operator opened **DNS Manager → Forward Lookup Zones → soc.home.lab → Properties → Name Servers**. One entry is listed:

| Name server FQDN | Listed IPv4 address | Evidence |
| --- | --- | --- |
| `soc_dc01.soc.home.lab.` | `10.10.10.10` | Zone Properties screenshot |

**Assessment:** The `soc.home.lab` Name Servers list contains the DC's internal VMnet1 IPv4 address only; the NAT address `192.168.111.130` does **not** appear in this list. There is **no evidence here that the Name Servers list itself needs changing**. This result is distinct from the direct `Resolve-DnsName` A-record query, which returned both internal and NAT addresses for the same hostname.

**Scope caution:** The Name Servers tab shows one zone's configuration; it does not establish what every NS/glue resolver might return, verify a separate `_msdcs.soc.home.lab` zone, or negate the undesirable NAT A records shown in DNS Manager.

### _msdcs Name Servers tab — screenshot verified 24 October 2026

In DNS Manager, the operator selected **Forward Lookup Zones → _msdcs.soc.home.lab → Properties → Name Servers**.

| Zone | Name server listed | IP address displayed in DNS Manager |
| --- | --- | --- |
| `_msdcs.soc.home.lab` | `soc_dc01.soc.home.lab.` | **Unknown** |

**Finding:** The separate forest locator zone has the expected server **name**, but the Name Servers tab displays **Unknown** for the address. The screenshot does **not** establish whether the underlying NS resource record is missing, invalid, or correct; nor does it establish whether DNS resolution currently fails. The same host's A record previously resolved to both `10.10.10.10` and `192.168.111.130` when queried directly against the internal DNS server. The GUI value may reflect an address-lookup/validation limitation and must not be equated with a broken DNS zone.

**No changes made.** Do not edit the NS row or change other records based on the GUI display alone.

### _msdcs NS record resolution — screenshot verified 24 October 2026

The operator ran this read-only query against the internal DNS server:

```powershell
Resolve-DnsName -Name _msdcs.soc.home.lab -Type NS -Server 10.10.10.10
```

| Response field | Value | DNS section |
| --- | --- | --- |
| Queried zone | `_msdcs.soc.home.lab` | Question |
| NS target | `soc_dc01.soc.home.lab` | Answer |
| NS TTL | 3600 seconds (1 hour) | Answer |
| Additional A record for `soc_dc01.soc.home.lab` | `10.10.10.10` | Additional |
| Additional record TTL | 3600 seconds (1 hour) | Additional |

**Finding:** The DNS server **successfully returns the expected NS record** and an additional A record pointing to the DC's internal address `10.10.10.10`. This provides positive evidence that the earlier `Unknown` address in DNS Manager's Name Servers tab is **not evidence of a missing or unresolvable NS record** in this direct test. The GUI discrepancy remains unexplained but does not currently justify editing the NS entry.

**Limits:** An NS response and a supporting Additional-section A record do not confirm the AD DC locator SRV records, workstation authentication, DNS client selection, or AD replication health. This query's Additional section lists the internal address; the earlier direct hostname A-record query independently returned **both** `10.10.10.10` and NAT `192.168.111.130`. **Do not infer that the NAT A-record problem has disappeared.**

### Active Directory DC locator SRV lookup — screenshot verified 24 October 2026

A PowerShell screenshot in the named `SOC_DC-1` VMware VM shows a successful read-only query:

```powershell
Resolve-DnsName -Name _ldap._tcp.dc._msdcs.soc.home.lab -Type SRV -Server 10.10.10.10
```

| Field | Observed value | DNS section |
| --- | --- | --- |
| Service name | `_ldap._tcp.dc._msdcs.soc.home.lab` | Answer |
| Type | SRV | Answer |
| Target | `soc_dc01.soc.home.lab` | Answer |
| Port | `389` (LDAP) | Answer |
| Priority | `0` | Answer |
| Weight | `100` | Answer |
| SRV TTL | `600` seconds | Answer |
| Additional host A | `10.10.10.10` | Additional |
| Additional A TTL | `3600` seconds | Additional |

**Finding:** The internal DNS server returns the expected LDAP domain-controller locator SRV record and includes the DC's VMnet1 IPv4 address in an Additional A record. This verifies **DNS service discovery data**, not actual LDAP TCP connectivity, authentication, Group Policy application, other locator records or healthy AD replication.

The earlier direct A lookup of `soc_dc01.soc.home.lab` returned **both** `10.10.10.10` and NAT `192.168.111.130`. The SRV query's Additional A value does **not** demonstrate the NAT record is gone. NAT DNS-client registration, all-interface DNS listening and possible stale records remain open findings.

**Next read-only diagnostic — DC DNS health:**

```powershell
dcdiag /test:dns /s:SOC_DC01
```

This runs Windows Domain Controller Diagnostics' DNS test against `SOC_DC01`; collect any failures or warnings without changing services or zones. Before remediation, verify a suitable Active Directory-aware backup and rollback procedure; do not treat a VM snapshot alone as a substitute for a proper system-state backup.

**Outstanding:** Assess Domain Controller DNS diagnostic health; verify Windows 10 client IP and AD DNS; evaluate whether `192.168.254.10` is stale; prepare an Active Directory-aware backup and rollback plan before changing DNS registration/listeners or deleting records.

**Other pending checks:** Determine whether `192.168.254.10` is stale, verify current Windows 10 guest address and whether it uses internal AD DNS, review DNS Server interface binding and NAT adapter registration, and create a rollback checkpoint before changing production-like AD lab settings.

### Attempted DNS remediation — screenshot verified 24 October 2026

PowerShell on `SOC_DC01` shows these commands run with **no visible errors**:

```powershell
Set-DnsClient -InterfaceAlias "Ethernet1" -RegisterThisConnectionsAddress $false
Set-DnsClientServerAddress -InterfaceAlias "Soc nw" -ServerAddresses 10.10.10.10
Set-DnsClientServerAddress -InterfaceAlias "Ethernet1" -ServerAddresses 10.10.10.10
```

The commands are visible more than once in the terminal history. Their lack of errors suggests execution, but the screenshot does not independently confirm the settings persisted; rechecking them is possible later if necessary. It does **not** confirm whether the DNS Server listener was restricted.

**Post-command verification:** Two direct queries of `soc_dc01.soc.home.lab` against `10.10.10.10` still returned both A records (`192.168.111.130` and `10.10.10.10`), each with TTL 1200 seconds.

**Interpretation:** Disabling NAT adapter dynamic DNS registration does **not automatically remove pre-existing A records**. Cleanup is still needed. Avoid removing records by name alone: the DC's correct internal `10.10.10.10` record must remain.

**Focused next action (not yet performed):** After confirming a valid backup/rollback, remove only the unwanted `soc_dc01` A record with `RecordData="192.168.111.130"`, using DNS Manager or `Remove-DnsServerResourceRecord -ZoneName "soc.home.lab" -Name "soc_dc01" -RRType A -RecordData "192.168.111.130"`. Independently review the zone-apex A records (`192.168.111.130`, `192.168.254.10`) and ensure a correct internal apex A record before changing those records. Re-query DNS and move back to verifying the other VMs; only revisit if unwanted records return.

### Attempted removal of NAT A record — failed (24 October 2026)

A PowerShell screenshot shows an attempt to remove the `soc_dc01` Host (A) record at `192.168.111.130` using `Remove-DnsServerResourceRecord -ZoneName "soc.home.lab" -Name "soc_dc01" -RRType A -RecordData "192.168.111.130"`.

The DNS management cmdlet returned **Win32 9714 / `DNS_ERROR_NAME_DOES_NOT_EXIST`**, reporting failure to get the `soc_dc01` record in `soc.home.lab`. It **did not delete** the requested record. The subsequent direct DNS query still returned both `192.168.111.130` and `10.10.10.10` (1200-second TTLs).

**Outcome:** NAT A record remains; no deletion success claimed. This error alone does not establish whether the queried A response was stale, whether the record is visible to the server-management interface, or why this particular cmdlet could not select it.

**Focused manual remediation (pending):** In DNS Manager → Forward Lookup Zones → `soc.home.lab`, find the specific `soc_dc01` A record with address `192.168.111.130`. After a verified rollback/backup, delete **only that address-specific record**, preserving the `soc_dc01` A record `10.10.10.10` and all NS/SRV records. Confirm removal by re-querying `soc_dc01.soc.home.lab` against `10.10.10.10`. The zone-apex NAT and potentially stale addresses remain separately identified and require careful review. Prioritise returning to the GitHub VM inventory after this focused correction.

### Deleted DNS Manager A records reported; old answer persists — 24 October 2026

The lab owner reports manually deleting all visible `192.168.111.130` DNS records from DNS Manager. A subsequent screenshot of `Resolve-DnsName soc_dc01.soc.home.lab -Type A -Server 10.10.10.10` **still returns** `192.168.111.130` and `10.10.10.10`, both TTL 1200 seconds. The deletion is **operator-reported**; the DNS query is **screenshot-verified**.

**Status at the time of this screenshot:** Unresolved. The returned NAT address may reflect DNS cache/state or re-registration by another DC service. A **later owner report** states the issue was fixed using Configure NIC teaming settings, but the exact change and post-fix DNS query were not supplied (see resolution note below).

**Focused next check (not yet executed):**

```powershell
Clear-DnsServerCache -Force
Clear-DnsClientCache
Resolve-DnsName -Name soc_dc01.soc.home.lab -Type A -Server 10.10.10.10
```

These commands flush DNS Server and local client caches and then query the internal DNS server again; they do **not** delete authoritative zone records. If the NAT A record persists, inspect zone data/automatic registrations only when returning to the issue rather than delaying the GitHub lab documentation indefinitely. The DNS server's Interfaces tab was previously set to listen on all addresses, which can also contribute to address registration.

### Data handling

The submitted screenshot contained MAC addresses, IPv6 link-local addresses and Windows guest UI information. To reduce unnecessary disclosure, these identifier values and the raw PDF were **not** committed; only the operationally relevant IPv4, DNS and adapter settings are transcribed.

### Domain Controller DNS troubleshooting disposition — owner report (24 October 2026)

The lab owner stated that they **fixed the DNS problem through “Configure NIC teaming” settings**. This is recorded as **operator-reported resolved** to preserve the outcome of their troubleshooting. No exact team settings, after-state DNS command output, or before/after NIC configuration was supplied, so the actual mechanism and record-level resolution remain **unverified by screenshot**. Prior diagnostic results above accurately describe the state when those tests were captured.

**Scope decision:** Stop the detailed DC DNS investigation and resume documenting the other VMware guests for the GitHub SOC lab portfolio. Do not assume NIC teaming exists on other machines or repeat this fix without verifying their topology.

## W.10.Pro_Soc-Lab Client — Windows 10 guest IPv4 configuration

> **Evidence:** VMware Workstation screenshot with `W.10.Pro_Soc-Lab Client` selected, displaying `ipconfig /all` in PowerShell on **24 October 2026, approximately 12:22 PM (Melbourne time)**. The visible console starts partway through the output, so the guest **hostname and domain membership are not independently shown**. An earlier lab note used hostname `DonW10`; do not mistake that historical hostname for a value verified in this screenshot.

### Internal SOC interface — Windows `Ethernet1`

| Property | Observed value |
| --- | --- |
| Network association | `VMnet1` host-only, inferred by matching the confirmed `10.10.10.0/24` subnet to VMware network settings |
| IPv4 address | `10.10.10.11` (Preferred) |
| Subnet mask | `255.255.255.0` (`/24`) |
| DHCP enabled | **No** — static IPv4 |
| Default gateway | **Blank / no gateway shown** |
| DNS server | `10.10.10.10` (lab Domain Controller) |
| NetBIOS over TCP/IP | Enabled |

### NAT interface — Windows `Ethernet0`

| Property | Observed value |
| --- | --- |
| Network association | `VMnet8` NAT, inferred by matching the confirmed `192.168.111.0/24` subnet to VMware network settings |
| IPv4 address | `192.168.111.132` (Preferred) |
| Subnet mask | `255.255.255.0` (`/24`) |
| DHCP enabled | **Yes** |
| Default gateway | `192.168.111.2` |
| DHCP server | `192.168.111.254` |
| DNS server | `192.168.111.2` |
| Connection-specific DNS suffix | `localdomain` |
| Primary WINS server | `192.168.111.2` |
| NetBIOS over TCP/IP | Enabled |

The visible `ipconfig /all` output reports **IP Routing Enabled: No**. Its visible DNS suffix search list contains `localdomain`. These observations do **not** verify domain join, successful authentication, or actual application connectivity.

**Correction to earlier inventory:** The client's previously recorded internal address `10.10.10.15` is **outdated** relative to the live screenshot: its internal IPv4 address is now `10.10.10.11`. The DNS Manager screenshot previously showed a `DonW10` record at `10.10.10.50`; that is *also not* the IP shown on the running Windows 10 interface. This discrepancy is documented only, not treated as an invitation to divert the architecture inventory into DNS troubleshooting.

**Scope:** This is a configuration transcription for the GitHub portfolio. No guest network changes were made and no raw screenshots, MAC addresses, or IPv6 host identifiers were committed.

## Ubuntu Wazuh — IPv4 interface inventory

> **Evidence:** VMware Workstation `Ubuntu Wazuh` selected, showing Linux console output of `ip -br -4 addr` on **24 October 2026 (approximately 4:52 PM, Melbourne time)**. No raw screenshot is stored in the repository.

### Guest identity and IPv4 addresses

| Field | Screenshot result |
| --- | --- |
| VMware display name | `Ubuntu Wazuh` |
| Linux hostname (terminal prompt / login banner) | `wazuh-server` |
| Operating system banner | Ubuntu 26.04.1 LTS (as displayed) |
| Interface `lo` | `127.0.0.1/8` — state `UNKNOWN` (loopback) |
| Interface `ens160` | `10.10.10.20/24` — state `UP` |
| Interface `ens192` | `192.168.111.128/24` — state `UP` (output also shows `metric 100`) |

**Network association:** The address `10.10.10.20/24` matches the verified VMnet1 host-only `10.10.10.0/24` subnet, and `192.168.111.128/24` matches VMnet8 NAT `192.168.111.0/24`. This is consistent with the earlier VMware adapter record (Adapter 1 Custom VMnet1; Adapter 2 NAT mode). The guest interface-to-VMnet association has not been directly corroborated by MAC matching, but the subnet evidence is strong.

**Initial screenshot scope:** `ip -br -4 addr` shows interface states and addresses but not gateways, DNS or service health. The follow-up screenshot below supplies routes and DNS settings.

### IPv4 routing and DNS — screenshot verified 24 October 2026

A second `Ubuntu Wazuh` VMware screenshot at approximately 4:57 PM (Melbourne time) includes output of:

```bash
ip -4 route
resolvectl status
```

**IPv4 routes visible:**

| Destination | Via / device | Source | Notes |
| --- | --- | --- | --- |
| `default` | `192.168.111.2` via `ens192` | `192.168.111.128` | DHCP, metric 100 |
| `10.10.10.0/24` | directly connected via `ens160` | `10.10.10.20` | Internal lab network |
| `192.168.111.0/24` | directly connected via `ens192` | `192.168.111.128` | NAT network, metric 100 |
| `192.168.111.2` | via `ens192` (scope link) | `192.168.111.128` | DHCP-learned host route, metric 100 |

**DNS resolver settings visible:**

| Linux interface | DNS servers listed | Current DNS server | Other |
| --- | --- | --- | --- |
| `ens160` | `10.10.10.10`, `1.1.1.1` | `1.1.1.1` | DNS DefaultRoute: yes |
| `ens192` | `192.168.111.2` | `192.168.111.2` | DNS DefaultRoute: yes |

`resolvectl` reports **resolv.conf mode: stub**. Its per-link `DefaultRoute: yes` applies to **DNS resolution routing**, not a second IPv4 default gateway. The IPv4 route table contains one default route via VMnet8 `192.168.111.2`. The route output identifies the VMnet8 NAT address as DHCP-sourced; the internal address's configuration mechanism is not proved by this command alone.

**Additional observation:** The screenshot also shows two `ping -c 4 10.10.10.11` attempts, each reporting 4 transmitted, 0 received and 100% packet loss. This is recorded as **ICMP echo non-response** from Wazuh to the Windows 10 internal address at capture time. ICMP non-response alone does **not** prove the endpoint is offline or that other traffic is blocked. This inventory step makes no troubleshooting changes.

**Status:** Wazuh IPv4 address, gateway, route and DNS inventory complete. Actual DNS query success, Wazuh service health, and full network connectivity remain outside the current documentation check.

## Splunk-SIEM — IPv4, routing and DNS verification

> **Evidence:** User-provided screenshot of VMware Workstation with `Splunk-SIEM` selected, showing `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` in the Linux terminal on **6 November 2026, around 18:03 AEDT**. Login banner shows 07:03 UTC, consistent with local time. Textual transcription only; no raw screenshot or identifying MAC addresses committed.

### Guest identity and IPv4 addresses

| Item | Screenshot-verified value |
| --- | --- |
| VMware VM name | `Splunk-SIEM` |
| Linux hostname (terminal prompt) | `splunk-siem` |
| `lo` | `127.0.0.1/8`, state UNKNOWN (loopback) |
| `ens33` | `10.10.10.69/24`, state UP (internal lab subnet) |
| `ens34` | `192.168.111.135/24`, state UP; metric 100 (VMware NAT subnet) |

The guest address subnets align with the previously recorded VMware wiring: Adapter 1 Custom `VMnet1` (`10.10.10.0/24`), Adapter 2 Custom `VMnet8` (`192.168.111.0/24`). The exact guest interface-to-VMnet correspondence is consistent with, rather than independently MAC-matched to, the VMware settings. The internal IPv4 matches the historical `10.10.10.69` record.

### IPv4 routes

| Destination | Next hop / interface | Source or details |
| --- | --- | --- |
| `default` | `192.168.111.2` via `ens34` | `proto dhcp`, src `192.168.111.135`, metric 100 |
| `10.10.10.0/24` | directly connected via `ens33` | src `10.10.10.69` |
| `192.168.111.0/24` | directly connected via `ens34` | src `192.168.111.135`, metric 100 |
| `192.168.111.2` | scope-link host route via `ens34` | `proto dhcp`, src `192.168.111.135`, metric 100 |

There is **one IPv4 default gateway**, `192.168.111.2` on the NAT interface. The NAT route is labelled DHCP in the output; the screenshot does not directly establish whether `ens33` uses static configuration.

### DNS configuration

`resolvectl status` shows `resolv.conf mode: stub` and the following per-link settings:

| Interface | Current scopes | DNS servers | Current DNS | DNS DefaultRoute |
| --- | --- | --- | --- | --- |
| `ens33` | none | None listed | None listed | no |
| `ens34` | DNS | `192.168.111.2` | `192.168.111.2` | yes |

**Interpretation:** The documented Splunk VM currently uses the VMware NAT DNS resolver on `ens34`. **DNS DefaultRoute** is a `systemd-resolved` DNS-routing decision; it is different from the kernel's IPv4 default route, although both point to the NAT side here. This screenshot does **not** establish whether Splunk's application forwards or receives logs successfully, or whether the VM is joined to the `soc.home.lab` domain.

**Status:** Network address, gateway, routes and resolver inventory complete. No settings were changed; we are not launching a DNS or connectivity investigation as part of the GitHub architecture inventory.

## Caldera-SOC — IPv4, routing and DNS verification

> **Evidence:** User-provided VMware Workstation screenshot with `Caldera-SOC` selected, showing `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` in the terminal on **6 November 2026 at approximately 18:11 AEDT**. The login banner reports 07:10 UTC. Textual transcription only; raw screenshot and identifiers are not committed.

### Guest identity and addresses

| Item | Screenshot result |
| --- | --- |
| VMware VM display name | `Caldera-SOC` |
| Guest hostname (terminal prompt) | `caldera-soc` |
| `lo` (loopback) | `127.0.0.1/8`, state UNKNOWN |
| `ens33` | `192.168.111.137/24`, state UP, metric 100 (NAT subnet) |
| `ens34` | `10.10.10.99/24`, state UP (internal SOC subnet) |

This matches the previously identified VMware adapter assignments: Adapter 1 Custom `VMnet8` (NAT) and Adapter 2 Custom `VMnet1` (host-only). The guest interface-to-VMnet mapping is consistent with the matching subnets and VMware adapter ordering; no MAC address correlation was performed. The current internal address confirms the earlier `10.10.10.99` record.

### IPv4 routes

| Destination | Route / interface | Details |
| --- | --- | --- |
| `default` | via `192.168.111.2` on `ens33` | `proto dhcp`, src `192.168.111.137`, metric 100 |
| `10.10.10.0/24` | directly connected on `ens34` | src `10.10.10.99` |
| `192.168.111.0/24` | directly connected on `ens33` | src `192.168.111.137`, metric 100 |
| `192.168.111.2` | scope-link host route on `ens33` | `proto dhcp`, src `192.168.111.137`, metric 100 |

There is one IPv4 default gateway: `192.168.111.2` on NAT `ens33`. The NAT routes explicitly say DHCP; this screenshot does not prove how the internal `ens34` address is persistently configured.

### DNS resolver configuration

`resolvectl status` shows `resolv.conf mode: stub`.

| Interface | Current scopes | DNS server(s) shown | Current DNS | DNS DefaultRoute |
| --- | --- | --- | --- | --- |
| `ens33` | DNS | `192.168.111.2` | `192.168.111.2` | yes |
| `ens34` | none | None listed | None listed | no |

**Interpretation:** This screenshot verifies CALDERA's guest network settings; it does **not** verify that CALDERA's application service is running, DNS resolution succeeds, or simulated activity is reaching the SOC monitoring tools. `systemd-resolved`'s DNS `DefaultRoute` is distinct from the IPv4 kernel default route.

**Status:** IPv4 addresses, routes, default gateway and DNS inventoried. No configuration changes made.

## Kali Linux — IPv4, routing and DNS observations

> **Evidence:** Two user-supplied VMware Workstation screenshots with `Kali Linux` selected, dated **6 November 2026 at approximately 18:21 AEDT**. They show terminal commands `ip -br -4 addr`, `ip -4 route` and later `resolvectl status`. Screenshot evidence is transcribed; no raw screenshots, MAC addresses or local account identifiers were committed.

### IPv4 interface inventory

| Interface | State | IPv4 address | Role / evidence |
| --- | --- | --- | --- |
| `lo` | UNKNOWN | `127.0.0.1/8` | Loopback |
| `eth0` | UP | `10.10.10.30/24` | Internal SOC network (VMnet1 subnet) |
| `eth1` | UP | `192.168.111.129/24` | NAT network (VMnet8 subnet) |
| `docker0` | UP | `172.17.0.1/16` | Local Docker bridge; **not** a third VMware network |

The guest hostname displayed in the terminal prompt is `kali-soc`. Interface-to-VMware association is consistent with prior owner-mapped VMware settings (Adapter 1 Host-only; Adapter 2 NAT); no guest MAC-to-VMware adapter match was performed.

### IPv4 routing table

| Destination | Via / interface | Source / metric |
| --- | --- | --- |
| `default` | `192.168.111.2` via `eth1` | DHCP; src `192.168.111.129`; metric 101 |
| `10.10.10.0/24` | Direct on `eth0` | src `10.10.10.30`; metric 100 |
| `172.17.0.0/16` | Direct on `docker0` | src `172.17.0.1` |
| `192.168.111.0/24` | Direct on `eth1` | src `192.168.111.129`; metric 101 |

There is one IPv4 default gateway: `192.168.111.2` on the NAT side. `proto dhcp` on the default route supports that it was learned via DHCP. The capture does not prove how the internal `eth0` IPv4 address is persistently provisioned.

### Resolver output and package errors

The first screenshot shows `resolvectl: command not found`, followed by an attempt to install `systemd-resolved` using APT. In the later screenshot, `resolvectl status` runs and reports `resolv.conf mode: stub`; none of the displayed interfaces (`eth0`, `eth1`, `docker0` and a temporary `veth` interface) has a **DNS server address listed** or DNS scope shown. `eth0` and `eth1` both display `DefaultRoute: no` in the **resolver** status; this is separate from the kernel's confirmed IPv4 default route on `eth1`.

The terminal also shows APT **temporary failure resolving `http.kali.org`** and failed archive downloads. This is an observed DNS/name-resolution problem during package retrieval, **not** proof of an internet routing outage or the root cause of resolver configuration. Since a package install was attempted between screenshots, it would be inaccurate to describe the entire session as read-only or claim no guest configuration may have changed.

**Scope:** This evidence completes the Kali guest address/routing inventory and captures the DNS information available at the time. Resolver server addresses were **not present in the displayed status**, so none are inferred. We are recording the observation and continuing the GitHub inventory rather than diagnosing Kali's package-install failure.

## Linux Mint — IPv4, routing and DNS verification

> **Evidence:** VMware Workstation screenshot with `Linux Mint` selected, showing `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` in a terminal on **6 November 2026 at approximately 18:26 AEDT**. Text-only evidence recorded; no user account, MAC address or raw screenshot committed.

### Guest identity and IPv4 interfaces

| Interface / item | Screenshot value | Role |
| --- | --- | --- |
| VMware display name | `Linux Mint` | Linux packet-monitoring / Wireshark VM |
| Terminal hostname | `donvi-Virtual-Machine` | As shown in terminal prompt |
| `lo` | `127.0.0.1/8`, state UNKNOWN | Loopback |
| `ens160` | `10.10.10.25/24`, state UP | Internal SOC / VMnet1 subnet |
| `ens192` | `192.168.111.133/24`, state UP | VMware NAT / VMnet8 subnet |

The interface addresses match the previously verified VMware Virtual Network Editor subnets and the earlier Library setting of **Adapter 1 Host-only + Adapter 2 NAT**. Association by address is consistent; no MAC-based binding comparison was performed. The screenshot does not show whether `ens160` is statically configured.

### IPv4 routing

| Destination | Via / interface | Details |
| --- | --- | --- |
| `default` | `192.168.111.2` via `ens192` | `proto dhcp`, src `192.168.111.133`, metric 101 |
| `10.10.10.0/24` | directly connected `ens160` | `proto kernel`, src `10.10.10.25`, metric 100 |
| `192.168.111.0/24` | directly connected `ens192` | `proto kernel`, src `192.168.111.133`, metric 101 |

The **single IPv4 default route** uses VMware NAT `192.168.111.2` through `ens192`.

### DNS resolver

The `resolvectl status` output reports `resolv.conf mode: stub`. Link 2 (`ens160`) shows `Current Scopes: none` with no server address listed. Link 3 (`ens192`) shows:

| Setting | Screenshot result |
| --- | --- |
| Current Scopes | DNS |
| Current DNS Server | `192.168.111.2` |
| DNS Servers | `192.168.111.2` |
| DNS Domain | `localdomain` |

`resolvectl`'s per-link `+DefaultRoute` for `ens192` refers to DNS request routing, not an additional kernel default gateway. No DNS server is shown on internal `ens160`.

**Status:** Linux Mint's guest addresses, routing, default gateway and resolver settings documented. This does not prove packet-monitor capture interfaces, traffic visibility, or Wireshark functionality. No changes made.

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
- [x] `SOC_DC-1`: DNS Server Interfaces tab shows `All IP addresses`, including NAT `192.168.111.130` (24 October screenshot)
- [x] `SOC_DC-1`: `soc.home.lab` Name Servers tab lists `soc_dc01.soc.home.lab.` with `10.10.10.10` only (24 October screenshot)
- [x] `SOC_DC-1`: Inspect `_msdcs.soc.home.lab` Name Servers tab — server `soc_dc01.soc.home.lab.` listed, GUI IP shown as `Unknown` (24 October screenshot)
- [x] `SOC_DC-1`: `_msdcs.soc.home.lab` NS query returned `soc_dc01.soc.home.lab` with Additional A `10.10.10.10` (24 October screenshot)
- [x] `SOC_DC-1`: Verify LDAP domain-controller locator SRV record (target `soc_dc01.soc.home.lab`, port 389, Additional A `10.10.10.10`; 24 October screenshot)
- [x] `SOC_DC-1`: Attempt DNS client remediation via PowerShell; no visible errors, but repeated DNS query still returns NAT and internal DC A records (24 October screenshot)
- [ ] `SOC_DC-1`: PowerShell NAT A-record deletion failed with Win32 9714 on 24 October; attempt precise GUI removal after backup, verify result and return to VM inventory
- [x] `W.10.Pro_Soc-Lab Client`: Verify guest internal IPv4 `10.10.10.11`/DNS `10.10.10.10` and NAT IPv4 `192.168.111.132`/gateway `192.168.111.2` from 24 October screenshot
- [x] `Ubuntu Wazuh`: Verify IPv4 `ens160=10.10.10.20/24` and `ens192=192.168.111.128/24`, both UP (24 October screenshot)
- [x] `Ubuntu Wazuh`: Capture IPv4 default gateway `192.168.111.2` via `ens192` and DNS servers on `ens160`/`ens192` (24 October screenshot); ICMP to Windows 10 did not respond
- [x] `Splunk-SIEM`: IPv4 `ens33=10.10.10.69/24`, `ens34=192.168.111.135/24`; default gateway `192.168.111.2` on `ens34`; DNS `192.168.111.2` on `ens34` (6 November 2026 screenshot)
- [x] `Caldera-SOC`: Verify IPv4 `ens34=10.10.10.99/24`, `ens33=192.168.111.137/24`, gateway `192.168.111.2` on `ens33`, DNS `192.168.111.2` via `ens33` (6 November screenshot)
- [x] `Kali Linux`: IPv4 `eth0=10.10.10.30/24`, `eth1=192.168.111.129/24`, Docker bridge `172.17.0.1/16`, NAT default gateway `192.168.111.2`; resolver shows no DNS servers, APT lookup failure observed (6 November screenshots)
- [x] `Linux Mint`: Verified `ens160=10.10.10.25/24`, `ens192=192.168.111.133/24`, NAT gateway/DNS `192.168.111.2`, resolver domain `localdomain` (6 November screenshot)
- [ ] `Metasploitable3`: Verify guest IP on VMnet1 and confirm NAT Adapter 2 remains disconnected when powered on
