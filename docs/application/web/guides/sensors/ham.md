# Human Activity Monitor

You can access and record human activity data from various [sensors](#supported-monitors) and [recorders](#supported-recorders-in-wearable-applications) on the device.

This feature is supported in mobile and wearable applications only.

The main features of the Human Activity Monitor API include the following:

- Retrieving data

  You can [collect monitor data](#retrieving-data).

- Managing data recording

  You can [record human activity data and retrieve the saved data](#managing-data-recording).

- Using user-defined intervals

  You can [change intervals for collecting data](#using-user-defined-intervals).

- Receiving notifications

  You can [detect changes in the monitor data](#receiving-notifications-on-pedometer-data-changes).

- Recognizing sleep activity

  You can determine whether the [user is sleeping](#detecting-sleep) and monitor [user's sleep](#monitoring-sleep).

## Prerequisites

To use the Human Activity Monitor API (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html) applications), the application has to request permission by adding the following privileges to the `config.xml` file:

```
<tizen:privilege name="http://tizen.org/privilege/healthinfo"/>
<tizen:privilege name="http://tizen.org/privilege/location"/>
```

## Retrieve data

Enabling the monitor and retrieving data is a basic Human Activity Monitor (HAM) management skill:

<a name="support"></a>
1. Check whether a sensor is supported using the `tizen.systeminfo.getCapability()` method to get the [appropriate capability](#table_capabilities).

2. To enable the monitor and start collecting data, use the `start()` method of the `HumanActivityMonitorManager` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) applications):

   ```
   var counter = 0;

   function onchangedCB(hrmInfo) {
       console.log('Heart Rate: ' + hrmInfo.heartRate);
       console.log('Peak-to-peak interval: ' + hrmInfo.rRInterval + ' milliseconds');

       counter++;
       if (counter > 10) {
           /* Stop the sensor after detecting a few changes */
           tizen.humanactivitymonitor.stop('HRM');
       }
   }

   tizen.humanactivitymonitor.start('HRM', onchangedCB);
   ```

3. When the heart rate monitor (HRM) is enabled, you can get the current data using the `getHumanActivityData()` method of the `HumanActivityMonitorManager` interface:

   ```
   function onsuccessCB(hrmInfo) {
       console.log('Heart rate: ' + hrmInfo.heartRate);
   }

   function onerrorCB(error) {
       console.log('Error occurred: ' + error.message);
   }

   tizen.humanactivitymonitor.getHumanActivityData('HRM', onsuccessCB, onerrorCB);
   ```

4. If the requested `HumanActivityType` type is `PEDOMETER` then `getHumanActivityData()` provides the `HumanActivityPedometerData` object. This object not only includes data gathered since the call of the `start()` method, but also the data gathered since the device boot:

    ```
    function onsuccessCB(pedometerData) {
        console.log('Steps made since start: ' + pedometerData.cumulativeTotalStepCount);
        console.log('Steps made since device boot: ' + pedometerData.accumulativeTotalStepCount);
    }

    function onerrorCB(error) {
        console.log('Error occurred: ' + error.message);
    }

    tizen.humanactivitymonitor.getHumanActivityData('PEDOMETER', onsuccessCB, onerrorCB);
    ```

5. To disable HAM when it is no longer required, use the `stop()` method of the `HumanActivityMonitorManager` interface:

   ```
   tizen.humanactivitymonitor.stop('HRM');
   ```

## Manage data record

The Human Activity Monitor API allows you to record and retrieve saved sensor data, , follow these steps to record and retrieve saved sensor data:

1. To check whether a sensor is supported, use the `getCapability()` method of the `SystemInfo` interface (in [mobile](../../api/latest/device_api/mobile/tizen/systeminfo.html#SystemInfo) and [wearable](../../api/latest/device_api/wearable/tizen/systeminfo.html#SystemInfo) applications):

   ```
   if (tizen.systeminfo.getCapability('http://tizen.org/feature/sensor.barometer') === false) {
       console.log('PRESSURE is not supported on this device.');

       return;
   }
   ```

2. To enable data recording, use the `startRecorder()` method of the `HumanActivityMonitorManager` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) applications). Optionally, you can also define an interval and period for the data recording:

   ```
   var type = 'PRESSURE';

   var options = {
       retentionPeriod: 1 /* 1 hour */
   }

   try {
       tizen.humanactivitymonitor.startRecorder(type, options);
   } catch (err) {
       console.log(err.name + ': ' + err.message);
   }
   ```

   To stop recording sensor data, use the `stopRecorder()` method of the `HumanActivityMonitorManager` interface:

   ```
   try {
       tizen.humanactivitymonitor.stopRecorder('PRESSURE');
   } catch (err) {
       console.log(err.name + ': ' + err.message);
   }
   ```

3. Before retrieving data, you can specify a time period to be retrieved using the `startTime` and `endTime` options in the `HumanActivityRecorderQuery` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityRecorderQuery) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityRecorderQuery) applications):

   ```
   /* To retrieve data from July 1, 2016 to July 31, 2016 */
   var query = {};
   query['startTime'] = (new Date(2016, 7, 1)).getTime() / 1000;
   query['endTime'] = (new Date(2016, 7, 31)).getTime() / 1000;
   ```

4. To get the data sliced by an interval, you can use a combination of the `anchorTime` and `interval` options in the `HumanActivityRecorderQuery` interface.

   Some human activity recorder types do not allow slicing the data by an interval:

   ```
   /* To retrieve data everyday at midnight */
   /* Time is 0:00 internally */
   query['anchorTime'] = (new Date(2016, 7, 1, 0, 0)).getTime() / 1000;
   query['interval'] = 1440; /* Day */
   ```

5. To read the human activity recorder data from the database, use the `readRecorderData()` method of the `HumanActivityMonitorManager` interface with the query.

   Even if your application never recorded any data, you can access any data that has been recorded in the database by other applications:

   ```
   function onerror(error) {
       console.log(error.name + ': ' + error.message);
   }

   function onread(data) {
       for (var idx = 0; idx < data.length; ++idx) {
           console.log('average pressure: ' + data[idx].average);
       }
   }

   var type = 'PRESSURE';

   try {
       tizen.humanactivitymonitor.readRecorderData(type, query, onread, onerror);
   } catch (error) {
       console.log(error.name + ': ' + error.message);
   }
   ```

## Use user-defined intervals

The Human Activity Monitor API allows the user to select their own intervals for collecting samples in a specified range using the `HumanActivityMonitorOption` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorOption) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorOption) applications). Such functionality can be used to build more power-efficient data collection applications (the less often the device gathers data, the less energy is used). You can change the interval according to the device state, for example, when the display is switched off, the sampling interval can be decreased.

1. If a human activity type allows setting the interval at which data is sent to the application or setting the sampling interval, the last parameter of the `start()` method can be used to specify this information:

   ```
   var myCallbackInterval = 240000;
   var mySampleInterval = 10000;

   function onchangedCB(gpsInfo) {
       console.log('this callback is called every ' + myCallbackInterval + ' milliseconds');
       console.log('the gpsInfo includes the GPS information that is collected every ' +
                  mySampleInterval + ' milliseconds');
   }

   function onerrorCB(error) {
       console.log('Error occurred. Name:' + error.name + ', message: ' + error.message);
   }

   var option = {
       'callbackInterval': myCallbackInterval,
       'sampleInterval': mySampleInterval
   };

   tizen.humanactivitymonitor.start('GPS', onchangedCB, onerrorCB, option);
   ```

2. When the heart-rate monitor (HRM) is enabled, you can get the current data using the `getHumanActivityData()` method of the `HumanActivityMonitorManager` interface:

   ```
   function onsuccessCB(hrmInfo) {
       console.log('Heart rate: ' + hrmInfo.heartRate);
   }
   function onerrorCB(error) {
       console.log('Error occurred: ' + error.message);
   }
   tizen.humanactivitymonitor.getHumanActivityData('HRM', onsuccessCB, onerrorCB);
   ```

3. To disable HAM when it is no longer required, use the `stop()` method of the `HumanActivityMonitorManager` interface:

   ```
   tizen.humanactivitymonitor.stop('HRM');
   ```

## Receive notifications on pedometer data changes

Learning how to register a listener for accumulative allows you to monitor the step count maintained by the system without enabling the Pedometer sensor and the `PEDOMETER` monitor is a basic Human Activity Monitor (HAM) management skill:

1. To register an event handler for accumulative pedometer changes, use the `setAccumulativePedometerListener()`  method of the `HumanActivityMonitorManager` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) applications):

   ```
   function onchangedCB(pedometerInfo) {
       console.log('Step status: ' + pedometerInfo.stepStatus);
       console.log('Speed: ' + pedometerInfo.speed);
       console.log('Walking frequency: ' + pedometerInfo.walkingFrequency);
       /* Deregisters a previously registered listener */
       tizen.humanactivitymonitor.unsetAccumulativePedometerListener();
   }

   tizen.humanactivitymonitor.setAccumulativePedometerListener(onchangedCB);
   ```

2. To stop receiving notifications about the accumulative pedometer changes, use the `unsetAccumulativePedometerListener()` method of the `HumanActivityMonitorManager` interface:

   ```
   tizen.humanactivitymonitor.unsetAccumulativePedometerListener();
   ```

## Monitor sleep

Learning how to monitor user's sleep is a basic Human Activity Monitor (HAM) management skill:

1. To enable the monitor and start collecting data, use the `start()` method of the `HumanActivityMonitorManager` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) applications):

   ```
   function onchangedCB(sleepInfo) {
       console.log('Sleep status: ' + sleepInfo.status);
       console.log('Timestamp: ' + sleepInfo.timestamp + ' milliseconds');
   }

   tizen.humanactivitymonitor.start('SLEEP_MONITOR', onchangedCB);
   ```

2. To disable the monitor when it is no longer required, use the `stop()` method of the `HumanActivityMonitorManager` interface:

   ```
   tizen.humanactivitymonitor.stop('SLEEP_MONITOR');
   ```

## Detect sleep

Learning how to detect whether the user is asleep is a basic Human Activity Monitor (HAM) management skill:

1. To enable the monitor and start collecting data, use the `start()` method of the `HumanActivityMonitorManager` interface (in [mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) and [wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html#HumanActivityMonitorManager) applications) with
`HumanActivityType` set to SLEEP_DETECTOR:

   ```
   function onchangedCB(sleepInfo) {
       console.log('Sleep status: ' + sleepInfo.status);
   }

   tizen.humanactivitymonitor.start('SLEEP_DETECTOR', onchangedCB);
   ```

2. To disable the monitor when it is no longer required, use the `stop()` method of the `HumanActivityMonitorManager` interface with
`HumanActivityType` set to SLEEP_DETECTOR:
   ```
   tizen.humanactivitymonitor.stop('SLEEP_DETECTOR');
   ```

## Supported monitors

The following table introduces the available monitor types and lists the monitor capabilities you can use to [determine whether a specific monitor is supported](#support) on a device.

<a name="table_capabilities"></a>
**Table: Human activity monitors and capabilities**

| Monitor                              | Capability                               | Notes                                    |
| ------------------------------------ | ---------------------------------------- | ---------------------------------------- |
| Pedometer and accumulative pedometer | `http://tizen.org/feature/sensor.pedometer` | When the pedometer sensor is started, a change callback is invoked when data changes. Use the `getHumanActivityData()` method to get the current data.<br> The accumulative pedometer sensor does not have to be started by your application as long as step counting is enabled by any other application or the system. Listener registered with the `setAccumulativePedometerListener()` method is called when accumulative counters are changed. |
| Heart rate monitor                   | `http://tizen.org/feature/sensor.heart_rate_monitor` | When the heart rate monitor (HRM) sensor is started, a change callback is invoked when data changes. Use the `getHumanActivityData()` method to get the current data. |
| GPS                                  | `http://tizen.org/feature/location.batch` | When the GPS sensor is started, a change callback is invoked when data changes. Use the `getHumanActivityData()` method to get the current data.<br> The GPS sensor provides both the current value and a short history of last recorded GPS positions. The sensor supports sampling intervals, which can be used to create more power-efficient applications. |
| Sleep monitor                        | `http://tizen.org/feature/sensor.sleep_monitor` | When the sleep sensor is started, a change callback is invoked when data changes. Use the `getHumanActivityData()` method to get the current data. |
| Sleep detector                        | `http://tizen.org/feature/sensor.sleep_monitor` | When the sleep sensor is started, a change callback is invoked when data changes. Use the `getHumanActivityData()` method to get the current data. |

## Supported recorders in wearable applications

The following table introduces the available recorder types and lists the capabilities you can use to [determine whether a specific recorder is supported](#support) on a device.

**Table: Human activity recorders and capabilities**

| Recorder | Capability                               | Notes                                    |
| -------- | ---------------------------------------- | ---------------------------------------- |
| Pressure | `http://tizen.org/feature/sensor.barometer` | Use the `startRecorder()` and `stopRecorder()` methods to record the pressure sensor data for the specific period of time. Use the `readRecorderData()` method to read the recorded pressure sensor data. |

## Related information
* Dependencies
  - Tizen 2.4 and Higher for Mobile
  - Tizen 2.3.1 and Higher for Wearable
* API References
  - [Mobile](../../api/latest/device_api/mobile/tizen/humanactivitymonitor.html)
  - [Wearable](../../api/latest/device_api/wearable/tizen/humanactivitymonitor.html)
