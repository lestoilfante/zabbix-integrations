# Template Watchguard Firewall

## Overview

Template for monitoring Watchuard Firebox with Fireware v12.X

### Items
* **General health checks** by SNMP
* **ICMP stats**
* **Public OS updates availability** based on website url defined in
	{$WG_DOWNLOAD_HOME} and {$WG_DOWNLOAD_URL}

### Discovery Rules
* **FireCluster** with health checks and triggers
* **Network Interfaces** speed and operational status


## Author

[lestoilfante](https://github.com/lestoilfante)

## Macros used

|Name|Description|Default|Type|
|----|-----------|-------|----|
|{$SNMP_COMMUNITY}|-|Inherited|Text macro|
|{$SNMP_TIMEOUT}|-|`5m`|Text macro|
|{$WG_HISTORY_L}|-|`90d`|Text macro|
|{$WG_HISTORY_M}|-|`1w`|Text macro|
|{$WG_HISTORY_S}|-|`1d`|Text macro|
|{$WG_DOWNLOAD_HOME}|WatchGuard website download landing page|`https://software.watchguard.com/`|Text macro|
|{$WG_DOWNLOAD_URL}|WatchGuard website download product page|`https://software.watchguard.com/SoftwareDownloads?current=true&familyId=`|Text macro|
|{$ICMP_LOSS_WARN}|-|`20`|Text macro|
|{$ICMP_RESPONSE_TIME_WARN}|-|`5m`|Text macro|

## Template links

There are no template links in this template.

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|FireCluster|FireCluster enabled discovering|`SNMP agent`|wgClusterEnabled.discovery|
|Network interfaces discovery|-|`SNMP agent`|wg.net.if.discovery|

## Items collected

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Active Connections|-|`SNMP agent`|wgSystemCurrActiveConns|
|CPU Utilization|-|`SNMP agent`|wgSystemCpuUtil1|
|Device description|-|`SNMP agent`|sysDescr|
|Device location|-|`SNMP agent`|sysLocation|
|Device name|-|`SNMP agent`|sysName|
|Device uptime|-|`SNMP agent`|sysUpTime|
|Free Memory|-|`SNMP agent`|memTotalFree|
|ICMP loss|-|`Simple check`|icmppingloss|
|ICMP ping|-|`Simple check`|icmpping|
|ICMP response time|-|`Simple check`|icmppingsec|
|Latest OS available|Latest OS available found on WatchGuard website|`Dependent item`|wgOShttpCheck<p></p><p>Using Javascript HttpRequest</p>|
|SNMP agent availability|-|`Zabbix internal`|zabbix[host,snmp,available]|
|Software version|Current OS|`SNMP agent`|wgSoftwareVersion|
|Total Memory|-|`SNMP agent`|memTotalReal|
|FireCluster "\{#MEMBER.ID.[id]\}"|-|`SNMP agent`|wg`[First\|Second]`MemberId[\{#MEMBER.ID.[id]\}]|
|FireCluster "\{#MEMBER.ID.[id]\}" Health|-|`SNMP agent`|wg`[First\|Second]`MemberSystemHealth[\{#MEMBER.ID.[id]\}]|
|FireCluster "\{#MEMBER.ID.[id]\}" HW Health|-|`SNMP agent`|wg`[First\|Second]`MemberHardwareHealth[\{#MEMBER.ID.[id]\}]|
|FireCluster "\{#MEMBER.ID.[id]\}" Port Health|-|`SNMP agent`|wg`[First\|Second]`MemberMonitorPortHealth[\{#MEMBER.ID.[id]\}]|
|FireCluster "\{#MEMBER.ID.[id]\}" Role|Member is Active or Standby|`SNMP agent`|wg`[First\|Second]`MemberRole[\{#MEMBER.ID.[id]\}]|
|FireCluster "\{#MEMBER.ID.[id]\}" Weighted Health|-|`SNMP agent`|wg`[First\|Second]`MemberWeightAvg[\{#MEMBER.ID.[id]\}]|
|Network interfaces discovery "\{#IFDESCR\}" Interface type|-|`SNMP agent`|wg.net.if.type[ifType.\{#SNMPINDEX\}]|
|Network interfaces discovery "\{#IFDESCR\}" Operational status|-|`SNMP agent`|wg.net.if.status[ifOperStatus.\{#SNMPINDEX\}]|
|Network interfaces discovery "\{#IFDESCR\}" Speed|-|`SNMP agent`|wg.net.if.speed[ifHighSpeed.\{#SNMPINDEX\}]|


## Triggers

|Name|Description|Expression|Priority|
|----|-----------|----------|--------|
|Update Available|Newer Fireware update available|-|`Information`|
|Unavailable by ICMP ping|-|-|`High`|
|No SNMP data collection|-|-|`Warning`|
|High ICMP ping response time|-|-|`Warning`|
|High ICMP ping loss|-|-|`Warning`|
|has been restarted|Device has been restarted|-|`Warning`|
|Failed to retrieve latest OS available|Newer Fireware update check failed|-|`Information`|
|CPU Utilization High|-|-|`Warning`|
|Available memory under 5%|-|-|`Information`|
|FireCluster "\{#MEMBER.ID.[id]\}" Overall Health|-|-|`Warning`|
|FireCluster active member has changed|-|-|`Information`|
|Network interfaces discovery "Interface \{#IFDESCR\}" Link down|-|-|`Average`|
|Network interfaces discovery "Interface \{#IFDESCR\}" has changed to lower speed than it was before|-|-|`Information`|
