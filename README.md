# homebridge-bond-ultra-low-support-eglo

An independent, unverified beta fork of [homebridge-bond](https://github.com/aarons22/homebridge-bond)
with optional EGLO ultra-low fan support. It uses the [Bond V2 API](http://docs-local.appbond.com).
The original project is MIT licensed; see [LICENSE](./LICENSE).

## Features

This plugin currently supports the following devices and features:

- Multiple Bonds
- Supported Devices
  - Ceiling Fan
    - Light on/off
    - Up/Down light
    - Fan Speeds 1-8 (dynamic based on bond configuration)
    - Light Dimming
  - Blinds
    - Open / Close
  - Generic Device
    - On / Off (appears as switch in HomeKit)
  - Fireplace
    - Flame On / Off (appears as switch in HomeKit)


You can view the backlog of features [here](https://github.com/aarons22/homebridge-bond/). Feel free to add a feature request in the Issues tab!

## Installation

### Bond Parameters

| Option             | Required | Explanation |
| ------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ip_address`   | true   | To get your Bond IP Address, follow the instructions [here](https://github.com/aarons22/homebridge-bond/wiki/Get-Bond-IP-Address). |
| `token` | true   | This can be found in the Bond app in your Bond Settings. Scroll down until you see `Local Token`. You can tap on the row to copy to your clipboard. |
| `hide_device_ids` | false  | Array of device ids to ignore (i.e. `["1111", "2222"]`) |
| `ms_between_actions` | false | Space out actions sent to the Bond by at least this many milliseconds. |

### Easiest Configuration

For the best experience setting up this plugin, please use [homebridge-config-ui-x](https://www.npmjs.com/package/homebridge-config-ui-x).

### HOOBS Configuration

In HOOBS, you need to provide the bonds (optionally, turn on opt-in features):
```json
[
    {
        "ip_address": "<IP_ADDRESS>",
        "token": "<TOKEN>",
        "hide_device_ids": ["123456"]
    }
]
```

<ins>Example Config in HOOBS:</ins>

![Config](./images/hoobs_configuration.png)

### Basic Configuration

Assuming a global installation of `homebridge`, install the plugin:

`npm i -g homebridge-bond-ultra-low-support-eglo@beta`

Do not run this fork and the original plugin for the same Bond at the same time: both use the
`Bond` platform name and could send duplicate commands. Back up your Homebridge configuration
before switching; Homebridge may recreate accessories or require you to reconfigure automations.

Add the `Bond` platform in your homebridge `config.json` file:

```json
"platforms": [
    {
        "platform": "Bond",
        "bonds": [
            {
                "ip_address": "<IP_ADDRESS>",
                "token": "<TOKEN>"
            }
        ],
    }
],
```

NOTE: If you have a "Smart by BOND" fan/device, you will need to add a Bond in the config for _each_ fan/device.

### Optional Platform Parameters

| Option             | Default | Explanation                                                                                                                                                         |
| ------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `include_dimmer`   | false   | If dimming is a valid action on a device, it will be included as additional switch on the accessory. When using this feature, turn the switch on to start dimming. When you reach the brightness level you would like, turn the switch off. |
| `fan_speed_values` | false   | Use fan speed values instead of percentages (i.e. Hey Siri set the Office Fan to 2) |
| `include_toggle_state` | false  | This will add a switch to single-action accessories to toggle the state (i.e. Shades, Fireplace, Generic, Lights). Fan speeds are not eligible for this option. |

### EGLO ultra-low configuration

First, learn the long-press signal on a separate Bond Generic Device, configure its `TurnOn`
action, and set its signal repeats to a value that actually triggers ultra low on your fan.
This plugin does not change signal repeats. Test that Generic Device in the Bond app first.
The fan must report five normal speeds and support `SetSpeed`, `TurnOn`, and `TurnOff`.

Add a mapping inside the corresponding entry in `bonds`:

```json
{
  "ip_address": "<BOND_IP>",
  "token": "<BOND_TOKEN>",
  "ms_between_actions": 1000,
  "hide_device_ids": ["<ULTRA_DEVICE_ID>"],
  "ultra_low_fans": [
    {
      "fan_device_id": "<FAN_DEVICE_ID>",
      "ultra_low_device_id": "<ULTRA_DEVICE_ID>"
    }
  ]
}
```

Replace the placeholders with IDs from Bond device settings. Hiding the Generic Device is
optional and does not prevent the plugin from sending its action. Each mapping only affects
that fan on that Bond. Other fans retain the upstream speed behavior.

HomeKit speed 1 sends the Generic Device's `TurnOn` action; speeds 2-6 send normal fan speeds
1-5. Off sends `TurnOff` to the fan. In percentage mode the steps are approximately 16.67%,
33.33%, 50%, 66.67%, 83.33%, and 100%. With `fan_speed_values`, they are 1-6 instead.
Transmit-queue-full errors are retried twice, and `ms_between_actions` applies to retries.

Bond cannot know that the Generic Device changed the fan speed. After a successful HomeKit
command, this plugin remembers fan power and speed only until Homebridge restarts and ignores
fan power/speed updates from Bond to avoid replacing that state with stale data. Physical
remote or Bond app changes can therefore leave HomeKit out of sync. After a restart Bond's
reported normal speed is used until HomeKit sends another command; ultra low cannot be
detected automatically. Light and direction updates remain independent.

The transition from ultra low to a normal speed is implemented by sending the normal
`SetSpeed` action directly, but must be verified on your actual fan before relying on it.

## Development

I'm more than happy to take PRs if you want to fix a bug you are having or take a shot at adding a new feature you're interested in. To compile the code in realtime, run:

```
npm run dev
```

To use the local version of the plugin in homebridge, I run `npm link` in the directory of the cloned repo.

## Support

Report issues with this fork at [its GitHub repository](https://github.com/christopherguzik/homebridge-bond/issues).
Please do not report fork-specific EGLO problems to the upstream project.
