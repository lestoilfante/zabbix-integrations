# Template Milestone XProtect

## Overview

A Zabbix template designed to monitor [Milestone XProtect VMS platform](https://www.milestonesys.com/) by HTTP without any external scripts.  

Uses native Milestone SOAP protocols and RESTful APIs allowing a full and documented integration with Management Server, Recording Server, Mobile Server and API Gateway.

Currently supports the discovery of Management Servers, Recording Servers, Mobile Servers, Services, Cameras, Storages, Archives.  
All these entities are identified by a tag like [MGM],[REC], etc on discovered Items name, no additional Hosts will be created.

### Environment

Tested on Milestone XProtect 23R3 with Zabbix 6.0

### Requirements

A Milestone XProtect user belonging to a role with at least these permissions:
* Info -> Allow Web Client Login
* Overall Security -> Management Server -> Connect
* Overall Security -> Management Server -> Status API
* Overall Security -> Cameras -> Read

To enable also TLS certificates checks a `Zabbix Agent 2` instance is required. Hopefully this will be superseded by a Simple check in the future (https://support.zabbix.com/browse/ZBXNEXT-8140).  
If Zabbix Agent 2 is not going to be installed on Milestone Management Server use this workaround:
1. On Zabbix add a secondary Host Interface of Type Agent to the defined Milestone Management Server Host
2. Point this secondary Host Interface to any available agent, i.e. 127.0.0.1 for Agent 2 running on Zabbix Server itself
3. Select this interface on Host's LLD rule.  

## Author

Copyright (C) 2024 [lestoilfante](https://github.com/lestoilfante)

## License

GNU General Public License version 3 (GPLv3)

## Configuration

1. Define a Host pointing to Milestone *Management Server* IP/DNS and set the macros `{$MILESTONE.USER}` and `{$MILESTONE.PASSWORD}`
2. Import template `template_Milestone_XProtect.yaml` and link it to Milestone *Management Server* Host

## Macros used

|Name|Description|Default|Required|
|----|-----------|-------|--------|
|{$MILESTONE.ARCHIVE.WARN}|Percentage of available archive space remaining that will fire trigger|`10`||
|{$MILESTONE.CERT.EXPIRY.WARN}|Number of days until the certificate expires|`7`||
|{$MILESTONE.CONN}|Connection protocol used for initial communication, use `https` or `http`|`https`||
|{$MILESTONE.ID}|Identifier uniquely identifying Zabbix calling instance. Typically, each ID should refer to a specific machine running this template|`ZABBIX_00`||
|{$MILESTONE.LLD_FREQ}|Low-level discovery frequency, keep lower than Milestone token duration which defaults to 60m|`50m`||
|{$MILESTONE.MOBILE.DH.PRIME}|Mobile Server expects the username and password to be encrypted with a Diffie-Hellman-Merkle shared secret. This is the prime number used (length 1024 or 2048 bit) in hexadecimal format|`F488FD584E49DBCD20B49DE49107366B336C380D451D0F7C88B31C7C5B2D8EF6F3C923C043F0A55B188D8EBB558CB85D38D334FD7C175743A31D186CDE33212CB52AFF3CE1B1294018118D7C84A70A72D686C40319C807297ACA950CD9969FABD00A509B0246D3083D66A45D419F9C7CBD894B221926BAABA25EC355E92F78C7`||
|{$MILESTONE.MOBILE.DH.USERANDOM}|Set to `1024` or `2048` if Zabbix has to generate a random prime of choosen length for DH key exchange. Requires huge cpu resources, potentially leading to item timeout. Leave empty if you are not sure.|||
|{$MILESTONE.PASSWORD}|XProtect password||*|
|{$MILESTONE.STORAGE.WARN}|Percentage of available storage space remaining that will fire trigger|`10`||
|{$MILESTONE.USER}|XProtect user||*|

## Items

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: Product name||`Dependent item`|mxp.info.pn|
|Milestone: Product SLC||`Dependent item`|mxp.info.slc|
|Milestone: Product version||`Dependent item`|mxp.info.pv|
|Milestone: XProtect Discovery|Retrieves data from Milestone Management Server feeding discovered items|`Script`|mxp.info|
|Milestone: XProtect Discovery status|Milestone Management Server data retrieval status|`Dependent item`|mxp.info.status|

## Triggers

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: XProtect Discovery status: error returned|Some error occurred during data retrieval from Milestone Management Server|`last(/Milestone XProtect/mxp.info.status)<>"OK"`|Warning||

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Archives||`Dependent item`|mxp.discovery.archives|
|Cameras|Discovers all enabled cameras|`Dependent item`|mxp.discovery.cameras|
|Management Servers||`Dependent item`|mxp.discovery.mgm|
|Mobile Servers||`Dependent item`|mxp.discovery.mob|
|Recording Servers||`Dependent item`|mxp.discovery.rec|
|Services|Registered services discovery|`Dependent item`|mxp.discovery.svc<br>By default filters out:<br> Mobile Server (already discovered by other LLD)<br> Management Server (already discovered by other LLD)<br> Api Gateway (already parsed as default item)|
|Storages||`Dependent item`|mxp.discovery.storages|

### Item prototypes for Archives discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [REC] {#REC_DNAME} [ARC] {#REC_ARCHIVE_DNAME}|Gets archive storage status.<br>Retrieved directly from recording server.|`Script`|mxp.rec.archive[{#REC_ID}.{#REC_ARCHIVE_ID}]|
|Milestone: [REC] {#REC_DNAME} [ARC] {#REC_ARCHIVE_DNAME} space available||`Dependent item`|mxp.rec.archive[{#REC_ID}.{#REC_ARCHIVE_ID}.sa]|
|Milestone: [REC] {#REC_DNAME} [ARC] {#REC_ARCHIVE_DNAME} status||`Dependent item`|mxp.rec.archive[{#REC_ID}.{#REC_ARCHIVE_ID}.status]|

### Trigger prototypes for Archives discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [REC] {#REC_DNAME} [ARC] {#REC_ARCHIVE_DNAME} space available is low||`last(/Milestone XProtect/mxp.rec.archive[{#REC_ID}.{#REC_ARCHIVE_ID}.sa])<{$MILESTONE.ARCHIVE.WARN}`|Warning||
|Milestone: [REC] {#REC_DNAME} [ARC] {#REC_ARCHIVE_DNAME} status||`last(/Milestone XProtect/mxp.rec.archive[{#REC_ID}.{#REC_ARCHIVE_ID}.status])<> 0`|High||

### Item prototypes for Cameras discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [REC] {#REC_DNAME} [CAM] {#CAM_DNAME} status|Gets camera status from recording server|`Script`|mxp.cam[{#CAM_ID}.status]|

### Trigger prototypes for Cameras discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [REC] {#REC_DNAME} [CAM] {#CAM_DNAME} status||`last(/Milestone XProtect/mxp.cam[{#CAM_ID}.status])<> 0`|Warning||

### Item prototypes for Management Servers discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [MGM] {#MGM_HOST} Certificate: Expires on|The date on which the certificate validity period ends|`Dependent item`|mxp.mgm.cert[{#MGM_ID}.notAfter]|
|Milestone: [MGM] {#MGM_HOST} Certificate: Fingerprint|The Certificate Signature (SHA1 Fingerprint or Thumbprint) is the hash of the entire certificate in DER form|`Dependent item`|mxp.mgm.cert[{#MGM_ID}.fingerprint]|
|Milestone: [MGM] {#MGM_HOST} Certificate: Get|Management server certificate.<br>Returns the JSON with attributes of certificate.|`Zabbix agent`|web.certificate.get[{#MGM_SSL_HOSTNAME},{#MGM_SSL_PORT},{#MGM_SSL_IP}]|
|Milestone: [MGM] {#MGM_HOST} sw version||`Script`|mxp.mgm[{#MGM_ID}.ver]|

### Trigger prototypes for Management Servers discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [MGM] {#MGM_HOST} Certificate: expires soon|The encryption certificate should be updated or it will become untrusted|`(last(/Milestone XProtect/mxp.mgm.cert[{#MGM_ID}.notAfter]) - now()) / 86400 < {$MILESTONE.CERT.EXPIRY.WARN}`|Warning||
|Milestone: [MGM] {#MGM_HOST} Certificate: Fingerprint has changed|The encryption certificate fingerprint has changed. If you did not update the certificate, it may mean your certificate has been hacked.<br>Ack to close.|`last(/Milestone XProtect/mxp.mgm.cert[{#MGM_ID}.fingerprint]) <> last(/Milestone XProtect/mxp.mgm.cert[{#MGM_ID}.fingerprint],#2)`|Information||

### Item prototypes for Mobile Servers discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [MOB] {#MOB_DNAME}|Master item implementing Mobile Server communication protocol|`Script`|mxp.mob.status[{#MOB_ID}]|
|Milestone: [MOB] {#MOB_DNAME} active streams||`Dependent item`|mxp.mob.status[{#MOB_ID}.streams]|
|Milestone: [MOB] {#MOB_DNAME} active users||`Dependent item`|mxp.mob.status[{#MOB_ID}.users]|
|Milestone: [MOB] {#MOB_DNAME} BW in|Bandwidth usage (inbound)|`Dependent item`|mxp.mob.status[{#MOB_ID}.bwin]|
|Milestone: [MOB] {#MOB_DNAME} BW out|Bandwidth usage (outbound)|`Dependent item`|mxp.mob.status[{#MOB_ID}.bwout]|
|Milestone: [MOB] {#MOB_DNAME} Certificate: Expires on|The date on which the certificate validity period ends|`Dependent item`|mxp.mob.cert[{#MOB_ID}.notAfter]|
|Milestone: [MOB] {#MOB_DNAME} Certificate: Fingerprint|The Certificate Signature (SHA1 Fingerprint or Thumbprint) is the hash of the entire certificate in DER form|`Dependent item`|mxp.mob.cert[{#MOB_ID}.fingerprint]|
|Milestone: [MOB] {#MOB_DNAME} Certificate: Get|Mobile server streaming media certificate.<br>Returns the JSON with attributes of certificate.|`Zabbix agent`|web.certificate.get[{#MOB_SSL_HOSTNAME},{#MOB_SSL_PORT},{#MOB_SSL_IP}]|
|Milestone: [MOB] {#MOB_DNAME} cpu usage||`Dependent item`|mxp.mob.status[{#MOB_ID}.cpu]|
|Milestone: [MOB] {#MOB_DNAME} http status|Check http service availability.<br>Expected http status code is 200, 0 on timeout.|`Script`|mxp.mob.http[{#MOB_ID}]|
|Milestone: [MOB] {#MOB_DNAME} hw accelerated count||`Dependent item`|mxp.mob.status[{#MOB_ID}.HwAcceleratedCount]|
|Milestone: [MOB] {#MOB_DNAME} server ip||`Dependent item`|mxp.mob.status[{#MOB_ID}.ip]|
|Milestone: [MOB] {#MOB_DNAME} status|Mobile Server communication status|`Dependent item`|mxp.mob.status[{#MOB_ID}.error]|
|Milestone: [MOB] {#MOB_DNAME} up since||`Dependent item`|mxp.mob.status[{#MOB_ID}.upSince]|
|Milestone: [MOB] {#MOB_DNAME} version||`Dependent item`|mxp.mob.status[{#MOB_ID}.version]|

### Trigger prototypes for Mobile Servers discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [MOB] {#MOB_DNAME} Certificate: expires soon|The encryption certificate should be updated or it will become untrusted|`(last(/Milestone XProtect/mxp.mob.cert[{#MOB_ID}.notAfter]) - now()) / 86400 < {$MILESTONE.CERT.EXPIRY.WARN}`|Warning||
|Milestone: [MOB] {#MOB_DNAME} Certificate: Fingerprint has changed|The encryption certificate fingerprint has changed. If you did not update the certificate, it may mean your certificate has been hacked.<br>Ack to close.|`last(/Milestone XProtect/mxp.mob.cert[{#MOB_ID}.fingerprint]) <> last(/Milestone XProtect/mxp.mob.cert[{#MOB_ID}.fingerprint],#2)`|Information||
|Milestone: [MOB] {#MOB_DNAME} communication fail|Communication with Mobile Server has failed|`last(/Milestone XProtect/mxp.mob.status[{#MOB_ID}.error])<>0`|Warning||
|Milestone: [MOB] {#MOB_DNAME} not responding||`last(/Milestone XProtect/mxp.mob.http[{#MOB_ID}]) <> 200`|Average||

### Item prototypes for Recording Servers discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [REC] {#REC_DNAME} Certificate: Expires on|The date on which the certificate validity period ends|`Dependent item`|mxp.rec.cert[{#REC_ID}.notAfter]|
|Milestone: [REC] {#REC_DNAME} Certificate: Fingerprint|The Certificate Signature (SHA1 Fingerprint or Thumbprint) is the hash of the entire certificate in DER form|`Dependent item`|mxp.rec.cert[{#REC_ID}.fingerprint]|
|Milestone: [REC] {#REC_DNAME} Certificate: Get|Recording server streaming media certificate.<br>Returns the JSON with attributes of certificate.|`Zabbix agent`|web.certificate.get[{#SSL_HOSTNAME},{#SSL_PORT},{#SSL_IP}]|
|Milestone: [REC] {#REC_DNAME} status|Recording server to Management server connection state.<br>Retrieved directly from recording server.|`Script`|mxp.rec.status[{#REC_ID}]|

### Trigger prototypes for Recording Servers discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [REC] {#REC_DNAME} Certificate: expires soon|The encryption certificate should be updated or it will become untrusted|`(last(/Milestone XProtect/mxp.rec.cert[{#REC_ID}.notAfter]) - now()) / 86400 < {$MILESTONE.CERT.EXPIRY.WARN}`|Warning||
|Milestone: [REC] {#REC_DNAME} Certificate: Fingerprint has changed|The encryption certificate fingerprint has changed. If you did not update the certificate, it may mean your certificate has been hacked.<br>Ack to close.|`last(/Milestone XProtect/mxp.rec.cert[{#REC_ID}.fingerprint]) <> last(/Milestone XProtect/mxp.rec.cert[{#REC_ID}.fingerprint],#2)`|Information||
|Milestone: [REC] {#REC_DNAME} status|Recorder Server to Management Server communication error|`last(/Milestone XProtect/mxp.rec.status[{#REC_ID}])<>"Connected"`|High||

### Item prototypes for Services discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [SVC] {#SVC_NAME}|Basic http GET check, returns ok as long as service responds|`Dependent item`|mxp.svc[{#SVC_TYPE}.{#SVC_ID}.reachable]|
|Milestone: [SVC] {#SVC_NAME} uri||`Script`|mxp.svc[{#SVC_TYPE}.{#SVC_ID}.uri]|

### Trigger prototypes for Services discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [SVC] {#SVC_NAME} is down||`last(/Milestone XProtect/mxp.svc[{#SVC_TYPE}.{#SVC_ID}.reachable])=0`|Warning||

### Item prototypes for Storages discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Milestone: [REC] {#REC_DNAME} [STR] {#REC_STORAGE_DNAME}|Gets recording storage status.<br>Retrieved directly from recording server.|`Script`|mxp.rec.storage[{#REC_ID}.{#REC_STORAGE_ID}]|
|Milestone: [REC] {#REC_DNAME} [STR] {#REC_STORAGE_DNAME} space available||`Dependent item`|mxp.rec.storage[{#REC_ID}.{#REC_STORAGE_ID}.sa]|
|Milestone: [REC] {#REC_DNAME} [STR] {#REC_STORAGE_DNAME} status||`Dependent item`|mxp.rec.storage[{#REC_ID}.{#REC_STORAGE_ID}.status]|

### Trigger prototypes for Storages discovery
|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Milestone: [REC] {#REC_DNAME} [STR] {#REC_STORAGE_DNAME} space available is low||`last(/Milestone XProtect/mxp.rec.storage[{#REC_ID}.{#REC_STORAGE_ID}.sa])<{$MILESTONE.STORAGE.WARN}`|Warning||
|Milestone: [REC] {#REC_DNAME} [STR] {#REC_STORAGE_DNAME} status||`last(/Milestone XProtect/mxp.rec.storage[{#REC_ID}.{#REC_STORAGE_ID}.status])<> 0`|High||

## Template links

There are no template links in this template.

