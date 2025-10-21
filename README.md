# elkmon2 

**elkmon2** updates Simon Gentry's [elkmon](https://github.com/sgentry/elkmon) package to support secure connections with the latest firmware (version 2.0.51) on the M1-XEP.  

Note that this firmware only supports TLS-1.2, which is deprecated TLS, but this should work on modern systems, unlike the older TLS 1.0 which has typically been removed.

If you have disabled username/passwords in your M1XEP (which is not recommended) then you can omit the `userName` and `password` properties when constructing the `Elk` instance.

## Features

* Supports both secure and non-secure communication with [Elk M1XEP](http://www.elkproducts.com/product-catalog/elk-m1xep-m1-ethernet-interface)
* All received messages are parsed into an ElkMessage

  Example
  ``` javascript
    {
      message: '1EAS000000001111111100000000000E', // Full message
      body: '00000000111111110000000000', // Parsed message
      type: 'AS', // Type of message
      hexLength: '1E',
      checkSum: '0E'
    }
  ```
* Some messages are parsed into a specific message type. For example, ZoneStatusReport (ZS) which includes an array of zones:

  ``` javascript
  {
    message: 'D6ZS33333333333333303333000000000000333300000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000034',
    body: '333333333333333033330000000000003333000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
    type: 'ZS',
    hexLength: 'D6',
    checkSum: '34',
    zones:
      [
        { id: 1, physicalStatus: 'Short', logicalState: 'Normal' },
        { id: 2, physicalStatus: 'Short', logicalState: 'Normal' },
        ...
      ]
  }
  ```
* Events are emitted when messages are received. You can listen for all (*) or by type (KC, ZD, ZC, etc).
* Requests for data return a Promise and can be chained together.

## Example Usage

```javascript
var Elk = require('elkmon');

// Instantiate a new Elk instance (non-secure)
var elk = new Elk(
  2101, // port
  '192.168.1.100' // M1XEP address
);

// Register any event handlers
elk.on('connected', () => {
  console.log('***connected***');

  // Request arming status report
  elk.requestArmingStatus()
    .then((report) => {
      console.log(report.areas);
      // Request zones status report
      return elk.requestZoneStatusReport()
    })
    .then((report) => {
      console.log(report.zones);
      elk.speak('all clear');
  });
});

// Listen for all messages
elk.on('*', (message) => {
  console.log(message);
});

// Listen for messages by type (Zone Change)
elk.on('ZC', (message) => {
  console.log('Zone Change Report: ', message);
});

elk.connect();

```

```javascript
// Instantiate a secure instance
var elk = new Elk(
  2601,
  '192.168.1.100', {
    secure: true,
    userName: 'SomeUser',
    password: 'YourPassword',
  }
);
```

# API

## connect()
  Connects to M1XEP.

## disconnect()
  Closes connection to M1XEP.

## arm(areaId, armMode, keypadCode)
  Arm Elk in specified arming mode

## disarm(areaId, keypadCode)
  Disarm Elk

## activateTask(taskId)
  Activates a task

## setOutputOn(outputId, seconds)
  Turns an output on

## setOutputOff(outputId)
  Turns an output off

## speak(message)
  Command Elk panel to speak a message over it's speaker.

## toggleOutput(outputId)
  Toggles a control Output On/Off.

## requestOutputStatusReport([timeout])
  Request the Control Output Status report from Elk panel.

## bypassZone(zoneId, areaId, keypadCode)
  Bypass a Zone.

## requestArmingStatus([timeout])
  Requests an Arming Status Report.

## requestAreas([timeout])
  Request Keypad Area assignments

## requestZoneDefinitionReport([timeout])
  Requests a Zone Definition Report.

## requestZonePartitionReport([timeout])
  Requests a Zone Partition Report.

## requestZoneVoltageReport(id, [timeout])
  Requests a Zone Voltage Report.

## requestZoneStatusReport([timeout])
  Requests a Zone Status Report.

## requestTextDescription(id, type, [timeout])
  Requests a text description.

## requestTextDescriptionAll(type)
  Requests the configured Text Descriptions, by type.