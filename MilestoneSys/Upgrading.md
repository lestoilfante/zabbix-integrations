# Upgrading Template Milestone XProtect

Template version is reported in `Milestone: XProtect Discovery` script as `zbx.version`.

## 1.1.0 to 1.2.0

Camera connection loss now fires on duration instead of first error, with flapping detection and thresholds per recorder or camera (see [README.md](README.md#camera-thresholds-per-recorder-or-camera)).

* Changes reach existing cameras on next discovery, use *Execute now* on `Milestone: XProtect Discovery` to apply them immediately
* Trigger `[CAM] {#CAM_DNAME} status` now covers only status 3, 4 and 5:
  * Connection loss (status 1) moves to `is down` (duration based), `is unstable` (flapping) and `lost connection` (Info)
  * Not licensed (status 2) moves to `is not licensed`
* Camera status history is kept. The item is now fed by `[CAM] {#CAM_DNAME} status check`, which is not stored and shows no value in Latest data. On Zabbix 6.0 use *Execute now* on the status check item
* When a status check fails no camera trigger changes state, failures are visible as unsupported `status check` items
* Camera status check timeout raised from 10s to 30s
