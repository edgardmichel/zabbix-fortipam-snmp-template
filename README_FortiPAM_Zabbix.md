# Zabbix Template for Fortinet FortiPAM via SNMP

Community Zabbix template for monitoring **Fortinet FortiPAM** appliances through SNMP.

This template was built and validated against a real FortiPAM appliance and focuses on useful operational, security, licensing, HA, authentication, configuration-change, session and logging metrics rather than importing every object exposed by the Fortinet MIB tree.

> **Tested with**
>
> - Fortinet FortiPAM **1.9.1**
> - Zabbix **7.4.14**
> - SNMP **v3 / authPriv**
> - Authentication protocol: **SHA-256**
> - Privacy protocol: **AES**
> - Net-SNMP tools on Ubuntu/Debian

Other FortiPAM and Zabbix versions may work, but have not been validated with this template.

---

## Features

The template includes monitoring for:

- System serial number and firmware version
- System uptime and reboot reason
- Global CPU and memory utilization
- CPU utilization per processor/core using Low-Level Discovery (LLD)
- Memory capacity, buffers, caches and conserve-mode thresholds
- Disk capacity, disk usage and utilization percentage
- Active IPv4/IPv6 sessions and session rates
- Advanced session counters such as queue-full, clashes and no-listener events
- Authentication request counters and authentication rate
- LDAP connection failures
- DNS request, response, retransmission, forwarding and timeout counters
- Administrator count and administrator-count changes
- FortiPAM license discovery and expiration monitoring
- HA operating mode and HA configuration
- HA member discovery, CPU, memory, sessions, synchronization and checksum information
- Configuration serial/checksum and detection of configuration changes
- Logging destination discovery
- Sent, relayed, cached, failed and dropped log counters
- Log delivery rate
- Log partition scan requirement

The template uses **numeric OIDs** at runtime. Fortinet MIB files are therefore **not required by Zabbix to execute the template**, but they are highly recommended for troubleshooting, validation and translating OIDs to symbolic names.

---

## Low-Level Discovery

The template currently uses the following LLD rules:

| Discovery rule | Purpose |
| --- | --- |
| `FortiPAM: Processor discovery` | Discovers processors/CPU cores and creates per-CPU metrics |
| `FortiPAM: License discovery` | Discovers license contracts and monitors the selected FortiPAM contract |
| `FortiPAM: HA member discovery` | Discovers HA members and creates per-member health/synchronization metrics |
| `FortiPAM: Log device discovery` | Discovers Fortinet logging destinations such as syslog and FortiAnalyzer slots |

The LLD implementation follows the Zabbix 7.4 model:

```text
SNMP walk[] master item
        |
        v
Dependent discovery rule
        |
        v
SNMP walk to JSON
        |
        v
{#SNMPINDEX} + custom LLD macros
        |
        v
Dependent item / trigger prototypes
```

Using a single master `walk[]` for a table significantly reduces the number of SNMP requests sent to the appliance.

---

## Main User Macros

The template exposes the following macros so thresholds can be customized without modifying the items or triggers:

| Macro | Default | Description |
| --- | ---: | --- |
| `{$FORTIPAM.CPU.WARN}` | `80` | Global CPU warning threshold |
| `{$FORTIPAM.CPU.HIGH}` | `90` | Global CPU high threshold |
| `{$FORTIPAM.CPU.CORE.HIGH}` | `95` | Individual CPU/core high threshold |
| `{$FORTIPAM.MEM.WARN}` | `80` | Memory warning threshold |
| `{$FORTIPAM.MEM.HIGH}` | `90` | Memory high threshold |
| `{$FORTIPAM.DISK.WARN}` | `80` | Disk warning threshold |
| `{$FORTIPAM.DISK.HIGH}` | `90` | Disk high threshold |
| `{$FORTIPAM.LICENSE.MATCHES}` | `^FPAM$` | Regex used to select the FortiPAM license contract |
| `{$FORTIPAM.LICENSE.WARN}` | `60d` | License warning period |
| `{$FORTIPAM.LICENSE.HIGH}` | `30d` | License high-severity period |
| `{$FORTIPAM.LICENSE.CRIT}` | `7d` | License critical period |

---

## Requirements

### FortiPAM

SNMP must be enabled on the FortiPAM management interface and the Zabbix server or proxy must be allowed to query it.

This template was tested using SNMPv3 `authPriv`.

Do **not** store SNMP authentication or privacy passwords inside the template. Configure them on the SNMP interface of the Zabbix host.

### Zabbix

- Zabbix Server/Proxy 7.4.x
- SNMP interface configured on the FortiPAM host
- The template imported and linked to the host

For generic interface monitoring (traffic, errors, discards, operational state, etc.), it is recommended to also link a generic SNMP network-device template based on IF-MIB instead of duplicating those items in this FortiPAM-specific template.

---

## Importing the Template

1. Download the YAML template file.
2. In Zabbix, open **Data collection → Templates**.
3. Click **Import**.
4. Select the YAML file.
5. Import the template.
6. Create or open the FortiPAM host.
7. Add an **SNMP interface** pointing to the FortiPAM management IP.
8. Configure the appropriate SNMPv3 security parameters on the host.
9. Link **Fortinet FortiPAM by SNMP** to the host.
10. Check **Monitoring → Latest data** after the first polling cycles.

---

# Fortinet MIB Files

For troubleshooting and development, the following Fortinet MIB files were used:

```text
FORTINET-CORE-MIB.mib
FORTINET-APPLICATION-MIB.mib
```

A relevant detail is that the file:

```text
FORTINET-APPLICATION-MIB.mib
```

declares the module internally as:

```text
FORTINET-FORTIGATE-MIB
```

This matters when using Net-SNMP commands such as `snmptranslate`, because Net-SNMP uses the **MIB module name declared inside the file**, not necessarily the filename.

You can confirm the declared module names with:

```bash
grep -n "DEFINITIONS" /usr/share/snmp/mibs/FORTINET-CORE-MIB.mib
grep -n "DEFINITIONS" /usr/share/snmp/mibs/FORTINET-APPLICATION-MIB.mib
```

Expected result:

```text
FORTINET-CORE-MIB DEFINITIONS ::= BEGIN
FORTINET-FORTIGATE-MIB DEFINITIONS ::= BEGIN
```

> The Fortinet MIB files are vendor-provided files and are **not included in this repository**. Obtain the appropriate MIB package from Fortinet or from the FortiPAM/Fortinet support resources applicable to your firmware version.

---

## Installing Net-SNMP Tools

On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y snmp
```

Depending on the distribution/repository configuration, standard MIB packages may also be available through:

```bash
sudo apt install -y snmp-mibs-downloader
sudo download-mibs
```

The second package is optional and may not be available in every Ubuntu/Debian repository configuration.

Copy the Fortinet MIB files to a directory known by Net-SNMP, for example:

```bash
sudo cp FORTINET-CORE-MIB.mib /usr/share/snmp/mibs/
sudo cp FORTINET-APPLICATION-MIB.mib /usr/share/snmp/mibs/
```

---

## Testing MIB Loading

To list the Fortinet MIB tree using the declared module names:

```bash
snmptranslate \
  -M +/usr/share/snmp/mibs \
  -m +FORTINET-CORE-MIB:FORTINET-FORTIGATE-MIB \
  -Tz
```

To inspect a specific object:

```bash
snmptranslate \
  -M +/usr/share/snmp/mibs \
  -m +FORTINET-CORE-MIB:FORTINET-FORTIGATE-MIB \
  -Td FORTINET-FORTIGATE-MIB::fgSysCpuUsage
```

If Net-SNMP reports missing modules such as `SNMPv2-MIB`, `SNMPv2-SMI`, `SNMPv2-TC`, `IF-MIB`, etc., install the standard MIB dependencies available for your Linux distribution before troubleshooting the Fortinet MIB itself.

---

# SNMP Validation Commands

Never place real SNMP credentials in scripts, documentation or Git repositories.

The examples below use placeholders:

```text
<AUTH_PASSWORD>
<PRIV_PASSWORD>
<FORTIPAM_IP>
```

## Test a scalar OID

Serial number:

```bash
snmpget \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.100.1.1.1.0
```

CPU utilization:

```bash
snmpget \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.101.4.1.3.0
```

## Test the FortiPAM system subtree

```bash
snmpbulkwalk \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.101.4
```

## Test the HA subtree

```bash
snmpbulkwalk \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.101.13.2.1
```

## Test the logging-destination table

```bash
snmpbulkwalk \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.101.21.2.1
```

## Test configuration-change objects

```bash
snmpget \
  -v3 \
  -l authPriv \
  -u Zabbix \
  -a SHA256 \
  -A '<AUTH_PASSWORD>' \
  -x AES \
  -X '<PRIV_PASSWORD>' \
  -On \
  <FORTIPAM_IP> \
  1.3.6.1.4.1.12356.101.22.1.1.0 \
  1.3.6.1.4.1.12356.101.22.1.2.0 \
  1.3.6.1.4.1.12356.101.22.1.3.0
```

---

# Important FortiPAM SNMP Behavior

During testing with FortiPAM 1.9.1, a full walk of the complete Fortinet enterprise subtree:

```text
1.3.6.1.4.1.12356
```

was **not reliable**.

The SNMP agent returned `genError` while traversing part of the session-related branch under:

```text
1.3.6.1.4.1.12356.101.11
```

A relevant characteristic observed during testing was:

```text
SNMP GET on an exact OID     -> works
SNMP GETNEXT on some OIDs    -> may return genError
large enterprise-wide walk   -> may stop with genError
narrow tested table walk     -> works
```

For that reason, this template intentionally uses:

- `get[OID]` for known scalar objects
- narrow `walk[OID,...]` master items only for tables that were individually tested
- dependent items and LLD preprocessing to reduce SNMP requests

Avoid replacing these targeted walks with a single large `walk[1.3.6.1.4.1.12356]` master item.

This behavior may be firmware-specific.

---

# HA Notes

FortiPAM may expose the HA statistics table even when the appliance is running in **Standalone** mode.

In Standalone mode, synchronization-related values can legitimately appear as:

```text
HA mode: Standalone
Synchronization status: Unsynchronized
HA group name: empty
Primary serial: empty
Synchronization timestamp: unavailable
```

The template therefore evaluates HA synchronization alarms only when the system is **not** in Standalone mode.

SNMP `DateAndTime` values that do not contain a valid year are discarded by JavaScript preprocessing instead of being stored as an invalid Unix timestamp.

---

# License Monitoring

The license LLD reads the Fortinet license-contract table and applies the regex stored in:

```text
{$FORTIPAM.LICENSE.MATCHES}
```

The default value is:

```text
^FPAM$
```

This prevents the template from creating unnecessary monitoring items for every Fortinet/FortiGuard contract exposed by the MIB.

The discovered FortiPAM license expiration date is converted to Unix time and can trigger alerts at:

```text
60 days
30 days
7 days
expired
```

Adjust the corresponding macros if different thresholds are required.

---

# Configuration Change Detection

The template monitors:

```text
fgConfigSerial
fgConfigChecksum
fgConfigLastChangeTime
```

These are used to identify changes to the appliance configuration.

The template also calculates how long ago the most recent configuration change occurred.

This is particularly useful for PAM/security appliances because a configuration change may be operationally important even when the device remains fully available.

---

# Logging Monitoring

The log-device LLD discovers the log destinations exposed by the Fortinet MIB, including available syslog and FortiAnalyzer slots.

For each discovered destination the template can monitor:

```text
Enabled state
Logs sent
Logs relayed
Logs cached
Logs failed
Logs dropped
Send rate
```

A log destination being present in the MIB does **not** necessarily mean that it is configured or enabled. Fortinet exposes predefined destination slots, so the template intentionally does not generate an alarm simply because a discovered destination has `Enabled = 0`.

Failed or dropped log counters are more meaningful operational indicators.

---

# Security Recommendations

- Prefer SNMPv3 `authPriv`.
- Use strong, unique authentication and privacy passwords.
- Restrict FortiPAM SNMP access to the Zabbix Server/Proxy IP addresses.
- Never commit SNMP credentials to Git.
- Do not hardcode credentials inside the Zabbix template.
- Rotate credentials that have been exposed in shell history, tickets, chat transcripts or documentation.
- Keep the FortiPAM firmware and Zabbix installation supported and patched.

---

# Troubleshooting

## The master walk has data, but no LLD items are created

Check:

```text
Data collection -> Hosts -> Discovery
```

Then test the discovery preprocessing.

For SNMP-table discovery, use **one** `SNMP walk to JSON` preprocessing step containing all required LLD fields. Do not chain one `SNMP walk to JSON` step per field.

## An LLD item was created but has no Last data

Check:

```text
Data collection -> Hosts -> Items -> Info
```

Then validate the dependent item's `SNMP walk value` preprocessing.

Remember that a dependent item receives a value when its master item receives a **new value**. After changing preprocessing, use **Execute now** on the master item.

## Date/time items become unsupported

Fortinet may return dates as strings or SNMP `DateAndTime` byte sequences.

The template uses JavaScript preprocessing to convert valid values to Unix timestamps and returns `null` for synchronization timestamps that are legitimately unavailable.

## `snmptranslate` cannot load the Fortinet application MIB

Remember that:

```text
Filename: FORTINET-APPLICATION-MIB.mib
Module:   FORTINET-FORTIGATE-MIB
```

Load the module using:

```bash
-m +FORTINET-CORE-MIB:FORTINET-FORTIGATE-MIB
```

rather than assuming the MIB module name is identical to the filename.

---

# Contributing

Contributions are welcome.

Useful contributions include:

- validation against other FortiPAM firmware versions
- additional FortiPAM-specific OIDs
- improvements to trigger thresholds
- SNMP trap support
- dashboards
- additional documentation
- bug reports for firmware-specific SNMP behavior

When reporting an SNMP issue, please include:

```text
FortiPAM version
Zabbix version
SNMP version/security level
Numeric OID
snmpget/snmpwalk output with credentials removed
```

Never include authentication or privacy passwords in issues or pull requests.

---

# Disclaimer

This is a community template and is not an official Fortinet or Zabbix template.

Fortinet, FortiPAM and related product names are trademarks of Fortinet, Inc.  
Zabbix is a trademark of Zabbix LLC.

Use this template in a test environment before deploying it to production.
