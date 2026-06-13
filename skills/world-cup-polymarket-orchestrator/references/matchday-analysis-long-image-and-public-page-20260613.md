# Matchday analysis long image + public page workflow (2026-06-13)

Use when the owner asks to turn daily World Cup/football betting analysis into a shareable long image and/or public web page.

## What worked

1. **Gather current facts first**
   - Use `date -u` and Beijing time for timestamps.
   - Use FIFA/ESPN for schedule, venue, status, and sportsbook snapshot when available.
   - Use Polymarket Gamma event + `*-more-markets` child event for win/draw/spread/totals tokens.
   - Use CLOB `/book?token_id=...` for current best bid/ask before displaying prices.

2. **Report/news summaries**
   - For “世界杯相关报道 + 20字总结”, collect major reports from FIFA, Al Jazeera, FOX Sports, Yahoo Sports, SI, ESPN, etc.
   - Output one item per report: title, URL, and an approximately 20-Chinese-character summary.
   - Keep the global takeaway short and avoid fabricating article details not read.

3. **Long image generation**
   - A deterministic PIL-rendered image worked well for a Telegram-ready long image.
   - Recommended size used successfully: `1080 x 2200` PNG.
   - Include for each match:
     - match + kickoff time in ET and Beijing time;
     - venue/group if known;
     - current Polymarket buy prices;
     - main direction, secondary direction;
     - score lean;
     - rating;
     - risk point.
   - Save to `/root/.hermes/media_cache/` and deliver with `MEDIA:`.
   - Verify with vision before final: clear text, no truncation, all requested matches present.

4. **Public page update**
   - The active public page in this session was served from:
     `/opt/polymarket-worldcup/public_betting_page/index.html`
   - The public server was a Python `http.server` on port 80 serving that directory.
   - Before overwriting, create a timestamped backup of `index.html`.
   - Update the HTML to match the same content class as the long image; do not reuse old USA/Canada cards or unrelated content.
   - Verify locally and publicly:
     - HTTP 200;
     - page title contains the requested topic;
     - body contains all key matches;
     - render a screenshot with Playwright and inspect visually.
   - For Chinese HTML verification with `requests`, explicitly set `r.encoding='utf-8'`; otherwise `requests` may display mojibake even though the page/browser is correct.

## Concrete verification checklist

```text
[ ] Backup current index.html
[ ] New HTML title matches requested matchday/topic
[ ] HTTP 200 on origin/public URL
[ ] Body includes every requested match name
[ ] Chinese text is checked with UTF-8 decoding
[ ] Full-page screenshot generated
[ ] Vision check: content class, text clarity, no obvious truncation
[ ] Return public URL and screenshot MEDIA path
```

## Pitfalls

- Do not treat “website update” as only editing a local file; verify the served public URL.
- Do not rely on a browser timeout alone; use a simple HTTP request as a faster/public content check, then screenshot if possible.
- Do not publish a page whose title/body still contains prior-day matches.
- Do not put private account names, exact personal holdings, or pool participant allocations into a public betting page unless the owner explicitly asks for internal-only disclosure.
