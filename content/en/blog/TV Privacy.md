---
title: "Debloating My Living Room TV, Part 2: The Tracking"
weight: 1
resources:
params:
date: 2026-10-30T10:00:00-07:00
showDate: false
draft: false
tags: ["blog"]
---

### Debloating My Living Room TV, Part 2: The Tracking
In [part 1](../tv-debloat/) I cleaned up the Chromecast plugged into my Vizio and got it running reasonably quicker. That covered speed. This part is about the other kind of bloat: everything these two devices quietly report about what I watch.

### Checking the Vizio Hadn't Undone Itself
Back in part 1, Claude turned off three things on the Vizio through its local API:

- **Viewing Data.** This is Vizio's ACR feature. It fingerprints whatever is on screen, whether that's an app, cable or even a game console on HDMI, and sends it back to Vizio.
- **Ad Replacement.** This lets Vizio swap its own ads into live TV.
- **Limit Ad Tracking.** I turned this on.

TVs have a habit of quietly re-enabling this kind of thing after a firmware update. So twelve days later, the first thing I did was have Claude read those settings again. All three were still where I left them, on the same firmware. Good.

There's no way to get rid of WatchFree+ or the SmartCast home screen; the API even marks WatchFree+ as not hideable. Since I start everything from the Chromecast remote now, I rarely see them anyway.

### The Chromecast Tracks Too
I'd been so focused on the Vizio that I never looked at Google's side. Google TV has its own advertising ID, usage reporting and crash reporting.

First hiccup: the Chromecast had forgotten our ADB pairing. Wireless debugging switches itself off after a restart, and turning it back on wasn't enough. I had to pair again with a fresh code, same as in part 1:

```bash
adb pair <chromecast-ip>:<pairing-port> <6-digit-code>
adb connect <chromecast-ip>:<connect-port>
```

### What Claude Could Change
**App-error reports.** This is a plain system setting, so it was a one-liner:

```bash
adb shell settings put global send_action_app_error 0
adb shell settings put secure send_action_app_error 0
```

**Cast stats.** These turned out to be off already. You can check by asking the Chromecast directly on port 8008:

```bash
curl -s http://<chromecast-ip>:8008/setup/eureka_info
```

The `opt_in` block showed crash and stats reporting both `false`.

**Advertising ID.** This one's stored by Google Play services, and there's no ADB setting for it. What Claude could do was open the Ads screen on the TV and press the buttons itself. That was trickier than it sounds. The screenshot didn't show which item was selected, so Claude dumped the screen layout with `uiautomator`. It turned out the selection was sitting on a "Learn more" link at the bottom, so Claude tapped "Delete advertising ID" directly. It read the confirmation screen before accepting, and afterwards checked that the screen now offered "Get new advertising ID". The old ID was gone.

### What It Couldn't
**Usage & diagnostics** is the one that got away. Google locks the screen behind it so only its own apps can open it. Going in through the normal Settings menu just showed a loading spinner that never finished. So this one is a job for the remote: Settings > Privacy > Usage & diagnostics, and patience.

### The Option I Didn't Take
The most effective privacy fix for the Vizio would be to take it off Wi-Fi completely. Everything I watch goes through the Chromecast, so the TV doesn't need the internet. Offline, it can't send anything anywhere.

I haven't done it yet. I'd lose casting and AirPlay straight to the TV, plus firmware updates. But it's tempting, and it's easy to undo.

### Looking Back
Part 1 was about making things faster. This part was mostly about checking. The settings I changed stayed changed. Google's side had a few switches I'd never looked at. And one setting simply can't be reached without picking up the remote. As before, every change went into the same log with its undo command, so nothing here is permanent.
