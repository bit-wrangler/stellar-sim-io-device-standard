# Edge Case Examples

## (1) Multiple Instances of the Same SSIODS Device 

In this set of examples, an airliner flight sim cockpit uses two of the same device (32x toggle switch panel) to provide the user with 64 toggle switches coupled to various functions in the aircraft. The device supports message ID configuration over CAN bus when in CONFIGURING mode.

The SSIODS device profile contains the following information:
```
Device Application ID: 73ab6411-8baa-4d57-8372-a047246ead02
Device status default message id: 0x790
Device CAN messages:
    1. Message Id: 0x030
       Payload: 
            4 bytes containing the on/off state of the 32 switches
            4 bytes showing which switches have been toggles since last message
Device configuration mode CAN messages:
    1. Message Id: 0x000
       Payload: 2 bytes containing the new device status message id
    2. Message Id: 0x001
       Payload: 2 bytes containing the new message id for message (1) (the switch states message)
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
    default_can_message_id: 0x030
    can_message_id: null
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
    default_can_message_id: 0x030
    can_message_id: 0x031
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



