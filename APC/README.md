# Template APC Smart-UPS SmartConnect

## Overview

This Zabbix template collects information from the APC SmartConnect Cloud platform, including voltage, current, power, status, temperature, battery usage, and relevant events.

### Requirements

* **Zabbix version 6.0 or later**
* **External script `APC-ScrapConnect.php` (https://github.com/lestoilfante/APC-ScrapConnect)**

### Items
* **Set inventory type**

### Discovery Rules
* **Device discovery** with health checks and triggers

## Author

Copyright (C) 2023 [lestoilfante](https://github.com/lestoilfante)

## License
GNU General Public License version 3 (GPLv3)

## Configuration

Import template `template_APC_SmartConnect.yaml`, assign to UPS Host, set the macros `{$APCSC.USER}` `{$APCSC.PASSWORD}` as per SmartConnect credentials and… and stop, done! The automated discovery will look for the first device registered on SmartConnect that has an IP equal to Host interface defined in Zabbix, or alternatively, if you have multiple devices with the same IP, set the macro `{$APC.SERIAL}` to proceed with discovery by serial number.

### Macros used

|Name|Description|Default|Type|Required|
|----|-----------|-------|----|--------|
|{$APC.SERIAL}|<p>Device SN.</p>|-|Text|*|
|{$APCSC.PASSWORD}|<p>APC SmartConnect password.</p>|-|Secret Text|*|
|{$APCSC.USER}|<p>APC SmartConnect username.</p>|-|Text|*|
|{$BATTERY.CAPACITY.MIN.WARN}|<p>Minimum battery capacity percentage for trigger expression.</p>|`50`|Text||
|{$BATTERY.TEMP.MAX.WARN}|<p>Maximum battery temperature for trigger expression.</p>|`55`|Text||
|{$INVENTORY.TYPE}|<p>Used to set inventory Type for each host using this template.</p>|`UPS`|Text||
|{$TIME.PERIOD}|<p>Time period for trigger expression.</p>|`15m`|Text||
|{$UPS.INPUT_FREQ.MAX.WARN}|<p>Maximum input frequency for trigger expression.</p>|`50.3`|Text||
|{$UPS.INPUT_FREQ.MIN.WARN}|<p>Minimum input frequency for trigger expression.</p>|`49.7`|Text||
|{$UPS.INPUT_VOLT.MAX.WARN}|<p>Maximum input voltage for trigger expression.</p>|`243`|Text||
|{$UPS.INPUT_VOLT.MIN.WARN}|<p>Minimum input voltage for trigger expression.</p>|`197`|Text||
|{$UPS.OUTPUT.MAX.WARN}|<p>Maximum output load in % for trigger expression.</p>|`80`|Text||

## Template links

There are no template links in this template.

## Discovery rules

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Device discovery|Discovery based on Host IP or device SN|`External check`|APC-ScrapConnect.php["{$APCSC.USER}","{$APCSC.PASSWORD}",discovery,"{$APC.SERIAL}","{HOST.IP}"]|

## Items collected

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Battery capacity|-|`Dependent item`|apcsc.battery.chargePercentage[{#ID}]|
|Battery EOL|-|`Dependent item`|apcsc.battery.EolDate[{#ID}]|
|Battery last replace date|-|`Dependent item`|apcsc.battery.installDate[{#ID}]|
|Battery model|-|`Dependent item`|apcsc.system.batteryRbcSku[{#ID}]|
|Battery runtime remaining|-|`Dependent item`|apcsc.battery.runtimeRemaining[{#ID}]|
|Battery temperature|-|`Dependent item`|apcsc.system.temperature[{#ID}]|
|Battery voltage|-|`Dependent item`|apcsc.battery.voltage[{#ID}]|
|Communication Status|UPS status on SmartConnect platform|`Dependent item`|apcsc.system.pluginStatus[{#ID}]|
|Device Firmware|Current firmware installed on the UPS|`Dependent item`|apcsc.system.firmware[{#ID}]|
|Device Firmware latest|Latest firmware available for this UPS|`Dependent item`|apcsc.system.firmwareLatest[{#ID}]|
|External battery packs count|-|`Dependent item`|apcsc.system.batteryFramesInstalledCount[{#ID}]|
|Get details|JSON data feeding child items|`External check`|APC-ScrapConnect.php["{$APCSC.USER}","{$APCSC.PASSWORD}",gwdetails,"{#ID}"]|
|Get Info|JSON data feeding child items|`External check`|APC-ScrapConnect.php["{$APCSC.USER}","{$APCSC.PASSWORD}",gwinfo,"{#ID}"]|
|Input efficiency|-|`Dependent item`|apcsc.input.efficiency[{#ID}]|
|Input frequency|-|`Dependent item`|apcsc.input.frequency[{#ID}]|
|Input voltage|-|`Dependent item`|apcsc.input.voltage[{#ID}]|
|Model|-|`Dependent item`|apcsc.system.model[{#ID}]|
|Name|-|`Dependent item`|apcsc.system.name[{#ID}]|
|Ongoing Error condition|Returns active UPS conditions with severity level 3, based on platform's discovered values.|`Dependent item`|apcsc.condition.error[{#ID}]|
|Ongoing Info condition|Returns active UPS conditions with severity level 1, based on platform's discovered values.|`Dependent item`|apcsc.condition.info[{#ID}]|
|Ongoing Warning condition|Returns active UPS conditions with severity level 2, based on platform's discovered values.|`Dependent item`|apcsc.condition.warning[{#ID}]|
|Output current|-|`Dependent item`|apcsc.output.current[{#ID}]|
|Output frequency|-|`Dependent item`|apcsc.output.frequency[{#ID}]|
|Output load|-|`Dependent item`|apcsc.output.load[{#ID}]|
|Output voltage|-|`Dependent item`|apcsc.output.voltage[{#ID}]|
|Selftest fail cause|-|`Dependent item`|apcsc.selftest.failing[{#ID}]|
|Selftest interval|-|`Dependent item`|apcsc.selftest.intv[{#ID}]|
|Selftest last result|-|`Dependent item`|apcsc.selftest.result[{#ID}]|
|Serial number|-|`Dependent item`|apcsc.system.sn[{#ID}]|
|Status|-|`Dependent item`|apcsc.system.OperatingMode[{#ID}]|
|UPS Condition|Array of active events demanding attention.|`Dependent item`|apcsc.system.condition[{#ID}]|
|Warranty Expiry Date|-|`Dependent item`|apcsc.system.warrantyEndDate[{#ID}]|

## Triggers

|Name|Description|Expression|Priority|
|----|-----------|----------|--------|
|Battery has high temperature|-|-|`High`|
|Battery has low capacity|-|-|`High`|
|Communication Status|-|-|`Average`|
|Ongoing Error condition|-|-|`High`|
|Ongoing Info condition|-|-|`Information`|
|Ongoing Warning condition|-|-|`Warning`|
|Operative Status|-|-|`High`|
|Output load is high|-|-|`High`|
|Selftest last result|-|-|`Warning`|
|Unacceptable input frequency|-|-|`High`|
|Unacceptable input voltage|-|-|`High`|
