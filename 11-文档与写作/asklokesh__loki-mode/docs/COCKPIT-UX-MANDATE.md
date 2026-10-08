# loki start opening experience - UX / engagement / stickiness mandate

Founder mandate (2026-07-09): the moment `loki start` opens must deliver the best
UX, engagement, and stickiness in the CLI space - "nothing anyone ever attempted
with cli and terminal." This shapes S1 (start handoff) and a dedicated polish pass.

## Principles

1. Time-to-first-wow < 1 second. The instant the user hits enter, they see a
   crafted, branded, alive moment - not a wall of logs. The Autonomi mark renders
   immediately; the build identity (spec, tier, what it is about to do) reads at a
   glance.
2. Alive, not static. A subtle, tasteful motion on open (the RARV loop igniting,
   a one-line "here is the plan" reveal). No gratuitous animation; one orchestrated
   moment that lands.
3. Confidence + control in the same breath. The user immediately understands: it
   is running, it is safe, and here is exactly how to watch / steer / stop. The
   handoff card IS the hook - it makes staying to watch feel rewarding.
4. Stickiness = the cockpit pull. "Both" (dashboard + cockpit) is the default and
   the card sells it: "watch it build, live." First run teaches `loki cockpit`;
   the remembered choice makes the second run instant.
5. Progress you feel. Show the first real signal fast (tier detected, first
   iteration reasoning) so the user gets a dopamine hit of momentum, not a spinner.
6. Honest always. Never fake progress or a wow that is not backed by real state.

## Concrete asks (for the polish pass after slices merge)

- A crafted open banner: Autonomi logo + "Loki" wordmark (Fraunces feel in ASCII/
  unicode), a one-line value framing of THIS run, then the handoff card.
- The countdown to auto-Both should feel inviting ("opening your cockpit in 8s -
  press a key to choose"), not like a timeout to dread.
- The reattach hints after detach should be delightful + copy-pasteable
  (`loki cockpit` front and center).
- Optional: a tiny "what Loki will do" 3-line plan preview (from detected tier +
  spec) so the user sees intent before backgrounding.
- Measure: time-to-first-frame, and whether users pick Both (engagement proxy).

This is a follow-up polish story (S6) layered on S1 once the core lands. Do not
fake it; every wow is backed by a real field.

## HARD CONSTRAINTS (founder, 2026-07-09)

- STARE-WORTHY: the open must be a VISUAL moment people stop and admire - the
  quality of a crafted product launch screen, in the terminal. Type, spacing,
  the Autonomi mark, colors (truecolor), and one alive beat. Not a log dump.
- ZERO NEW TOOLING / ZERO FRICTION: the user installs NOTHING extra and learns NO
  new tool. No chafa, no headless Chrome as a requirement, no new binary, no
  config step. It must work out of the box on `loki start`. The dependency-free
  pure-bash render path (truecolor ANSI + optional in-terminal image where the
  terminal already supports it, silent graceful fallback otherwise) is mandatory;
  anything that would make a user run an install command is disqualified.
- IT JUST HAPPENS: the amazing open is the DEFAULT of `loki start`, not an opt-in
  flag the user must discover. No extra ceremony.
