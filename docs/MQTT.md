<SPAN ALIGN="CENTER" STYLE="text-align:center">
<DIV ALIGN="CENTER" STYLE="text-align:center">

[![homebridge-unifi-protect: Native HomeKit support for UniFi Protect](https://raw.githubusercontent.com/hjdhjd/homebridge-unifi-protect/main/homebridge-protect.svg)](https://github.com/hjdhjd/homebridge-unifi-protect)

# Homebridge UniFi Protect

[![Downloads](https://img.shields.io/npm/dt/homebridge-unifi-protect?color=%230559C9&logo=icloud&logoColor=%23FFFFFF&style=for-the-badge)](https://www.npmjs.com/package/homebridge-unifi-protect)
[![Version](https://img.shields.io/npm/v/homebridge-unifi-protect?color=%230559C9&label=Homebridge%20UniFi%20Protect&logo=ubiquiti&logoColor=%23FFFFFF&style=for-the-badge)](https://www.npmjs.com/package/homebridge-unifi-protect)
[![UniFi Protect@Homebridge Discord](https://img.shields.io/discord/432663330281226270?color=0559C9&label=Discord&logo=discord&logoColor=%23FFFFFF&style=for-the-badge)](https://discord.gg/QXqfHEW)
[![verified-by-homebridge](https://img.shields.io/badge/homebridge-verified-blueviolet?color=%23491F59&style=for-the-badge&logoColor=%23FFFFFF&logo=homebridge)](https://github.com/homebridge/homebridge/wiki/Verified-Plugins)

## HomeKit support for the UniFi Protect ecosystem using [Homebridge](https://homebridge.io).
</DIV>
</SPAN>

`homebridge-unifi-protect` is a [Homebridge](https://homebridge.io) plugin that provides HomeKit support to the [UniFi Protect](https://unifi-network.ui.com/video-security) device ecosystem. [UniFi Protect](https://unifi-network.ui.com/video-security) is [Ubiquiti's](https://www.ui.com) video security platform, with rich camera, doorbell, and NVR controller hardware options for you to choose from, as well as an app which you can use to view, configure and manage your video camera and doorbells.

### MQTT Support

[MQTT](https://mqtt.org) is a popular Internet of Things (IoT) messaging protocol that can be used to weave together different smart devices and orchestrate or instrument them in an infinite number of ways. In short - it lets things that might not normally be able to talk to each other communicate across ecosystems, provided they can support MQTT.

I've provided MQTT support for those that are interested - I'm genuinely curious, if not a bit skeptical, at how many people actually want to use this capability. MQTT has a lot of nerd-credibility, and it was a fun side project to mess around with. :smile:

`homebridge-unifi-protect` will publish MQTT events if you've configured a broker in the controller-specific settings. The plugin supports a rich set of capabilities over MQTT. This includes:

  * Camera-specific RTSP information.
  * Doorbell message events. See [doorbell message events](#doorbell-messages) for additional details.
  * Doorbell ring events.
  * Liveview-related events, including the security system accessory and security alarm.
  * Motion events.
  * Snapshot events, including publishing the actual images over MQTT.

### How to configure and use this feature

This documentation assumes you know what MQTT is, what an MQTT broker does, and how to configure it. Setting up an MQTT broker is beyond the scope of this documentation. There are plenty of guides available on how to do so just a search away.

You configure MQTT settings within a `controller` configuration block. The settings are:

| Configuration Setting | Description
|-----------------------|----------------------------------
| `mqttUrl`             | The URL of your MQTT broker. **This must be in URL form**, e.g.: `mqtt://user:password@1.2.3.4`.
| `mqttTopic`           | The base topic to publish to. The default is: `unifi/protect`.

To reemphasize the above: **mqttUrl** must be a valid URL. Simply entering in a hostname without specifying it in URL form will result in an error. The URL can use any of these protocols: `mqtt`, `mqtts`, `tcp`, `tls`, `ws`, `wss`.

When events are published, by default, the topics look like:

```sh
unifi/protect/1234567890AB/motion
unifi/protect/ABCDEF123456/doorbell
```

In the above example, `1234567890AB` and `ABCDEF123456` are the MAC addresses of your cameras or doorbells. We use MAC addresses as an easy way to guarantee unique identifiers that won't change. `homebridge-unifi-protect` provides you information about your cameras and their respective MAC addresses in the homebridge log on startup. Additionally, you can use the UniFi Protect app or webUI to lookup what the MAC addresses are of your cameras, should you need to do so.

### <A NAME="publish"></A>Topics Published
The topics and messages that `homebridge-unifi-protect` publishes are:

| Topic                 | Message Published
|-----------------------|----------------------------------
| `alarm`               | `true` or `false` when a UniFi Protect sensor detects a recognized alarm sound.
| `ambientlight`        | Ambient light level, in lux, on a UniFi Protect sensor.
| `contact`             | `true` or `false` when a UniFi Protect sensor detects contact. Note: UniFi Protect will set this state differently depending on the placement type you select in the Protect app or the Protect webUI.
| `doorbell`            | `true` when the doorbell is rung. Each press of the doorbell will trigger a new event.
| `humidity`            | Humidity percentage level on a UniFi Protect sensor.
| `leak`                | `true` or `false` when a UniFi Protect sensor detects a leak.
| `message`             | `{"message":"Some Message","duration":60}`. See [Doorbell Messages](#doorbell-messages) for additional documentation.
|                       |
| `liveviews`           | `[{"name": "LiveviewName", "state": true},{"name": "AnotherLiveview", "state": false}]`. `state` can be `true` or `false`, indicating whether a liveview scene is active.
| `motion`              | `true` when motion is detected. `false` when the motion event is reset.
|                       |
| <CODE>motion/smart/<I>object</I></CODE>            | `true` when a smart motion event is detected for *object*. `false` when the smart motion event is reset. Valid *object* types are currently: `person`, `vehicle`. If what you want to listen for is whether a motion event is sent to HomeKit, use the `motion` topic instead.</BR>
|                       |
| `rtsp`                | `{"Name": "URL"}`. Represents a JSON containing all the valid RTSP URLs that can be used to stream from this camera. `Name` is the name assigned by UniFi Protect to the RTSP URL. `URL` represents the URL that can be used for streaming.
|                       |
| `securitysystem`      | One of `Alarm`, `Away`, `Home`, `Night`, `Off`. This message is published every time the security state is set.
|                       |
| `snapshot`            | A [data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs) containing a base64-encoded JPEG of the snapshot that was requested (either by HomeKit or MQTT).
|                       |
| `telemetry`           | All UniFi Protect telemetry from the realtime events API. This is the raw feed - you're on your own to parse through it for the messages you may be interested in. This is published to the NVR MAC address.
| `temperature`         | Temperature, in Celsius, on a UniFi Protect sensor.

Messages are published to MQTT when an action occurs on a camera, controller, or doorbell that triggers the respective event, or when an MQTT message is received for one of the topics `homebridge-unifi-protect` subscribes to. For example, snapshot images are published every time HomeKit requests a snapshot as well as when a request is received through MQTT to trigger a new snapshot.

### <A NAME="subscribe"></A>Topics Subscribed
The topics that `homebridge-unifi-protect` subscribes to are:

| Topic                   | Message Expected
|-------------------------|----------------------------------
| `alarm/get`             | `true` will trigger a publish event of the current alarm state for a UniFi Protect sensor.
| `ambientlight/get`      | `true` will trigger a publish event of the ambient light level, in lux, for a UniFi Protect sensor.
| `contact/get`           | `true` will trigger a publish event of the current contact state for a UniFi Protect sensor. Note: UniFi Protect will set this state differently depending on the placement type you select in the Protect app or the Protect webUI.
| `humidity/get`          | `true` will trigger a publish event of the humidity level, as a percentage, for a UniFi Protect sensor.
| `leak/get`              | `true` will trigger a publish event of the current leak sensor state for a UniFi Protect sensor.
| `liveviews/get`         | `true` will request that the plugin publish the current state of all liveviews to the `liveviews` topic.
| `liveviews/set `        | A JSON-compatible array in the format `[{"name": "view1", "state": true }, ...]` This will activate or deactivate one of more liveviews, depending on the respective state.
|                         |
| `message/get`           | `true` will request that the plugin publish a message to the `message` topic containing the current message JSON for the doorbell. See [Doorbell Messages](#doorbell-messages) for additional documentation.
| `message/set`           | A JSON in the format `{"message":"Some Message","duration":60}`. See [Doorbell Messages](#doorbell-messages) for additional documentation.
|                         |
| `motion/trigger`        | `true` will trigger a motion event on the camera or doorbell.
|                         |
| `rtsp/get`              | `true` will request that the plugin publish a message to the `rtsp` topic containing a JSON of RTSP URLs for the camera or doorbell.
|                         |
| `securitysystem/get`    | `true` will request that the plugin publish the current state of the security system to the `securitysystem` topic.
| `securitysystem/set`    | One of `AlarmOff`, `AlarmOn`, `Away`, `Home`, `Night`, `Off`. This will set the respective state on the security system accessory.
|                         |
| `snapshot/trigger`      | `true` will trigger the camera or doorbell to generate a snapshot.
| `temperature/get`       | `true` will trigger a publish event of the temperature, in Celsius, for a UniFi Protect sensor.

Some messages, such as those for the liveviews and securitysystem topics, are controller-specific. To use these topics, make sure you use the controller MAC address when you create your topic strings.

### <A NAME="doorbell-messages"></A>Doorbell Messages
Doorbell messages are a fun feature available in UniFi Protect doorbells. You can read the [doorbell documentation](https://github.com/hjdhjd/homebridge-unifi-protect/blob/main/docs/Doorbell.md) for more information about what the feature is and how it works.

Doorbell messages are published to MQTT using the topic `message`. `homebridge-unifi-protect` will publish the following JSON every time the plugin sets a message to the `message` topic:

```js
{ "message": "Some Message", "duration": 60}
```

| Property          | Description
|-------------------|----------------------------------
| `message`         | This contains the message that's set on the doorbell. An empty message, `""`, will reset the message display on the doorbell.
| `duration`        | This contains the duration that the message is set for, in seconds.

The accepted values for `duration` are:

| Duration Value    | Description
|-------------------|----------------------------------
| `0`               | This specifies that the message will be on the doorbell screen indefinitely.
| `number`          | This specifies that the message will be on the doorbell screen for `number` seconds, greater than 0.
| none              | A missing duration property will use the UniFi Protect default value of 60 seconds.

`homebridge-unifi-protect` subscribes to messages sent to the topic `message/set`. If you publish an MQTT message to the `message/set` topic containing a JSON using the above format, you can set the message on the doorbell LCD. This should provide the ability to arbitrarily set any message on the doorbell, programmatically.

`homebridge-unifi-protect` subscribes to messages sent to the topic `message/get`. If you publish an MQTT message containing `true` to the `message/get` topic, a message will be published to the `message` topic containing the current doorbell message and remaining duration in the JSON message format above.

### Some Fun Facts
  * MQTT support is disabled by default. It's enabled when an MQTT broker is specified in the configuration.
  * MQTT is configured per-controller. This allows you to have different MQTT brokers for different Protect controllers, if needed.
  * If connectivity to the broker is lost, it will perpetually retry to connect in one-minute intervals.
  * If a bad URL is provided, MQTT support will not be enabled.
