# homebridge-modbus-2

Hombridge plugin for controlling appliances through ModbusTCP.
This is an extended/bugfixed version of
[homebridge-modbus of berkus](https://github.com/berkus/homebridge-modbus),
which in turn is a extended/bugfixed version
of the original [homebridge-modbus](https://github.com/sclavel/homebridge-modbus).

## Usage of new Characteristics

Define your Service type and add `correction` or `scale`.
```json
{
    "platforms": [
        {
       	    "platform": "Modbus",
            "ip": "192.168.1.201",
            "port": 502,
            "pollFrequency": 1000,
            "accessories": [
            	{
                    "name": "Bedroom",
                    "type": "Thermostat",
                    "TargetTemperature": {
                        "address": "r1",
                        "scale": "*10",
                        "correction": "/10"
                    },
                    "CurrentTemperature": {
                        "address": "i1",
                        "correction": "/10"
                    },
                    "CurrentHeatingCoolingState": {
                        "address": "r3",
                        "readonly": true,
                        "mask": "2",
                        "map": {"0": 0, "2": 1},
                        "validValues": [0, 1],
                    },
                    "TargetHeatingCoolingState": {
                        "value": 1,
                        "validValues": [1]
                    }
            	}
            ]
        }
    ]
}
```

> **Attention**
> `scale` is only applied when writing a value,
> `correction` is only applied when a value is updated!

## Original

You must define each accessory in the config.json file, with a `name`, a `type` (that matches any HomeKit Service type), and one or more characteristics (that match HomeKit Characteristics for this Service).
If you want to group several services under the same accessory, just reuse the same accessory name, and add a `subtype` field if services are of the same type.
The characteristics value in the json is the modbus address associated with it.
Coils are defined with the letter 'c' followed by the coil number, holding registers are defined with the letter 'r' followed by the register number, input registers are defined with the letter 'i' followed by the register number.

If you want more advanced control, the characteristic value can instead be an object having `address` as the coil/register type and number, and optional elements like `validValues`, `maxValue`, `value` to configure HomeKit, `mask` and/or `map` to map different values between modbus and homekit, `readOnly` to force holding registers or coils to not be written on modbus (input registers are always read-only).
You can apply a simple correction to the values by specifying "correction" key.

Example config.json:
```json
{
    "platforms": [
        {
       	    "platform": "Modbus",
            "ip": "192.168.1.201",
            "port": 502,
            "pollFrequency": 1000,
            "accessories": [
            	{
                    "name": "Bedroom ventilation",
                    "type": "Fan",
                    "On": "r1"
            	},
            	{
                    "name": "Bedroom light",
                    "type": "Switch",
                    "On": "c1"
            	},
            	{
                    "name": "Bedroom",
                    "type": "Thermostat",
                    "TargetTemperature": "r2",
                    "CurrentTemperature": "i1",
                    "CurrentHeatingCoolingState": {
                        "address": "r3",
                        "readonly": true,
                        "mask": "2",
                        "map": {"0": 0, "2": 1},
                        "validValues": [0, 1],
                        "correction": "/10"
                    },
                    "TargetHeatingCoolingState": {
                        "value": 1,
                        "validValues": [1]
                    }
            	},
            	{
                    "name": "Water Heater",
                    "type": "TemperatureSensor",
                    "subtype": "from boiler",
                    "CurrentTemperature": "i2"
            	},
            	{
                    "name": "Water Heater",
                    "type": "TemperatureSensor",
                    "subtype": "to boiler",
                    "CurrentTemperature": "i3"
            	}
            ]
        }
    ]
}
```
