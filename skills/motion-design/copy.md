# Copy — on screen and in voice

Every sentence is reviewed as an argument, not as a string. The notes below are verbatim critiques; each one is a rule.

## The three things that get a line cut

**1. It tells the user something they already assume.**

> *"It reads your real store — not a copy, not a summary."*
> → *"this copy is problematic because the user already know its not a copy or summary obv in a product it would use live data"*

Never defend against a doubt the viewer does not have. You are drawing attention to a weakness nobody suspected.

**2. It brags with a count.**

> *"Eighty-five things it can reach, across every part of it. / Fifty-three it only reads. The rest have to ask you first."*
> → *"no this is bad because we dont want user to know how many things it can do but what it can do"*

Numbers of internal things are for the builders. Say the *kind* of thing, not how many. The replacement was a list of domains — products and prices, orders and refunds, payouts and payments — with the real list on screen underneath doing the counting silently.

**3. It implies a limit you do not mean.**

> *"And then it stops."*
> → *"dont want to imply it stops"*

Read every sentence for what it concedes as well as what it claims.

## Other standing rules

**No scene-setting.** A draft opened with a morning routine. *"buddy i dont want that morning part please."* Start on the subject.

**No dangling referents.** *"every one of these what? explain it."* If a sentence needs the viewer to carry a noun from two beats ago, name the noun. This is why the on-screen statement uses the product's own badge inline instead of the word "it" — it is the one place in the film that makes a claim about reach, so it spends the name there.

**Open on the problem, in the viewer's words.** *"everything that goes wrong with your store it should start with this"*, and earlier *"you can make the tile section more error focused and longer to discuss the problem"*, *"we need to redo the script talk more about the problem initially"*.

**Name the principle, not the mechanic.** The closing line went from *"Your press is what does it"* to **"Your consent is what does it."** A press is what the user physically does; consent is what it means. When both readings are available, take the one that states the idea.

**Say a thing once.** The close originally had a wordmark, a caption, and a spoken line, over a mark that already said all three. All three were removed; the final close is one glyph. *"in close i only need [the] logo in middle but smaller."* An ending is the first place to cut.

**One line, one sentence.** *"still too much text to scan you should not add multiple lines only one line on sentence."*

**Match the voice to the type.** Both on-screen statements are spoken verbatim. An earlier draft went silent over them, on the grounds that they are the film's own voice, and that is what left the script full of holes. The voice is slower than the type (≈3.5s against 1.7s to fill), so the type finishes and the voice carries through the hold.

**Emphasis in the read must match emphasis in the type.** If the statement lights *Nothing … until you approve it* at full ink and mutes the rest, the read stresses the same words. Otherwise the two argue.

## Naming

The agent had been called `<company> agent`, and the note was that this *"sounds too generic like 11labs have elevenlabs architect shopify has sidekick etc"*. It got a short human name instead.

A product's agent gets a short human name, lowercase in body text, set in the product's own badge component when it appears in a sentence. The company name can live outside the badge, with only the agent's name inside it.

Treatment notes, in order, so you can see how fine the tuning gets: white pill → make the badge black → make it white again → undo that, *"keep the pill dark only"* → remove its border → *"the strands dont need a background of their own also pill should animate width as text appears"* → *"spacing only required between [the name] and strands not on both sides of strands"* → *"strands a bit too small and not aligned in center with [the name]"* → *"would be great if we can align [the] badge text with text baseline"*.

The point is not the final values. The point is that a badge in a sentence gets measured against the baseline, the leading, the cap height and the word spacing of the line it sits in, and every one of those will be checked.

## Structure of the argument

The film's order, after it was reversed once on their instruction (*"i am thinking we put the tile section first then the intro then the white section"*):

1. **The problem**, dark, dense, fast — everything that goes wrong, each small enough to ignore.
2. **The name**, on a pinch and a push to white.
3. **What it knows and what you can ask it**, light, on the product's own UI.
4. **The reach**, as a fall past the real list of capabilities.
5. **The gate** — the one sentence that says nothing happens without approval.
6. **The proof** — a change proposed and waiting for you, dark.
7. **The mark**, alone, five seconds, no words.

The failure mode this structure exists to avoid, in their words: *"looks like im watching a boring presentation ppt"* and *"you're making bland stuff by just showing the UI (which is OK, but that is only it)"*.
