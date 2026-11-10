# Motion

Everything here was arrived at by being told it was wrong. The notes are quoted so you can tell which rule a new situation resembles.

## The curve family

```ts
/** Tight through the middle, flat to order n-1 at both ends. n IS the peak velocity. */
export function tightInOut(x: number, n = 4): number {
  const k = Math.min(1, Math.max(0, x))
  return k < 0.5 ? 2 ** (n - 1) * k ** n : 1 - (-2 * k + 2) ** n / 2
}

/** The same curve with a dead end cut off and the rest stretched to fill. */
export function trim(f: (x: number) => number, a = 0, b = 1): (x: number) => number {
  const lo = f(a)
  const span = f(b) - lo
  return (x) => (f(a + (b - a) * Math.min(1, Math.max(0, x))) - lo) / span
}
```

Two other shapes earn their place:

```ts
/** Ninth-order smoothstep: zero in the first FOUR derivatives at both ends, 2.5x average
 *  through the middle. For crossing an empty frame — a tenth of the way in it is doing a
 *  ten-thousandth of its peak, so the first three frames move less than a pixel. */
const CROSS = (x: number) => 126*x**5 - 420*x**6 + 540*x**7 - 315*x**8 + 70*x**9

/** Quintic with p(0)=0, p(1)=1, p'(0)=a, p'(1)=0 and both second derivatives zero.
 *  `a` is the speed the previous move is still doing, in units of this move's average.
 *  At a=0 it is smootherstep. This is how a move inherits velocity instead of starting
 *  from rest. */
const carry = (a: number) => (x: number) =>
  a*x + (10 - 6*a)*x**3 + (8*a - 15)*x**4 + (6 - 3*a)*x**5
```

### Why not a cubic-bezier

You cannot state peak velocity with one. The entire argument they kept having with the motion — *"tight ease in out"*, *"tight on center and really smooth on ends"*, *"TOO LINEAR SMOOTH ON START THEN FAST THEN SMOOTH AT END"* — is an argument about peak velocity and end flatness. With `tightInOut` those are the two things you type.

### Choosing n, with the history

- **4 — a camera.** Ran on the product's `EASE.resize` first. That is an ease-out: leaves at full speed, spends the rest settling. Correct for a panel resizing under a hand, wrong for anything with mass. The crossing between two halves of an act read as a lurch that then eased in.
- **5 — a close/pinch.** Enough bite to read as an action.
- **7 — an open/push.** The tile opening, the dot growing into the frame.
- **9 (`CROSS`) — a crossing over an empty frame.** Three curves preceded it: `tightInOut(x, 6)` on the reasoning that the frame is empty mid-crossing so the speed may as well be spent where nothing can see it — right arithmetic, wrong conclusion, because ninety pixels a frame between two stillnesses reads as a snap however little is on screen to snap. Then smootherstep (fixed the peak, left the ends leaving at a visible clip). Then the seventh order, which was close. The middle costs almost nothing across all of these — 2.5 against 1.9 of average, four pixels a frame. **All the difference is at the two ends, which is where it was wanted.**

Their notes along that path: *"in ride moving between screen is too fast ease in out and very smooth at start and end middle doesnt have to be too tight"*, then *"nah moving between screen in ride is not smooth at all it should start and end very very slow"*.

## The dwell problem, fully

Symptom: a hold of 2–6 frames at a junction where two moves meet. They see it every time.

Cause: `tightInOut` is flat at **both** ends by construction. End-to-end, two flat ends are a stop. Nothing is mistuned — the curves are each doing what they promise.

Three fixes, in order of preference:

1. **One move instead of two.** If the second move's start is the first move's end, write the whole thing as one function of time. This is what `dotX` does: the outward split and the inward pinch are one expression, with the pinch multiplying down what the split produced.
2. **`trim` the ends that meet.** Trim the incoming move's tail and the outgoing move's head so the pair hands over mid-velocity. Verify: sample position every frame across the junction and assert no frame moves less than 0.4px while the gesture is nominally in motion.
3. **`carry(a)`** when the second move must start from the first move's *exact* speed and that speed is known. `a = v_handover * duration / distance`.

A worked case: the collapse-to-dot-to-split chain. `PART = trim(x => tightInOut(x, 7), 0.25)` — the split leaves at the pace the collapse had reached. `CLOSE = trim(x => tightInOut(x, 5), 0, 0.84)` — the pinch arrives moving and stops dead, which is right only because the zoom takes over in the opposite direction on that frame.

### Solve the handover point, don't guess it

The moment a tween "has arrived" is usually not its end. Bisect for it:

```ts
// the x where the eased value first exceeds 1 - 1e-3
let lo = 0, hi = 1
for (let i = 0; i < 40; i++) { const m = (lo + hi) / 2; if (OPEN(m) < 0.999) lo = m; else hi = m }
const ARRIVED = lo   // came out at 0.78824 for tightInOut(x, 7)
```
Then place the next event at `MERGED - DUR * ARRIVED` rather than at `MERGED - DUR`. Forty steps on a monotone function is exact to well under a frame.

## Simultaneity

*"zoom out and move up shouldnt stop till end they should run together"* → *"start together and end together"* → *"the end of zoom out and start of move up should be intertwined"*.

Build scale and travel as one move over one duration on one curve. Their peaks then land on the same frame by construction, which is the point: the frame opens out at exactly the speed the content is leaving through the top of it.

Rejected alternatives, both tried: giving the scale back first and then travelling (reads as two moves); leading the scale over the first 40% while the camera only drifts (*"the drift is a camera waiting, however alive the number keeps it"*).

## Speed laws

When the note is about *the rate of arrivals* rather than one move's shape, write `dv/dd` and integrate. Their notes: *"it should stop on the list first render first item then second slowly starts moving then becomes faster then stops again at the end item"*, *"we want to show the first three and four with increasing pace from item to item till we catch up a significant speed"*, *"the speed gain on the list should be smooth and so with speed decel. rn its accelerating too much"*, *"the speed gain still feels a bit sudden"*.

**Logistic in distance:**

```
dv/dd = (v/L) · (1 − v/V)
```

- While `v` is small this is `v = V0·e^(d/L)` — the only law where every row is a fixed fraction quicker than the one above it, and `L` *is* that fraction. That is the part the eye can read.
- As `v` approaches `V` the gain closes itself off, so the run **arrives** at top speed instead of hitting it.

**Why the obvious version fails.** A bare exponential clipped at `V` has a constant gain per row, which is a pole in time: `v = V0·L/(L − V0·t)`. It spends nearly all its duration slow and does its last doubling in the final few percent — twelve pixels a frame to forty-six in a quarter second, then it stops accelerating dead against the clip. Halving `L` smooths the rows and the fall still whips, because the shape in time is the same shape wherever you cut it off and the corner at the top is still a corner. Under the logistic, peak acceleration is `4V²/27L` against the clipped exponential's `V²/L` — a sixth — and it falls to zero at both ends of the rise.

**Pick V from the content, not from taste.** One row per frame is the speed at which a list stops being a list: every row falls on the same pixels as the one before it, so nothing is readable however long you look. Going faster buys nothing visible and costs a lot, because the acceleration needed to reach a speed goes as its square.

`t(d)` has a closed form; `d(t)` does not — bisect, same as everything else.

There is **no cruise phase** and there does not need to be one: the flat top of the rise is the cruise. The run is a rise, its mirror, and then it hands its velocity to the next leg. It does not brake. A constant brake from `V0` to standstill is under a pixel a frame — a camera that has visibly stopped — and then the next leg starts from rest, and the join reads as a hold between two moves. *"ok it should not stop at the end of the list"* / *"keep moving"*.

## Chain lag (the "pulled from above" stretch)

They asked for *"a natural stretch effect too of being pulled from above"*, then *"the stretch should be staggered"*, then diagnosed the implementation precisely: *"why is the pull seems from both bottom and top instead of just top???"*, *"the last line moves down implying it is not staggered and there is a pull from bottom too!"*, *"the bottommost line it moves down from its actual position when it should not cross its original baseline and be pulled from the top"*.

**The wrong model:** compute one strain across the column and scale every block by it. That is every row stretching simultaneously by a different amount, which reads as the list being squeezed, and rows below the anchor move *down*.

**The right model:** a delay by depth. A row at depth `d` below the frame's top edge renders where the camera had it `LAG_T · d / frameDepth` seconds ago.

```ts
const was = (d: number) => here.y - yAt(t - (LAG_T * Math.max(0, d)) / span)
const head = was(top - edge), foot = was(top + h - edge)
node.style.transform = `translateY(${head}px) scaleY(${1 + (foot - head) / h})`
```

Three bugs disappear for free, each of which had been patched by hand before:
- **Nothing ever moves backwards.** The offset is `yAt(t) − yAt(t−τ)` and `yAt` only increases, so each row's own motion is `yAt'(t−τ)`, never negative. A link moves up *less* than the one above it; it never moves down.
- **It is zero when the camera is still**, largest when quickest, and collapses on its own as the camera stops — a delay of a stationary signal is the signal. No release curve, no cap, no bell to place.
- **Strain works out to `LAG_T/frameDepth · v`**, so the column stretches in proportion to how hard it is being pulled, which is what a stretch is supposed to mean.

Because property one holds for *any* `LAG_T`, that number can be set by eye alone — which is why it is the one number here that was tuned rather than solved. 0.12 → 25px of trail, arithmetic can see it and the eye cannot. **0.32 → 60px, which reads.** 0.46 → 90px, the column coming apart rather than trailing.

Their calibration notes on intensity: *"increase intensity of stretch and pull so its noticeable"*, *"nice just increase the intensity of the pull a tad bit so i can notice it"*, then *"no no undo increase intensity earlier was fine"*.

One write per wrapper. Rows carry GSAP transforms from their own reveals, and two writers on one `transform` is a fight neither wins — and is the usual cause of the sub-pixel vibration they report.

## Motion blur

```ts
const step = Math.abs(camAt(t).y - camAt(t - 1/FPS).y)
const sd = Math.min(30, (step * 0.75) / Math.sqrt(12))
blurNode.setAttribute('stdDeviation', `0 ${sd.toFixed(2)}`)
```

- `0.75` is a 270° shutter. `sqrt(12)` is the Gaussian matching a box of that length.
- It had been divided by 2.2 — about three times a real exposure — so the column dissolved into soup at under one row a frame and the blur stopped agreeing with the apparent speed.
- **Per-axis.** CSS `blur()` is isotropic.
- **Filter everything in frame, generously.** A layer just outside the frame unfiltered next to one just inside it shows a visible seam. Reach = `frameHeight/scale + 620`.
- **Statements get it too.** The camera crosses an empty screen to reach a sentence at thirty pixels a frame; hard edges with no smear is exactly the condition for strobing, and no easing fixes it because it is not a problem with the curve.

## Arrivals driven by the camera

Nothing is preloaded. *"you need to draw stuff one by one on the ride not preloaded stuff"*, *"also list stuff as the movement on y happens not preloaded boring list"*, *"why are some subitems in end of orders are pre-rendered instead of staggering after each other"*.

A row shows at `max(sceneStart, timeAtY(v − LEAD), timeAtY(v) − NEAR)` — a lead in *time* (0.3s) capped by a lead in *distance* (280px). Standing still that puts the arrival mid-screen; at speed the cap takes over and it lands in the bottom third, where it has to be, because at 46px a frame a row appearing at centre would already be leaving.

`timeAtY` is a bisection on the camera function, which is why the camera must be a pure function of time and not a tween you can only sample once per frame.

Two consequences they found by watching:
- **The camera must pass every row**, or rows past its resting point all resolve to the same instant and arrive in a clump. Rest on the last row, not the middle of the last block. *"always keep the end item in center of screen whether main item or subitem."*
- **Guard the ordering.** Two rules place these rows and which one wins changes down the column. A running maximum makes out-of-order arrival unrepresentable. It has been wrong once.

Direction of arrival: **from below, never from the left.** *"instead of animating the list items and subitems from left animate them from bottom"*. Rows slide in from −20 x in the product; here that was the one thing moving across the frame while everything else moved up it.

## Camera rig

One function of time, driven per frame, not a tween — because its derivative is needed in the same frame for the blur, and a GSAP tween cannot be asked for its value at two times.

```ts
const tx = ax * FRAME.w - x * s
const ty = ay * FRAME.h - y * s
strip.style.transform = `translate3d(${tx}px, ${ty}px, 0) scale(${s})`
```

A scene takes the camera by pushing a `(t) => Framing | null` driver; returning null gives it back. Park the tweened camera on the landing frame just before the boundary so the next beat starts from where this one actually finished.

Half-frame guards at both ends: `t < t0 - EDGE || t >= t0 + dur - EDGE`, `EDGE = 1/120`.

**x stays constant for an entire act if you can manage it.** The column hung right of centre once and the camera swung out to it and back — a lateral move in the middle of a vertical one, which fights the fall for no gain. *"the list i dont want the camera to move left instead you can make the list in center with their sub connectors."*

## Spacing between stations

One screen of nothing between sections: `GAP = round(FRAME.h / SCALE)`. *"between each section on the rail there should be gap of 1 screen size"* — then *"nah 1 screen diff only"* when two were tried.

At the fitted gaps it had before (328, 220, 410, 500 — each derived from fitting one thing next to another) a section was always leaving while the next was arriving, so the fall read as one long page being scrolled rather than as a journey between places. **The empty screen is what makes each arrival an arrival.**

## Things deleted for being redundant

They remove aggressively, and are usually right:
- The rail — a 1px rule joining every station. *"you know what we dont need the rail line just camera movement is okay."* Also *"we dont need rail line in between list categories"*: a connector that joins seventeen things in a row joins nothing, it just draws a dashed line down the middle of the list.
- The line drawn around four circles, frames 284–384 — *"redundant"*.
- A heading over the capability column — a caption saying what the statement the camera just left already said.
- A zoom-out at the end — *"the zoom out in end is not needed"*.
- A blue gradient ball — *"redundant and ugly af"*.
- A blue beam in a connector line — *"dont want the blue beam in the line"*.
