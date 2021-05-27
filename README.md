# 1 - Integrating-SLES-15-Active-Directory


# 2 - Configure and syncronize NTP
```bash
ntpq -p
```

# 3 - instalation of required packages 
```bash
Winbind: krb5-client samba-client openldap2-client samba-winbind samba-winbind-32bit
```

# 4 - Edit Krb5 configuration file /etc/krb5.conf:

```bash

[libdefaults]
        dns_canonicalize_hostname = false
        rdns = false
        default_realm = CJF.LOCAL
        default_ccache_name = /tmp/krb5cc_%{uid}

[realms]
        CJF.LOCAL = {
                kdc = cjf.local
                default_domain = cjf.local
                admin_server = cjf.local
}

[logging]
  kdc = FILE:/var/log/krb5/krb5kdc.log
  admin_server = FILE:/var/log/krb5/kadmind.log
  default = SYSLOG:NOTICE:DAEMON

[domain_realm]
.cjf.local = CJF.LOCAL
cjf.local = CJF.LOCAL

```
# 5 - Configure SMB you'll need to use the REALM as setup /etc/samba/smb.conf file:

```bash
 [global]
        workgroup = EXAMPLE
        kerberos method = secrets and keytab
        realm = EXAMPLE.COM
        security = ADS
        template homedir = /home/%D/%U
        template shell = /bin/bash
        winbind refresh tickets = yes
        winbind use default domain = yes
        winbind enum groups = yes
        winbind enum users = yes
        idmap uid = 10000-20000
        idmap gid = 10000-20000
        
```
# 6 - Verify your nsswtich configuration file /etc/nsswitch

```bash
passwd: compat winbind
group:  compat winbind
shadow: compat


hosts:          files dns
networks:       files dns


services:       files
protocols:      files
rpc:            files
ethers:         files
netmasks:       files
netgroup:       files nis
publickey:      files


bootparams:     files
automount:      files nis
aliases:        files

```

# 7 - finish the configuration in winbind /etc/security/pam_winbind.conf:

```bash
  [global]
    cached_login = yes
    krb5_auth = yes
    krb5_ccache_type = FILE

```

# 8 - Configure openldap2 client /etc/openldap/ldap.conf

```bash
URI ldap://cjf.local
BASE dc=cjf,dc=local


REFERRALS OFF
```

# 9 - Establish Kerberos Connection using "kinit" with a privileged AD user 

```bash
kinit Administrator
```

# 10 - Create the computer account. The "-k" flag uses the Kerberos ticket created in the previous step for authentication. Alternatively one could use the "-U" flag with the administrative user and password.

```bash
net ads join -k
```

# 11 - Configure PAM stack using pam-config:

```bash
pam-config -a --winbind
pam-config -a --mkhomedir
```
# 12 - Enable and start your configured service:

```bash
systemctl enable sssd
systemctl start sssd
```
