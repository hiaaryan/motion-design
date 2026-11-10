# UI fidelity

When a film shows the product, it shows **the product** — not an impression of it. This is the rule that has caused the most rework, and the only one they have restated as a near-threat: *"strictly use our ui no other card ui acceptable"*.

## Before building anything that looks like the product

Go read the real components. Not the design system docs — the components, in the app that ships. The first version of one film was rejected with *"the dashboard ui is not matched cut to cut at all not even close"*, and the instruction that followed was *"you can use [the app] and [the shared ui package] to match the ui"*, then *"extract every piece every edge case of the ui"*.

Build an inventory first: every card, row, badge, input, connector, orb, ring, chart, empty state and loading state you might need, with its real dimensions, radii, type sizes, colours and spacings taken from source. Confirm the inventory before you animate anything.

## The deviations they have caught

Every one of these is a real note. They are listed because each one is a *class* of mistake, not a one-off:

| Note | Class |
|---|---|
| *"the thinking orb dots are too dense in chat why arent they exactly like in the dash ui?"* | sub-component parameters |
| *"why arent strands in center?"* | alignment within a component |
| *"also why does strand not go when chat has started like in actual ui"* | state transitions the product has |
| *"why it doesnt show thought for n secs and todo with connectors when they are present in current ui?"* | whole features omitted |
| *"the connector should start below the icon not from center of it"* | anchor points |
| *"when 1/2 is done it should show progress ring like in ui"* | progress affordances |
| *"when bars are rising they have correct radius but when they settle the radius decreases?"* | correct at rest, wrong in motion |
| *"why bar radius so inconsistent"* | radius drift across states |
| *"the thinking orb looks so low res when zoomed"* | drawn small then scaled up |
| *"connectors are a bit weird now and mismatching from ui after the squircle fix"* | a global change breaking a local match |

**The pattern:** they zoom in. Anything you approximated at normal size will be caught at 4× — *"also when typing is happening in chat input i need you to super zoom and follow the cursor in that"*. Draw every element at the largest size it will ever appear and scale **down**.

## Absolute rules

**Never invent a colour.** Use the theme's tokens. If something needs a value the theme does not have, say so instead of choosing one. (The one documented exception carried its own justification in the code: an inline badge on a white statement keeps the product's own black ground, because the strand shader is emissive and washes out to pastel over a light fill.)

**Never show developer surface.** *"why are you showing code??? we are not making this for devs"*, *"dont show json in approval card"*. No code, no JSON, no ids, no internal counts. If the real UI shows a technical value, the film shows the human-facing one.

**No card appears in two sections.** *"also the cards are too repetitive across sections, no section should have same card."* Each beat earns its own object. If two beats genuinely need the same component, one of those beats is redundant — cut it rather than repeat.

**Roundness is squircle.** *"i want all the roundness to be squircle like apple on the corners."* And when something is meant to be a pill — badges, approve buttons, user message bubbles — it is fully round, not nearly: *"approved badge should be fully rounded"*, *"has to this button should be fully rounded"*, *"same with user chat message box"*.

Watch for the global-change-breaks-local-match failure: a squircle pass once changed radii everywhere and broke connectors that matched the product. After any sweeping style change, re-check every component against source.

## Reproducing behaviour, not just appearance

The product's *interactions* are part of the UI. Things they required because the real app does them:

- A submitted message becomes a chat bubble — *"the chat doesnt come up in chatbox when submitted"*.
- Thinking state, with a duration — *"add a thought for 1 sec after todo card too"*.
- Results arrive **staggered**, never all at once — *"there is waiting loading in thinking and results appear staggered not instantly"*.
- To-dos tick one by one, then the card that follows them — *"first todo should show up with ticking animation 1by one of the tasks and then the analytics card after it"*.
- Typing is letter by letter, with a visible cursor, and the cursor moves smoothly — *"i want the cursor to show when typing and also add letter by letter"*, *"i want letter reveal and cursor movement to be smooth not snappy"*.
- The caret starts where the real input puts it — *"why is the cursor after placeholder initially?"*.

**Scrolling is its own problem.** It was wrong in both directions and they said so both times: *"the chat moving and scrolling too snappy"*, then after a fix *"you messed up the chat scroll i want it switched back it feels so lethargic now"*, then *"the scrolling in ask card chatbox is not 60fps seems to lag make it smooth af"*, then *"now scrolling in chat is too slow"*. Scroll speed is a per-surface tuning job; change it only when asked, and change it by small increments.

## Layout notes they have given

- **Centre things in the frame.** *"the coupons card is also not in the middle of the screen"*, *"the chat card isnt in center when zoomed"*, *"why arent strands in center?"*
- **Connectors leave from the middle of an edge**, not a corner: *"the connector should start from middle bottom of the ask card instead of left and also in middle top of the coupons card"*.
- **Mask long rules at their ends** so they fade rather than stop: *"the vertical line should have mask from 60% on y ends"*; the same instinct on the badge strands, *"add mask y from 90 in strands in pill"* → *"make it 80"*.
- **Padding is checked top against bottom.** *"padding is not even below and top of inner card add more in top"*.
- **Cards are shorter than you build them.** Three separate notes: *"this chatbox card height is too much man"*, *"shorten this card height will you?"*, *"the agent card height is too big"*. Then the opposite, once, about a chart: *"increase chart card height why did you decrease it"* — so shrink the containers, not the content.
- **Agent text is narrower and tighter** than the frame allows: *"shorten width of the agent chat responses by a bit and keep its chat on the left side"*, *"the agent text line height should be reduced"*.
- **Two labels near each other is a bug.** *"v_07 v_08 labels are too close to each other"*, *"coupon expired and order stuck on hold errors are too close"*. Note: a speech-style block grows **downwards** from its `top`, so moving one down to separate it can make the overlap worse. Check which way each block grows before moving it.
- **Backgrounds stay plain.** *"dont like the checkered boxes and gradients in backgrounds"*, *"i dont like the blue gradient"*, *"that blue gradient ball in the end is redundant and ugly af"*.

## Theme is meaning, not decoration

Dark is the problem; white is the solution. *"because when the scene changes to white then we need to discuss the solution as vibe has changed"*. The film opens dark on everything that goes wrong, turns white when the agent is introduced, and returns to dark for the moment of approval: *"when the card returns switch to dark mode"*.

Pace follows the same split — *"we also need a few sections to slow down pace and some to be in dark mode"*, *"and increase pace since ask"*.

When a theme change coincides with a beat, the change **is** the beat: cut to it, do not cross-fade into it.
