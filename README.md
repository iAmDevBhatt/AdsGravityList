# Pi-hole gravity list — Netflix / YouTube / Amazon / JioHotstar (India)

## Set expectations first

Pi-hole blocks at the **DNS level** — it can refuse to resolve a hostname,
nothing finer-grained than that. That works great against third-party ad
networks (DoubleClick, Amazon's ad system, mobile ad SDKs) that live on
their own distinct domains.

It does **not** work against in-stream video ads on these four services,
because those ads are server-side inserted — stitched into the exact same
CDN stream as the show/video itself:

- **YouTube** (app/TV especially) — ad chunks come from `googlevideo.com`,
  the same host serving the video.
- **Netflix** ad-tier — no public ad domain exists; ads are muxed into the
  stream.
- **Prime Video** ad-tier — same story; some banner/companion ads are
  blockable (see list), the in-video ad breaks aren't.
- **JioHotstar** — especially during live sports (IPL etc.), ads are
  inserted server-side into the live stream.

If you want ads gone from in-browser YouTube, a browser extension
(uBlock Origin) will do far better than DNS, because it can inspect and
rewrite the page/player, not just block hostnames.

What this list *does* get you: fewer banner/companion ads, far less
tracking/telemetry traffic from these apps, and (per one unverified report)
possibly fewer ad calls on JioHotstar specifically.

## Files

- `streaming-ads-india.txt` — the actual domain list, in standard
  adlist/hosts syntax (one domain per line, `#` for comments).

## How to add it to Pi-hole

Pick whichever fits your setup:

1. **Bulk-paste in the admin UI** — Pi-hole Admin → *Domains* → paste the
   non-comment lines from the "SAFE TIER" section into the multi-add box,
   set type to *Deny / Exact*, pick your default group, Save.

2. **CLI, if you can SSH into the Pi-hole host:**
   ```
   pihole -b doubleclick.net googleads.g.doubleclick.net amazon-adsystem.com ...
   ```

3. **As a hosted adlist (true "gravity" source)** — push
   `streaming-ads-india.txt` to a GitHub Gist or repo, copy its *raw* URL,
   add that URL under *Settings → Adlists*, then *Tools → Update Gravity*.
   This is the way to go if you want it to stay in sync with future edits.

Only add the **SAFE TIER** domains at first.

## Testing the EXPERIMENTAL TIER safely

For `s.youtube.com`, `hesads.akamaized.net`, `aiv-delivery.net`:

1. Open Pi-hole's **Query Log** (Tools → Query Log) and leave it open on
   a second screen/tab.
2. Add **one** experimental domain as Deny.
3. Open the app (YouTube / JioHotstar / Prime Video) on the device you
   care about and actually try to play something, including live content
   if it's JioHotstar.
4. If playback fails or stalls: remove that domain immediately
   (Pi-hole Admin → Domains → delete, or `pihole -b -d <domain>`), it's
   not safe for your setup.
5. If playback is fine: watch the Query Log while an ad would normally
   play — if you see the blocked domain being queried right then, it's a
   plausible ad-ping; keep it. If it was already in your log for a
   different reason, be more cautious.
6. Move to the next experimental domain. Never add more than one at a
   time — you lose the ability to tell which change caused a break.

## Finding more domains yourself (the reliable way)

Since Netflix's ad-tier and JioHotstar's live-sports ads have no public
blocklist, the practical approach is:

1. Put the streaming device on its own Pi-hole **group**, so changes are
   isolated from your other devices.
2. Watch the **Query Log**, filtered to that device, while an ad is
   actually playing.
3. Look for domains that are clearly ad-tech (names containing `ads`,
   `adserver`, `pixel`, `track`, `analytics`, third-party ad-network
   company names) rather than the service's own core CDN hostnames.
4. Deny just that domain, retest playback end-to-end (including resuming
   after an ad break), keep it only if nothing breaks.

This is slower than importing a ready-made list, but for JioHotstar and
Netflix specifically, it's currently the only approach that's actually
based on your own traffic rather than a guess.

## If something breaks

Whitelist immediately rather than troubleshooting under pressure:

```
pihole -w <domain>
```
or Admin → Domains → add as *Allow*. Then remove it from the Deny list
too, so it doesn't fight the allow rule.

## Beyond the original four: other Indian ad-tech

Added a new section to `streaming-ads-india.txt` for ad-tech/engagement
SDKs that show up across other Indian apps (MX Player, ZEE5, SonyLIV,
news sites, etc.), not just the original four services:

- **Confirmed and safe to add:** CleverTap's actual tracking host
  (`wzrkt.com` — WHOIS + multiple tracker-blocklist sources agree),
  MoEngage's documented SDK hosts, and Taboola/Outbrain (the native-ad
  widgets at the bottom of nearly every Indian news site).
- **"Needs verification" section:** Affle, InMobi Glance (lock-screen
  ads on Xiaomi/Samsung phones), Times Internet's "Colombia" ad
  exchange, Tyroo, Vserv — real India-focused ad-tech companies, but I
  could not confirm their actual ad-serving hostnames from independent
  sources. Don't bulk-add these; find the real hostname in your own
  Query Log first.

**Known risk, specific to this broader set:** SonyLIV and MX Player are
both reported (Pi-hole/NextDNS forums) to actively detect DNS-level ad
blocking and show a "please disable your ad blocker" prompt instead of
playing. If that happens, it's almost always one specific domain — use
the Query Log method above (block everything, then watch what's blocked
when the nag screen shows up, allowlist just that one) rather than
giving up on the whole app.
