# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-09-28

### Added

- Initial public release of the `Fortinet FortiPAM by SNMP` template.
- Support for Zabbix 7.4.x.
- Validation against FortiPAM 1.9.1.
- SNMPv3 `authPriv` monitoring.
- System serial number and firmware monitoring.
- Global CPU, memory and disk monitoring.
- Per-processor CPU discovery using Low-Level Discovery (LLD).
- Memory cache, buffer and conserve-threshold monitoring.
- IPv4 and IPv6 session monitoring.
- Session-rate monitoring.
- Advanced session counters, including queue-full and clash conditions.
- Authentication request counters and authentication request rate.
- LDAP authentication failure monitoring.
- DNS request, response, forwarding, retransmission and timeout monitoring.
- Administrator-count monitoring and change detection.
- FortiPAM license discovery and license-expiration triggers.
- HA mode and HA configuration monitoring.
- HA member discovery with:
  - serial number
  - hostname
  - CPU utilization
  - memory utilization
  - network usage
  - sessions
  - packet and byte counters
  - synchronization status
  - synchronization timestamps
  - global checksum
  - primary serial
- Configuration-change monitoring using Fortinet configuration serial/checksum objects.
- Time-since-last-configuration-change calculated item.
- Logging destination discovery.
- Logging metrics for sent, relayed, cached, failed and dropped logs.
- Log send-rate dependent item.
- Log partition scan monitoring.
- Reusable user macros for CPU, memory, disk and license thresholds.
- Value maps for Fortinet boolean, HA, logging and related states.
- Documentation for Fortinet MIB loading and Net-SNMP validation.
- Documentation for known FortiPAM SNMP walk limitations.

### Notes

- The template uses numeric OIDs at runtime and does not require Fortinet MIB files to be installed on the Zabbix server.
- Fortinet MIB files are recommended for troubleshooting, development and OID translation.
- A full walk of the complete Fortinet enterprise subtree may fail on some FortiPAM firmware versions. The template therefore uses exact `get[]` operations and narrow, validated `walk[]` calls.

