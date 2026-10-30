# Windows Authentication

## Overview

Windows Authentication is automatically used when `{$MILESTONE.USER}` is in the form `DOMAIN\username`.  
Protocol is selected by `{$MILESTONE.AUTH.WINDOWS}`:
* `HTTPAUTH_NTLM` (default) for maximum compatibility, no additional configuration required
* `HTTPAUTH_NEGOTIATE` to use Kerberos, requires the configuration below

## Kerberos requirements

Zabbix must be configured according to the official [Zabbix Kerberos guide](https://www.zabbix.com/documentation/current/en/manual/appendix/items/kerberos), moreover:
* A valid `HTTP/<MANAGEMENT_SERVER_FQDN>` SPN matching the hostname used in the request is mandatory
* A valid `/etc/krb5.conf` is required to map the DNS domain to the Kerberos realm and locate the KDC
* A TGT must already exist in the credential cache accessible to the Zabbix process

When Zabbix script engine calls `setHttpAuth(HTTPAUTH_NEGOTIATE, 'DOMAIN\\username', 'password')` it does not acquire the initial TGT.  
`HttpRequest` can only use an existing TGT to acquire the `HTTP/<MANAGEMENT_SERVER_FQDN>@REALM` service ticket.

## Testing

Acquire the initial TGT as `zabbix` user:
```bash
sudo -u zabbix bash -c '
    read -s PASSWORD
    echo $PASSWORD | kinit username
    unset PASSWORD
'
```

Now either Zabbix Script item or curl can acquire and use a service ticket for `HTTP/<MANAGEMENT_SERVER_FQDN>`:
```bash
sudo -u zabbix bash -c '
    curl -vk --negotiate \
      -u : -H "Content-Type: application/x-www-form-urlencoded" \
      --data "grant_type=windows_auth&client_id=winauthclient" \
      "https://<MANAGEMENT_SERVER_FQDN>/IDP/connect/token"
'
```

Without a TGT acquired by `kinit` this alone can't work, even passing explicit credentials:
```bash
sudo -u zabbix bash -c '
    read -s PASSWORD
    curl -vk --negotiate \
      -u "DOMAIN\\username:$PASSWORD" \
      -H "Content-Type: application/x-www-form-urlencoded" \
      --data "grant_type=windows_auth&client_id=winauthclient" \
      "https://<MANAGEMENT_SERVER_FQDN>/IDP/connect/token"
    unset PASSWORD
'
```

## Known limitations

TGT acquisition is not handled by the template.  
A possible out-of-the-box workaround, not currently implemented, would be an additional External Script item running `kinit` before the Script item, allowing `HttpRequest` to obtain the HTTP service ticket.
