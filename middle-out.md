# Middle-Out

<dl>
<div><dt>Problem</dt><dd>An analyser finds everything on a spine MRI and returns numbers. Nobody knew what those findings should look like on the image for a radiologist to actually work from them.</dd></div>
<div><dt>Role</dt><dd>Designed and wrote both the drawing library and the headless generator that runs it — interface, graphics, logic, algorithms, implementation.</dd></div>
<div><dt>Purpose</dt><dd>So a radiologist sees every finding on the slice it came from, beside the evidence for it, and can reject any of them.</dd></div>
<div><dt>Built with</dt><dd>TypeScript · Canvas 2D · React · a headless browser for the baked frames</dd></div>
<div><dt>Where it runs</dt><dd>Three products at Remedy Logic and the diagnostic viewers of their partners, read by radiologists every working day.</dd></div>
</dl>

## The idea

A line on a canvas is a first-week exercise. A dozen lines with numbers and panels on one spine slice is another job. The reader takes it in at a glance and may argue with any part of it.

Little of that is drawing. It is deciding where each thing goes. There are a couple of hundred pixels of dense grey and a set of rules that contradict one another. Which rule gives way is a question about how radiologists read. Drawing was the easy part.

The library was built at [Remedy Logic](https://remedylogic.com) by the author of this post. The rendering engine and the layout solver. The rules about what may be printed. The configuration every consumer reads, and the headless generator that runs all of it.

A spine MRI arrives as sagittal and axial series[^planes] and leaves as a report. Protrusion at one level. Stenosis at another. Canal diameter in millimetres.[^spine] Everyone downstream rebuilds the spine from those words, with no images in front of them. Findings get missed on the way, and that much is measured.

- **The abnormality is never seen.** The day-to-day error rate in radiology runs at three to five per cent. Sixty to eighty per cent of those errors are perceptual: the finding is on the image and nobody saw it.[^perception]
- **The finding is not what the study was for.** A lumbar spine MRI holds the whole abdominal aorta. In a cohort of 1922 patients over fifty-five, an aneurysm was present in 4.4 per cent and reached the report in 31 per cent of those.[^aorta]

A radiologist reads all shift in the dark with a queue waiting. The overlay exists to help with that.

![An axial slice at a lumbar disc: the abdominal aorta measured at 34.4 mm in red, and a panel beside it naming aortic aneurysm as present](https://illinifellow.com/i/004-GEkd644SgTx.webp)
*An aneurysm on a study ordered for the spine. The red is the analyser's verdict rather than the drawing's emphasis.*

<aside><h4>What the figures count</h4>Those numbers count what reached the report, rather than what a radiologist saw. Whether drawing a finding on the image makes it less likely to be missed has never been shown. What follows is a position rather than a result.</aside>

The analyser that finds those things is a colleague's work inside the company. Per slice it hands over contours and the measurements taken across them. Then the findings, with their severity. None of it is a picture. This work starts where the numbers arrive.

<aside><h4>Cleared</h4>The analyser is an FDA-cleared device: <a href="https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfPMN/pmn.cfm?ID=K241108">K241108</a>, decided 30 October 2024. The values on the frame are its values, printed exactly as they arrived. Hence a rule that otherwise reads as pedantry: never trim a number to fit the space.</aside>

## What a radiology co-pilot is

The product reads alongside the radiologist. It says what it saw and where, including the finding the reader has not reached yet. The radiologist still decides. One available decision is that the finding is wrong.

The analyser's own 510(k)[^clearance] summary puts the checking of its outputs on the user.[^summary] Almost every rule below comes out of that one sentence.

> An assistant you cannot disagree with is one you must either trust blindly or switch off.

Three things follow. The evidence for each finding stays on the image. Every finding handed over gets drawn. Labels stay off the anatomy they describe.

So the reader is never simply told there is something at a level. They are looking at it. The measurement is drawn across the thing measured, in a colour that says how bad it is. Claim and evidence in one glance.

That leaves the layout one rule it cannot argue with. Every value goes beside the thing it measures, however crowded the slice.

> A number trimmed to fit is a different number.

## What is drawn, and what for

**Outlines** say which structure the analyser believes it is looking at. Contours rather than filled masks. The claim is about the anatomy underneath, and a filled shape covers up the evidence for it. **Linear measurements** are drawn across the thing they measure. The line shows the value was taken in the right place. **Areas** carry the same duty. **Angles** describe alignment. That is the one quantity the eye cannot guess.

<figure data-abreast>
<figure><img src="https://illinifellow.com/i/005-rUnEOlPetzL.png" alt="a sagittal cervical series with the analyser's contours drawn on it, the topmost ones sitting off the bone"><figcaption>Nothing here was nudged to look right, and a filled shape would have hidden the miss politely.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/006-Rd7ciBqRRNH.webp" alt="a sagittal lumbar slice with the lordotic angle drawn along two vertebral endplates and its value printed at the vertex"><figcaption>Both rays run off the frame. The vertex is the only place left for the value.</figcaption></figure>
</figure>

**Colour** encodes severity. Colour is the loudest channel on the frame. The eye sorts it before attention arrives. It can be spent once, and it is spent on the grade. **The findings panel** says in words what was found at this level. Always in the same corner. **The navigation strip** shows where in the stack the slice sits. Losing it is how people scroll past the level they wanted.

![An axial slice with the aorta measured in green and the thecal sac in red, both values repeated in a panel at the left](https://illinifellow.com/i/007-Y4jae8a9u_u.jpg)
*Two colours are two severities. The value on the anatomy and the value in the panel come from one row of the grade table. They cannot disagree.*

<aside><h4>Two colours in one sentence</h4>A stenosis can be severe on the right and mild on the left. The panel prints both words on one line, each in its own colour. A canvas measures a string and paints it in one colour. So the library carries a typesetter of its own: a markup parser, a line breaker working in measured widths, and a painter that lays the runs back out across the plate.</aside>

![A panel over an axial slice listing five findings for the L5/S1 level, three of them carrying a severity word](https://illinifellow.com/i/008-U7Y2mCemtF2.webp)
*One level's findings, worded by the configuration. "Bilateral" is a sentence of its own, because it names neither side. Two of the five carry no grade. For those, only the fact is reported.*

Every layer switches on, off, recolours or reshapes from one configuration. No code. Of the 329 visibility switches in the base, 263 are off. The base is the vocabulary. The consumer writes the sentence.

<aside><h4>Why the file repeats itself</h4>The configuration is 5854 lines. A third of it is repetition typed out by hand. Compressing it with references broke the build for every consumer. Twice. A YAML parser refuses a reference graph past a built-in limit. Raising that limit locally only moves the problem into other people's builds. The repetition stayed. It is still typed out by hand.</aside>

<figure data-abreast>
<figure><img src="https://illinifellow.com/i/009-YXFw_-DR5oZ.webp" alt="a sagittal lumbar slice with every vertebra and disc outlined, the level names on the bodies and six measurements led out to labels"><figcaption>Everything the analyser returned, switched on at once: the busiest a frame is allowed to get.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/010-5FSE-v8YlkV.webp" alt="the same slice with the vertebra and disc outlines switched off, the level names and five measurements still drawn"><figcaption>One switch in the configuration. The layout is solved again from scratch in the room that leaves.</figcaption></figure>
</figure>

The overlay draws less than it is handed, on purpose. A measurement is dropped when the drawn line would misrepresent the anatomy under it. An axial slice cut at a strong tilt, for one. A line a reader cannot check is worse than no line. Over thirty-eight studies, drawing everything on offer would add 8.9 per cent more measurements and 3.8 per cent more text.

<aside><h4>What is left off</h4>On a slice cut at a strong tilt the distance across the slice is not the distance across the structure. How much tilt is too much differs between discs and vertebral bodies, and both thresholds sit in the configuration. The two percentages come from one pass over thirty-eight studies, counting objects rather than pixels.</aside>

> The picture is not an illustration of the result. It is how the result is read.

## How it reaches the reader

A headless generator[^headless] renders the frames ahead of time in a private cluster. They are packed into a series beside the original study.[^dicom] Most radiologists meet the overlay that way. The second path draws the same overlay live, on a viewport the reader pans and zooms.

<aside><h4>A second series, never the original</h4>The enriched frames are always an addition. The study keeps the series it arrived with, so the pixels the drawing talks about are still there to be measured. The new series says what it is in its own description. A series a radiologist could mistake for the original would be a defect.</aside>

<figure data-index="01" data-label="FROM THE SCANNER TO THE READER">
<div data-diagram="chain"><div data-track><div data-step><span data-dot></span><b data-name data-caps>Study</b><span data-detail data-caps>Slices</span></div><div data-step><span data-dot></span><b data-name data-caps>Analysis</b><span data-detail data-caps>Findings</span></div><div data-step data-accent><span data-dot></span><b data-name data-caps>The library</b><span data-detail data-caps>Layout and drawing</span></div><div data-fork><div data-step><span data-dot></span><b data-name>Baked into a series</b><span data-detail>Beside the original</span></div><div data-step><span data-dot></span><b data-name>Drawn on a viewport</b><span data-detail>Or on any other canvas</span></div></div></div></div>
</figure>

<figure>
<figure><img src="https://illinifellow.com/i/011-nQZPqZcQx_u.jpg" alt="the overlay shown inside the eUnity diagnostic viewer, the vendor's own interface around it"><figcaption>Read in eUnity by radiologists who never learned the overlay had a name.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/012-ETqfW2PRdVR.jpg" alt="a sagittal lumbar slice in eUnity with a measurement drawn on it in red and a measurements panel beside it"><figcaption>The same viewer, a measurement and its panel. Both are pixels of the frame.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/013--kFi56xV0f7.jpg" alt="an axial lumbar slice inside Fujifilm Synapse with a panel of findings drawn on the frame"><figcaption>Fujifilm Synapse, another maker’s reading station. Nothing there knows what a finding is.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/014-kN_ovF1agaH.jpg" alt="the series list of a study in Fujifilm Synapse, the enriched series standing among the ones the scanner produced"><figcaption>The enriched series stands in the study’s own list, beside the ones the scanner sent.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/015-Kh4XGxaUt2H.jpg" alt="the enriched series and the original side by side inside the CARPL platform"><figcaption><a href="https://carpl.ai">CARPL</a> carries many vendors' models. Nothing was written on either side to make this one fit.</figcaption></figure>
</figure>

<aside><h4>Why it is a picture at all</h4>The imaging standard's annotation objects each refuse half the job. A one-bit overlay plane is a single-colour stencil. A mask can say only filled regions. A vector presentation format is displayed by almost no viewer in the field. A burnt-in frame is the one form every reader already opens. The price is that its pixels can no longer be measured.</aside>

## Why this had to be built

A finding is worth something on the slice it came from, beside the evidence. Move the number into a table and the link goes with it. The reader rebuilds it from a level name, against a picture they are no longer looking at.

Most products in spine imaging keep the numbers off the anatomy. A table, or an empty margin where placing them costs nothing.

<aside><h4>One quantity down the levels</h4>Several CT products measure vertebral body height and nothing else: Siemens AI-Rad Companion MSK, Nanox.AI HealthOST, Avicenna CINA-VCF Quantix. The values go in a column beside the spine, each row level with its own vertebra, and no leader is needed. Stacked levels and an empty margin are not a placement problem.</aside>

[CoLumbo](https://columbo.me), from Smart Soft Healthcare, is the one product that does the hard thing. It puts several kinds of measurement onto the anatomy of a lumbar MRI slice, each value on the caliper that took it. Its own published frames are the best evidence of what the approach runs into.[^frames]

Three things on those frames. An area stands on the lower border of the sac it measures. A value inside the sac is crossed by two nerve-root contours. A cross-reference line runs through a level name.[^columbo]

![Two panels from CoLumbo: a sagittal lumbar slice with level names in red and a value against its caliper, and an axial slice where the area of the dural sac is printed on the sac's own outline](https://illinifellow.com/i/021-L4ai2qM2hKF.webp)
*Chosen by the people who built it, to show the feature working. Screenshot: CoLumbo, Smart Soft Healthcare — [columbo.me/help/dural-sac-area/](https://columbo.me/help/dural-sac-area/)*

![An axial slice with a disc outlined in green, a measurement printed across the outline, and a value inside a cyan region obscured by two magenta contours](https://illinifellow.com/i/019-6bI2GGV3Hft.webp)
*Screenshot: CoLumbo, Smart Soft Healthcare — [columbo.me/help/disk-herniation-size/](https://columbo.me/help/disk-herniation-size/)*

<aside><h4>The same product, five years earlier</h4>Its <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8158737/">2021 validation paper</a> shows the anatomy filled with opaque colour and not one value on the frame. Every measurement sat as text under the picture. Today the values are on the anatomy. Moving them there brings the crowding with it.</aside>

One rule accounts for all three: the value goes where the measurement ended. That is a drawing rule rather than a placement system. It has no answer for two things that want the same pixels.

Every one of those three is impossible here, by design. Everything already on the frame declares the room it occupies before a label is placed. Anatomy, panels, level names, other labels. An arrangement that puts a plate on a shape is refused before it is scored, so it cannot win by being better elsewhere. What is left to trade is leader length, and a leader is the one thing a reader can follow back.

Which is why the library was written as the one thing every consumer draws from, rather than as the labelling inside a single viewer.

<figure data-index="02" data-label="THREE WAYS TO PUT A NUMBER ON A SCAN" data-note="The same six lumbar measurements in millimetres, arranged three ways. A comparison of strategies, and no panel stands for a particular product.">
<div data-diagram="panels"><div data-trio><div data-panel><b data-name>On the anatomy</b><div data-art><svg viewBox="-60 0 320 260" role="img" aria-label="six measurements printed on the anatomy, two of them standing on what they measure and two pairs printed over each other" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.55" stroke-width="2.6" stroke-dasharray="0 5.18" fill="none" stroke-linecap="round" d="M99.44 70L114.32 70.62L127.34 73.71L137.26 78.67L144.08 86.11L146.56 94.79L144.08 102.23L138.5 106.58L132.3 109.06L129.82 114.02L130.44 118.98L140.36 120.84L151.52 121.46L161.44 119.59L168.88 115.88L165.16 123.94L155.24 128.28L144.08 130.14L137.88 135.71L130.44 143.16L123 151.83L115.56 160.51L109.98 169.82L106.26 179.11L103.16 186.56L99.44 190.28L95.72 186.56L92.62 179.11L88.9 169.82L83.32 160.51L75.88 151.83L68.44 143.16L61 135.71L54.8 130.14L43.64 128.28L33.72 123.94L30 115.88L37.44 119.59L47.36 121.46L58.52 120.84L68.44 118.98L69.06 114.02L66.58 109.06L60.38 106.58L54.8 102.23L52.32 94.79L54.8 86.11L61.62 78.67L71.54 73.71L84.56 70.62L99.44 70M99.44 107.82L108.74 110.3L114.94 116.5L116.8 124.56L112.46 131.99L105.64 136.34L99.44 138.19L93.24 136.34L86.42 131.99L82.08 124.56L83.94 116.5L90.14 110.3L99.44 107.82"/><path stroke="currentColor" opacity="0.8" stroke-width="5" fill="none" stroke-linecap="round" d="M127.96 87.36v.01M71.54 89.83v.01M106.88 110.91v.01M79.6 130.76v.01M121.14 131.99v.01M92 161.76v.01"/><text x="110" y="100" font-size="18" fill="currentColor" opacity="0.7">8.6</text><text x="58" y="104" font-size="18" fill="currentColor" opacity="0.7">12.4</text><text x="126" y="126" font-size="18" fill="currentColor" data-accent>7.1</text><text x="136" y="136" font-size="18" fill="currentColor" data-accent>4.2</text><text x="48" y="150" font-size="18" fill="currentColor" data-accent>5.3</text><text x="58" y="160" font-size="18" fill="currentColor" data-accent>3.8</text></svg></div></div><div data-panel><b data-name>In a table beside it</b><div data-art><svg viewBox="0 0 320 260" role="img" aria-label="the anatomy left bare, every measurement moved into a table at the side with nothing joining the two" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.75" stroke-width="2.6" stroke-dasharray="0 5.18" fill="none" stroke-linecap="round" d="M99.44 70L114.32 70.62L127.34 73.71L137.26 78.67L144.08 86.11L146.56 94.79L144.08 102.23L138.5 106.58L132.3 109.06L129.82 114.02L130.44 118.98L140.36 120.84L151.52 121.46L161.44 119.59L168.88 115.88L165.16 123.94L155.24 128.28L144.08 130.14L137.88 135.71L130.44 143.16L123 151.83L115.56 160.51L109.98 169.82L106.26 179.11L103.16 186.56L99.44 190.28L95.72 186.56L92.62 179.11L88.9 169.82L83.32 160.51L75.88 151.83L68.44 143.16L61 135.71L54.8 130.14L43.64 128.28L33.72 123.94L30 115.88L37.44 119.59L47.36 121.46L58.52 120.84L68.44 118.98L69.06 114.02L66.58 109.06L60.38 106.58L54.8 102.23L52.32 94.79L54.8 86.11L61.62 78.67L71.54 73.71L84.56 70.62L99.44 70M99.44 107.82L108.74 110.3L114.94 116.5L116.8 124.56L112.46 131.99L105.64 136.34L99.44 138.19L93.24 136.34L86.42 131.99L82.08 124.56L83.94 116.5L90.14 110.3L99.44 107.82"/><path stroke="currentColor" opacity="0.4" stroke-width="5" fill="none" stroke-linecap="round" d="M127.96 87.36v.01M71.54 89.83v.01M106.88 110.91v.01M79.6 130.76v.01M121.14 131.99v.01M92 161.76v.01"/><path stroke="currentColor" opacity="0.3" stroke-width="2" stroke-dasharray="0 5.22" fill="none" stroke-linecap="round" d="M186 56H308M186 208H308M186 56V208M308 56V208"/><path stroke="currentColor" opacity="0.2" stroke-width="1.8" stroke-dasharray="0 5.1" fill="none" stroke-linecap="round" d="M198 82L300 82M198 106L300 106M198 130L300 130M198 154L300 154M198 178L300 178"/><path stroke="currentColor" opacity="0.4" stroke-width="3.4" fill="none" stroke-linecap="round" d="M200 68v.01M200 92v.01M200 116v.01M200 140v.01M200 164v.01M200 188v.01"/><text x="292" y="74" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">8.6</text><text x="292" y="98" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">12.4</text><text x="292" y="122" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">7.1</text><text x="292" y="146" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">4.2</text><text x="292" y="170" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">5.3</text><text x="292" y="194" font-size="18" fill="currentColor" opacity="0.7" text-anchor="end">3.8</text></svg></div></div><div data-panel><b data-name>In a column on leaders</b><div data-art><svg viewBox="-24 0 320 260" role="img" aria-label="the same six measurements gathered into a column beside the anatomy, each on its own leader, none of them crossing" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.75" stroke-width="2.6" stroke-dasharray="0 5.18" fill="none" stroke-linecap="round" d="M99.44 70L114.32 70.62L127.34 73.71L137.26 78.67L144.08 86.11L146.56 94.79L144.08 102.23L138.5 106.58L132.3 109.06L129.82 114.02L130.44 118.98L140.36 120.84L151.52 121.46L161.44 119.59L168.88 115.88L165.16 123.94L155.24 128.28L144.08 130.14L137.88 135.71L130.44 143.16L123 151.83L115.56 160.51L109.98 169.82L106.26 179.11L103.16 186.56L99.44 190.28L95.72 186.56L92.62 179.11L88.9 169.82L83.32 160.51L75.88 151.83L68.44 143.16L61 135.71L54.8 130.14L43.64 128.28L33.72 123.94L30 115.88L37.44 119.59L47.36 121.46L58.52 120.84L68.44 118.98L69.06 114.02L66.58 109.06L60.38 106.58L54.8 102.23L52.32 94.79L54.8 86.11L61.62 78.67L71.54 73.71L84.56 70.62L99.44 70M99.44 107.82L108.74 110.3L114.94 116.5L116.8 124.56L112.46 131.99L105.64 136.34L99.44 138.19L93.24 136.34L86.42 131.99L82.08 124.56L83.94 116.5L90.14 110.3L99.44 107.82"/><path stroke="currentColor" opacity="0.38" stroke-width="2.05" stroke-dasharray="0 6.04" fill="none" stroke-linecap="round" d="M127.96 87.36L200 56M71.54 89.83L200 92M106.88 110.91L200 128M121.14 131.99L200 164M79.6 130.76L200 200M92 161.76L200 236"/><path stroke="currentColor" opacity="0.2" stroke-width="2.05" stroke-dasharray="0 6" fill="none" stroke-linecap="round" d="M200 56L200 236"/><path stroke="currentColor" opacity="0.8" stroke-width="5" fill="none" stroke-linecap="round" d="M127.96 87.36v.01M71.54 89.83v.01M106.88 110.91v.01M79.6 130.76v.01M121.14 131.99v.01M92 161.76v.01"/><path stroke="currentColor" opacity="0.45" stroke-width="3.4" fill="none" stroke-linecap="round" d="M200 56v.01M200 92v.01M200 128v.01M200 164v.01M200 200v.01M200 236v.01"/><text x="210" y="62" font-size="18" fill="currentColor" opacity="0.7">8.6</text><text x="210" y="98" font-size="18" fill="currentColor" opacity="0.7">12.4</text><text x="210" y="134" font-size="18" fill="currentColor" opacity="0.7">7.1</text><text x="210" y="170" font-size="18" fill="currentColor" opacity="0.7">4.2</text><text x="210" y="206" font-size="18" fill="currentColor" opacity="0.7">5.3</text><text x="210" y="242" font-size="18" fill="currentColor" opacity="0.7">3.8</text></svg></div></div></div></div>
</figure>

## Four rules that cannot all hold

Take an axial lumbar slice. That is the plane where the disc, the canal and the nerve roots are seen together. A couple of hundred pixels of dense grey, with a dozen calipers inside them. Each wants a legible label near its own point. Four rules say where it may go:

- No two labels may overlap.
- No label may sit on the anatomy it describes.
- No leader may cross another.
- Every label has to be drawn.

Any one of them is easy. All four together are impossible. That is the whole job.

<aside><h4>Three places the rules come from</h4>Whether an arrangement exists at all is geometry. Which rule is the hard one comes from how a radiologist reads. What may be shown comes from what the imaging formats carry. No one of the three can check the other two.</aside>

Contours are the easy half. A contour sits where the image put it. Two of them never argue about which one moves. A label has no place of its own. Its position is chosen, and every choice takes pixels from something else. Ten labels on leaders is a packing problem. On a dense slice it can have no solution at all.

<aside><h4>Markup was tried first</h4>Panels, notices and strip began as ordinary page elements over the picture. That works only in a browser under your own control. A headless render and another vendor's viewer both leave the markup with nowhere to live. The browser had been wrapping lines, measuring them and drawing plates behind them for free. All of it had to be written from nothing. A browser does a great deal for free. The bill arrives all at once.</aside>

![An axial lumbar slice with the disc and the canal outlined, nine measurements led out to one column of labels on the right, and a dashed rectangle drawn round every object on the frame](https://illinifellow.com/i/020-g7T4yoEelUP.webp)
*The case in question, solved, with the layout guides switched on. Nine measurements of seven different quantities. The dashed rectangles are the room each object claimed before anything was placed.*

## Middle-out

The first version did not search. It offered a label forty-two positions in a fixed order and took the first that fitted. If all forty-two failed, the label was not drawn at all. Forty-two positions felt like thoroughness at the time.

> When nothing fitted, the code drew nothing. That is the one outcome the rules exist to forbid.

What replaced it sent the labels out from the centre like a fan. Each one travelled away from its own measurement until it found clear space.

Middle-out.

![Three men moving their hands in sync](https://illinifellow.com/i/023--tHLxx0Fnvq.gif)

It looks orderly. That is what makes it deceptive. A label buys clear space with leader length. A longer leader crosses more neighbours. Sending every label as far out as the frame allows maximises the very quantity that needed minimising.

The idea was elegant. The pictures were a mess.

<figure>
<figure><img src="https://illinifellow.com/i/024-VaFGZQYjs7q.png" alt="a sagittal lumbar slice with four measurement labels sent outwards at angles, two of them overlapping at the bottom"><figcaption>The fan, working exactly as designed.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/025-I1MIZj3XLi0.jpg" alt="six labels all reading 4.9 mm thrown out from the middle of a sagittal slice in every direction, each on its own leader"><figcaption>Six labels and one value. Nothing in the algorithm preferred any direction over another.</figcaption></figure>
</figure>

Letting a crowded label leave its column and find its own room came next. Every version made a better frame and a worse stack. The same measurement landed somewhere new on each slice, so the reader had to hunt it down again at every one. That mechanism was deleted too. A predictable crowd beats an unpredictable position.

<aside><h4>Panels that used to move</h4>Everything on the frame used to negotiate for space. Five attempts at making the findings panels behave all ended the same way: a panel jittering when the slice moved by a single pixel. Deleting the mechanism beat tuning it a sixth time. Only labels and their leaders move now. The shoving went with it.</aside>

<figure data-index="03" data-label="WHERE THE WORDS WENT" data-note="Crossings found by segment intersection and marked. The column order is the one of all 720 that crosses nowhere.">
<style>@media (prefers-reduced-motion: no-preference) {[data-figure="middleOut"] [data-plot]:hover [data-flow],[data-figure="middleOut"] [data-plot]:active [data-flow] {animation: middleOutCrawl 1.1s linear infinite}}@keyframes middleOutCrawl {from {stroke-dashoffset: 0} to {stroke-dashoffset: -6}}</style>
<div data-diagram="panels" data-figure="middleOut"><div data-pair><div data-panel><b data-name>Out from the middle</b><div data-plot><div data-art><svg viewBox="0 0 480 300" role="img" aria-label="six findings sent out from the centre of the object, three pairs of leaders crossing" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.8" stroke-linecap="round" fill="none" stroke-width="3.1" stroke-dasharray="0 6.2" d="M240 74.83L258.6 75.6L274.88 79.47L287.27 85.67L295.8 94.97L298.9 105.82L295.8 115.12L288.82 120.55L281.07 123.65L277.98 129.85L278.75 136.05L291.15 138.38L305.1 139.15L317.5 136.82L326.8 132.18L322.15 142.25L309.75 147.68L295.8 150L288.05 156.97L278.75 166.28L269.45 177.12L260.15 187.97L253.18 199.6L248.53 211.22L244.65 220.53L240 225.18L235.35 220.53L231.47 211.22L226.82 199.6L219.85 187.97L210.55 177.12L201.25 166.28L191.95 156.97L184.2 150L170.25 147.68L157.85 142.25L153.2 132.18L162.5 136.82L174.9 139.15L188.85 138.38L201.25 136.05L202.03 129.85L198.93 123.65L191.18 120.55L184.2 115.12L181.1 105.82L184.2 94.97L192.72 85.67L205.12 79.47L221.4 75.6L240 74.83M240 122.1L251.62 125.2L259.38 132.95L261.7 143.03L256.27 152.32L247.75 157.75L240 160.07L232.25 157.75L223.72 152.32L218.3 143.03L220.62 132.95L228.38 125.2L240 122.1"/><path stroke="currentColor" opacity="0.38" stroke-linecap="round" fill="none" stroke-width="2.05" stroke-dasharray="0 6" data-flow d="M205.12 99.62L144 262M275.65 96.53L336 266M249.3 125.97L144 38M215.2 150.78L144 150M267.12 152.32L336 30M230.7 189.53L336 146"/><path stroke="currentColor" opacity="0.2" stroke-linecap="round" fill="none" stroke-width="2.05" stroke-dasharray="0 6" d="M144 38L144 262M336 30L336 266"/><path stroke="currentColor" opacity="0.85" stroke-linecap="round" fill="none" stroke-width="6" d="M205.12 99.62v.01M275.65 96.53v.01M249.3 125.97v.01M215.2 150.78v.01M267.12 152.32v.01M230.7 189.53v.01"/><path stroke="currentColor" opacity="0.45" stroke-linecap="round" fill="none" stroke-width="3.6" d="M144 262v.01M144 38v.01M144 150v.01M336 266v.01M336 30v.01M336 146v.01"/><path style="stroke:var(--accentColor)" stroke-linecap="round" fill="none" stroke-width="8.4" d="M185.99 150.46v.01M284.52 121.43v.01M298.75 161.4v.01"/></svg></div><span data-pin data-side="left" data-caps style="--y:12.67%">8.0 MM</span><span data-pin data-side="left" data-caps style="--y:50%">7.4 MM</span><span data-pin data-side="left" data-caps style="--y:87.33%">6.1 MM</span><span data-pin data-side="right" data-caps style="--y:10%">5.2 MM</span><span data-pin data-side="right" data-caps style="--y:48.67%">3.7 MM</span><span data-pin data-side="right" data-caps style="--y:88.67%">4.9 MM</span></div></div><div data-panel><b data-name>Gathered in a column</b><div data-plot><div data-art><svg viewBox="0 0 480 300" role="img" aria-label="the same six findings gathered into one column at the edge, no pair of leaders crossing" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.8" stroke-linecap="round" fill="none" stroke-width="3.1" stroke-dasharray="0 6.2" d="M240 74.83L258.6 75.6L274.88 79.47L287.27 85.67L295.8 94.97L298.9 105.82L295.8 115.12L288.82 120.55L281.07 123.65L277.98 129.85L278.75 136.05L291.15 138.38L305.1 139.15L317.5 136.82L326.8 132.18L322.15 142.25L309.75 147.68L295.8 150L288.05 156.97L278.75 166.28L269.45 177.12L260.15 187.97L253.18 199.6L248.53 211.22L244.65 220.53L240 225.18L235.35 220.53L231.47 211.22L226.82 199.6L219.85 187.97L210.55 177.12L201.25 166.28L191.95 156.97L184.2 150L170.25 147.68L157.85 142.25L153.2 132.18L162.5 136.82L174.9 139.15L188.85 138.38L201.25 136.05L202.03 129.85L198.93 123.65L191.18 120.55L184.2 115.12L181.1 105.82L184.2 94.97L192.72 85.67L205.12 79.47L221.4 75.6L240 74.83M240 122.1L251.62 125.2L259.38 132.95L261.7 143.03L256.27 152.32L247.75 157.75L240 160.07L232.25 157.75L223.72 152.32L218.3 143.03L220.62 132.95L228.38 125.2L240 122.1"/><path stroke="currentColor" opacity="0.38" stroke-linecap="round" fill="none" stroke-width="2.05" stroke-dasharray="0 6" data-flow d="M205.12 99.62L336 30M275.65 96.53L336 78M249.3 125.97L336 126M215.2 150.78L336 222M267.12 152.32L336 174M230.7 189.53L336 270"/><path stroke="currentColor" opacity="0.2" stroke-linecap="round" fill="none" stroke-width="2.05" stroke-dasharray="0 6" d="M336 30L336 270"/><path stroke="currentColor" opacity="0.85" stroke-linecap="round" fill="none" stroke-width="6" d="M205.12 99.62v.01M275.65 96.53v.01M249.3 125.97v.01M215.2 150.78v.01M267.12 152.32v.01M230.7 189.53v.01"/><path stroke="currentColor" opacity="0.45" stroke-linecap="round" fill="none" stroke-width="3.6" d="M336 30v.01M336 78v.01M336 126v.01M336 222v.01M336 174v.01M336 270v.01"/></svg></div><span data-pin data-side="right" data-caps style="--y:10%">6.1 MM</span><span data-pin data-side="right" data-caps style="--y:26%">4.9 MM</span><span data-pin data-side="right" data-caps style="--y:42%">8.0 MM</span><span data-pin data-side="right" data-caps style="--y:58%">5.2 MM</span><span data-pin data-side="right" data-caps style="--y:74%">7.4 MM</span><span data-pin data-side="right" data-caps style="--y:90%">3.7 MM</span></div></div></div></div>
</figure>

## How a crowded picture is read

The eye fixates. It takes in a small sharp region. Then it jumps, and throws away everything between. A label beside its measurement sits inside the same fixation and costs nothing. A label at the far end of the frame costs a jump out and a jump back. The value has to be held across both. For one number.

The clinical question settles how labels group. A reader wants the same quantity at adjacent levels: canal diameter down the lumbar spine, disc height above and below. Six numbers clinging each to its own point are six searches. The same six in a column are one region and one movement of the eye.

Occlusion is the one rule with no acceptable amount. A label over the anatomy it describes destroys the evidence for its own claim: the number is standing on the thing the reader would check it against. Elsewhere that is cosmetic. Here it is a finding nobody can verify.

![Two labels reading 2.2 mm crowded against each other at the top edge of the frame](https://illinifellow.com/i/026-ssqGWYly9NS.webp)
*Two measurements of the same size want the same point. The loser ends up outside the frame.*

## Searching instead of trying

The forty-two tries gave way to a search. No formula gives a position. The constraints contradict each other, so any closed form that satisfies one breaks another. What exists is a space of arrangements, most of them bad. The work is searching it against two tiers and a fixed order of sacrifice:

```yaml
ranking: # how a whole arrangement is judged, top down
  refuse: [leaders_crossed, plate_on_shape]
  concede: [leader_over_reading, leader_over_plate]
  settle: [longest_haul, haul_evenness, haul_sum]
budget:
  reseat_rounds: 9 # how often one plate may be reseated
  step_rungs: 5 # how far along the ladder it may be pushed
  exhausted: return_best_seen # never empty, never an error
```

The order inside the settling terms runs against instinct. Score the spread first, and "push everyone equally far" beats "leave almost everyone alone". A reader tolerates a crowd of short leaders and refuses one number exiled across the frame. So the longest single haul is scored first.

<aside><h4>How long the search may run</h4>Nine reseats for one plate, five rungs along the ladder, and then the best arrangement seen so far is returned. On a lumbar slice there is often no room for a clean answer, so what gets given up is settled before the search starts rather than argued at each site.</aside>

The first search took the first arrangement that satisfied the hard rules. A reasonable thing to do. It produced visibly worse pictures than it had to.

> An arrangement that breaks no rule can still be the wrong picture.

It is scored as a whole composition now, and the search returns the least damaged arrangement. The first scoring counted violations and took any move that lowered the count. Repairs have a price ceiling now.[^haul]

## Order, not room

Labels are gathered into columns, so every leader in a column arrives on the same vertical. For any two of them, crossing comes down to the order they sit in. It is a property of the assignment. Two labels can have all the space in the world and still cross.

<figure data-index="04" data-label="WHY A PAIR OF LEADERS CROSSES" data-note="The same three anchors and the same three slots. Only the assignment differs.">
<div data-diagram="panels"><div data-pair><div data-panel><b data-name>Order inverted</b><div data-art><svg viewBox="36 58 320 244" role="img" aria-label="three leaders in the wrong order, two of them crossing" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.36" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.478" d="M98 180L256 78M46 78L256 180M62 274L256 282"/><path stroke="currentColor" opacity="0.3" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.4" d="M262 64H350M262 92H350M262 166H350M262 194H350M262 268H350M262 296H350"/><path stroke="currentColor" opacity="0.3" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.667" d="M262 64V92M350 64V92M262 166V194M350 166V194M262 268V296M350 268V296"/><path stroke="currentColor" opacity="0.55" stroke-linecap="round" fill="none" stroke-width="4" d="M46 78v.01M98 180v.01M62 274v.01"/><path style="stroke:var(--accentColor)" stroke-linecap="round" fill="none" stroke-width="5.5" d="M165.8 136.2v.01"/></svg></div></div><div data-panel><b data-name>Order restored</b><div data-art><svg viewBox="36 58 320 244" role="img" aria-label="the same three leaders in the right order, none crossing" xmlns="http://www.w3.org/2000/svg"><path stroke="currentColor" opacity="0.52" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.468" d="M46 78H256M98 180H256M62 274L256 282"/><path stroke="currentColor" opacity="0.3" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.4" d="M262 64H350M262 92H350M262 166H350M262 194H350M262 268H350M262 296H350"/><path stroke="currentColor" opacity="0.3" stroke-linecap="round" fill="none" stroke-width="1.9" stroke-dasharray="0 4.667" d="M262 64V92M350 64V92M262 166V194M350 166V194M262 268V296M350 268V296"/><path style="stroke:var(--accentColor)" stroke-linecap="round" fill="none" stroke-width="4" d="M46 78v.01M98 180v.01M62 274v.01"/></svg></div></div></div></div>
</figure>

The repair is small. Take a pair and swap their slots. Keep the swap only if the total number of crossings went strictly down. A column already clean comes out identical, which is what made it safe to ship.

<aside><h4>One order in seven hundred and twenty</h4>Six labels can sit in a column seven hundred and twenty ways. On a crowded slice one of those orders may be the only one that crosses nowhere. Swapping pairs finds it without ever listing them, and stops the moment no swap lowers the count.</aside>

Nobody counted crossings before that. A comment in the code asserted that sorting labels by the height of their points made crossings impossible, and on the strength of that one sentence they never were.[^sorted]

![A sagittal lumbar slice with five canal measurements led out to five labels standing in one vertical column](https://illinifellow.com/i/028-2ibFi5AxOAP.webp)
*One movement of the eye down the column, in place of five searches across the frame.*

Order settles which slot a label takes. Where the column stands is answered by the anatomy rather than by the window:

```yaml
ladder:
  lanes_per_side: 4 # verticals available on each side
  stands_off: anatomy # pinned to the shapes, not to the frame
  gap_from_anatomy: 24 # clearance from the outermost shape, px
  lane_width_percent: 130 # of the widest plate in it, margin included
crowding:
  never: compress_gaps # the space between plates is never taken
  outer_lane: folds_inward # it collapses, its plates move in
  inner_lane: pulled_off_wall # the last one steps back from the edge
```

<aside><h4>What the window may decide</h4>The columns used to be clipped to the visible area. A wider panel pushed them outward, and the same slice read differently in two products. The window decides whether a label fits. Where the ladder stands is the anatomy's business.</aside>

## Away from the line

When a label does not fit where it wants to be, it moves away from the measurement line. Away, and only away. The leader then leaves at a sensible angle instead of doubling back over its own measurement. A moved label pushes its neighbour, so the rule has to hold at every step of the chain.

The version that read best on paper spread a column symmetrically about its anchor. It moved the label nearest the measurement towards the line, and towards is the one direction the rule forbids.[^symmetry]

<aside><h4>When away has no direction</h4>Two identical measurements at the same place give a direction of zero length. The old code picked a side silently. The label then changed sides mid-stack with nothing on the image to explain it. Teaching the planner to work the direction out properly changed nothing. The renderer was re-seating labels on a criterion of its own. It had been overruling the planner all along.</aside>

## What a label has to go round

What a label has to avoid is declared on the thing to be avoided, as one number:

```yaml
vertebra: { clearance: 14 } # go round, and hold 14 px clear
disc_quiet: { clearance: ~ } # not an obstacle: write over a calm level
disc_bulging: { clearance: 0 } # hug the outline, which is not off
board_left: { clearance: 5 } # a panel obstructs like anatomy
caliper_arm: # no key at all -> the margin comes from the kind,
  kind: reading # never from zero, because forgetting must not disarm
```

Labels are free to use the space over the levels nobody needs to look at. The keep-outs say where the interesting anatomy is by saying where not to write.

![A sagittal lumbar frame with the layout guides switched on, a dashed rectangle drawn round every vertebra, disc, level name, label plate and text panel](https://illinifellow.com/i/029-oHsYAP5rla5.png)
*Every object on the frame declares a rectangle before anything is placed, labels and panels included. What the rectangles did not claim is the room the next label gets.*

<aside><h4>A field that went missing</h4>One entry was missing a padding field. That gave a keep-out rectangle made of non-numbers. Every intersection test against it answered "overlaps". Any comparison with a non-number is false, and the test was the negation of one. So the label was dropped and the logs stayed quiet about it. The defect arrived as a person asking where a label had gone.</aside>

An area is a polygon, and the leader has to land on its boundary. The two obvious ways of choosing that point both fail.[^anchor] What works is a cost rather than a distance. Vertical offset is traded against horizontal reach, so the point chosen faces the label.

![An axial slice with two nested outlines, their areas led out to labels on opposite sides, a linear measurement above and a panel of pathologies for the level](https://illinifellow.com/i/030-wJIJG-1crl4.webp)
*Each leader lands on the side of the boundary facing its own label. That took a cost rather than a nearest point. The findings panel sits in this corner on every slice in the stack.*

## One picture, three hosts

The three consumers see different worlds.[^hosts] The requirement is one sentence. The same study with the same configuration produces the same picture in all three, to the pixel.

<aside><h4>The strip that ran backwards</h4>The navigation strip ran backwards on one plane or the other depending on which host drew it. The plane's normal points opposite ways for sagittal and axial. Any single global choice of direction is wrong on one of them. The strip now follows the order it was handed the collection in.</aside>

> A special case for one host is a different picture for everyone.

The drawing surface is a 2D canvas.[^canvas] Pixel identity across three hosts is a bitmap problem, and the canvas is the surface all three share. Text is measured by the canvas itself, so the font ships inside the library.[^metrics]

<aside><h4>Twenty-three variants of one frame</h4>Choosing the face took a day and twenty-three rendered variants of one frame. A face changes the apparent size of a label even when the declared height is identical. Apparent size decides whether a label fits. A monospaced one won.</aside>

There are four coordinate systems: the analyser's pixels, the layout's units, the screen and the device. Each has its own type and one named conversion across. A number from one used as a number from another now fails to compile. Every placement bug the library ever had was that mistake. Making it unrepresentable retired the class.

<aside><h4>The tests hid the class</h4>Every stub held the second conversion factor at one. That is the value the headless renderer uses, where the conversion does nothing. A quantity that skipped it looked correct in every test and came out about a fifth off in a browser. The developer sandbox failed the same way from the other side, never reporting its own area. The fix landed in the tests rather than in the drawing.</aside>

![The overlay drawn live in a two-plane viewer: contours over both series, measurements and a findings panel on the axial one](https://illinifellow.com/i/031-qaBWwfBfjx1.jpg)
*The live path: the same drawing code over a viewport the reader pans and zooms. Under a gesture the overlay adjusts nothing. It asks the viewer again where every millimetre has landed.*

<figure>
<figure><img src="https://illinifellow.com/i/033-FkXtd9UJfr7.webp" alt="a sagittal lumbar slice at native scale with eighteen measurements led out to one column of labels"><figcaption>Native scale, eighteen measurements.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/034-qupwJ--h9XV.webp" alt="the same slice magnified twice, the anatomy twice the size and the labels unchanged, the column re-seated"><figcaption>Magnified twice. The anchors ride the anatomy while the type stays put, because only positions take the scale.</figcaption></figure>
</figure>

<aside><h4>A cap on reach</h4>Magnify enough and the column rides outward with the anatomy until it leaves the frame, so leader reach is capped. Hiding the overlay during a gesture was the cheap way to stay quick. It was written, then refused. A reader moving the image is the reader using it.</aside>

Behaviour is pinned by properties rather than by expected pictures.[^properties] Behind them sits a corpus of slices, one for every placement defect ever found.

<aside><h4>What a snapshot can pin</h4>There is no correct picture to write down, so the picture that came out is recorded and watched. Updating a snapshot takes one keystroke, and the updated file then records whatever the code does now as the thing it ought to do. That has pinned a defect as correct behaviour at least once. A slice also has to be reproduced at the height it is judged at, since the crowding differs at another size.</aside>

## What it cost to learn

**Rules the code calls inviolable.** A label moves away from the line. A leader never cuts across a caliper. A column does not move. Each is written down as absolute. Each is in practice a concession the solver may buy at a price. The documentation had recorded the intention as the behaviour.

**A mechanism that cancelled out.** An outside review found a mechanism tying the layout to the height of the composition. A public property, a fallback constant, tests and a stack of documentation. All of it the author's own. The reviewers were unanimous: the quantity cancels out of every comparison the solver can make. The jitter it had been built to remove had been there the whole time.

> Code can be correct, tested, documented, and still change nothing.

**Fallback values in the code.** They all came out. The configuration is the only source of truth. Almost none had ever fired, and several that could have were stale. After the removal the picture was identical to the pixel.

<aside><h4>Frames that came back black</h4>In one run seven overlays came back empty. Six kilobytes against a median of four hundred and fifty. All seven jobs reported success. The canvas size was set as a markup property and re-applied one step before the drawing, and assigning a size to a canvas wipes its pixels. The file's documentation said the defect was fixed. The code was the unfixed version.</aside>

## From part of one viewer to a standard

The library began as one JavaScript file inside a single viewer, with no tests and no way to run one. It has close to three hundred now. Twenty-five thousand eight hundred lines across sixty-two files, behind one published surface. The last production run drew 2738 slices across thirty-six jobs with no errors and no warnings. That had never happened before.

<aside><h4>What a clean run means</h4>No job reported an error and no slice was dropped. The counts come from the generator's own job records rather than from a test harness. Whether every label sat where a radiologist would have put it is a different question, and no counter answers it.</aside>

None of that is what sold it. The first prototypes stood at industry events while the engine behind them was still being built. The picture was enough. People stopped. Conversations turned into introductions and later into integrations. What stopped them was the slice with the findings already on it.

<figure data-abreast>
<figure><img src="https://illinifellow.com/i/035-X5qQQk6F6ku.webp" alt="visitors round a monitor at a trade-show stand, the slices with the findings on screen"><figcaption>A stand, one monitor, and whoever stopped.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/036-LQYzS3YNzeq.webp" alt="an auditorium of about a hundred watching the same slices on three stage screens"><figcaption>A product pitch, about a hundred people. The drawing does the selling.</figcaption></figure>
<figure><img src="https://illinifellow.com/i/037-BGbehOEL-Lj.webp" alt="a stand on the exhibition floor with the sagittal and axial series running on two monitors"><figcaption>The engine behind this was still being written.</figcaption></figure>
</figure>

> The library did the work silently but surely.

[^planes]: An MRI takes its pictures with a magnet and radio waves, and no radiation at all. It shows soft tissue, which is why a spine is examined this way: the discs and the nerves, not only the bone. A study is not one picture but stacks of thin slices, cut two ways. Sagittal cuts run down the body from front to back, so the spine is seen from the side with the levels one above the other. Axial cuts run across it, so a single level is seen from above, with the disc, the canal and the nerves in the same picture. Almost everything difficult in this post happens on an axial slice.

[^spine]: These are the words a spine report is written in. A disc is the pad between two vertebrae, and a protrusion is that pad pushed out of place. The canal is the tunnel down the middle of the spine carrying the cord and the nerves; stenosis is that tunnel being narrow, which is what presses on them. Canal diameter is how wide it is at one level, in millimetres. A report is a list of such statements, level by level, and everyone who reads it afterwards has no picture in front of them.

[^perception]: Brady, [Error and discrepancy in radiology: inevitable or avoidable?](https://doi.org/10.1007/s13244-016-0534-1), *Insights into Imaging* 2017, covering data back to 1949. The day-to-day rate quoted there is Berlin's, and retrospective review comes out at around thirty per cent. That retrospective figure is an average over targeted studies rather than a rate any one department measured, and the spread behind it is wide.

[^aorta]: Asmundo et al., [Incidental diagnosis and reporting rate of abdominal aortic aneurysms on lumbar spine MRI](https://doi.org/10.21037/qims-24-1291), *Quantitative Imaging in Medicine and Surgery* 2025, for the 1922 patients. A separate series of 395 studies found an aneurysm in 9.6 per cent, reported in 10.5 per cent: [Lumbar Spine MRI: Missed Opportunities for Abdominal Aortic Aneurysm Detection](https://pubmed.ncbi.nlm.nih.gov/31151692/), *Current Problems in Diagnostic Radiology* 2019. Both count what the report mentioned, which is a different question from what the reader saw. The smaller the aneurysm the less often it appears, and in the second series most were under four centimetres.

[^clearance]: A 510(k) is how most medical devices reach the American market. The maker shows the [Food and Drug Administration](https://www.fda.gov/medical-devices/premarket-submissions-selecting-and-preparing-correct-submission/premarket-notification-510k) that the new device is as safe and as effective as one already sold there, and the agency clears it. Cleared is not the same as approved: approval is the heavier route, kept for devices that carry more risk. The number is the file, K241108 here, and what the maker wrote in it is binding. The sentence about who checks the output is one of those promises.

[^summary]: The wording is the manufacturer's, filed with the clearance: the outputs "are reviewed, analyzed, confirmed or corrected by the user before any such content is included in the user's final report". It settles what the drawing is for. A claim the reader can check, and refuse.

[^headless]: A headless browser is an ordinary browser with no window on any screen, started and driven by a program instead of by a person. It exists so that a server can render a page exactly as a reader's browser would and then keep the result as a file, rather than show it to anybody. [Chrome runs this way](https://developer.chrome.com/docs/chromium/headless) from a switch on the command line. Here it means the frames are drawn by the same code that draws them live, on a machine with no screen at all.

[^dicom]: [DICOM](https://www.dicomstandard.org) is the file format and the language of medical imaging. It was agreed in 1993 by the American College of Radiology and the [National Electrical Manufacturers Association](https://www.nema.org), so that a picture from one maker's scanner opens in another maker's viewer, with the patient, the machine and the settings carried inside the file. It also fixes how pictures are grouped: a study is one visit, and a series is one stack of slices inside that study. Here the drawn frames are added as a new series in the same study. That is why they arrive wherever the original arrives, and why no viewer had to be taught anything.

[^frames]: The frames are the vendor's own, published on its help pages to show the feature working: one version, chosen by the people who built it. That makes them fair to reason from and no basis for a frequency. Nobody else in the niche puts this many kinds of measurement on the anatomy, so there is little else to reason from. A still says nothing about how often the crowding happens.

[^columbo]: The area on the sac's own border is on the vendor's [dural sac area](https://columbo.me/help/dural-sac-area/) page, the value under two contours on [herniation size](https://columbo.me/help/disk-herniation-size/). The cross-reference line through a level name is on the sagittal frame of the same pair. Each page was chosen to show the feature working, so these are the arrangements the vendor was content to publish.

[^haul]: On a real slice the count-only scoring hauled one measurement a fifth of the way across the picture to spare its leader a single plate. The neighbour at the same level sat comfortably beside its own caliper. The count went down, so the move was kept. That is the arrangement the ceiling now refuses.

[^sorted]: The premise is false. Height decides where a leader ends, and where it leaves its caliper sideways decides its path just as much, so two labels sorted by height can still cross. Nothing counted crossings, so nothing complained for as long as the comment stood.

[^symmetry]: Least total movement, tidy on a whiteboard, and the obvious answer. An independent pass over the placement found every hard rule satisfied and one thing wrong: the nearest label had travelled thirteen pixels in the forbidden direction. The rule is about direction rather than distance, so the size of the offset settles nothing.

[^anchor]: The nearest point on the boundary sends the leader back over itself, on exactly the shapes that matter. The point at the label's own height wanders round to the far side of the polygon. Both are cheap to compute, which is why they get tried first.

[^hosts]: The headless generator chooses its own frame size. The live viewport takes whatever the page left it, then gets panned, zoomed and resized. Underneath sit viewers built by other people, [OHIF](https://ohif.org) among them.

[^canvas]: A canvas is a rectangle of pixels inside a web page that a program paints into, one stroke at a time. It is [part of the web standard](https://html.spec.whatwg.org/multipage/canvas.html), and every browser has one. It remembers nothing: once a line is painted there is no line any more, only coloured pixels, and nothing can be moved, asked about or re-wrapped afterwards. That is exactly why it is used here, since the same instructions give the same pixels everywhere. It is also why everything a page normally does for free — wrapping a line of text, drawing a plate behind it — had to be written from nothing.

[^metrics]: What the canvas answers depends on the platform's rasteriser and on whether the font had finished loading. A width measured before the face arrives is the width of a fallback. Shipping the file inside the library removes the second half of that.

[^properties]: The properties: every label drawn, every label docked to its own column, plates and leaders clear of each other. All of it is measured on the rectangles the painter actually draws, rather than on the numbers the planner handed it. A property holds across every slice in the corpus at once.

<img data-cover src="https://illinifellow.com/i/middle-out.gif" alt="Frames from the overlay generator: labelled measurements drawn over a scan, redrawn again and again as the layout settles" />
