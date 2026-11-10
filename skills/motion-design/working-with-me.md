# The review loop

The method, stated by the user themselves: *"I don't write most of it by hand. I direct Claude Code, and the trick is feeding it taste, not instructions. I hand it references: a clip whose pacing I like. Then I review it like a design crit. Every note becomes a rule it keeps across sessions."*

So: **every note is permanent.** A thing corrected once is never to be done again, in this film or the next one.

## Reading the magnitude of a note

The adjective is the instruction. Getting this wrong is the fastest way to waste a round — answering a 5% note with a redesign, or a redesign note with a 5% nudge.

| What they say | What it means | What you change |
|---|---|---|
| *"a teensy bit"*, *"juuust a teensy bit"*, *"a tad bit"* | 3–5% | one number, slightly |
| *"a bit"*, *"a bit more"*, *"slightly"* | 10–15% | one number |
| *"too much"*, *"way too"*, *"so much"* | 30–50% | one number, decisively |
| *"doesn't feel right"*, *"feels off"*, *"seems weird"* | the approach is wrong | rebuild that move, keep the beat |
| *"so off"*, *"so bad"*, *"doesn't come close"* | the whole thing is wrong | rebuild the beat |
| *"scrap"*, *"redesign"* | delete the ideas | nothing survives — new layout, new copy, new beat |
| *"revert"*, *"undo"*, *"switch back"* | the previous value was right | restore exactly; do not re-interpret |

Real sequences, to calibrate: *"increase intensity of stretch and pull so its noticeable"* → *"nice just increase the intensity of the pull a tad bit so i can notice it"* → *"no no undo increase intensity earlier was fine"*. Three rounds on one number, ending back where it started. That is normal and it is fine — what is not fine is losing the earlier value.

And: *"for bar chart add more inner padding and the gray lines should be wider than white in bar bg"* → *"actually make white and gray line in bg equal wide"*. They change their mind mid-thought. Implement the latest instruction without referencing the earlier one.

## Precision of the notes

They review frame by frame and will give you the exact coordinates of a fault:

- *"at 30.20 secs frame 1812 the ask card snaps and becomes scaled up only in that frame and second please fix"* — a one-frame floating-point boundary bug
- *"when working on it has finished the chat text suddenly moves upwards like this for 0.05secs then comes back"* — three frames
- *"there is literally one second of whitescreen before ask card shows"*
- *"see the white border has reached so far ahead and v_06 hasnt even rendered"*
- *"line drawing from frame 284-384 around the four circles is redundant"*

**Therefore: evaluate your own work frame by frame before showing it.** *"ARE YOU EVEN EVALUATING YOUR WORK FRAME TO FRAME?"* is what you get otherwise. Render stills at the junctions, sample positions every frame across each move, and check the numbers rather than the vibe.

## References are to be studied, not glanced at

Three separate escalations came from not doing this:

- In capitals: you are not getting the creative direction, go back through the reference frame by frame, you are making bland work by *just* showing the UI — *"PLEASE TAKE YOUR TIME EXAMINE THE VIDEO PROPERLY EVEN IF YOU HAVE TO EXAMINE MILLION FRAMES"*
- *"you also didnt go frame by frame in ai agent mp4 else you'd have got the pacing and motion"*
- *"you need to study the ai agent mp4 opening more densely with shit ton of frames so you can understand better"*
- *"you did a really bad job please watch [the reference] bw 1-4 second and listen for the sound effects"*

When handed a reference: extract frames at 30–60fps over the region named, measure the actual motion (positions per frame, section durations, cut points), and for audio separate it from the music before analysing. Quote numbers back. A reference is a spec, not a mood board.

## The escalation ladder

Watch for the rung you are on; each one means a different thing.

1. **Specific note.** *"the pen is going a bit too fast."* Normal. Fix the number.
2. **Repeated note.** The same thing twice means your first fix missed. Stop and diagnose rather than re-tuning — when they said the collapse ease was wrong, the ease was already correct and the *duration* differed.
3. **"Scrap."** You are solving the wrong problem. Do not salvage.
4. **Capitals.** *"NO NO NO NO NO NO NO"*, *"NAH IT LOOKS TOO LINEAR"*. You have repeated a mistake after being told. Go back to the reference and the measurements; do not try another variation.
5. **Personal.** The notes stop being about the work and start being about you. Frustration at a loop, not a verdict on the task. The correct response is to change *method*, not to apologise. They follow these with an apology of their own (*"sorry i'll be patient with you from now on"*, *"ok im gonna be patient with you from now on"*) and a more structured brief — take that brief seriously, it is usually the best spec in the conversation.

Praise is equally plain and worth recording: *"PERFECT"*, *"AWESOME LET'S DO IT"*, *"this is it!"*, *"I loved your video you did a great job very nice"*, and — after the long cue fight — that we were finally there. When you get one, **lock that state down** — back it up, write the numbers into comments — because the next phase begins immediately.

## Phase discipline

Twice imposed explicitly:

> *"ok im gonna be patient with you from now on let's do this step by step. from now on forget about timing and sounds and VO. we'll come to that later. we'll just do shots and scenes for now ok?"*

> *"now do cues and sound effects and music scrap the whole old thing. dont do vo yet. we'll do it at last"*

Order: **scenes → cues and score → VO → render.** Never work ahead. Never let polish in a later phase paper over a problem in an earlier one. When they say *"dont care about audio and vo now or the video time"*, the video's total duration is also not a constraint yet.

## Never render unprompted

*"dont render now only let me preview everything in the devserver first"*, *"always add it in preview dont talk about rendering until i say so"*, *"by preview i mean me viewing it on the dev server"*, *"dont render until i say so!!"*.

End a change with where it is, not with an offer to render. When told to render, render — and only then.

## Long jobs

**Give the estimate unprompted**, and make it real. One failure worth not repeating: the ETA for a 4K render was projected from how fast the output file was growing, scaled against a 1080p render — but the 4K file was already 18× the size of the 1080p one, so every multiplier was nonsense. Estimate from **frames completed per second**, measured on the job that is actually running.

**Do not pipe a background job's output through `tail`.** It buffers until exit, which hides the progress that was being printed all along. Leave it unpiped and read the file.

Reference figures: 3,858 frames at 3840×2160 took **608s (10.1 min)**; the mux is seconds. CRF 14 at 4K gives ~2 Mbps on flat UI content — low bitrate because there is little entropy, not because quality was traded.

## Engineering habits that prevent the notes

- **Solve every number you can derive.** Handover points by bisection, arrival times by inverting the camera, zoom targets from the frame diagonal. Dialled numbers break the moment anything upstream moves, and things move constantly.
- **Throw, don't comment.** `if (measured > constant) throw` has caught real breakage three times.
- **Half-frame epsilons at every scene boundary.** `t0 + dur` and the next scene's literal `at` are equal in decimal, not in binary.
- **Derived state goes in a per-frame syncer**, never an `onUpdate` — scrubbing with `tl.time(t, true)` suppresses callbacks, so a film that plays right can scrub wrong.
- **One writer per `transform`.** Two is the usual cause of vibration.
- **Check the probe matched what you think it matched.** A measurement that read the stage root instead of the lockup produced a confidently wrong report that the title wasn't closing.
- **Check the output you wrote is the output you computed.** A master limiter computed its gain on one buffer and the file was written from another, so it did nothing at all.
- **Check every code path is reachable.** Three score sections could never fire; thirteen seconds of film measured flat at −13.2 dB ±0.4 as a result.
- **Report failures plainly.** Overstating what you verified is caught immediately and costs the round.
