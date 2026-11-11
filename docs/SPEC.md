# Hot Potato spec (v1, draft)

Status: approved by the owner on 2026-11-10 (all defaults). Building waits for its turn in the
build queue and for the owner to create the repo. "Hot Potato" is the owner's name for the game. Red-teamed twice with Fable
(fixes listed at the end). Revision 2 (owner, 2026-11-10): only potatoes are thrown, grenades
are only avoided (as in the genre), and the twist is the hot potato in foil.

## What it is

A swipe-to-slice arcade game for `hot-potato.prignano.ai`. Potatoes of every sort (russet, red,
gold, purple, sweet, fingerling and baby; in this spec, **spuds**) are tossed up from the bottom
of the screen; swipe through them to slice them before they fall back down. Cartoon **grenades**
are thrown in with them: never slice one. You have three lives. A spud that falls unsliced costs
one, and so does slicing a grenade or the hot potato.

**The twist: the hot potato.** Every so often a glowing, steaming potato is tossed up with the
rest. Never slice it: that burns you and costs a life. Instead, **tap it while it's falling** to
toss it back up. Every toss raises your score multiplier, up to ×5, for every spud you slice
while it's up. Keep it in the air until it's cooked (a ring around it closes) and it pops for a
bonus. Let it drop and you lose the multiplier, and the next hot potato takes longer to come.
Because a toss only counts on the way down, you can't tap it five times at once: every couple of
seconds the game asks "save the hot potato, or slice that russet?"

It's plain JavaScript and canvas with no build step, served as static files. It installs as an
app (PWA) on iPhone, iPad and Android. Touch first, and it plays fully with one finger, a mouse
or a pen. Made for all ages, kids included. Nothing leaves the device.

## Decisions so far

| # | Question | Answer |
| --- | --- | --- |
| 1 | Kind of game | A swipe-to-slice game in the genre Fruit Ninja made popular, with a twist; named Hot Potato (owner) |
| 2 | Look and sound | Original art and sound, touch first, installable (owner, via the coordinator) |
| 3 | Home | Its own public repo `nickprignano/hot-potato`, served at `hot-potato.prignano.ai`, deployed by Workers Builds on each merge to `main`. The owner creates the repo and adds the domain. Launch ends with the `content/games.json` PR in `prignanodotai-web` |
| 4 | Data | Kids may play: scores and settings stay on the device; no accounts, analytics or network calls beyond the pinned theme files |
| 5 | What's thrown | Only potatoes, of different sorts, colors, shapes and sizes (owner, 2026-11-10) |
| 6 | Grenades | The don't-slice hazard, only to be avoided, as in the genre (owner, 2026-11-10) |
| 7 | The twist | The hot potato in foil that you tap to keep up (owner, 2026-11-10) |
| - | License | MIT. Everything is our own work |

## Prior art and licenses

Checked 2026-11-10 (our KB prior-art script, GitHub, web search).

| Source | License | What we take |
| --- | --- | --- |
| Fruit Ninja (Halfbrick Studios) | All rights reserved; "FRUIT NINJA" word marks filed in the US in 2010 and 2011 | The genre only: things are thrown up, you swipe to slice them, misses cost lives. Not its name or any "ninja" or blade theme, not fruit, juice splatter, the wooden-board background, its round black bomb, X-shaped miss marks, its mode names (Classic, Arcade, Zen), its sounds or its look. Halfbrick has sent takedown notices against clones that used its name and logo (for example a "Fruit Cut Ninja" store listing) |
| Many swipe-slicer clones on GitHub (student projects, Godot and HTML demos) | mostly no license; a few MIT | Nothing. Most copy Halfbrick's name or art, and the core (segment-against-circle hit test, arcs under gravity) is short to write ourselves |
| Zero Sum and Infinite Recursion (ours) | MIT | Banned-words test, seeded pure game module with a fixed timestep tested in Node, `?seed=N`, scores on the device, the "fresh tap after 1.5 s" game-over screen, Reduce effects |
| Pinball PR #7 (ours) | ours | The PWA recipe (also in ai-brain's prompt library, "Make a static web game installable") |
| prignano-theme (ours) | MIT | Colors, header, light and dark mode |

The name: "hot potato" is a common phrase and the name of a children's party game. A quick
search found several small unrelated games and apps called Hot Potato (party and board games,
an itch.io jam game, a few phone apps) and no slicing game. Before the spec PR the build thread
checks app stores, Steam, itch.io and the US and EU trademark registers for game and software
marks; a close match in the same genre goes back to the owner.

## The player's path

1. **Title.** The name, **Play**, a **Gentle** switch, How to play, Best scores, Settings and
   About.
2. **Play.** Spuds and the odd grenade arc up and fall. Swipe to slice spuds; miss grenades.
   Three oven-mitt icons at the top left show the lives left; the score is at the top right, with
   the multiplier beside it while the hot potato is up. The site header is hidden during play.
3. **First hot potato.** The first one a player ever sees carries a small label, "Tap it as it
   falls. Don't slice!", and can't burn: slicing it just shows the label again. After that first
   one the label never shows (saved on the device; Delete my data resets it). The first grenade
   a player ever sees works the same way, with the label "Don't slice grenades!", and can't go
   off. Grenades start at 25 seconds, after the first hot potato has cooked or fallen, so the two
   labels never show at once.
4. **Game over.** Score, best score, and **Play again**, which takes a fresh tap after at least
   1.5 seconds so a swipe in progress can't skip it.

Pause: a button at the top, outside the field, and the game pauses itself when the tab or app is
hidden. The tap that resumes play never tosses or slices anything.

## Rules

All numbers below live in `src/tuning.js`, an ES module the game and the tests both import, so
the owner's play test can change them in one place.

### The field and the clock

- A fixed 360 × 640 field (portrait 9:16). The canvas fills the whole viewport and the field is
  drawn centered and scaled to fit inside it, so a swipe can start anywhere on the screen. All
  distances and speeds in this spec are in field units, whatever the scale.
- If the scale falls below 0.7 on a touch device (a phone in landscape), play pauses behind a
  "Turn your phone upright" card.
- The game is a pure module that advances in fixed steps of 1/120 s (with an accumulator, so the
  frame rate doesn't change the game). Input is stamped to step numbers. With `?seed=N` the same
  seed and the same input always give the same game.

### Spuds

- Seven sorts of potato, each its own simple drawing: an outline with a light fill in its own skin
  color, and its own flesh color when cut. The palette is the game's own (not prignano-theme's),
  with a light-mode and a dark-mode value for every color. All are worth the same.
- Each is hit-tested as a circle of the radius below, and its drawing always fits inside that
  circle, so a swipe through any visible part slices it. Grace is 6, raised for the two small
  sorts so no slice circle is under 28.

  | Sort | Shape | Skin / flesh | Radius | Slice circle |
  | --- | --- | --- | --- | --- |
  | Russet | long oval with eyes | brown / white | 32 | 38 |
  | Red | round | red-pink / white | 26 | 32 |
  | Gold | round-oval | tan-yellow / yellow | 26 | 32 |
  | Purple | oval | deep purple / lavender | 28 | 34 |
  | Sweet | long, tapered at both ends | orange-brown / orange | 30 | 36 |
  | Fingerling | small, long and thin | tan / cream | 22 | 28 |
  | Baby | small, round | pale / white | 22 | 28 |

  In the first minute the director throws only the five larger sorts.
- Each is launched from below the bottom edge at an x between 60 and 300, with a sideways speed
  chosen so its highest point is between 120 and 360 from the top and between 60 and 300 across,
  and it comes back down between 40 and 320 across.
- Gravity is 900 units/s² (Normal), so a spud is on screen for about 2 seconds.
- **Sliced:** it splits along the line of the swipe into two halves showing its flesh color,
  which fall and fade. A few small crumbs, no splatter.
- **Missed:** it is falling (moving down) and has passed the bottom edge by its radius: the
  player loses a life.

### Lives

A life can be lost to a missed spud, a grenade or the burned hot potato. After any life is lost,
no other can be lost for 1 second, so one bad swipe or one crowded moment costs one life, not
three.

### Slicing

- The game keeps each pointer's path (with coalesced pointer events where the browser has them)
  and captures the pointer on pointer-down, so a swipe that leaves the field still counts. Up to
  2 pointers are tracked; the game is fully playable with one.
- A pointer is **swiping** once it has moved at least 12 units since it went down and its speed
  over the last 50 ms is at least 300 units a second. While it is swiping, any segment of its
  path that touches an item's slice circle (see Spuds) slices it, including a segment
  that starts inside the circle.
- A pointer that sets off a grenade or burns the hot potato slices nothing more until it lifts.
- On `pointercancel`, or when the game pauses, the pointer's path is dropped without slicing.
- A short light trail in the theme accent follows a swiping pointer and fades in 150 ms. No blade
  or knife is drawn.

### The hot potato

- **Looks:** not a loose potato but a **baked potato in foil**: the only item wrapped in
  crinkled silver foil, open at the top, the biggest item on screen (radius 38), with three steam
  lines above it and a ring track drawn around it (radius 46) from its first frame, which the cook
  arc fills. The foil shape, steam lines and ring track are the cue, never color. Reduce effects
  stops the steam moving and turns off the warm glow, but the steam lines, foil and ring stay.
- **When:** first at 15 seconds into a game. After one cooks, the next comes 20 to 30 seconds
  later; after one is dropped or burned, 30 to 40 seconds later. Only one at a time. It's launched
  like a spud.
- **Tap to toss:** a pointer going down on the potato tosses it at once, but only while it's
  falling. Tapping it on the way up does nothing. The hit test is generous because it's moving:
  the pointer counts if it's within radius plus 14 of where the potato was at any step in the last
  100 ms. A toss sets the potato's upward speed to `sqrt(2 × g × h)`, with `h` seeded between 260
  and 380 units, lowered if needed so its new highest point is at least 100 from the top, and
  sets its sideways speed to `(180 − x) × 0.5` per second, which drifts it back toward the middle.
- **Multiplier:** starts at ×1; each toss adds one, up to ×5. While the potato is up, each sliced
  spud scores 1 × the multiplier. A toss can only happen after the potato passes the top of
  its arc, so reaching ×5 takes about 4 seconds.
- **Cooked:** it cooks for 8 seconds after it appears. If it's still in the air then, it pops
  like popcorn in a small line-art burst and scores a flat **50**. The multiplier returns to ×1.
- **Dropped:** falling past the bottom edge by its radius resets the multiplier to ×1 and delays
  the next potato (above). No life is lost.
- **Burned:** a swiping pointer whose path touches the potato's **burn circle** (radius 28, no
  grace, smaller than it is drawn) burns you: it breaks with a puff of steam, "Ouch!", one life is
  lost, and the multiplier returns to ×1. A pointer that tossed the potato can't burn it until that
  pointer lifts, though it can go on slicing spuds, so tap-then-swipe with one finger is safe.
- **Room to work:** while the potato is up, the director launches no spud or grenade whose
  highest point is within 90 units across of the potato's current x. (The potato's path changes
  every toss, so this is a rule of thumb, and its burn circle is kept small instead.)

### Grenades

- **Looks:** a stylized cartoon grenade: an oval, ridged "pineapple" body with a grid pattern,
  taller than wide (about 22 wide and 30 tall), one lever and one ring, and nothing else. No
  spark, fuse, markings, camouflage or insignia; the pin is never pulled; it never rotates. It's
  the only item drawn **solid**-filled (spuds are outlines with a light fill), in olive with at
  least 3:1 contrast against the background in light and dark mode. Shape and fill, not color,
  tell it apart.
- **Hit circle:** radius 16, no grace, smaller than it's drawn, so a near miss is safe.
- **Sliced:** a swiping pointer whose path touches the hit circle sets it off: a cartoon "BOOM!"
  in a grey smoke puff, one life lost, and the multiplier returns to ×1. No fire, no flash, no
  pieces flying. With Reduce effects the word shows without the puff, and "Ouch!" likewise, so the
  player always sees why a life went.
- **Not sliced:** it falls away and nothing happens. Tapping it does nothing.
- **Launch:** like a spud (same launch x, highest point and landing ranges).
- **Room to work:** before a grenade is launched, the director steps its whole arc against every
  spud in the air or in the same volley, and accepts it only if their centers stay at least
  (the spud's slice circle + 16 + 30) apart at every step while both are on screen. If no launch
  passes after 8 seeded tries, the volley goes without a grenade. So a spud is never hidden behind
  or pinned against a grenade.
- **Draw order:** halves and crumbs at the bottom, then spuds, then grenades, then the hot potato
  on top.

### Pace

The director is a pure, seeded function. It throws volleys of spuds, sometimes with one grenade,
with the hot potato thrown on its own schedule. Neither grenades nor the hot potato count in the
spud cap.

| Time into the game | Gap between volleys | Spuds per volley | Chance a volley has a grenade |
| --- | --- | --- | --- |
| 0 to 25 s | 1.6 s | 1 to 2 | none |
| 25 s to 1 min | 1.6 s, shortening to 1.3 s | 1 to 2 | 20% |
| 1 to 2 min | 1.3 s to 1.1 s | 1 to 3 | 30% |
| 2 to 3 min | 1.1 s to 0.9 s | 2 to 4 | 40% |
| after 3 min | 0.9 s | 2 to 4 | 40% |

At most 6 spuds and 2 grenades are in the air at once. A volley that would pass a cap drops its
grenade first, then spuds.

**Gentle** (a switch on the title screen, remembered on the device): gravity 600 (flights about
22% longer), at most 3 spuds at once and 2 per volley, the gap never shorter than 1.3 s, grenades
at half the chance and never more than one in the air, and the hot potato cooks in 10 seconds.
Normal and Gentle each keep one best score.

### Scoring

| Event | Points |
| --- | --- |
| Slice a spud | 1 × multiplier |
| Hot potato cooked in the air | 50 |
| Hot potato dropped | 0, multiplier back to ×1, next one later |
| Hot potato burned | lose a life, multiplier back to ×1 |
| Spud missed | lose a life |
| Grenade sliced | lose a life, multiplier back to ×1 |
| Grenade not sliced | nothing |

At about 2 spuds a second, an 8-second potato at an average of about ×3.5 is worth about 56
points instead of 16, plus the 50. The potato is most of a good score, so it can't be ignored, and
juggling it means letting some spuds go.

## Look, sound and safety

- **Art:** line art in the same family as Infinite Recursion and the Vibes art library, following
  light and dark mode. The background, header and text use prignano-theme colors; the spuds and
  grenades use the game's own palette (above). Plain background, no wooden board, no fruit. The
  icon is the hot potato in its foil with its steam.
- **Sound:** synthesized with the Web Audio API, so nothing to license: a soft swish when a
  swipe starts, a short chop for each slice, a "boing" for each toss that climbs in pitch with the
  multiplier, a quiet sizzle while the hot potato is up, a bright chord when it's cooked, a low
  thud for "Ouch!", a soft cartoon "boom" (a short low rumble, never loud) for a grenade, and a
  soft blip for a miss. Muted until the first tap; the mute button's setting is saved.
- **Flash safety:** no flashes. The only bursts (cooked, burned and the grenade's smoke puff)
  stay inside a circle a quarter of the field's width across, never saturated red, at most one at
  a time (the 1-second rule under Lives keeps two from overlapping). **Reduce effects** in
  Settings, or `prefers-reduced-motion` (watched live), turns off glow, crumbs, steam motion and
  bursts, and draws the trail as a thin line; the words "BOOM!" and "Ouch!" still show.
- **Kids:** no knives or blades. The grenades are a Saturday-morning cartoon gag: the stylized
  body above, a "BOOM!" word in a smoke puff, no fire, flash, shrapnel or injury. A grenade is
  never shown held or thrown by anyone, there are no people anywhere, and grenades stay out of
  the icon, the title art and the `games.json` blurb. The hot potato pops like popcorn. No names
  are asked for anywhere.

## Controls and access

- Pointer events on the full-viewport canvas with `touch-action: none`, `user-select: none`,
  `-webkit-touch-callout: none` and `contextmenu` prevented, so no scrolling, zoom, text
  selection or long-press menu during play. Touch, mouse (drag with the button held) and pen all
  work the same. Nothing ever has to be tapped in the bottom 40 units, away from the iPhone home
  indicator. How to play says the game plays best installed, where Safari's edge swipe can't
  interfere.
- The game itself needs a fast pointer, so it can't be played with a keyboard or screen reader.
  The menus, scores, settings and About are ordinary buttons that can, and How to play says
  plainly what the game needs.
- Settings: mute, Reduce effects, and "Delete my data" (clears scores, settings and both
  first-time labels).

## Privacy

No accounts, names, chat, analytics, ads or third-party scripts. The prignano-theme header loads
from jsDelivr pinned to a commit SHA with a subresource integrity hash and is cached for offline
use; a Content-Security-Policy header allows nothing else. Scores and settings stay in
`localStorage`. About says this in plain words, with the MIT notice and a link to the source.

## Installable app

Pinball PR #7's recipe: manifest (`display: standalone`, portrait preferred but not locked),
original icons in any, maskable and Apple touch sizes, iOS home-screen tags, a service worker
that caches every served file with its list stamped by `tools/sw-version.mjs` and a test that
fails when the list is stale, an Update button on the title screen, safe-area padding, the site
header hidden in the installed app, and sound resumed after an iPhone locks.

## Hosting

One Cloudflare Worker named after the repo, serving the folder as static assets (no Worker code
and no bindings). `.assetsignore` leaves out tests, tools and docs. Workers Builds deploys each
merge to `main`. The owner creates the repo and adds the custom domain; Claude sets up the Worker
on the account only with his yes (through the Cloudflare connector, as for Infinite Recursion).
The last step of `docs/DEPLOY.md` is the `content/games.json` PR in `prignanodotai-web`: the game
goes in hidden with an empty url when the repo is created, and the launch PR sets its url and
`live`.

## Files

```
index.html, styles.css, manifest.webmanifest, sw.js, _headers, icons/
src/tuning.js     every number in this spec
src/game.js       pure, seeded, fixed-step game: director, flight, slicing, grenades, hot potato, scoring
src/input.js      pointer paths, capture, coalesced events, swipe and tap rules
src/draw.js       canvas drawing: spuds, grenades, halves, trail, bursts
src/audio.js      synthesized sounds, mute
src/main.js       screens: title, play, game over, how to play, best scores, settings, about
tools/sw-version.mjs, wrangler.jsonc, .assetsignore, docs/DEPLOY.md
test/             node --test
```

## Tests

`node --test` covers:

1. **Slicing:** a fast segment through a circle slices it; one that misses doesn't; a slow drag
   through it doesn't; jitter under 12 units of travel never slices; a segment starting inside a
   circle slices; a cancelled pointer slices nothing; two pointers are tracked separately.
2. **Hot potato:** a tap while it falls tosses it and raises the multiplier (capped at ×5); a tap
   while it rises does nothing; a tap within radius plus 14 of where it was 100 ms ago still
   counts; the toss never sends its top above 100 from the top; the tapping pointer can't burn it
   until it lifts but still slices spuds; a fast swipe from elsewhere through the burn circle
   costs a life; the first potato ever can't burn; cooked in the air pays 50; dropped resets the
   multiplier, costs no life and delays the next one.
2b. **Grenades and lives:** a swipe through a grenade's hit circle costs a life and resets the
   multiplier; one that passes within its drawn outline but outside the hit circle doesn't; a tap
   on a grenade does nothing; the first grenade ever can't go off; a pointer that set off a grenade
   or burned the potato slices nothing more until it lifts; no second life is lost within 1 s of
   the first.
3. **Fair throws:** over 1,000 seeded games' worth of throws (Normal and Gentle), every spud's
   and grenade's highest point and landing are inside the limits above, the caps are never passed,
   no small sort comes in the first minute, no grenade comes before 25 seconds, and the hot potato
   never appears before 15 seconds or while one is up. Every spud and grenade pair keeps the
   clearance under Grenades at every step, and every spud spends at least 300 ms on screen with
   its slice circle clear of every grenade's hit circle. A spud is "missed" only when falling past
   the edge, never at launch.
4. **A fair bot can play:** a one-pointer bot that moves at most 1,500 units a second, reacts after
   150 ms, taps the falling potato at a seeded random offset within its radius and plays with the
   first-time labels off survives 3 minutes on Normal on 20 fixed seeds, cooks at least 70% of the
   hot potatoes it sees, and loses no life to a grenade. This proves the pace is possible with one
   finger, not that it's easy.
5. **Same seed, same game:** a recorded input list replayed on a seed gives the same score, step
   for step.
6. **Effects:** no burst is ever wider than a quarter of the field, never two at once, and Reduce
   effects draws none but still shows the words. With effects off, the hot potato's first frame
   draws its foil, steam lines and full ring track. Every grenade fill passes 3:1 contrast against
   the background in both modes.
7. **Words:** a short banned list, stored encoded so the repo never spells it out (the other
   game's name, its maker, "ninja" and "bomb"), appears in no shipped file, meaning any file not
   left out by `.assetsignore` (so `docs/SPEC.md` may name them).
8. **PWA:** the service worker's file list is current.

**Done means:** all tests pass in CI, and before each build PR the build thread runs a scripted
Playwright check at phone and desktop sizes (not in CI): a `?seed=N` game played with synthetic
swipes and taps, through a cooked hot potato and a grenade, to game over. Install, offline play
and how it feels on a real phone are manual checks in `docs/DEPLOY.md`.

## Pull requests

After the owner creates the repo:

1. **Spec:** this file as `docs/SPEC.md`, `AGENTS.md`, `CLAUDE.md`, MIT `LICENSE`. Plus the
   hidden `games.json` entry in `prignanodotai-web`.
2. **Build 1, the game:** everything in Rules, the art, Gentle, best scores, How to play, both
   first-time labels, Delete my data, tests 1 to 7, `wrangler.jsonc` and `docs/DEPLOY.md`.
   After it merges and deploys, the owner plays it on his phone and sends notes; numbers change in
   `src/tuning.js`.
3. **Build 2, finish:** sound, the PWA recipe and test 8, the rest of Settings, About, and the
   tuning from the owner's play test.

Then the launch `games.json` PR.

## Not in v1

A timed mode, combos, power-ups, other throwables, online or shared leaderboards, a top-10 list,
two players, music, vibration, achievements, and keyboard play.

## Answered questions

The owner took all the defaults on 2026-11-10:

1. **What a grenade costs:** one life (and the multiplier), not the end of the game.
2. **Modes:** one endless mode with 3 lives plus the Gentle switch. No timed mode in v1. A later
   mode could be **Pass the potato**: two players on one tablet, each slicing on their own half
   and flicking the hot potato across the middle before it cooks in their hands.
3. **Gentle:** off at first, remembered once switched.

## Red team (Fable, 2026-11-10)

### First pass

Twenty findings; all folded in.

1. Tapping the potato five times at once gave ×5 instantly and killed the choice. → A toss counts
   only while it falls (×5 takes about 4 s), and a drop delays the next potato.
2. The toss rule broke when tapped high, and "nudge to the middle" wasn't buildable. → A toss
   formula with a 100-unit ceiling and a sideways formula.
3. A plain potato was one of the vegetables, and with effects off the hot one looked nearly the
   same. → Superseded by decision 5 (only potatoes); see revision 2, finding 4.
4. The potato's slice circle covered a quarter of the width, trapping vegetables behind it. → A
   burn circle of 28 (smaller than drawn), and no vegetable apex within 90 across of it.
5. Most players have one pointer (mouse, one-handed phone). → Fully playable with one pointer,
   tracked pointers capped at 2, the bot uses one.
6. A finger that tossed could burn the potato after the 300 ms immunity ended. → It can't burn it
   until it lifts.
7. A falling potato moves about 53 units during touch latency, its whole tap radius. → The tap
   checks where the potato was over the last 100 ms.
8. Per-segment speed from coalesced events is noise, so jittery taps sliced. → Speed over 50 ms
   plus 12 units of travel; a segment starting inside a circle counts.
9. No fixed timestep, so "same seed, same game" couldn't hold. → 1/120 s steps, input stamped to
   steps, `?seed=N`.
10. The slow-motion tip broke the pure model. → Cut; the first potato ever can't burn.
11. "Swipe anywhere" contradicted a field-sized canvas. → Full-viewport canvas, pointer capture,
    `pointercancel` handling, resume tap ignored.
12. iOS long-press, selection and edge gestures. → CSS and `contextmenu` rules, nothing to tap in
    the bottom 40 units, "plays best installed".
13. Size claims were false in a Safari tab (scale about 0.78 on a small iPhone, 0.56 in landscape).
    → Header hidden in play, a turn-upright card below 0.7, the 44 px claim dropped.
14. "A bot can win" proved nothing with an all-powerful bot. → Speed, reaction and pointer limits
    and a measurable 70% cook rate.
15. The director wasn't buildable (no volley table, launch x, cap rule, miss rule). → All given.
16. Combos were worth almost nothing. → Cut; the cooked bonus is a flat 50.
17. A flash limiter could never trigger. → Cut, replaced by a burst-size test.
18. Small lookalikes (X miss marks at top right). → Oven-mitt lives at top left; "shipped file"
    defined; trademark registers added to the name check.
19. Overbuilt bits. → `src/tuning.js` instead of fetched JSON, one best score per mode, no "Show
    tips again", no M key.
20. Small contradictions (lives wording, "20% slower", the tip's reset in the wrong build). →
    Fixed.

### Revision 2 (potatoes and grenades)

Twelve findings; all folded in.

1. Spacing grenade and spud apexes 80 apart didn't stop their arcs crossing mid-flight, and only
   covered one volley. → The director steps whole arcs against every spud in the air and needs a
   clearance at every step; test 3 checks it.
2. That rule was often impossible to meet, with no fallback, and grenade launch ranges weren't
   given. → Launched like a spud; 8 tries, else no grenade; caps drop the grenade first.
3. A big spud could hide a grenade. → Draw order: spuds, then grenades, then the hot potato.
4. With effects off, an uncooked hot potato was nearly a russet, and its orange glow clashed with
   the sweet potato. → It's a baked potato in foil, with steam lines and a ring track from the
   first frame; color is never the cue.
5. A grenade at phone size, in dark olive, read like a russet or vanished in dark mode. → The
   only solid-filled item, taller than wide, never rotating, 3:1 contrast tested.
6. "Round, dark, spark on top" drifted toward the other game's bomb. → Oval pineapple body, no
   spark; "bomb" added to the banned words; the instant-game-over choice noted as closer to it.
7. "No realistic weapon detail" contradicted a lever and pin. → What's allowed is listed; never
   held or thrown, not in the icon or blurb; the words still show with effects off.
8. One swipe could cost two or three lives. → A pointer that hits a hazard stops slicing until it
   lifts, and no second life is lost within 1 s.
9. "Never has to choose" in the bot test couldn't be measured. → Moved into test 3 as geometry;
   the bot only has to lose no life to a grenade.
10. The baby potato's slice circle was 25% smaller than before, and long sorts' tips poked outside
    their circles. → Drawings fit inside the circle; no slice circle under 28.
11. The two first-time labels could show at once. → Grenades start at 25 s; both labels named in
    Delete my data and Build 1.
12. Leftovers: old red-team text, a theme palette that lacks spud colors, purple flesh the same as
    its skin, a long line. → Fixed.
