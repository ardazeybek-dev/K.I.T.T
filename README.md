# K.I.T.T.

A Knight Rider cockpit dashboard that runs in the browser. One HTML file, no build
step. Every sound in it is synthesised in code — there is not a single audio file in
this repository — and the car talks back to you.

**▶ [Try it live](https://kitt-arda.shipstatic.com)**

> Built for fun. It is a nostalgia toy, not a product: non-commercial, free, and
> happy to be laughed at. **K.I.T.T. himself speaks Turkish** — the persona, the
> replies and the synthesised voice are Turkish on purpose, because the voice *is*
> the character. The interface and the ops log are in English. Type to him in Turkish.

## What it does

- **Pedals with hold-to-press physics** — gas and brake accelerate and slow the car up
  to 220 MPH, with an engine growl and a hydraulic brake hiss built from oscillators
- **A perspective warp tunnel** that stretches and accelerates with the speedometer
- **Conversation** — ask K.I.T.T. something and he answers out loud, in character
- **A cabin media deck** — a UHF tuner and a 133 BPM drum sequencer, both synthesised
- **Propulsion and power switches** — plasma shields and ion thrusters with animated
  heat bars, a live odometer, and 7-segment neon readouts for speed, comms and climate
- **Works on a phone** — the wide cockpit grid folds into a vertical scroll panel

## How it works

This is the part worth reading.

**Every sound is synthesised, through one bus.** Oscillators and filtered noise, no
samples. Two details make it hold together. The white-noise buffer — used by the
engine, the tyre screech and the turbo whoosh — is built once, lazily, and reused;
filling a fresh 1.5-second buffer on every effect would allocate constantly during a
pedal hold. And every voice connects to a single master `GainNode` rather than to
`ctx.destination`, which is what makes the mute button actually mute: with per-effect
muting, any sound started after the toggle would slip through.

**Browsers do not just block audio — they suspend it.** Creating the `AudioContext` on
the first user gesture is the well-known half. The other half is calling `resume()`
when the context comes back `suspended`, because the context is also suspended when
the tab loses focus. On mobile Safari `resume()` returns a promise that can reject,
so the rejection is swallowed deliberately: a failed resume should leave the page
silent, not throw.

**The voice is deepened on desktop and left alone on mobile.** `pitch = 0.28` with
Microsoft Tolga on Windows gives the deep, flat K.I.T.T. delivery the character needs.
The same setting on iOS Safari produces crackling, distorted speech — the mobile
engines do not pitch-shift cleanly. So mobile plays the phone's own Turkish voice
unmodified at pitch 1.0. Same character, two different paths to it, because the
platforms genuinely differ.

**Picking the voice is a preference ladder, not a lookup.** Named high-quality Turkish
male voices first (Tolga on Windows, Cem on iOS), then any Turkish voice, then whatever
the browser hands over. The list is also cached and refreshed from `voiceschanged`,
because Chrome returns an empty array from the first `getVoices()` call — reading it
once at startup gets you nothing.

**The AI brain has three layers, tried in cost order.** An exact local command match
answers instantly with no network at all. Otherwise the question goes to Groq's
LLaMA-3. If that fails — no key, rate limit, no connection — a local Turkish
dictionary keyed on substrings answers instead. The user is never shown an API error;
K.I.T.T. simply replies from a smaller brain. A dashboard that goes mute because a
third party is down is a worse dashboard.

**No API key lives in this repository.** The key is read from `localStorage` and
entered by whoever is using the page. Cloning the repo gets you the local brain and
nothing to leak.

**Speech is cancelled the moment the gas pedal goes down.** Otherwise a sentence
that is still being spoken plays underneath the engine growl and both turn to mud.

**Speed is three mutually exclusive timers.** Gas, brake, and a passive deceleration
that only runs when neither pedal is held — and clears itself at zero instead of
ticking forever against a stationary car.

**Errors go to the ops log, never to a dialog.** Every audio and speech path is
wrapped, and what fails is printed onto the dashboard in character. An `alert()` in
a cockpit would break the illusion and block the page at the same time.

## Running it

```bash
git clone https://github.com/ardazeybek-dev/K.I.T.T.git
cd K.I.T.T
```

Open `index.html` in a browser. No server, no build step, no package install.

> Browsers block audio until you interact with the page, so click anywhere on the dark
> background first — then K.I.T.T. greets you. That is a browser rule, not a bug.

To give him the LLaMA-3 brain, paste a [Groq API key](https://console.groq.com/keys)
into the config field on the dashboard; it is kept in your browser only. Without one
he falls back to the local dictionary and still answers.

## Tech

| | |
|---|---|
| Stack | HTML, CSS, JavaScript — one file, no build step |
| Styling | TailwindCSS, Google Fonts and Lucide icons, all from a CDN |
| Audio | Web Audio API — `OscillatorNode`, `GainNode`, `BiquadFilterNode`, a shared noise buffer, one master gain bus |
| Speech | `SpeechSynthesis`, Turkish voice, pitch-shifted on desktop only |
| AI | Groq `llama-3` over `fetch`, with an offline Turkish dictionary fallback |
| Size | ~100 KB, ~2,000 lines |

## License

MIT — use it, break it, make it worse.
