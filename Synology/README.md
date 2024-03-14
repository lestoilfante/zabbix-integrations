# Template Synology Hyper Backup

## Overview

Checks Synology Hyper Backup jobs via DSM Web Api with pure Zabbix Script implementation.

### Environment

Created and tested with:
* Synology DSM 6.2 and 7.2
* Zabbix 6.0

### Items
* **Hyper Backup: Get status** returning JSON array of all Task objects

### Discovery Rules
* **Tasks discovery** backup tasks

## Author

Copyright (C) 2023 [lestoilfante](https://github.com/lestoilfante)

## License
GNU General Public License version 3 (GPLv3)

## Configuration

Import template `template_synology_hyperbackup.yaml`, assign to Synology Host, set the macros `{$SYNO.REST.USER}` `{$SYNO.REST.PASSWORD}`, this user has to be in Synology `administrators` group and allowed access to DSM Application.

### Macros used

|Name|Description|Default|Type|Required|
|----|-----------|-------|----|--------|
|{$SYNO.REST.API.BACKUP.LLD_INT}|<p>LLD Interval.</p>|`15m`|Text||
|{$SYNO.REST.API.PORT}|<p>DSM https port.</p>|`5001`|Text||
|{$SYNO.REST.PASSWORD}|<p>DSM password.</p>|-|Secret Text|*|
|{$SYNO.REST.USER}|<p>DSM user.</p>|-|Text|*|

## Template links

There are no template links in this template.

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Tasks discovery|Discovery of backup tasks|`Script`|syno.rest.HyperBackupTasks|

## Items collected

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Hyper Backup Task {#TASK_NAME} last result|Last task result|`Dependent item`|syno.rest.HyperBackupLastResult[{#TASK_ID}]|
|Hyper Backup Task {#TASK_NAME} last run|Last time/date task was run|`Dependent item`|syno.rest.HyperBackupLastRun[{#TASK_ID}]|
|Hyper Backup Task {#TASK_NAME} next run|Next time/date task will run|`Dependent item`|syno.rest.HyperBackupNextRun[{#TASK_ID}]|
|Hyper Backup Task {#TASK_NAME} status|Current backup task activity|`Dependent item`|syno.rest.HyperBackupStatus[{#TASK_ID}]|

## Triggers

|Name|Description|Expression|Priority|
|----|-----------|----------|--------|
|Hyper Backup Task {#TASK_NAME} last result: {#TASK_LAST_RESULT}|-|-|`Warning`|
|Hyper Backup Task {#TASK_NAME} unexpected status|-|-|`Warning`|
