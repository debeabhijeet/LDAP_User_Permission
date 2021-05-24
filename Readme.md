# Sudo Permissions for LDAP Group & User

## Configuring Server  

### Adding Sudo schema

```shell
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/sudo.ldif
```

### OpenLDAP SUDOers Organization Unit

Create SUDOers ou on your Organization directory structure.

```shell
vi sudoersou.ldif

dn: ou=SUDOers,dc=example,dc=com
objectClass: organizationalUnit
ou: SUDOers
```

Updating the OpenLDAP database with the SUDOers organizational unit entry above.

```shell
ldapadd -Y EXTERNAL -H ldapi:/// -f sudoersou.ldif
                OR
ldapadd -x -D cn=Manager,dc=example,dc=com -W -f sudoeresou.ldif
```

### Create a Ldap group with sudo power

```shell
vi sudogroup.ldif

dn: cn=admingroup,ou=Group,dc=example,dc=com
objectClass: posixGroup
cn: admingroup
gidNumber: 1020

dn: cn=%admingroup,ou=SUDOers,dc=example,dc=com
objectClass: top
objectClass: sudoRole
cn: %admingroup
sudoUser: %admingroup
sudoHost: ALL
sudoRunAsUser: root
sudoCommand: ALL
sudoOption: !authenticate
```

Making entry of an ldap group with sudo power to openldap database

```shell
ldapadd -x -D cn=Manager,dc=example,dc=com -W -f sudogroup.ldif
```

### Adding LDAP user to the LDAP group

Here we will be creating an LDAP user and while doing that we also add this user to the LDAP group with sudo powers.

```shell
vi usersudogroup.ldif

dn: uid=john,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: john
sn: kelly
userPassword: {SSHA}xxxxxxxxxxxx
loginShell: /bin/bash
uidNumber: 9010
gidNumber: 1020
homeDirectory: /home/john
```

Making entry of the LDAP user associated to the LDAP group with sudo power in the openldap Database.

```shell
ldapadd -x -D cn=Manager,dc=example,dc=com -W -f usersudogroup.ldif
```

## Configuring Client

### Updating the sssd.conf file

```shell
vi /etc/sssd/sssd.conf

[domain/default]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://server.ip.or.dns
ldap_search_base = dc=example,dc=com
cache_credentials = True
ldap_tls_cacertdir = /etc/openldap/certs
ldap_tls_reqcert = allow
sudo_provider= ldap

[sssd]
config_file_version = 2
services = nss, pam, sudo
domains = default
```

### Permission for sssd.conf

Set the read/write access to /etc/sssd/ for the owner (root).

```shell
chmod 600 -R /etc/sssd
```

### Updating nsswitch.conf file

```shell
vi /etc/nsswitch.conf

sudoers:    compat sss
```

### Restarting the sssd & nscd service

```shell
systemctl restart sssd nscd
```
