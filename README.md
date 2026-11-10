# motion-design

An agent skill for building motion films **in code** — TypeScript, DOM, GSAP, Playwright stepping the clock one frame at a time, ffmpeg. No After Effects project, no mockup handed to an engineer.

It is not a tutorial. It is one director's taste, written down as rules an agent can follow: which easing curve, why that exponent, when two moves are allowed to meet, how loud the voice sits over a bed that never ducks, and what to do when the note is "feels off".

Every rule in it came from a note on real work, and carries the reasoning so it generalises past the case that produced it.

## Install

```bash
npx skills add hiaaryan/motion-design
```

## What's in it

| File | |
|---|---|
| `SKILL.md` | The working agreement, the motion law, the dwell rule, type and emphasis, the mix law, and a checklist to walk before showing anyone the work |
| `motion.md` | Curve algebra — why the exponent *is* peak velocity, why `trim` exists, logistic speed laws, the chain-lag stretch, blur as real distance covered |
| `sound.md` | Score arrangement, cue design, the mix, the VO chain |
| `ui-fidelity.md` | Reproducing a shipping product UI inside a film |
| `copy.md` | On-screen sentences and voiceover scripts |
| `working-with-me.md` | The review loop, the magnitude vocabulary, the escalation ladder |

## The short version

- **Tight through the middle, very soft at both ends.** `tightInOut(x, n)`, where `n` is the peak velocity in units of the move's own average. Never linear.
- **Two moves that are one gesture never pause between them.** Two curves that are each flat at the ends, meeting end to end, is a stop — trim the ends that meet.
- **Solve the number, never dial it.** Handover points by bisection, arrival times by inverting the camera's own curve.
- **The bed does not duck.** No sidechain, no automation. If the voice does not sit, the voice moves.
- **Summary statistics are not a sound.** Flatness, crest and centroid can all match while the result is audibly wrong, because what is missing is structure.
- **If the product draws it, draw the product's version.** Pixel for pixel, including the states it has in motion.

## License

MIT
