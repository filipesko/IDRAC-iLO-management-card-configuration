# Automated BMC configuration

Standalone Ansible automation for HPE ProLiant iLO 5/6/7 and Dell PowerEdge
iDRAC 9/10 management controllers.

The playbook discovers the controller vendor through Redfish and automatically
runs the matching HPE or Dell configuration path. It configures:

- time zone
- static NTP servers
- management-controller hostname and domain
- static DNS servers
- SNMP version 1 and an SNMPv1 trap destination

## Requirements

- Ansible Core on the control host
- HTTPS connectivity to the management controller
- a BMC account with configuration privileges
- Redfish enabled on the controller

Install the Dell OpenManage collection:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Configuration

The playbook accepts exactly four runtime input variables:

```yaml
bmc_ip: "192.0.2.10"
bmc_username: "Administrator"
bmc_hostname: "bmc-pve01"
region: "EMEA"
```

`region` is case-insensitive and must be one of `EMEA`, `NASA`, or `APAC`.
The matching file is loaded automatically:

- `vars/emea_vars.yml`
- `vars/nasa_vars.yml`
- `vars/apac_vars.yml`

These files contain the remaining settings, including the password, shared
`domain_name`, DNS, NTP, timezone, certificate policy, and SNMP
destination/community.
Replace all example values before use.

Each regional file specifies its timezone once using the IANA-style convention
accepted by iDRAC:

```yaml
bmc_timezone: "Europe/Brussels"
```

For HPE iLO, the playbook converts this value through
`ilo_timezone_name_map` and selects the matching `Name` or `Value` advertised
by that controller's Redfish `TimeZoneList`. The supplied map includes common
US, European, and APAC zones.

Encrypt the regional files because they contain credentials and SNMP community
strings:

```bash
ansible-vault encrypt vars/emea_vars.yml
ansible-vault encrypt vars/nasa_vars.yml
ansible-vault encrypt vars/apac_vars.yml
```

## Run

From the project root:

```bash
ansible-playbook --ask-vault-pass playbooks/configure-bmc.yml \
  -e bmc_ip=192.0.2.10 \
  -e bmc_username=Administrator \
  -e bmc_hostname=bmc-pve01 \
  -e region=EMEA
```

The first play queries `/redfish/v1/Managers`, recognizes HPE iLO or Dell
iDRAC, and prints the detected controller type. Unsupported BMCs stop with a
clear error before any configuration change is attempted.
