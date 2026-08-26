# Weather for codriver

Current conditions and the next four hours for a city you choose, in a codriver widget slot. Data from [Open-Meteo](https://open-meteo.com) — no key, one request per refresh.

A **codriver app**: a small page that codriver embeds in a sandboxed,
cross-origin iframe beside the map in a Tesla's browser. It is built to the
contract at <https://developer.codriver.io/guides/build-an-app>.

- **Live:** <https://weather-codriver.pages.dev>
- **Slot size:** ~300 × 130 CSS px

## Install it

1. Go to <https://codriver.io/account> → **Apps**
2. Add **Custom page** and paste `https://weather-codriver.pages.dev`
3. Pick a screen slot

(Once it is a catalogue entry it will appear in the marketplace directly, with
its own settings form. `codriver-app.json` in this repo is the manifest for
that submission.)

## Settings

- **City** — any city name; Paris if you leave it alone. Resolved once through
  Open-Meteo's geocoder and cached, so the usual case is a single request.

Units follow **codriver's own setting**, so the panel shows °C or °F to match
the rest of the car, and switching units re-renders without a refetch.
Precipitation is only shown at 20% or above: a row of `0% 0% 0% 0%` is four
numbers that say nothing.

Hours are the **city's** local clock. The API is asked for `timeformat=unixtime`
for this reason — its default ISO strings carry no offset, so a browser parses
them in its own timezone and silently picks the wrong hours whenever the car and
the city differ.

## Develop it

```bash
npm install          # wrangler only
npm run serve        # http://localhost:8792/dev.html
```

`public/dev.html` fakes the codriver host: it posts a `context` message,
answers the widget's `ready`, and re-posts on resize — the same handshake the
car performs. Change the theme, units, uiSize or settings and watch the panel
react. The slot is resizable so you can prove the layout holds.

```bash
npm run deploy       # wrangler pages deploy
```

## What it does not do

- **It never learns where you are.** codriver does not pass location, speed or
  heading to an extension, so the city is a setting rather than your position — which is also why it shows the weather where you are going, not where you are.
- It cannot read your codriver session, your route, or your account. It runs on
  its own origin; the browser's same-origin policy is what enforces that, not a
  promise in this README.
- No analytics, no tracking, no cookies. The only network calls are to the data
  API named above.
- No `innerHTML` anywhere in the page: every value from the API is written with
  `textContent` onto constructed nodes.

## Notes on the car

It shares an eight-year-old GPU with a 3D map on older Teslas, so: no
animation, no `requestAnimationFrame`, one low-frequency timer, and work stops
when the frame is hidden. Chromium throttles hidden frames hard, so returning
from hidden re-checks freshness rather than assuming the timer kept running.

MIT licensed.
