# HearingHistoryLab
# Part 1 — Taking the test

## What this is

A hearing self-test for adults roughly 20 to 80 that tracks hearing
over time: a short session once a month, or after a loud concert. Each
session produces one honest measurement per pitch and ear that you
compare with your own past measurements, on your own equipment.

Everything is relative to your own history. The app never claims
absolute or clinical values.

Live test at this URL https://alanbork.github.io/HearingHistoryLab/hearing_history_lab.html

## Why history is the whole point

A single session is a snapshot; only repeat sessions show whether your
hearing is *changing*. Because every session starts from your past
thresholds instead of from scratch, check-ups get faster the longer
your history grows — and the snapshot chart can overlay your past
results with your usual range, so drift stands out at a glance instead
of hiding in a table.

## How to run it

Double-click `hearing_history_lab.html`. It opens in Chrome, Edge, or Firefox via
`file://` with no install, no build step, no libraries, and no network
requests of any kind. Your data never leaves your browser.

**Headphones — the single most important part.** Use a good pair of
wired over-ear or in-ear headphones that seal comfortably and stay put.
Wired matters: Bluetooth headphones apply their own processing and
volume quirks that drift between sessions, and every session needs to
sound identical. Never use speakers — your room changes the sound more
than your ears do.

**The same setup, every time.** Reuse the same headphones, the same
device, and the same fit each session. If anything does change, later
sessions open with a quick check that flags it honestly.

**Volume to maximum, enhancements off.** Set the device volume to max,
and switch off any equalizer, bass boost, spatial audio, or loudness
normalization. Those features quietly move levels around behind our
back.

**A quiet room.** Background noise buries the quietest beeps, which are
exactly the ones that pin down your thresholds. A closed door and a
paused TV are enough.

## What a session feels like

First time only, a warm-up: a beep repeats while you raise a slider
from silent until you just hear it, at each pitch in each ear. Then you
pick which pitches to measure on a rough preview chart.

Then the real measurement, one of two ways (your choice, remembered):

- **Ascending tone (recommended).** Beeps rise automatically and you
  tap the side you hear the moment you hear anything — one tap per
  run. Runs flow into each other silently; when the session is done a
  voice says "Session complete", so it works with your eyes closed.
- **Staircase.** Each trial plays two quick beeps or a single longer
  tone in one ear, and you answer both *which ear* and *which
  pattern* on a two-by-two grid.

Everything starts silent and climbs only with your input; nothing ever
starts loud. Set aside about 20–30 undisturbed minutes the first time.
If your ears tire, pause between pitches — a rested guess is better.
Every trial is saved the moment it is scored, so an interrupted
session can resume exactly where you left off, or be kept as-is with
unfinished spots marked approximate.

## What you get

A snapshot audiogram in plain words: lower marks mean keener hearing
(you heard quieter sounds), X is your left ear, O is your right ear,
and an arrow marks spots not heard even at the loudest setting. Flip
between sessions with the switcher — hovering a step button ghosts the
other session faintly over the chart (on touch, press to preview and
release to switch). A slider under the chart overlays your past
results with your usual range, so a today that moved shows up
instantly. A history chart plots every session over time, and every
session is one expandable row showing its full threshold grid, with mean
seconds per trial on each row. A "Start from" dropdown chooses "Continue
from most recent data" or "Fresh slider checks" for the next session.

The home screen always prints what is stored — sessions, trials, and
thresholds — on one line, and the results screen prints a save receipt
with the same totals, so there is never any doubt about what was kept.
Export JSON downloads everything as one file, with a matching Import
JSON to restore it; a Reset everything screen, guarded by five
checkboxes plus an export-first escape hatch, wipes back to scratch.

---

# Part 2 — How it measures, honestly

## What a threshold means here

Each reported threshold is the quietest level where you reliably
*identified* the sound — getting both which ear and which pattern
right about 75% of the time. A wrong-ear answer means the sound wasn't
heard in that ear at that level; a wrong count means it was heard but
miscounted. Both count as misses, but wrong-ear answers are tallied
separately as "not heard" events.

## The two methods

The staircase is an adaptive identification task. Each ear-and-pitch
track starts clearly audible, takes big steps down first, then samples
near the edge: after early misses it refits a psychometric curve to
everything that track has seen and places each next trial where the
fit predicts the edge, ending with the fitted curve's 75% crossing.

The ascending tone is an automatic slider. Each run plays beeps that
rise until you tap the side you hear. Every spot opens with an
unscored practice sweep from clearly audible, then one quick unscored
coarse sweep that finds the neighborhood fast, then up to three slow
scored sweeps — stopping after two when they agree. The threshold sits
one step below the tap center (the mean of an agreeing pair, or the
median of three so one bad tap can't drag it), which also roughly
cancels reaction-time overshoot at this ramp rate. A wrong-side tap
restarts the run from quiet.

Sessions record their method everywhere, and the results screen
compares against your last session of the other method — mean
difference plus direction, plus mean seconds per trial — so the two
methods can be baked off on both accuracy and speed.

## Units and calibration, plainly

Levels are the app's own volume scale ("app dB"): they behave like
decibels for comparing you with yourself, but depend on your
headphones, device, and volume setting. Differences in app dB are real
dB; absolute values are hardware-dependent and only loosely
calibrated — never clinical values. The bottom of the scale is true
digital silence, and later sessions start each pitch from your most
recent saved levels (older sessions backfill pitches the latest one
skipped), falling back to the original slider checks only for spots
with no data at all.

## Comparability

Returning sessions open with a short setup check at 1000 Hz in each
ear. This only verifies your setup: if either ear differs too far from
your stored curve, the session is flagged as possibly not comparable.
The flag travels with the session through history, charts, and
exports.

## What is stored, in brief

Each session keeps its full trial list, per-spot results (threshold,
trial/miss counts, status), the settings in force, and a debug log.
Spots end as measured, approximate (little data, disagreement, or an
early finish), not tested, or not heard at maximum. The original
slider checks live on as "session zero" alongside real sessions. A
live draft is rewritten after every trial, so a reload or crash loses
nothing; uncaught errors, dead audio, and corrupt data stop the app on
a loud full-state dump instead of failing silent.

## Has it been checked?

A seeded 120-track virtual-listener simulation gates the staircase at
mean ≤ 16 trials per track with ≥ 60% of estimates within 3 dB and
≥ 75% within 5 dB of truth — currently 15.3 trials, 65.0%, 78.3%.
Real field sessions are refit to show estimates converging next to
their miss clusters within about 11 trials. Known weak spots: shallow
slopes need more trials than this short format provides, and any
systematic offset between the two methods stays visible in the
bake-off rather than being assumed away.

---

# Part 3 — Implementation detail

## Architecture

One file: `index.html` holds the app, its styles, and one inline
script, with zero dependencies and zero network use — no external
references, `fetch`, or `XMLHttpRequest`. Headless tests boot that
real script in Node; browser tests drive the real file over CDP.

## Where the data lives

All data stays in your browser's `localStorage`, under eight keys
(kept under their original `hearingLog.*` names so older browsers
keep working):

- `hearingLog.sessions.v1`: every session — date, full trial list, and per
  ear-and-band results (threshold, slope, trial/miss/not-heard counts,
  status, retry count), plus the comparability flag.
- `hearingLog.earSwap.v1`: one setting, the permanent ear-label swap.
- `hearingLog.slider.v1`: slider-phase progress (run list plus seeds so
  far), so checks can resume or be reused instead of repeated.
- `hearingLog.bands.v1`: your last band choice, so every band picker
  (home screen and snapshot) pre-ticks it instead of starting over.
- `hearingLog.startSource.v1`: whether returning sessions start each
  pitch from the most recent saved levels or from fresh slider checks.
- `hearingLog.draft.v1`: the live session draft — every scored trial, both
  staircase states, scheduler, beep balance, and seeds — rewritten after
  every trial, so a reload or crash loses nothing; cleared when the
  session finishes.
- `hearingLog.mode.v1`: the chosen measurement method (staircase or
  ascending tone), offered as one button each on the home and snapshot
  screens.
- `hearingLog.history.v1`: how many months of past sessions the snapshot
  chart overlays (0 hides them; the default), so the history depth you
  picked is still there next launch.

Return-check values persist inside each session's stored log (setup
verification only, never used as thresholds). Export JSON downloads
everything as one file, generated locally from a Blob: all sessions —
each with every trial, per-spot start provenance, the full debug log,
and the settings in force (build, ear swap, bands tested, beep
balance, early-finish mark) — plus slider checks, band choice, mode,
start source, history depth, ear swap, any live draft or in-progress
session, and the build tag. New exports carry the `HearingHistoryLab`
app tag; imports still accept the original `Hearing Log` tag, preview
new versus duplicate sessions, restore checks only when none are
saved, apply the file's settings, and never import drafts.

The console command `copy(HearingHistoryLabDebug.csv())` gives the
same trials as CSV, one row per scored trial across all sessions:
session date, method, trial number, band, ear and level presented,
beeps presented, ear and count answered, correctness, the track's
final threshold and slope, retry count — plus run number (ascending,
0 for the unscored practice sweep, 1 for the coarse sweep) and
track-trial number with staircase/fitter phase (staircase). Timing
columns record, per trial, the start timestamp and wall duration, the
answer latency and replay count (staircase), and the run time,
first-beep latency, beeps played, wrong-side restarts, and overlay-pause
time (ascending); sessions also stamp their end time and total duration.
Old sessions predate the newest columns and leave them blank except
trial number, which falls back to row order. Voided skips are not rows
(they were never scored); they stay in the debug log with their levels.
Fields stay blank for trials that predate them.
(`HearingLogDebug` remains as a legacy alias.)

## Staircase parameters

Each track starts +16 dB above its seed with one orientation trial,
then two 8 dB dives per correct answer, one 4 dB bridging step, then
2 dB steps until the first miss. From there: down 2 dB after a correct
answer, up 6 dB after a miss (that ratio settles near 75% correct).
After the first reversal the steps halve to down 1 / up 3. After 2
misses it switches to the fitter: refit a logistic curve after every
trial, place the next trial at the fit's predicted 70% point at
tenth-of-a-dB precision (half-dB nudge as backstop so no level repeats
twice running). A track ends after 8 fitter trials (or 30 trials at
most); the reported threshold is the final curve's 75% crossing.
Fitter placements round to 1 dB; a degenerate intermediate fit falls
back to one staircase step so the track keeps exploring instead of
stalling. A final degenerate fit still yields an approximate threshold
from the best grid-edge fit, with a 10-trial continuation for that
track alone (retries don't consume them) that may extend up to 10
trials past the cap. When one ear's track finishes first, the rest of
the band goes to the other ear, and retried trials don't consume beep
balance.

## Ascending parameters

Runs open with a 0.8 s lead-in; taps before the first beep are
ignored. Beeps rise every 0.3 s. Practice starts 8 dB above the
estimate; the coarse sweep runs 2 dB steps from 10 dB below; scored
sweeps run 0.5 dB steps from 4 dB below the running estimate, up to
three, stopping after two that agree within 3 dB. Threshold = tap
center minus one 0.5 dB step (mean of two, median of three). Up to 3
wrong-side restarts per run; the fourth tap is accepted but flagged
approximate. A scored run with no tap after 12 beeps at maximum ends
the spot as not heard. Runs end silently; the session end speaks
"Session complete" through a device-local voice with a three-note
rising cue as backup. Arrow keys answer left/right during a sweep
(held-key repeats ignored, dead while an overlay is open), and the
retry button cleanly restarts the run. Opening the progress or debug
overlay pauses the sweep and resumes it on close. The progress overlay
draws tap marks with a dashed line at the threshold, exactly like the
staircase's fitted line. Ascending thresholds seed later sessions of
either method (treated as the curve middle with a default slope), and
the staircase's starts do the same in reverse.

## Audio details

The setup-check tone is 1000 Hz (2 kHz sounded harsh). Level mapping
is logarithmic (dB), the perceptually uniform taper; at or below
−80 dB no tone is scheduled at all (true digital silence). Stereo is
full left/right channel routing with a ChannelMerger fallback. Slider
runs repeat a single beep about every 0.3 s; the staircase's single
tone is one continuous 800 ms beep matching the two-beep pattern's
total length.

## Interface mechanics

The slider screen is one persistent control set: the slider stays
live, three buttons answer everything ("I hear it on the left /
right", "I don't hear it at all" — the last unlocks only at maximum),
the recorded value is the slider's final position, and Back redoes the
previous check (dropping its seed). A snapshot screen sits between checks and measurement:
a rough audiogram plus pitch tick-boxes (audiologist convention —
blue X left, red O right, upward arrows for no response); bands with
no response start unticked. A pre-session briefing names any needed
checks with a way back, and a get-ready screen with the setup reminder
is the last tap before the first sound. The answer grid is live from
tone start; answering cuts the tone; a green check acknowledges each
response for half a second. Tapping the centered speaker icon replays
without consuming the trial; two skips in a row nudge a track 2 dB
easier. Beep counts stay session-balanced by forcing the minority side
past a lead of two. The ear-label swap is offered when loud trials go
wrong-eared too often. Snapshot ghost buttons preview on hover,
touch-down ghosts immediately, dragging tracks the ghost, and release
commits only over the button (long-press menus suppressed). History
dots are faint per-session marks with mean ± spread bars; hovering
reads the numbers.

Debug mode is on by default (home-screen and snapshot checkboxes, or
`?debug=0` in the address, switch it off): it shows the previous
trial's right answer, level, verdict, and full track detail in a card
below the answer grid, never blocking the next trial. "Copy debug log"
buttons and the console commands
`copy(HearingHistoryLabDebug.dump())` and
`copy(JSON.stringify(HearingHistoryLabDebug.session()))` export the
log and the live session any time, mid-session included. The failure
dump shows what failed, the build tag, every stored key, live session
state, and the debug log with select-all and copy; the routing
decision (start source, bands, and the per-spot reason for every fresh
check) is logged too. Slider ear order is randomized (random opening
ear, coin flip per band) while bands stay in fixed order; every band
picker pre-ticks your last band choice, and returning users boot into
the snapshot built from their last session. If the browser can't store
data at all, a banner says so instead of silently losing results, and
a first-run screen with leftover settings warns that past sessions may
have been cleared elsewhere. The ear-label swap is offered when loud
trials (at or above −12 dB, at least 8 of them) go wrong-eared more
than 60% of the time. "See progress so far" and "View trial data"
graph every trial per track — green dots for correct, red crosses for
misses, dashed fitted threshold — at any point.

## Running the tests

`node tests/run.mjs` (or `npm test`) runs the whole suite with no
dependencies beyond Node. Fourteen files, each also runnable on its
own: boot and routing, snapshot audiogram and band picker (including
a label-collision guard), debug default, staircase behavior plus the
120-virtual-listener accuracy sim with pass gates, replay/skip-hint
behavior driving a real trial with fake audio, the progress-graph
overlay, a replay harness that refits the estimator on real field
sessions in `sessions/`, a draft/resume E2E that crashes a real run
mid-band, an ascending-tone suite (sweep state machine, timing and
restarts, crash-resume, bake-off plumbing), a session-comparison suite
(switcher, expander rows, slider session zero, CSV columns), a
slider-briefing suite (briefing, setup checks, most-recent-start
backfill, ghost guards), a history-overlay suite (past-results dots
with usual-range bars, the depth slider, import/export of the
setting), and a fatal-dump suite (loud failure screen, no-audio and
unloadable-data guards, routing-decision logging).
`node tests/staircase.mjs matrix` re-runs the staircase design
variants. `npm run test:browser` boots the real file in headless
Chrome/Edge over CDP (smoke plus a real-touch ghost test).

Built with Muse Code and Muse Spark 1.3 during the Meta Global AI Developer Hackathon, November 2026.
