# Edge Case Examples

## (1) Multiple Instances of the Same SSIODS Device 

In this set of examples, an airliner flight sim cockpit uses two of the same device (32x toggle switch panel) to provide the user with 64 toggle switches coupled to various functions in the aircraft. The device supports message ID configuration over CAN bus when in CONFIGURING mode.

The SSIODS device profile contains the following information:
```
Device Application ID: 73ab6411-8baa-4d57-8372-a047246ead02
Device status default message id: 0x790
Device CAN messages:
    0. Message Id: 0x030
       Payload: 
            4 bytes containing the on/off state of the 32 switches
            4 bytes showing which switches have been toggles since last message
Device configuration mode CAN messages:
    1. Message Id: 0x000
       Payload: 16 bit unsigned integer containing the new device status message id
    2. Message Id: 0x001
       Payload: 16 bit unsigned integer containing the new message id for message (0) (the switch states message)
```

The device ships with the following default configuration file:
```yaml
device_id: 73ab6411-8baa-4d57-8372-a047246ead02
device_name: MyFlightSimAvionics 32x Switch Panel Device
instance_id: 0
hub_instance: 0
can_instance: 0
default_status_message_id: 0x790
status_message_id: null
messages:
  - id: 0
    name: Switch State
    default_can_message_id: 0x030
    can_message_id: null
    forwarding: null
```


### (1.A) Multiple Instances of the Same SSIODS Device on the Same CAN Bus

In this specific example, the same SSIODS devices are installed on an SSIODS hub with a single CAN bus. Since both instances of the device have the same CAN bus message IDs, one of the devices must be reconfigured to eliminate the message ID collisions between the two devices.

The first instance of the device will use the default configuration file.

The second instance of the device will use a modified configuration file that will provide overrides:
```yaml
device_id: 73ab6411-8baa-4d57-8372-a047246ead02
device_name: MyFlightSimAvionics 32x Switch Panel Device
instance_id: 1
hub_instance: 0
can_instance: 0
default_status_message_id: 0x790
status_message_id: 0x791
messages:
  - id: 0
    name: Switch State
    default_can_message_id: 0x030
    can_message_id: 0x031
    forwarding: null
```

The following overrides exist in the above configuration:

1. `instance_id` was changed from `0` to `1`. This was changed to differentiate between instances of the `73ab6411-8baa-4d57-8372-a047246ead02` devices within the consuming application.
2. `status_message_id` was changed from `null` to `0x791`.
3. `messages[0].can_message_id` was changed from `null` to `0x031`.

The configration procedure for conflicting devices may look like the following (assuming all SSIODS devices are already connected to the hub):

1. Disconnect the first instance of the device's CAN bus connection or power. This will ensure that the hub can only communicate with the second instance of the device.
2. Configration utility sends message to put all devices into STANDBY mode. Utility should wait until all devices are in STANDBY mode.
3. Configuration utility sends message to put the target device into CONFIGURING mode.
4. Configuration utility sends message to set the target device status CAN message ID to `0x791`.
5. Configuration utility sends message to set the target device's 0 message's CAN message ID to `0x031`.
6. Configuration utility sends message to put all devices into STANDBY mode. Utility should wait until all devices are in STANDBY mode.
7. Configuration utility sends message to put all devices into NORMAL mode.
8. Reconnect the first instance of the device's CAN bus connection or power.

With the two devices operating at the same time, the messages from the first instance of the device will be emitted using CAN message ID `0x030` and will be mapped to `device: 73ab6411-8baa-4d57-8372-a047246ead02; instance: 0; id: 0;` messages by the interface service for consumption by the end application. The messages from the second instance of the device will be emitted using CAN message ID `0x031` and will be mapped to `device: 73ab6411-8baa-4d57-8372-a047246ead02; instance: 1; id: 0;` messages by the interface service for consumption by the end application.

### (1.B) Multiple Instances of the Same SSIODS Device on the Different CAN Busses of the Same Hub

In this specific example, the same SSIODS devices are installed on an SSIODS hub with two CAN busses. Since each instance of the device is on a different CAN bus, there is no collision between CAN messages.

The first instance of the device will use the default configuration file.

The second instance of the device will use a modified configuration file that will provide overrides:
```yaml
device_id: 73ab6411-8baa-4d57-8372-a047246ead02
device_name: MyFlightSimAvionics 32x Switch Panel Device
instance_id: 1
hub_instance: 0
can_instance: 1
default_status_message_id: 0x790
status_message_id: null
messages:
  - id: 0
    name: Switch State
    default_can_message_id: 0x030
    can_message_id: null
    forwarding: null
```

The following overrides exist in the above configuration:

1. `instance_id` was changed from `0` to `1`. This was changed to differentiate between instances of the `73ab6411-8baa-4d57-8372-a047246ead02` devices within the consuming application.
2. `can_instance` was changed from `0` to `1`. This tells the interface service that the device will be on CAN bus `1` of the hub `0`.

### (1.C) Multiple Instances of the Same SSIODS Device on Different Hubs

In this specific example, the same SSIODS devices are installed on an SSIODS hub with two CAN busses. Since each instance of the device is on a different CAN bus because the devices are connected to different hubs, there is no collision between CAN messages.

The first instance of the device will use the default configuration file.

The second instance of the device will use a modified configuration file that will provide overrides:
```yaml
device_id: 73ab6411-8baa-4d57-8372-a047246ead02
device_name: MyFlightSimAvionics 32x Switch Panel Device
instance_id: 1
hub_instance: 1
can_instance: 0
default_status_message_id: 0x790
status_message_id: null
messages:
  - id: 0
    name: Switch State
    default_can_message_id: 0x030
    can_message_id: null
    forwarding: null
```

The following overrides exist in the above configuration:

1. `instance_id` was changed from `0` to `1`. This was changed to differentiate between instances of the `73ab6411-8baa-4d57-8372-a047246ead02` devices within the consuming application.
2. `hub_instance` was changed from `0` to `1`. This tells the interface service that the device will be on hub `1`.

## (2) Interhub Forwarding of Messages 

In this example, a flight sim cockpit setup uses two hubs. One hub has two CAN busses and the other hub has one CAN bus. Connected to one of the CAN busses of the first hub, there is an encoder knob SSIODS device that is intended to be used to set the backlight illumination level on various instruments in the cockpit panel. These backlight illuminated indicators are spread across the CAN busses of both hubs, so the messages from the encoder knob device need to be forwarded to all the CAN busses of both hubs.

The SSIODS device profile contains the following information:
```
Device Application ID: c284a632-94ca-471a-b519-e3f2cef2588f
Device status default message id: 0x791
Device CAN messages:
    0. Message Id: 0x040
       Payload: 
            16 bit integer with encoder value
Device configuration mode CAN messages:
    1. Message Id: 0x000
       Payload: 16 bit unsigned integer containing the new device status message id
    2. Message Id: 0x001
       Payload: 16 bit unsigned integer containing the new message id for message (0)
    3. Message Id: 0x200
       Payload: 16 bit unsigned integer containing the max value of the encoder
```

The device ships with the following default configuration file:
```yaml
device_id: c284a632-94ca-471a-b519-e3f2cef2588f
device_name: MyFlightSimAvionics 20 PPR Encoder Knob Device
instance_id: 0
hub_instance: 0
can_instance: 0
default_status_message_id: 0x791
status_message_id: null
messages:
  - id: 0
    name: Encoder Value
    default_can_message_id: 0x040
    can_message_id: null
    forwarding: null
```

To achieve forwarding to all CAN busses across all the devices, the following configuration is used:
```yaml
device_id: c284a632-94ca-471a-b519-e3f2cef2588f
device_name: MyFlightSimAvionics 20 PPR Encoder Knob Device
instance_id: 0
hub_instance: 0
can_instance: 0
default_status_message_id: 0x791
status_message_id: null
messages:
  - id: 0
    name: Encoder Value
    default_can_message_id: 0x040
    can_message_id: null
    forwarding:
      - hub_instance: null
        can_instance: null
```

The following overrides exist in the above configuration:

1. `messages[0].forwarding` was changed from `null` to an object with properties `hub_instance: null` and `can_instance: null`. This configuration tells interface service to forward these messages to all hubs (since `null` was set as the destination for the `hub_instance`) and to configure all hubs to forward the message on all CAN busses present on the hub.