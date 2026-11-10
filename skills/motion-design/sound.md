# Sound

Three separate things with three separate rules: the **score** (continuous, never moves), the **cues** (sound design locked to the picture), and the **voice** (sits on top, is the only thing allowed to move).

## The mix law: the bed does not duck

No sidechain. No automation on the bed. No per-line gain rides. Ever.

This is the rule they state most often and in the most ways:
- *"yo this is nice but too much fast paced also why volume is going up and down in some sections"*
- *"from 17-26 volume lowers down and music becomes blurry same 25-41 same from 46 to end"*
- *"dont change music volume at all during vo no change in music or cues (current track) please"*

An older mix pulled the music 8.2 dB under a narration covering 64% of the film. That left **4.8 seconds in the whole piece** where the score was audible on its own — and is most of why three different scores all sounded alike. If the voice does not sit, **the voice moves**: raise `VO_GAIN`, re-record the line, or re-place it. The music stays where it is.

**What dynamics come from instead:** the arrangement changing at section boundaries. *"we need to find a way to kill that harmonic music completely since ask and transition to a completely new upbeat happy uplifting music"*, *"and increase pace since ask"*. Write each section its own instrumentation and let the change of material be the change of energy.

**A flat trim is not ducking and is allowed.** *"reduce whole audio sound a bit except vo"* → one constant applied to the score **and** the cue mix. Both, because "the rest of the audio" means both: trimming only the music leaves a chime landing over a quietened bed at its old level.

That trim is also the answer to the headroom problem. A voice cannot be put six decibels over a bed that is itself mastered to the ceiling — there is no room, and trying buys distortion or a master limiter pulling everything down. Three decibels off the bed costs the score almost nothing and is worth more than any further squeezing of the voice.

### Numbers that worked

```
BED_TRIM  0.71   (−3 dB, applied to musicShape AND the cue mix)
VO_GAIN   0.9
result    peak −1.0 dBFS · rms −16 dBFS · voice-to-bed separation 9.4 dB
          (per line 7.6–14.5) · voice crest 10.3 dB · added distortion −62 dB
```

## Score

**Register wanted:** heavy, premium, satisfying, elegant, uplifting, fast-paced, futuristic.
**Register rejected, in their words:** *"childish"*, *"gamey and 8bitey"*, *"are we in kindergarden lesson?"*, *"mysterious not uplifting or revealing"*, *"awkward"*, *"not satisfying and childish"*.

**Separate the arrangement from the timbre before you throw anything away.** *"sounds too gamey and 8bitey you need to change it completely, but i liked how you paced it between frames and sections."* That is a note about instruments wearing a note about structure. Ask which one is wrong; usually the structure survives.

**Back up the current track before replacing it.** *"PERFECT now time for VO and sound effect cues backup this original music in case we need to start over."*

**Noise and texture:** wanted, in small amounts, and they will calibrate it precisely. *"the hiss is too much tone it down and also add random crackling like in vinyls but not too much"*, *"a tiny bit of noise in the audio juuust a teensy bit"*. Vinyl crackle yes; hiss as a bed, no — twice rejected as *"hissing sound in the back which is so annoying"*.

**No whooshes.** Standing instruction.

**Place the biggest lift on the mark.** The music is arranged so its peak lands on the logo.

**Watch for unreachable arrangement code.** Three section entries in the score table could never fire because the lookup returned 0 before the act boundary, and the result measured flat at −13.2 dB ±0.4 for thirteen straight seconds. A section table deserves the same guard as a layout constant: assert every entry is reachable.

## Cues

### What they are for

*"there are also no synced vfx cues when elements are being shown on screen."* Every element that arrives gets a sound, placed by the scene, in film time. Scenes do not play audio — they push `cue(at, sound, gain, variant)` and the mix assembles offline against the same timeline. Move a scene and the cues move with it; this is why the soundtrack cannot drift from the picture.

### The brief, in their words

*"futuristic computery alien thing"*, *"glitch and distortion effect"*, *"FUTURISTIC COMPUTER SOUND GLITCHY TYPES"*, *"computery crystally sounds"*.

Three structural requirements:
1. **Exactly synced.** *"they should exactly be synced with errors showing."* Frame-exact, derived from the same constants that place the element.
2. **May ring on past the element.** *"the cues can be longer too and can go on after the element is visible yk dont have to be an instant."* A cue is not required to be a click.
3. **One family, every member audibly different.** *"this is it! but they need to start and end different for every cue like sound different but same pattern."* Earlier, the complaint was the opposite failure: *"ITS SO BAD AND MONOTONOUS FOR EVERY ERROR THE TONES DONT MATCH EVEN 1 BIT."* Implement with two rng streams — one seeded per family for what is shared, one per cue for the pitched content.

Density is theirs to set and they changed their mind: *"ok dont cue for every single error but every alternate one"* → *"actually cue for all errors its ok"*. Ask once, implement, don't argue.

### The six failures, so you do not repeat them

This took six rejected attempts, and the notes went from specific to flat refusal — *"ours doesnt even come close"* — with the volume rising each round. Every failure had the same root cause in a different costume.

| # | What was done | Why it failed |
|---|---|---|
| 1 | Peak-picked onsets in the reference | Found the **bed**, not the cues |
| 2 | Averaged spectral flux over grains | Returned "flat" — scattered pitches **average** flat |
| 3 | Band-limited onset detection | Found the **drums** |
| 4 | Matched flatness, crest and centroid | All three matched; the result was audibly nothing like it |
| 5 | Built from `band()` noise | `band()` is two one-pole filters — **6 dB/oct**. A "narrow" band passes nearly everything |
| 6 | Drove the result harder to add bite | Added distortion, not structure |

**What finally worked, in order:**
1. **Frame-diff the picture** at 30fps to find where the cues actually are. The video tells you the timestamps; the audio doesn't.
2. **Separate harmonic from percussive** (median filtering) and analyse the percussive residue — *"REFERENCE-cue-isolated.mp3 is so much better without speech"* is the same instinct, stated by hand.
3. **Look at the spectrogram.** It showed horizontal resonant bands that flat noise cannot produce, and silence between events. Neither fact is visible in any summary statistic.
4. Rebuild with **`reson()`** — an SVF band-pass leg at high Q, which actually rings — struck by a 2 ms noise burst, hard-gated, then `reverb(hp(out, 2500), 0.45, 0.12)`.

**The lesson, which generalises past audio: summary statistics are not a thing.** Flatness, crest and centroid all matched while the result was wrong, because what was missing was *structure* — narrow resonances and silence between events. When a measurement says you match and a human says you don't, the human is describing a structure your measurement cannot see. Go look at the thing.

**Secondary lesson: check your DSP does what its name says.** A function called `band` that is two one-pole filters is a 6 dB/oct tilt, not a band-pass. Nothing built on it was ever narrowband.

### Gain staging cues

Check every cue family's actual peak against the rest of the kit before shipping. One (`rise`) sat **20 dB under** everything else and was inaudible at all seven of its call sites — not a mix problem, a synthesis bug, and nobody hears a missing sound.

## Voiceover

### Process

1. **Script first, always.** *"first show me script before making vo."* Never generate audio before the script is approved.
2. **Avoid silence.** *"scrap whole vo make new one and avoid silences (you can keep them where you genuinely think are needed)."* Gaps of 0.3–0.5s between lines are breath; anything over 0.7 needs a reason you can state, tied to what the picture is doing.
3. **One voice, one model, one take per line**, numbered by sequence. Keep the budget per line (how long until the next one starts) in the takes README.
4. **Re-record only the lines that changed.** Takes are files; the mix picks up whatever is there and reports `N/16`.

### The chain

```
decode → peak-normalise to 0.89
       → compress(-28 dB, 4:1, attack 6 ms, release 160 ms)
       → gain
       → look-ahead limiter (ceiling 0.9, lookahead 6 ms, release 120 ms)
       → normalise AFTER limiting
       → trim leading silence (first sample over −45 dBFS, less 20 ms)
       → place at line.at
```

Two things here were learned the hard way:

**Never use a tanh waveshaper to hold a ceiling.** *"the vo sounds mushy / like bad quality."* A waveshaper holds a ceiling by *generating harmonics*, which is distortion by definition. A look-ahead limiter turns the gain *down* instead — two passes, backwards to ramp into peaks, forwards to release. Measured difference: added harmonic distortion went to **−62 dB**.

**Normalise after limiting, not before.** Normalising to a peak the compressor failed to catch drags the whole line down — a 2 ms attack cannot catch a plosive transient, and the "fix" made the voice quieter while raising crest from 17 to 18.8 dB. Limiting first makes the final peak deterministic.

### Loudness

They asked for a louder voice three times in a row (*"the vo should be a lot louder instead"*, *"the vo should be more more more louder"*) while simultaneously forbidding any change to the music. Those are compatible: raise `VO_GAIN`, and if there is no headroom left, trim everything that is not the voice by a constant. Do not reach for the bed's automation.

### Pronunciation

A model can refuse to say a word correctly and no amount of respelling will fix it. Six spellings were tried on one name — lowercase, capitalised, preceded by "That's", hyphenated, and two single-word forms. The hyphen fixed the vowel and broke the stress; nothing fixed both. Their call was to cut the line out of the film entirely.

**Spend at most three attempts on a pronunciation, then say it cannot be fixed and propose cutting or rewording the line.** Do not burn ten minutes a take discovering this slowly.

### Judging a take

You can measure its length and its level. You cannot hear its delivery. **Say so** rather than implying a take is good — stress and emphasis are exactly what went wrong last time, and the user is the only one who can check it.

## Generation logistics

- Parallel generation is refused on the free tier; one at a time, or the user upgrades. Ten minutes for one sentence is normal on a shared queue.
- When a queue is slow, offer the manual route early: *"let me do it from the web ui if you want. tell me whats left and i'll place them by sequence in a folder."* Give them the numbered list and the target folder, don't keep waiting.
