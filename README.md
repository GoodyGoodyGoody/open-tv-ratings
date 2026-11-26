# Open TV Ratings: how to add your viewers

**Draft 0.1, 26 November 2026. Not open yet:** the intake address below goes live once our fake-traffic filters pass their tests. Comments are welcome now.

Broadcast Atlas publishes open TV ratings: a free, public-domain ranking of the live channels people around the world actually watch. Today the numbers come from viewers using Broadcast Atlas's own players. This rulebook explains how any player or IPTV app can add its viewers too, anonymously, in one small message a minute.

Everything here (this rulebook and the ratings) is CC0, public domain. You can copy it, change it and build on it.

## Our promise to your viewers

We count minutes. We never count people.

- **No identity.** No account, no advertising ID, no device serial number, no install key, no cookie. Nothing you send can single out a viewer.
- **No network address kept.** We use the address once, when the message arrives, to work out the country and to drop data-centre and VPN traffic. Then we throw it away.
- **Live channels only.** No on-demand video and no catch-up. Only channels in the Broadcast Atlas catalog.
- **Checked on the device first.** Your app compares the stream that is playing with our catalog before it sends anything. Anything not in the catalog is never sent, so we never learn about it.
- **Nothing exact.** No timestamps, no viewing history. Each message says only "one more minute of this stream, in this app, on this kind of device".

## What counts as a watched minute

This is the same rule Broadcast Atlas uses for its own players, word for word.

A minute counts after each full 60 seconds that the channel is actually playing with the sound on:

- **Playing:** the video clock is moving, and the player is not paused, stalled, buffering, in an error or retrying. Time spent stalled or retrying adds nothing.
- **Sound on:** not muted and the volume is above zero. Muted previews and muted autoplay never count.
- **Not idle:** after three hours with no interaction, the player asks "Still watching?". Nothing counts while that question is on screen, or after playback has stopped for inactivity.
- **Background is fine:** playback in a hidden tab or a background app still counts, as long as it really plays with sound.
- **Tuning counts nothing:** starting, tuning in or flipping past a channel earns nothing by itself. Changing channel drops the unfinished minute.

Some apps can't see all of this (for example, an add-on that only learns "started" and "stopped"). Those apps send **event minutes**: one message per minute between start and stop. Apps that apply the full rule send **measured minutes**. We keep the two apart, and the published ratings say which kind each number is.

## How to send a minute

Send one message for each counted minute, at the moment the minute completes.

`POST https://broadcastatlas.com/api/v1/heartbeat` (not live yet)

```json
{
  "spec": "open-ratings/0.1",
  "stream": "3f6c2a9e-1b7d-4c55-9e0a-7a1b2c3d4e5f",
  "app": "ExamplePlayer",
  "app_version": "2.4.1",
  "device": "tv",
  "measure": "measured"
}
```

| Field | What it is |
|---|---|
| `spec` | This rulebook's version. |
| `stream` | The Broadcast Atlas stream ID from the catalog file (below). Never a URL. |
| `app` | Your app's public name. It is a label, not a password. |
| `app_version` | Your app's version. |
| `device` | One word: `tv`, `phone`, `tablet` or `desktop`. |
| `measure` | `measured` (you apply the full rule above) or `events` (start/stop only). |

Nothing else, ever. A message with any other field is refused.

**Replies:** `204` means counted or politely ignored (we don't tell you which). `400` means the message is malformed: fix your code, don't retry. `429` means slow down. Never retry a minute more than once, never send minutes in a batch, and never send a total ("I watched 45 minutes"). Totals are refused.

## The catalog file

`GET https://broadcastatlas.com/api/v1/catalog/streams` (not live yet)

The catalog file lists every stream that can be counted: its stream ID, its channel's public ID (the iptv-org ID, such as `France3.fr`, where there is one), and a fingerprint of its address. Your app downloads it once a day. Before sending a minute, it checks that the address it is actually playing matches a catalog entry.

Match on the **stream address**, never on the channel name or a playlist label. Labels can't be trusted: a pirated copy can carry the same label as the real channel.

## Asking your viewers

- **Europe and the UK:** ask once before sending anything, for example: "Count my viewing anonymously in the open TV ratings? (No personal data, live channels only.)" Send nothing until they say yes.
- **Everywhere else:** a clear setting to turn it off, explained in a sentence.
- **Your privacy policy:** say what you send. Suggested text: "If you allow it, [App] sends Broadcast Atlas one anonymous message for each minute you watch a live channel from its catalog: the stream, the app's name and version, and the kind of device. No account, identifier or viewing history is sent. Broadcast Atlas uses your network address only to estimate your country, then discards it."

## What we do on our side

Open ratings will be gamed, so the server decides what counts.

- **Only real runs count.** We credit a minute only as part of a believable run of consecutive messages, and never more minutes than the clock allows.
- **A daily code instead of an ID.** Messages from one network address and one app share a code that is rebuilt every day and never stored past that day. We use it only to apply sensible limits (no household watches 40 channels at once, or 30 hours in a day). Two screens in one home watching the same channel count once. That small undercount is the price of keeping no identifier.
- **Data centres and VPNs are dropped,** as podcast and advertising audits do.
- **Strange patterns are flagged.** Examples: a flat 24-hour curve, impossible audiences, or one channel dominating a single app.
- **Small numbers are never published,** so a handful of fake devices can't push a channel onto the chart.
- **Our method is public.** The ratings are labelled "self-reported", and we publish this method and how much traffic our filters remove.

## What you get back

- **The ratings themselves:** a free API, a daily download (JSON and CSV) and an MCP for AI assistants. See broadcastatlas.com/developers.
- **Popularity order.** Sort your channel list the way Broadcast Atlas does: by minutes watched, worldwide or for your viewer's country. Load our ready-sorted playlist, or re-order your own list by matching each channel's iptv-org ID (the `tvg-id` in most IPTV playlists) against the ratings API or the daily file. The more apps that send minutes, the better this order gets for everyone.
- **The catalog:** stable channel IDs, logos and stream health, so your users hit fewer dead channels.
- **Credit:** the ratings name the apps that contribute (unless you'd rather not be named).

## Status and versions

- This is draft 0.1. The intake opens when the fake-traffic tests pass. We'll announce it on the developers page.
- Breaking changes get a new `spec` number. Older versions keep working for at least six months after a new one is announced.
- Questions, comments and suggested changes are welcome.
