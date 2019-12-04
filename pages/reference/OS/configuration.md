---
title: Configuration
excerpt: Instructions for editing and customizing config.json for modifying the behavior of {{ $names.os.lower }}
---

# Configuring {{ $names.os.lower }}

The behavior of {{ $names.os.lower }} can be configured by editing the `config.json` file that resides in the boot partition of the device. This file accepts a [range of fields](#valid-fields) to modify the behavior of the host OS.

__Note:__ The `config.json` file is different from the [`config.txt`](https://www.balena.io/docs/reference/OS/advanced/#configtxt) file also located in the boot partition, which is used by the Raspberry Pi to set device configuration options.

## Using config.json to provision a device

The `config.json` file is located in the root of the `resin-boot` partition, and you may edit this file by mounting a flashed SD card and editing the file directly. Editing this file will only work if you edit before the device's first boot. Once the device has been provisioned, edits to this file have no effect as the contents of the file are copied to the device boot partition at `/mnt/boot/config.json`.

## Sample config.json

The following example provides all available configuration options available in `config.json`. Full details about each value may be found in the [valid fields](#valid-fields) section.

```json
{
  "hostname": "my-custom-hostname",
  "persistentLogging": true,
  "country": "GB",
  "ntpServers": "ntp-wwv.nist.gov resinio.pool.ntp.org",
  "dnsServers": "208.67.222.222 8.8.8.8",
  "os": {
    "network": {
      "connectivity": {
        "uri": "https://api.balena-cloud.com/connectivity-check",
        "interval": "300",
        "response": "optional value in the response"
      },
      "wifi": {
        "randomMacAddressScan": false
      }
    },
    "udevRules": {
      "56": "ENV{ID_FS_LABEL_ENC}==\"resin-root*\", IMPORT{program}=\"resin_update_state_probe $devnode\", SYMLINK+=\"disk/by-state/$env{RESIN_UPDATE_STATE}\"",
      "64": "ACTION!=\"add|change\", GOTO=\"modeswitch_rules_end\"\nKERNEL==\"ttyACM*\", ATTRS{idVendor}==\"1546\", ATTRS{idProduct}==\"1146\", TAG+=\"systemd\", ENV{SYSTEMD_WANTS}=\"u-blox-switch@'%E{DEVNAME}'.service\"\nLBEL=\"modeswitch_rules_end\"\n"
    },
    "sshKeys": [
      "ssh-rsa AAAAB3Nza...M2JB balena@macbook-pro",
      "ssh-rsa AAAAB3Nza...nFTQ balena@zenbook"
    ]
  }
}
```

## Valid fields

The following fields can be used to modify the behavior of the host OS.

### hostname

(string) The configured hostname of the device, otherwise the device UUID is used.

### persistentLogging

(boolean) Enable or disable persistent logging on the device. Defaults to false.

### country

(string) The country in which the device is operating. Used for setting the WiFi regulatory domain.

### ntpServers

(string) A space-separated list of NTP servers to use for time synchronization. Defaults to `resinio.pool.ntp.org` servers.

### dnsServers

(string) A space-separated list of preferred DNS servers to use for name resolution.

* When `dnsServers` is not defined, or empty, the default value of Google's DNS (8.8.8.8) is used.
* When `dnsServers` is `"null"` (a string), the system will not alter the dnsServers to the default server as detailed above.
* When `dnsServers` is defined and not `"null"` it will be considered as such without any additional servers.

### os

An object containing settings that customize the host OS at runtime.

#### network

##### wifi

An object that defines the configuration related to Wi-Fi.

* "randomMacAddressScan" (boolean) Configures MAC address randomization of a Wi-Fi device during scanning

The following example disables MAC address randomization of Wi-Fi device during scanning:

```json
"os": {
  "network" : {
    "wifi": {
      "randomMacAddressScan": false
    }
  }
}
```

##### connectivity

An object that defines configuration related to networking connectivity checks.

* "uri" (string) Value of the url to query for connectivity checks. Defaults to `$API_ENDPOINT/connectivity-check`.
* "interval" (string) Interval between connectivity checks in seconds. Defaults to 3600.
* "response" (string). If set controls what body content is checked for when requesting the URI. If it is an empty value, the HTTP server is expected to answer with status code 204 or send no data.

The following example configures the connectivity check by passing the balena cloud connectivity endpoint with a 5-minute interval.

```json
"os": {
  "network" : {
    "connectivity": {
    "uri" : "https://api.balena-cloud.com/connectivity-check",
    "interval" : "300"
    }
  }
}
```

#### udevRules

An object containing one or more custom udev rules as `key:value` pairs.

To turn a rule into a format that can be easily added to `config.json`, use the following command:

`cat rulefilename | jq -sR .`

For example:

```shell
root@resin:/etc/udev/rules.d# cat 64.rules | jq -sR .
"ACTION!=\"add|change\", GOTO=\"modeswitch_rules_end\"\nKERNEL==\"ttyACM*\", ATTRS{idVendor}==\"1546\", ATTRS{idProduct}==\"1146\", TAG+=\"systemd\", ENV{SYSTEMD_WANTS}=\"u-blox-switch@'%E{DEVNAME}'.service\"\nLBEL=\"modeswitch_rules_end\"\n"
```

The following example contains two custom udev rules that will create `/etc/udev/rules.d/56.rules` and `/etc/udev/rules.d/64.rules`. The first time rules are added/modified, udevd will reload the rules and re-trigger.

```json
"os": {
  "udevRules": {
    "56": "ENV{ID_FS_LABEL_ENC}==\"resin-root*\", IMPORT{program}=\"resin_update_state_probe $devnode\", SYMLINK+=\"disk/by-state/$env{RESIN_UPDATE_STATE}\"",
    "64" : "ACTION!=\"add|change\", GOTO=\"modeswitch_rules_end\"\nKERNEL==\"ttyACM*\", ATTRS{idVendor}==\"1546\", ATTRS{idProduct}==\"1146\", TAG+=\"systemd\", ENV{SYSTEMD_WANTS}=\"u-blox-switch@'%E{DEVNAME}'.service\"\nLBEL=\"modeswitch_rules_end\"\n"
  }
}
```

#### sshKeys

(Array) An array of strings containing a list of public SSH keys that will be used by the SSH server for authentication.

```json
"os": {
  "sshKeys": [
    "ssh-rsa AAAAB3Nza...M2JB balena@macbook-pro",
    "ssh-rsa AAAAB3Nza...nFTQ balena@zenbook"
  ]
}
```

## Managing config.json on a provisioned device

Once the device has been provisioned, the contents of `config.json` are copied to the device boot partition at `/mnt/boot/config.json`. You can view the contents of this file on a provisioned device with the following command:

```shell
balena ssh <uuid>
cat /mnt/boot/config.json | jq '.'
```

Special care should be taken when modifying any values of `config.json` on a provisioned device. This file is also used by the device supervisor, and mistakes to this file can leave a device inaccessible. There are various ways of safely modifying the values of `config.json`:

* Using the supervisor API, you can update the device [hostname](/reference/supervisor/supervisor-api/#patch-v1devicehost-config).
* Using the device dashboard [configuration variables](/learn/manage/configuration/), you may modify the persistent logging setting.
* Using the [configizer project](https://github.com/balena-io-playground/configizer) you may apply updates remotely via the {{ $names.cli }}. Note that this has not been thoroughly tested and should be treated with caution. Always test any changes first on a development device.

Alternatively, you can always reprovision a device with an updated `config.json` file.

**Note:** Editing config.json on a provisioned device is not recommended, as mistakes to this file can leave a device inaccessible. If you need to manually edit this file, see the [balenaOS masterclass instructions](https://github.com/balena-io-projects/balenaos-masterclass#12-advanced-editing-configjson-by-hand).
