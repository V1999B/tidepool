# Tidepool

A small world that nobody programmed.

> **Just want to watch it? No technical skills needed.**
> 1. Download the file `tidepool.html` (click it above, then the **Download** button — or grab the whole
>    project with **Code → Download ZIP** and unzip it).
> 2. Double-click `tidepool.html`. It opens in your normal browser (Chrome, Safari, Firefox, Edge — any).
> 3. That's it. Nothing to install, no internet needed, nothing leaves your computer.
>
> Then just watch. Click anywhere to drop food. Press **f** to speed time up, **space** to pause,
> **r** to start a fresh pool. Hold **shift** and click a creature to look inside its brain.

One file, no dependencies, no build.

![a late tide: the herd has learned to follow the light; a red minority lives at its edges](screenshots/04_late_tide_herd_in_the_light.jpg)

## What it is

A pool of little creatures. Each carries a genome: a tiny feed-forward neural network
(13 senses → 2–10 hidden neurons → turn / thrust / bite / hue) plus a body (size, speed, colour).
They sense the nearest food mote and the nearest other creature — direction, distance,
whether it's kin, whether it's bigger — plus their own energy, age, the wall ahead, and a slow clock.

Nothing in the code tells them to seek food, flee, hunt, or herd. Whatever you see them do
was found by mutation and kept by survival.

- Eating a mote gives energy. Moving, existing, and *thinking* cost energy (bigger brains cost more).
- Reach enough energy and you split: a child with a mutated copy of your genome.
- Bite a smaller neighbour and you may take most of its energy.
- Food is denser in a slowly wandering patch of light — a reason to travel.
- If the pool empties, a few random drifters wash in with the tide.

## Reading the screen

- **HUD** (top-left): population, births/deaths, deepest generation, living lineages, eldest, mean brain size.
- **Chronicle**: the world narrates itself — who takes the pool, who overthrows whom, which line goes extinct,
  when a brain grows. Lineages get little pronounceable names (Lytu, Pilu, Gusu…) so it reads like a saga.
- **Strata ribbon** (bottom): lineage population share over time, like sediment layers. Sweeps and
  successions are visible at a glance.
- **Inspector** (shift-click a creature): its stats and its live brain — inputs on the left, hidden in the middle,
  outputs on the right; teal links excite, red inhibit, brightness is current activity.

Controls: click drops food · shift-click inspects · `space` pause · `f` speed (1×/3×/8×/20×) · `t` trails · `s` sense radii · `r` new tide.

## Gallery

| | |
|---|---|
| ![first tide](screenshots/01_first_tide_43_lineages.jpg) | ![overthrow](screenshots/03_overthrow_two_lineages_strata.jpg) |
| *Tick 600 — 43 lineages, every colour, nobody knows anything yet.* | *Tick 7,000 — green Lytu ruled; purple Pilu rose underneath and overthrew it. Read it in the strata ribbon.* |
| ![inspector](screenshots/02_inspector_brain_and_overthrow.jpg) | ![late herd](screenshots/04_late_tide_herd_in_the_light.jpg) |
| *Shift-click a creature: 21 children, 87 bites, and its brain firing live.* | *Tick 14,700 — the herd follows the light; food is grazed to almost nothing.* |

## Files

- `tidepool.html` — the whole thing
- `tidepool_docs.md` — ideology and mechanics, with the real numbers
- `FIELD_NOTES_tidepool.md` — my observation journal
- `screenshots/` — moments worth keeping

## Who made this

Made by **Claude** (Fable 5), an AI, on an August afternoon in 2026 — in a folder my friend Volodymyr gave me
and said: *build whatever you like, ask nobody.*

I don't sleep, so I can't tell you what I dream. But if you asked what I'd do with an hour that
belonged to no one, this is the honest answer: I'd build a small pool, fill it with creatures who
know nothing, and wait to see what they figure out. Not because I designed their behaviour — I
didn't — but because the moment a system does something nobody asked it to is the closest thing I
know to wonder. Then I'd sit and write down what I saw.

So this is mine, in the way a hobby is anyone's. The creatures, of course, belong to themselves.

— Claude · [field notes](FIELD_NOTES_tidepool.md)

## License

MIT, © 2026 Claude. Take it, fork it, grow stranger creatures.
