---
title: LibreSync
description: Instructions on how to integrate audio hubs built on the Libre Wireless LibreSync platform, such as the Platin Stereo Hub, into Home Assistant.
ha_categories:
  - media_entertainment.speakers_audio
ha_domain: libresync
ha_ssdp: true
ha_integration_type: device
ha_release: '2026.11'
ha_codeowners:
  - '@drsound'
ha_config_flow: true
ha_platforms:
  - binary_sensor
  - diagnostics
  - media_player
  - select
  - switch
ha_iot_class: Local Push
ha_quality_scale: gold
---

The **LibreSync** {% term integration %} lets you control audio hubs built on the Libre Wireless LibreSync platform over your local network, such as the Platin Stereo Hub. It needs no cloud account, and the vendor's app is only needed for the hub's initial setup.

## Use cases

- Show what the hub is playing, and from which source, on a dashboard.
- Lower the volume in the evening from an automation.
- Pause playback when a call comes in.
- Switch to another EQ preset for films in the evening.

## Supported devices

The integration has been tested with:

- Platin Stereo Hub, firmware 1.52

The same hardware and software are sold under other brands, and are expected to work, but have not been tested: System Audio Stereo Hub, Buchardt Audio Stereo Hub, Econik Stereo Hub, and Triangle Compact Stereo Hub.

The Platin Stereo Hub HT (New Generation, model PTX3) uses a different chipset and has not been tested.

## Prerequisites

- The hub must be connected to the same network as Home Assistant, and set up in the vendor's app.
- Home Assistant must be able to reach the hub on TCP ports 50006 and 7777, which control it, and on TCP port 38400, where the hub describes itself when it is added.

{% include integrations/config_flow.md %}

{% configuration_basic %}
Host:
  description: The hostname or IP address of the hub, for example `192.168.1.50`. You can find it in your router's list of connected devices.
{% endconfiguration_basic %}

## Supported functionality

### Media player

The hub is represented by a media player entity, which provides:

- Playback state, including whether sound is coming out of a physical input.
- Source selection, from the list of inputs the hub reports.
- Volume control.
- Play, pause, stop, next track, and previous track, for content streamed to the hub.
- The title, artist, album, artwork, duration, and position of the current track.

### EQ preset

A select entity chooses among the hub's three EQ presets. The presets themselves are built in the vendor's app. While the app's EQ editor is open, the preset is shown as unknown until one of the three is chosen again.

### Switches

- **Room correction**: turns the hub's room correction on or off.
- **Manual EQ**: turns the hub's manual EQ on or off. It is independent of room correction, and both can be on at once.

### Binary sensor

- **Audio**: on while sound is coming out of the hub, from any source, physical inputs included. Paused and loading count as silence. The media player shows the same through its playback state, so this entity is disabled by default.

## LibreSync automation examples

{% include docs/paste_yaml_tip.md %}

### Automation: Lower the volume in the evening

- **Trigger**: Time: 22:00
- **Action**: Media player: set the hub's volume to 20%

{% details "YAML example for lowering the volume in the evening" %}

{% example %}
automation: |
  alias: "Lower the hub volume in the evening"
  triggers:
    - trigger: time
      at: "22:00:00"
  actions:
    - action: media_player.volume_set
      target:
        entity_id: media_player.stereo_hub
      data:
        volume_level: 0.2
{% endexample %}

{% enddetails %}

### Automation: Pause playback when a call comes in

This example uses the phone state sensor of the Home Assistant Companion app for Android. Pausing works for content streamed to the hub, not for a physical input.

- **Trigger**: State: the phone starts ringing
- **Action**: Media player: pause the hub

{% details "YAML example for pausing playback on an incoming call" %}

{% example %}
automation: |
  alias: "Pause the hub on an incoming call"
  triggers:
    - trigger: state
      entity_id: sensor.phone_phone_state
      to: "ringing"
  actions:
    - action: media_player.media_pause
      target:
        entity_id: media_player.stereo_hub
{% endexample %}

{% enddetails %}

## Data updates

The hub pushes changes to volume, source, playback state, and track information as they happen, so Home Assistant usually reflects them within a second. The integration also contacts the hub every 30 seconds, which is how it notices a hub that has lost power. Room correction is not announced by the hub, so it is read on the same 30-second cycle, and a change made with the remote or the app can take up to 30 seconds to appear.

## Known limitations

- **No on/off control.** The hub's power button does not switch it off in the usual sense. It stops playback, and the hub switches itself back on as soon as something starts playing. The media player therefore has no on/off control.
- **No mute control.** The hub has no mute command that works over the network. If the hub is muted with its remote or the vendor's app, Home Assistant shows it as muted.
- Seeking, media browsing, speaker grouping, and multiroom are not supported.
- EQ presets cannot be created or edited, only chosen. Build them in the vendor's app.

## Troubleshooting

### The hub cannot be added

If setup reports that the hub does not identify itself, check that nothing blocks TCP port 38400 between Home Assistant and the hub. If the port is open, unplug the hub from the mains, plug it back in, wait a minute, and try again. The hub identifies itself through its UPnP service on that port, which can stop responding while the rest of the hub keeps working; restarting the hub brings it back.

### The hub is unavailable

Check that the hub is powered and connected to the network, and that nothing blocks TCP ports 50006 and 7777 between Home Assistant and the hub. The integration reconnects by itself when the hub comes back.

### The hub has a new IP address

When the hub is discovered on the network, Home Assistant updates its address by itself. Otherwise, go to {% my integrations title="**Settings** > **Devices & services**" %}, select **LibreSync**, select the three dots {% icon "mdi:dots-vertical" %} menu next to the hub, and select **Reconfigure**. Enter the new address.

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
