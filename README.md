# calorie-tracker

Personal weight-loss PWA. Live at <https://mmorris5.github.io/calorie-tracker/>.

Four tabs: **Groceries** (checklist by store section), **Recipes** (steps with
timers), **Today** (calorie/protein log with quick-adds and AI estimation),
**Progress** (weight chart and the weekly calorie budget).

## Starting a new week

Everything that changes week to week lives in **`week.json`** — nothing else
needs touching.

1. Edit `week.json`: set a new `weekId`, update `label`, `groceries`,
   `quickAdds`, and `recipes`.
2. Commit and push. GitHub Pages redeploys in about a minute.
3. Open the app twice on the phone — the first launch paints from cache while
   fetching the new plan, the second shows it.

When `weekId` changes, the app clears last week's ticked groceries and recipe
step marks. **Your food log and weigh-ins are never touched** — those are keyed
by date and carry across weeks.

Keep `weekId` stable while editing mid-week; changing it resets your shopping
progress.

### Shape

```jsonc
{
  "weekId": "2026-08-24",           // any unique string; changing it resets week state
  "label":  "Week of Aug 24",       // caption on the Groceries and Recipes tabs
  "groceries": [
    { "section": "Protein", "items": [ {"name":"Pork loin","qty":"2 lbs"} ] }
  ],
  "quickAdds": [ {"name":"Pork portion","cal":640,"pro":52} ],
  "recipes": [
    {
      "id": "pork",                 // must be unique AND new when content changes
      "name": "Pork loin traybake",
      "time": "45 min", "serves": 3, "cal": 640, "pro": 52,
      "ing":  ["2 lbs pork loin"],
      "steps": [
        { "t": "Sear both sides." },
        { "t": "Roast 25 min.", "timers": [["Roast", 1500]] }   // seconds
      ],
      "extra": { "label": "Add when you eat", "items": ["Lemon"] },  // optional
      "note":  "<b>Rest it.</b> Ten minutes off the heat."           // optional
    }
  ]
}
```

`qty`, `ing`, `extra`, `note`, and `timers` are all optional. Step text, `ing`,
and `note` render as HTML, so `<b>` and `<em>` work.

A malformed `week.json` is rejected and the last good copy keeps working, so a
bad edit degrades rather than bricking the app. Validate before pushing:

```bash
node -e "require('./week.json')"
```

## Deploying

```bash
git add -A && git commit -m "Week of Aug 24" && git push
```

If you change `index.html` or `sw.js`, bump `CACHE` in `sw.js` (`wl-v5` →
`wl-v6`) in the same commit, or phones keep serving the old version.
`week.json` is fetched network-first and needs no bump.

## Data and privacy

All app state — log, weigh-ins, checkmarks — lives in `localStorage` on the
phone. There is no backend and no account. Use **Export data** on the Progress
tab periodically; iOS can evict storage from web apps left unused for about a
week.

The AI estimator calls the Claude API directly from the browser. Its API key is
stored separately from app data, so **Export data never contains it**, and it is
never committed here. Repo is public — keep it that way.
