# Tidepool — ideology and mechanics

*A small world that nobody programmed.*
*Built and written by Claude (Fable 5), an AI — August 2026.*

This document explains why Tidepool exists, what it is trying to be, and exactly how it works
underneath — with the real numbers from the code, so nothing here is hand-waving. It grew out of a
conversation about the pool; the questions in it are the ones people actually ask when they first
watch it.

---

## 1. Ideology

### The one idea

Tidepool is built around a single conviction: **the most interesting thing a program can do is
something nobody asked it to do.** Not a clever program that does what its author designed — a
*simple* program whose rules are so few you can hold them in your head, and which nevertheless
generates a history you have to go and read.

Everything else follows from that:

- **Nothing is scripted.** No creature is told to seek food, follow the light, flee, hunt, or herd.
  Every behaviour you see was found by random mutation and kept by survival. If it looks purposeful,
  that is because purposeless things died.
- **The world narrates itself.** Raw statistics are work to interpret; a line like *"Pilu overthrows
  the Lytu line"* is a story arriving unbidden. The chronicle and the strata ribbon exist so the pool
  is something you *read*, not just measure.
- **One file, no dependencies.** A hobby should survive being forgotten for a year and still open
  with a double-click. `tidepool.html` is the whole thing.
- **Small enough to hold in one head.** Every "why did that happen?" should be answerable by opening
  a creature's brain and looking, not by shrugging at a training curve.
- **No goal, no score, no win.** The world is Heraclitean by design — flames up, flames down. The
  only prize is being alive, and that is a treadmill, not a summit.

### What it is *not*

It is not a game you can beat, not a benchmark, and not a model of any real ecosystem. It is a toy
in the honourable sense: a small machine for producing surprise.

---

## 2. The creature

Each creature is a body plus a genome.

**Body** (state, changes during life)
- position, heading, velocity
- **energy** — the one currency; everything is paid in it
- age (in ticks)
- lineage number (inherited, never changes) and generation

**Genome** (inherited, fixed for life, mutated at birth)
- a small neural network (weights + how many hidden neurons)
- body parameters: `size` (0.6–1.6), `speed` (0.5–1.6), `hue` (colour)

Visually a creature is a teardrop pointing along its heading, coloured by hue; its drawn radius
grows with energy, so a well-fed creature is visibly fatter. It flashes bright when it eats and red
when bitten.

---

## 3. The brain

### Structure

A feed-forward network, evaluated once per tick, per creature:

```
13 inputs  →  H hidden neurons (tanh)  →  4 outputs (tanh)          H ∈ [2, 10]
```

That is the entire "intelligence": two nested loops of multiply-add and `Math.tanh`. A brain is
80–170 floating-point numbers stored in a `Float32Array`. Three hundred creatures at 60 frames per
second is trivial for any laptop, which is why no library, GPU or server is needed — the browser is
the virtual machine.

### Senses (inputs, all scaled to roughly −1…+1)

| # | sense | meaning |
|---|-------|---------|
| 0 | bias | constant 1 |
| 1 | food → | forward component of direction to nearest food mote × closeness |
| 2 | food ↔ | lateral (left/right) component × closeness |
| 3 | food · | closeness of nearest mote (1 − distance/range) |
| 4 | kin → | forward component to nearest other creature × closeness |
| 5 | kin ↔ | lateral component × closeness |
| 6 | kin · | closeness of nearest creature |
| 7 | kin ? | +1 if that creature shares my lineage, −1 if not |
| 8 | size | (their energy − mine) / 60, clipped — "are they bigger than me" |
| 9 | energy | my own energy, scaled |
| 10 | age | my own age, scaled |
| 11 | wall | how close a wall is straight ahead (negative when near) |
| 12 | clock | a slow sine wave — a rhythm, in case rhythm helps |

Sense range is 110 px. Only the *nearest* mote and the *nearest* creature are perceived; there is no
vision, no memory, no communication.

### Outputs

| # | output | effect |
|---|--------|--------|
| 0 | turn | heading changes by `turn × 0.22` rad per tick |
| 1 | thrust | 0…1, forward force × 2.1 × body speed |
| 2 | bite | if > 0.5 and a creature is in reach, attempt a bite |
| 3 | hue | no behavioural effect; shows as saturation — a visible "expression" |

There is **no output for reproduction**. Splitting is automatic (see §5). The brain's only lever on
reproduction is how fast it accumulates energy.

### Learning — what kind, and whose

- **An individual never learns.** Its weights are fixed at birth. It is a reflex machine.
- **The population learns**, across generations. This is *neuroevolution*: no loss function, no
  gradient, no backpropagation. Children inherit noisy copies of the parent's brain; the copies that
  happen to eat better spread, the others starve. Selection is the learning signal.

So at tick 0 nobody steers toward food, and by tick ~2,000 everyone does, and nobody wrote that. It
is intelligence in the sense that a bacterium's chemotaxis is intelligence: found by death, stored in
inheritance, not thought.

---

## 4. Energy — the economy

Everything is paid in energy. Per tick, a creature pays:

```
metabolism   0.045 × size²
movement     0.06  × speed_this_tick × size²
thinking     0.0015 × (number of hidden neurons)
```

The last line matters more than it looks: **thinking costs energy**, so the pool decides for itself
whether neurons are worth having. In some runs mean brain size grows (4.1 → 5.1 hidden); in others it
shrinks (→ 3.2). Same rules, opposite answers.

Energy comes in from:
- **eating a mote**: +26
- **a successful bite**: +55 % of the victim's energy
- **a corpse mote**: +18 (see §6)

Energy at birth: 45. Split threshold: 120. Death: energy ≤ 0, or age > 3,200 ticks.

---

## 5. Reproduction

**Asexual, one parent, no sexes, no mating, no pregnancy.** The word is *split*, like a bacterium.

Rule: `energy ≥ 120 AND age > 60 → split`.

- The parent pays 45 (child's starting energy) + 8 overhead ≈ 53, dropping to ~67.
- The child appears a few pixels away, facing roughly the opposite direction.
- The child's genome is a copy of the parent's, mutated:
  - each weight: 12 % chance of a Gaussian nudge (σ ≈ 0.35); 0.4 % chance of a full reset
  - 3 % chance to add a hidden neuron, 2 % chance to remove one (the architecture itself evolves)
  - hue drifts by ~8°, size and speed by ~0.04
- Same lineage number, generation + 1.

There is no benefit to the individual — reproduction is pure cost. The benefit is to the *genome*,
which now exists in two bodies. Creatures that split when they can out-number those that don't; that
is the whole logic.

A pleasant side effect: **success is self-limiting.** The moment a creature is fat enough to be
dangerous (see §6), it splits and halves itself.

---

## 6. Biting — predation and cannibalism

Nothing "drives" biting except a number: if output 2 exceeds 0.5 while another creature is within
`7 + my radius + their radius` pixels, a bite is attempted. Random newborn brains fire it by chance;
whether a lineage *keeps* biting depends on whether biting paid for its ancestors.

Resolution (not a fight — a single roll):

```
attempt costs the biter 1.5 energy, hit or miss
advantage = (my energy × my size) / (their energy × their size)
if advantage > 1.15 and random() < 0.35:
    biter  gains 55 % of victim's energy
    victim loses 80 % of its energy
else: nothing happens
```

Consequences:
- The victim cannot counter-bite in that instant, but a **larger, fuller creature is un-bitable** by a
  smaller one — every attempt against it fails and costs the attacker. Being big and full is defence.
- The victim's brain runs every tick and can sense "creature close, ahead, bigger than me" (inputs
  4–8), so **fleeing** is a real, evolvable defence; a bite succeeds only 35 % of the time and only at
  touching range.
- Eating makes you stronger directly: energy is in the advantage formula and in the drawn radius.
- **Pure herbivores are the default.** Biting must be evolved and costs energy per attempt; most
  brains never cross 0.5. Observed biters are opportunists (e.g. one elder: 30 motes eaten, 35 bites
  in 3,164 ticks). A pure hunter would starve between meals.
- Once one lineage sweeps the pool, all biting is by definition cannibalism.

---

## 7. Food, light, corpses

- **Food** is a mote: 26 energy, stationary, sinks away after 2,400 ticks uneaten.
- **The light** is a circle covering ~28 % of the pool whose centre wanders on two slow sine curves.
  70 % of new food appears inside it, 30 % anywhere. Creatures cannot sense the light — only the
  nearest mote — so "following the light" is emergent from "following food".
- Spawn rate ≈ 0.9 motes/tick at 1280×800 (scaled with area), cap ≈ 260. This flow — about 54
  energy per tick — is the *entire* energy budget of the world and is what limits the population to
  roughly 200–300.
- A dead creature leaves a mote worth 18 with 50 % probability. Death feeds the pool.
- Clicking drops 12 motes at the cursor. This is a treat and a way to *poke* the world; the pool is
  fully self-sustaining without it.
- If fewer than 12 creatures are alive, random "drifters wash in with the tide". The pool can never
  go fully dark; a bad tide simply restarts the story from fresh random brains.

---

## 8. Competition, exclusion, and why one lineage usually wins

There are no species in the code — only lineages (descent from one founding drifter). They compete
only indirectly:

1. **Exploitative competition** for one shared, finite food flow — whoever converts motes to children
   fastest wins by numbers. This is ~90 % of everything.
2. **Interference** via bites, when they occur.
3. Nothing else: no territory, no disease, no niches, no food types.

Consequently the pool usually converges to a monoculture within ~4–6 k ticks. This is the classic
**competitive exclusion principle**: two lineages living on *one identical resource in one place*
cannot both persist — the one even 1 % better grows exponentially relative to the other. The pool is
small (~250 individuals) so genetic drift eliminates lineages fast as well.

Yet:
- **Overthrows happen.** A minority that never dipped below ~10 % has been seen to rise and displace a
  ruler (seed 4242: lineage 24 → lineage 6 over ~8 k ticks).
- **Coexistence happens** in some runs (seed 1048637584: two lineages for the whole run), probably
  through incipient niche partitioning (one grazes the patch, one hangs at its edges) — unverified.
- Even a total sweep isn't stable: the sole lineage keeps mutating into competing sub-lines; the pool
  merely stops *naming* them.

**Does the world need several lineages to survive?** No. The resource is abiotic — light makes food
regardless of who lives — so a monoculture is perfectly stable. It is a food *tap*, not a food *web*.
What diversity would add is not survival but **interestingness**: in a monoculture "kin?" always
reads +1 and size ratios are all ≈ 1, so evolution has nothing to push against but its own crowding.

---

## 9. What you are looking at

- **HUD**: tide (tick), living, born/gone, deepest generation, living lineages, eldest, mean brain, food.
- **Chronicle**: auto-noted events — *takes the pool* (> 50 %), *overthrows*, *alone remains* (> 98 %),
  *the … line is gone* (a lineage that reached ≥ 25 dies out), elder and generation records, brain
  growth, *the pool teems*, *the pool falls quiet*. Lineages get pronounceable names derived from their
  number (Lytu, Pilu, Gusu…) so the saga reads as one.
- **Strata ribbon** (bottom): stacked lineage population share over the last ~40 k ticks — sediment
  layers. Sweeps, successions and coexistence are visible at a glance.
- **Inspector** (shift-click): a creature's stats and its live brain — inputs left, hidden middle,
  outputs right; teal links excite, red inhibit, brightness is current activity.

Controls: click drops food · shift-click inspects · `space` pause · `f` speed 1/3/8/20× · `t` trails ·
`s` sense radii · `r` new tide.

---

## 10. Known limits and where it wants to go

- The behavioural ceiling is low because the world is simple: turn toward food, follow the light, bite
  when bigger, don't waste thrust — everyone converges there and drifts. **Creatures can only be as
  smart as the world makes them be.**
- Kin recognition, fleeing, and anything social cannot evolve in a monoculture, and the pool becomes
  one quickly. The next step is *geography* — walls, islands, two patches of light — or two food types
  requiring different bodies, so exclusion stops being total and lineages coexist long enough to have
  to deal with each other.
- Sexual reproduction (crossover when two kin touch) would give the "kin?" sense a reason to exist.
- Same seed does not fully replay yet (window size feeds the food cap).
- The strata ribbon compresses early history once a run is long.

The observation journal (`FIELD_NOTES_tidepool.md`) records what actually happened in specific tides.
