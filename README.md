# Frigate LLM Notification [iOS-aware]

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-blue?logo=homeassistant)
![Frigate](https://img.shields.io/badge/Frigate-NVR-orange)
![LLM Vision](https://img.shields.io/badge/LLM%20Vision-AI-green)

An advanced Home Assistant automation blueprint that combines **Frigate NVR**, **LLM Vision**, and **interactive mobile notifications** to create intelligent, context-aware security alerts with actionable buttons.

This version keeps the original iOS-aware notification flow, but also includes the following practical fixes and improvements:

- Correct Frigate media endpoints for snapshots, clips, HLS streams, thumbnails, and review previews
- Downloader support using the corrected event/review URLs
- iOS-aware clip handling with HLS for Apple devices
- A dedicated **Enable Video Analysis** switch so end-event notifications can be sent **with or without** LLM clip analysis
- More reliable downloader waiting logic by matching on filenames instead of full URLs

---

## Key Features

### iOS and Android support
- Uses **HLS (`.m3u8`)** for iOS clip playback
- Uses **MP4 clips** for Android and other non-iOS devices
- Lets you define which notification devices are iOS
- Keeps attachments and click actions device-aware

### Interactive notifications
- Separate button sets for **New Event** and **End Event**
- Up to **3 buttons** per notification
- Built-in **Silence** action with timer input
- Optional **Call** action
- Supports:
  - View Live
  - View Snapshot
  - View Clip
  - Silence
  - Call

### AI-powered analysis
- Optional **snapshot analysis** for the initial alert
- Optional **clip analysis** for the follow-up alert
- Separate provider/model inputs for snapshot and clip analysis
- Custom prompts for image and clip analysis
- Optional LLM Vision memory/timeline features

### Filtering and control
- Camera filter
- Severity filter
- Zone filter
- Require any zone or all zones
- Object filter
- Sub-label filter
- Vacant-zone logic using person entities
- Custom condition support

### Reliability improvements
- Optional HA Downloader integration
- Works with direct Frigate media URLs
- Correct Frigate event and review endpoints
- Parallel automation mode
- Cooldown support to reduce notification spam

---

## What changed in this version

This blueprint differs from the earlier version in a few important ways:

### Correct media URLs
The blueprint now uses the working Frigate endpoints:

- Snapshot: `{{ host }}/api/events/{{ id }}/snapshot.jpg`
- Thumbnail: `{{ host }}/api/events/{{ id }}/thumbnail.jpg`
- Clip: `{{ host }}/api/events/{{ id }}/clip.mp4`
- iOS clip stream: `{{ host }}/api/vod/event/{{ id }}/master.m3u8`
- Review preview: `{{ host }}/api/review/{{ review_id }}/preview`

### Better downloader waits
The downloader wait steps now match only on:
- `filename: "{{ id }}_snapshot.jpg"`
- `filename: "{{ id }}_clip.mp4"`

This avoids brittle failures caused by matching the full URL string.

### Optional video analysis switch
A dedicated toggle allows you to disable clip analysis while still keeping end-event notifications:

- **On** → sends clip to LLM Vision and uses the AI result
- **Off** → skips clip analysis and still sends the end-event notification using the normal fallback message

---

## Prerequisites

### Required
1. **Home Assistant**
2. **Frigate** with MQTT configured
3. **LLM Vision** with at least one configured provider
4. **Home Assistant Companion App** on the target mobile devices

### Optional
5. **Downloader integration**
   - Recommended if you want Home Assistant to save snapshots and clips locally before sending them to LLM Vision

---

## Installation

### 1. Import the blueprint
Import the blueprint into Home Assistant using your normal blueprint workflow.

### 2. Create an automation from the blueprint
Start with a minimal setup:

- **Frigate Camera:** your Frigate camera entity
- **Severity:** `alert` and `detection`
- **Notify Devices:** your mobile devices
- **iOS Devices:** only your Apple devices
- **Provider For Image Analysis:** your LLM Vision provider
- **Provider For Clip Analysis:** your LLM Vision provider
- **LLM Vision Snapshot:** enabled or disabled as desired
- **LLM Vision Clip:** enabled or disabled as desired

### 3. Test with a simple person event
Recommended first test:
- camera with person detection
- no custom zones
- one phone
- one snapshot button
- one live-view button

---

## Important inputs

## Frigate options

### Frigate Camera
Select the Frigate camera entity to monitor.

### Severity
Choose which Frigate review severities should trigger the automation:
- `alert`
- `detection`

### Frigate Zones
Optional Frigate zones required for the event.

### Custom Zones
Use this for zones that are not exposed in the selector.

### All Zones
If enabled, all selected zones must be entered before processing continues.

### Required Objects
Optional object filter such as:
- `person`
- `car`
- `dog`
- `package`

### Sub Labels Required
Optional Frigate sub-label filter.

### Frigate Host
Set your Frigate base URL, for example:

`http://192.168.1.1:5000`

Use the direct Frigate host here, not a broken proxy path.

---

## LLM Vision options

### Provider For Image Analysis
Provider used for the initial snapshot analysis.

### AI Model For Image Analysis
Model used for the snapshot analysis.

### Provider For Clip Analysis
Provider used for the clip analysis.

### AI Model For Clip Analysis
Model used for the clip analysis.

---

## Snapshot Analyse

### LLM Vision Snapshot
Enable or disable LLM snapshot analysis for the initial notification.

### Prompt
Custom prompt for the snapshot analysis.

### Generate Title
If enabled, LLM Vision can generate the title for the initial notification.

---

## Review Clip Analyse

### LLM Vision Clip
This is the new switch that controls clip analysis.

- **Enabled** → end-event clip is analyzed by LLM Vision
- **Disabled** → end-event notification is still sent, but the clip is not sent to the LLM

### Prompt
Custom prompt for clip analysis.

### Max Frames
How many frames to analyze from the clip.

### Generate Title
If enabled, LLM Vision can generate the title for the clip update notification.

---

## Downloader options

### Use The Downloader Integration
If enabled, Home Assistant downloads the snapshot/clip locally and uses the local files for LLM Vision.

### Downloader Directory
Root downloader directory.

### Downloader Sub Directory
Optional subdirectory inside the downloader root.

### Image Download Max Wait Time
Maximum wait time for snapshot downloads.

### Clip Download Max Wait Time
Maximum wait time for clip downloads.

---

## Notification options

### Notify Devices
Choose the mobile devices that should receive the notifications.

### iOS Devices
Choose which of those notify devices are iOS devices.

### Click Action New Event / End Event
These let you define where the notification should open when tapped.

### Attachment New Event / End Event
These let you define which image/video/GIF is attached to each notification.

### Cooldown
Delay between retries / repeated processing to avoid spamming.

### Button actions
You can configure up to 3 buttons for:
- **New Event**
- **End Event**

Supported button actions:
- `VIEW_LIVE`
- `VIEW_SNAPSHOT`
- `VIEW_CLIP`
- `SILENCE`
- `CALL`

---

## Recommended configurations

### Simple front door setup
- Camera: front door
- Objects: `person`
- Severity: `alert`, `detection`
- Snapshot analysis: enabled
- Clip analysis: enabled
- Button 1: View Live
- Button 2: View Snapshot
- Button 3: Silence

### Fast and lightweight setup
- Snapshot analysis: enabled
- Clip analysis: disabled
- Downloader: enabled
- One or two buttons only

This is a good option if you want quick initial alerts but do not want to spend extra time or tokens on clip analysis.

### No-AI clip update setup
- Snapshot analysis: enabled
- Clip analysis: disabled

This keeps:
- initial AI snapshot summary
- end-event notification
- clip/snapshot buttons

but skips clip analysis entirely.

---

## Media behavior

### Initial notification
Usually uses:
- thumbnail or snapshot as the attachment
- snapshot or custom action as click target

### End-event notification
Usually uses:
- review preview or iOS clip stream as the attachment
- clip or stream as the click target

### iOS handling
iOS devices receive:
- HLS stream links for clip playback
- device-specific attachment handling

### Non-iOS handling
Non-iOS devices receive:
- standard MP4 clip links
- review GIF or static image attachments depending on your configuration

---

## Template variables you can use

Useful variables in prompts and messages include:

- `{{ input_objects }}`
- `{{ objects }}`
- `{{ camera_name }}`
- `{{ zone_names }}`
- `{{ before_zones }}`
- `{{ after_zones }}`
- `{{ image }}`
- `{{ thumb }}`
- `{{ video }}`
- `{{ video_ios }}`
- `{{ gif }}`
- `{{ detections }}`
- `{{ id }}`
- `{{ review_id }}`

---

## Notes

- This blueprint is intended for **one Frigate camera per automation instance**
- Use multiple automation instances for multiple cameras
- If Downloader is enabled, maintain the download directory separately
- If clip analysis is disabled, the blueprint still sends the end-event notification
- If you use iOS devices, make sure they are selected in **iOS Devices** so the automation sends the HLS clip URL

---

## Troubleshooting

### Snapshot or clip links do not work
Check:
- your **Frigate Host**
- that Frigate is reachable from Home Assistant
- that your notification attachments and click actions use the expected variables

### Snapshot analysis works but clip analysis fails
Try:
- turning **LLM Vision Clip** off temporarily
- lowering `Max Frames`
- using Downloader
- testing the clip URL directly from Frigate

### Notifications arrive but previews do not
Check:
- the selected attachment values
- iOS device selection
- Frigate host reachability from the mobile device

### Too many missed events
Reduce cooldown or review your filtering settings.

---

## Suggested defaults

### Balanced setup
- Snapshot analysis: on
- Clip analysis: on
- Downloader: on
- Cooldown: 2 minutes

### Lightweight setup
- Snapshot analysis: on
- Clip analysis: off
- Downloader: on
- Cooldown: 1 minute

### Lowest-complexity setup
- Snapshot analysis: off
- Clip analysis: off
- Downloader: off

This turns the blueprint into a Frigate-driven smart notification automation without LLM processing.

---

## Credits

Original blueprint by **whag**.

This version keeps the original blueprint structure and selectors while updating the working Frigate media paths, downloader behavior, iOS-aware media handling, and clip-analysis toggle.
