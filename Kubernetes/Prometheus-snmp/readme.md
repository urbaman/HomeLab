# snmp v3 example

Cisco:
 version: 3
 auth:
 username: snmpUser
 password: yourPassword
 auth_protocol: SHA
 priv_protocol: DES
 security_level: authPriv
 priv_password: privacyPassword
 walk: