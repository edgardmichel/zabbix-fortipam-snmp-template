# Contributing

Thank you for your interest in improving the Fortinet FortiPAM Zabbix template.

Contributions, validation reports and bug fixes are welcome.

## How to Contribute

1. Fork the repository.
2. Create a branch for your change.
3. Test your changes against a FortiPAM appliance whenever possible.
4. Export the updated template from Zabbix 7.4.x.
5. Verify that no credentials, IP addresses, usernames, serial numbers or other environment-specific secrets are included.
6. Update `CHANGELOG.md` when the change affects template behavior.
7. Open a pull request describing:
   - what changed
   - why it changed
   - FortiPAM version used for validation
   - Zabbix version used for validation

## Bug Reports

When reporting an issue, please include:

- FortiPAM firmware version
- Zabbix version
- SNMP version and security level
- affected item/discovery rule/trigger
- numeric OID involved
- relevant Zabbix error message
- sanitized `snmpget`, `snmpwalk` or `snmpbulkwalk` output

Example:

```text
FortiPAM version: 1.9.1
Zabbix version: 7.4.14
SNMP: v3 / authPriv
Affected OID: 1.3.6.1.4.1.12356.x.x.x
Error: ...
```

## Security and Privacy

Never include any of the following in issues or pull requests:

- SNMP authentication passwords
- SNMP privacy passwords
- API tokens
- private keys
- internal credentials
- production secrets

Sanitize hostnames, public/private IP addresses and serial numbers when necessary.

## Template Design Guidelines

Please keep the following design principles:

- Prefer numeric OIDs in template runtime configuration.
- Use `get[OID]` for scalar objects.
- Use narrow `walk[]` master items for validated tables.
- Use dependent items for values extracted from master walks.
- Use LLD for variable-length tables.
- Avoid a full `walk[1.3.6.1.4.1.12356]`.
- Avoid duplicating generic IF-MIB interface monitoring that is better handled by an existing generic SNMP network template.
- Prefer macros over hard-coded thresholds.
- Avoid triggers that create duplicate alerts for warning/high states.
- Keep vendor-specific behavior documented.

## Testing Checklist

Before submitting a template change:

- [ ] YAML imports successfully into Zabbix.
- [ ] No duplicate item keys.
- [ ] No duplicate UUIDs.
- [ ] Master SNMP items return data.
- [ ] Dependent items receive data.
- [ ] LLD rules discover expected entities.
- [ ] Item prototypes are created correctly.
- [ ] Trigger prototypes resolve correctly.
- [ ] No unexpected `Unsupported` items.
- [ ] No credentials or environment-specific secrets are present.
- [ ] README documentation is updated when necessary.

