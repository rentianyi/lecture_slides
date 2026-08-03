# Module 1 — Recording Script

*Deep Learning for Medical Image Analysis · University of Washington*
Deck: `main.tex` (31 active frames, 4 parts) → `CV_OpenCourse_clean.pdf`

---

## How to use this

Narration is in `>` blocks — speak it more or less as written, but stay loose; these
are lines, not a legal document. `[CLICK]` marks an advance to the next Beamer overlay
on the **same** frame; a new `###` heading means a new frame.

Timings assume ~135 wpm (lightboard pace, slower than conversational). Total ≈ **44 min**,
which splits into three recordable segments matching the ~15-min targets in
`Lecture Outline.pdf`:

| Segment | Frames | Content | Time |
|---|---|---|---|
| **1.1 — Why medical imaging is hard** | 1–11 | Modalities, tasks, data challenges | ~14 min |
| **1.2 — Computer vision before deep learning** | 12–21 | Edges, Sobel, convolution, template matching, the ceiling | ~16 min |
| **1.3 — Neural networks & classification** | 22–31 | Feature hierarchy, MLP, learning, softmax | ~14 min |

⚠️ **Fix these three slides before recording** — see [Pre-recording fixes](#pre-recording-fixes)
at the bottom. One of them (the softmax frame) makes the script wrong if left as-is.

---

# Segment 1.1 — Why Medical Imaging Is Hard

## Title page — ~45 s

> Welcome to Deep Learning for Medical Image Analysis. I'm Tianyi Ren, and I teach this
> course with Mehmet Kurt at the University of Washington.
>
> This is an open course, so everything — the slides, the notebooks, the code — is free
> and yours to reuse.
>
> Here's the arc. Module 1, today, is fundamentals: what medical images actually are,
> what we ask a computer to do with them, and how the field got from hand-written image
> filters to neural networks. Module 2 is hands-on — you'll train three real models.
> Module 3 goes beyond task-specific models into self-supervision and foundation models.
> Module 4 is about what breaks when you take a model to an actual hospital.
>
> For today you need no prior deep learning. Some linear algebra and calculus will help
> when we get to gradients, but I'll walk through every step.

**Board note:** camera-facing, no writing. Establish eye contact here — this is the only
frame with no visual competing for attention.

---

## F1 · Imaging volume and complexity are increasing — `main.tex:138` · ~75 s

> Let's start with why anyone cares about this problem.
>
> Two things are growing at once. The first is the number of imaging studies — more
> patients, more scans per patient, more indications for imaging every year. The second
> is the size of each study. A head CT twenty years ago might have been thirty slices.
> Today a routine multiphase study is several hundred, and a research protocol can be
> thousands of images for one patient.
>
> The number of radiologists is not growing at that rate. It can't — training a
> radiologist takes about thirteen years from the start of medical school.
>
> So you have a widening gap between the volume of pixels produced and the number of
> expert eyes available to look at them. That gap is the entire commercial and clinical
> motivation for this field. Every technique in this course is, one way or another, an
> attempt to close some part of it.

---

## F2 · Medical Knowledge is highly subspecialized — `main.tex:151` · ~70 s

> But volume isn't the only problem. Look at this scan.
>
> Three experienced readers can look at this and give you three different answers.
> Glioma. ADEM — that's acute disseminated encephalomyelitis. HIV encephalopathy.
> All three are defensible from this image alone.
>
> Getting the right answer here isn't a matter of looking harder. It needs a
> neuroradiologist who's seen hundreds of each, plus clinical context the image doesn't
> contain — the patient's age, symptom onset, immune status.
>
> I want you to hold onto this slide, because it has a consequence we'll come back to
> for the rest of the course. When we train a model, we train it on labels. And the
> labels come from people looking at images like this one. So the ground truth we're
> fitting to is itself uncertain, expensive, and sometimes wrong. That is not a footnote.
> It shapes what's achievable.

---

## F3 · Trends of AI in Radiology — `main.tex:246` · ~85 s

> Here's a famous quote. In 2016, Geoffrey Hinton — who a few years later won the Nobel
> Prize in Physics for his work on neural networks — said, and I'm quoting:
> "It's just obvious that within five years deep learning is going to do better than
> radiologists."
>
> That was almost ten years ago. So how did it turn out?
>
> Radiologists are still here. In fact there's a shortage. Hinton was wrong about
> replacement — but look at the chart. He was completely right that something was about
> to happen. The number of AI-enabled medical devices cleared by the FDA takes off right
> around 2016 and hasn't stopped, and the large majority of them are in radiology.
>
> So the real answer is: the technology arrived, but it arrived as narrow assistive
> tools. A model that flags a possible large-vessel occlusion and moves that study to the
> top of the queue. A model that measures a nodule so the radiologist doesn't have to.
> Not one model that reads the scan and writes the report.
>
> Understanding *why* it went that way — why narrow and assistive instead of broad and
> autonomous — is genuinely one of the things this course is about. Module 4 is entirely
> that question.

---

## F4 · Primary Modalities in Medical Imaging — `main.tex:265` · ~90 s

> Before we can process medical images, we have to know what they are. And "medical
> image" is not one thing — the physics differs, so the failure modes differ, so the
> right model differs.
>
> Three primary modalities. Read this slide left to right as increasing dimensionality
> and increasing soft-tissue detail.
>
> **X-ray** is a two-dimensional projection. High-energy electromagnetic radiation goes
> through the body, dense tissue absorbs more of it, and you get a shadow. It's fast,
> cheap, and available everywhere — fractures and chest imaging.
>
> **CT** is the same physics, but you rotate the source and detectors around the patient
> and reconstruct cross-sections. Now you have a three-dimensional volume, and the
> intensities are calibrated — the Hounsfield scale — so a number actually means a
> tissue density. Trauma, tumors.
>
> **MRI** is completely different physics. Strong magnetic fields and radio pulses
> excite hydrogen protons, and you read out the signal as they relax. No ionizing
> radiation, and dramatically better soft-tissue contrast. Brain, spinal cord, ligaments.
>
> Keep this slide in your head, because when we start talking about distribution shift,
> the fact that these are three different physical processes — plus vendor and protocol
> differences *within* each one — is exactly what causes the trouble.

---

## F5 · Medical Imaging: X-Ray — `main.tex:372` · ~75 s · **4 clicks**

> Let's take each one properly. X-ray first.
>
> [CLICK] **Mechanism.** High-energy electromagnetic radiation is transmitted through
> the body and captured on a detector on the other side.
>
> [CLICK] **Two technical features you need.** *Attenuation* — dense structures like
> bone absorb more photons, so less radiation reaches the detector, so bone shows up
> bright. Everything you see in an X-ray is a map of how much each path absorbed.
>
> And *projection* — and this is the important limitation. You are collapsing a
> three-dimensional body onto a two-dimensional plane. Everything along a given ray gets
> summed into one pixel. So structures overlap, and a lesion can hide behind a rib.
>
> [CLICK] **Clinically**: bone fractures, dental imaging, and chest screening — which is
> why chest X-ray is far and away the most common medical image in public datasets, and
> why it's the benchmark you'll see in half the papers in this field.

---

## F6 · Medical Imaging: CT — `main.tex:467` · ~75 s · **4 clicks**

> CT solves the projection problem.
>
> [CLICK] **Mechanism.** Instead of one shot from one direction, you rotate an X-ray
> source and a detector array around the patient, collect projections from many angles,
> and mathematically reconstruct the cross-sections.
>
> [CLICK] **Technical features.** *Hounsfield Units* — CT is quantitative. Water is
> zero by definition, air is minus one thousand, dense bone is up in the thousands. That
> means a number in a CT means the same thing on Monday as on Tuesday, and on a Siemens
> scanner as on a GE scanner. That's rarer than you'd think, and it matters enormously
> for machine learning — it's why CT preprocessing is usually just clipping to a window
> and rescaling, with no per-image normalization needed.
>
> And *tomographic reconstruction* — many projections in, a stack of slices out, which
> you can treat as a 3D volume.
>
> [CLICK] **Clinically**: complex fractures, trauma, internal bleeding, tumor detection
> and staging. It's the workhorse of emergency medicine because it's fast.

---

## F7 · Medical Imaging: MRI — `main.tex:553` · ~85 s · **4 clicks**

> MRI. Different physics entirely.
>
> [CLICK] **Mechanism.** You put the patient in a strong magnetic field, which aligns
> the hydrogen protons in their tissue. You hit them with a radiofrequency pulse to knock
> them out of alignment. Then you listen as they relax back, and reconstruct an image
> from that signal.
>
> [CLICK] **Three features.** *No ionizing radiation* — so you can image repeatedly,
> which is why longitudinal studies and pediatric imaging lean on MRI.
>
> *Soft-tissue contrast* — far better than CT. For brain, muscle, ligaments, MRI is
> simply the correct tool.
>
> And *T1 versus T2*. This one trips people up, so: these are different pulse sequences
> — different ways of exciting and reading out the same tissue. On T1, fat is bright.
> On T2, fluid is bright. Same patient, same scanner, same session, completely different
> looking images. A typical brain MRI protocol gives you four or five of these sequences,
> and they're all inputs to your model. That's why you'll see medical imaging models take
> multi-channel input where the channels are *sequences*, not colors.
>
> [CLICK] **Clinically**: brain and spinal cord, ligament and tendon injury, pelvic and
> abdominal soft tissue.

---

## F8 · Common Tasks in Medical Image Analysis — `main.tex:668` · ~55 s

> So we have images. What do we actually ask a computer to do with them?
>
> This figure is from the MedSAM paper — Wang and colleagues, *Nature Communications*,
> 2024 — and I'm showing it first because of its sheer variety. Every panel is a
> different modality, a different organ, a different target. CT, MRI, endoscopy,
> pathology, ultrasound, dermoscopy.
>
> Notice that the *task* here is the same in every panel — outline the structure of
> interest — but almost nothing else is. That tension, between one task and enormously
> heterogeneous data, is the story of Module 3.
>
> Let's organize the tasks properly.

---

## F9 · Common Tasks — the pipeline (static) — `main.tex:702`

> **[SKIP THIS FRAME IN RECORDING.]**

This is the un-animated twin of F10 — identical content, no shroud overlay. It exists as
a print/handout version. Advance straight past it, or comment it out before the recording
build. See [Pre-recording fixes](#pre-recording-fixes).

---

## F10 · Common Tasks — the pipeline (animated) — `main.tex:814` · ~2 min · **4 clicks**

> Here's the map for the whole field. Four stages, left to right, following what happens
> to an image from the scanner to a clinical decision.
>
> [CLICK] **One — preparation.** Before you analyze anything, you often have to fix the
> image. *Super-resolution* takes a fast, low-resolution acquisition and recovers detail
> — which matters because scan time is expensive and patients move. *Registration*
> aligns images into a common coordinate frame — the same patient across two timepoints,
> or a PET and an MRI of the same head. Nothing downstream works if your images aren't
> in the same space.
>
> [CLICK] **Two — classification.** Assign a label to an image or a region. At the top,
> identifying cell types in pathology. At the bottom, the classic one: is this lesion
> benign or malignant? One image goes in, one label comes out. This is Lecture 1 of
> Module 2, and it's the last thing we'll cover today.
>
> [CLICK] **Three — localization.** Classification tells you *what*; these tell you
> *where*. *Detection* puts a box around something — here, flagging a candidate lesion.
> *Segmentation* goes further and labels every single pixel, which is what you need if
> you want to measure a volume rather than just say "there's a tumor." Segmentation is
> Lecture 2 of Module 2.
>
> [CLICK] **Four — outcome.** This is the part that actually changes patient care.
> *Monitoring* — has this lesion grown between January and June? *Prediction* — given
> this scan, will this patient respond to chemotherapy?
>
> Everything in this course lives somewhere on this diagram. When you read a paper in
> this field, the first question to ask is: which column is this?

---

## F11 · Medical Image Data is Intrinsically Challenging — `main.tex:949` · ~2 min · **6 clicks**

> Now the bad news. If medical imaging were just computer vision with grayscale pictures,
> this course would be one lecture long. It isn't, for four reasons.
>
> **Data scarcity.** [CLICK] Medical images are enormous — a single CT volume can be
> half a gigabyte — but you have very few *patients*. ImageNet has fourteen million
> labeled images. A well-funded medical imaging study has a few hundred subjects. High
> resolution, tiny sample size, which is the exact opposite of what deep learning was
> designed for.
>
> [CLICK] **And the distribution is long-tailed.** Look at that curve. Common conditions
> dominate your dataset, and rare diseases — which are frequently the ones you most want
> to catch, because they're the ones that get missed — sit out in the tail with a handful
> of examples. If you train naively, your model learns to be very good at the diagnosis
> the radiologist would have made anyway.
>
> [CLICK] **Distribution shift.** Medical imaging is not standardized. A Siemens scanner
> and a GE scanner produce visibly different images of the same patient. Protocols vary
> between hospitals, and between departments in the same hospital. A model trained at one
> site routinely loses substantial performance at another.
>
> [CLICK] **And the labels are subjective.** Remember that scan from a few slides ago
> with three plausible diagnoses? Give the same image to two radiologists and you get
> disagreement — inter-observer variability. So there's a ceiling on your accuracy that
> has nothing to do with your architecture.
>
> [CLICK] **Privacy.** Medical data is legally protected — HIPAA in the US, GDPR in
> Europe — so it stays siloed in the institution that collected it. You often cannot
> simply pool data from ten hospitals, even when everyone wants to.
>
> [CLICK] And de-identification is harder than stripping the header. From a head CT you
> can reconstruct the patient's face well enough to identify them. Removing the name from
> the file does not make the data anonymous.
>
> Four problems: scarcity, shift, subjectivity, privacy. Every one of them gets a
> dedicated lecture in Module 4. For now, just know that this is why medical imaging is
> its own field rather than a subfolder of computer vision.

**Board note:** natural place for a breath and a beat before the part transition.

---

# Segment 1.2 — Computer Vision Before Deep Learning

## Part divider · History of Computer Vision — ~30 s

> Now let's rewind.
>
> I want to spend the next stretch on how people did computer vision *before* neural
> networks. Not for historical interest — because the operation at the heart of the
> convolutional neural network was invented here, by hand, decades earlier. If you
> understand what a Sobel filter does, a CNN stops being magic and becomes something
> fairly obvious.

---

## F12 · Traditional Computer Vision — `main.tex:1154` · ~2 min · **6 clicks**

> Classical computer vision was a toolbox of hand-designed operations. Five of the big
> ones:
>
> [CLICK] **Thresholding.** The simplest possible operation: pick a value, and every
> pixel above it becomes white, everything below becomes black. On the image at right,
> that alone separates the coins from the background. Crude, but for controlled lighting
> it's often enough — and it's still the first thing people try on segmentation problems.
>
> [CLICK] **Edge detection.** Find locations where brightness changes sharply. This is
> the one we're going to do properly, because it's the ancestor of the convolution layer.
>
> [CLICK] **Segmentation.** Group the regions enclosed by those edges into objects.
>
> [CLICK] **Curve detection.** Link adjacent edge pixels into continuous contours and
> shapes — the Hough transform is the classic version.
>
> [CLICK] **Optical flow.** Given two consecutive frames, estimate how each pixel moved.
> The basis of all motion analysis.
>
> [CLICK] And notice the shape of the whole thing. **Pixels, to edges, to shapes, to
> motion.** Each stage takes the output of the previous one and abstracts a little
> further. A human designed every stage and decided how they connect.
>
> Hold onto that structure. In about fifteen minutes I'm going to show you a neural
> network that learned almost exactly this hierarchy on its own, without anyone
> specifying it.

---

## F13 · Feature Engineering: Sobel Filters — `main.tex:1274` · ~2 min · **6 clicks**

> Let's do edge detection properly.
>
> [CLICK] First, a definition worth being precise about. **An edge is a location where
> image intensity changes abruptly.** That's it. Not "the outline of an object" — the
> image doesn't know about objects. Just: intensity changed fast here.
>
> [CLICK] The intuition. Inside a flat region — the middle of an organ, a patch of sky
> — neighboring pixels have nearly the same value, so the change is small. At an object
> boundary, you cross from one material to another and the value jumps.
>
> "Changes fast" should be setting off a bell. That's a derivative. So **treat the image
> as a function** — `f(x,y)` returns the intensity at position x, y — and take its
> gradient. Partial derivative in x, partial derivative in y. Two numbers per pixel: how
> fast intensity is changing horizontally, and how fast vertically.
>
> [CLICK] The gradient is a vector, and we usually want a single number for "how much of
> an edge is here." So take its magnitude — root of the sum of the squares. Big magnitude
> means strong edge. And the **goal** is exactly that: find the pixels where the gradient
> magnitude is large.
>
> [CLICK] **Why we bother.** Two reasons. It extracts object boundaries, which is what
> we actually care about. And it's an enormous reduction in data — you go from a million
> intensity values to a sparse set of boundary pixels. That mattered a great deal when
> computers were slow.
>
> [CLICK] One problem. A digital image isn't a smooth function — it's a grid of integers.
> You can't take a derivative analytically. You need a numerical approximation.
>
> [CLICK] And that approximation is the **Sobel filter.**

---

## F14 · Sobel Filter: Vertical Edge Detection — `main.tex:1353` · ~2 min · **2 clicks**

> Here's the whole thing on one slide. Let's compute it by hand.
>
> On the left, a three-by-three patch of an image. The left two columns are dark — value
> ten. The right column is bright — value two hundred. So there's a strong vertical edge
> running down the middle of this patch. Our filter should say so.
>
> In the middle is the Sobel kernel — call it G-x. Three by three. Negative weights on
> the left column, zero in the middle, positive on the right. And note the middle row is
> weighted double: minus two, zero, plus two. That's a small amount of smoothing built
> in, so a single noisy pixel doesn't produce a false edge.
>
> The operation is convolution — that's the asterisk. Line the kernel up on the patch,
> multiply each pair, and add everything up.
>
> [CLICK] So: minus one times ten, from the top-left. Zero times ten. Plus one times two
> hundred. Do that for all three rows, remembering the middle row is weighted double, and
> the total is **seven hundred and sixty.**
>
> [CLICK] Read what happened. **The negative weights suppress the dark side, the positive
> weights amplify the bright side.** If the patch were uniform — say two hundred
> everywhere — the negatives and positives would cancel exactly and you'd get zero. You
> only get a large response when the two sides differ. Seven-sixty is a big number, so:
> strong vertical edge.
>
> And here's the sentence to remember for the rest of the course. **That's it. That's
> the operation at the core of a convolutional neural network.** The only thing a CNN
> changes is where the nine numbers in the kernel come from. Sobel wrote them down in
> 1968. A CNN learns them from data.

**Board note:** if you write anything on the lightboard in this segment, write it here —
sketch the `−1 0 1 / −2 0 2 / −1 0 1` grid as you say it. This is the load-bearing frame
of the whole module.

---

## F15 · Sobel Filter with Convolution — `main.tex:1476` · ~85 s

> One kernel position gives you one number. To get an edge *image*, you slide the kernel
> over every position. This diagram unrolls that.
>
> Left plane: the image matrix. The highlighted three-by-three window is the region we're
> looking at right now — values around a hundred, so this is a fairly flat patch.
>
> Middle plane: the kernel, lined up with that window. Note this is the *other* Sobel
> kernel — minus one, minus two, minus one across the top, zeros through the middle,
> plus one, plus two, plus one along the bottom. It's the previous kernel rotated ninety
> degrees, so it responds to *horizontal* edges instead of vertical ones. In practice you
> always run both and combine them.
>
> Right plane: the output matrix. The red box shows the receptive field — which input
> pixels contributed — and the blue lines show that all nine of them collapse into a
> single output value. That value is **minus eight**: small, because this patch is nearly
> flat. No edge here.
>
> One window, one output pixel. Now move the window.

---

## F16 · Sobel Filter with Convolution, shifted — `main.tex:1597` · ~50 s

> Same picture, one step later. The window has moved down by one row — you can see the
> highlighted region and the values have shifted.
>
> Nine new inputs, same nine kernel weights, one new output pixel: **minus nine.**
>
> That's the entire algorithm. Slide, multiply, sum, write down one number, repeat until
> you've covered the image. Out comes a new image where bright means "strong edge here."
>
> Two things worth naming while we're looking at this. First, **the kernel weights don't
> change as you slide** — the same nine numbers everywhere in the image. That's called
> weight sharing, and it's why a convolution layer with a three-by-three kernel has nine
> parameters no matter whether the image is a hundred pixels wide or a thousand. Second,
> **the output is smaller than the input** — the window can't hang off the edge. That's
> where padding comes from, and it's why you'll spend an evening someday debugging a
> tensor shape mismatch.

> ⚠️ *This frame's title says "Slide Right + 1" but the window actually moves* down *one
> row. Fix the title before recording — see the bottom of this document.*

---

## F17 · Feature Engineering: Template Matching — `main.tex:1723` · ~50 s

> Edges are one hand-designed feature. Here's a second one, using the exact same
> machinery for a different purpose.
>
> The problem: I give you a small patch — a template — and a large image, and ask you to
> find where the patch occurs. This is Where's Waldo as an algorithm. Practically: find
> every instance of this cell type on the slide, or find the vertebra in this scan.
>
> Take a moment with the image before I show the answer. How would you do it?

**Board note:** actually pause. Two or three seconds of silence, camera-facing.

---

## F18 · Template Matching — the answer — `main.tex:1742` · ~55 s

> Here's the answer, and the method is the one you already know.
>
> Slide the template over every position in the image. At each position, multiply the
> template against the patch underneath it and sum. That number is a *similarity score* —
> high where the patch under the window looks like the template, low where it doesn't.
>
> The result is a whole map of scores. Take the maximum, and that's your match.
>
> Notice what just happened. Same sliding window, same multiply-and-sum, same output map.
> The only thing that changed is what we put in the kernel and how we read the output.
> With Sobel weights, we called the output an edge map. With an image patch as the
> weights, we call it a similarity map.
>
> This is worth sitting with, because it's the reason CNNs work. **A convolution kernel
> is a template, and convolving is asking "how much does this look like my template?" at
> every location.** A trained CNN is a very large collection of learned templates.

---

## F19 · Template Matching — cross-correlation — `main.tex:1761` · ~55 s

> The formal name for this operation is **cross-correlation**, and here it is in the same
> three-plane diagram we used for Sobel.
>
> Image on the left, with the current window highlighted. Kernel in the middle — but now
> the entries are written abstractly, `k` sub `i,j`, because the point is that these are
> just *some* nine numbers. Output on the right: one cell, labeled `corr`.
>
> Mathematically, cross-correlation and convolution differ only by whether you flip the
> kernel before applying it. In deep learning we ignore that distinction entirely, and
> what PyTorch calls `Conv2d` is technically cross-correlation. It doesn't matter,
> because the weights are learned — if a flip were needed, the network would just learn
> the flipped kernel. But you'll see both words used interchangeably, so now you know why.
>
> So: one operation, two classical applications, and it's the same operation deep
> learning uses. What changed?

---

## F20 · Evolution into Neural Networks (build 1) — `main.tex:1939` · ~90 s · **4 clicks**

> Here's what changed.
>
> [CLICK] In **classical vision, we hand-engineer the kernels.** Somebody sat down,
> thought about what an edge is, and wrote down nine numbers. And that approach worked —
> genuinely worked. Edge detection is one of the most successful primitives in the
> history of the field, and it's still in production everywhere.
>
> [CLICK] But it has a cost, and the cost is **human intuition.** Every feature is a
> research project. Someone has to have the idea, design it, tune it, and then hand-build
> the pipeline that connects it to the next stage. Nothing is end-to-end — you're
> assembling a machine by hand, stage by stage.
>
> [CLICK] And it has a limit: **poor generalizability.** Sobel works because "an edge" is
> a simple, universal, writable-down concept. Now write me the nine numbers that detect
> a face. Or a malignant lesion. You can't. Nobody can. The concept has too much
> variation — different sizes, orientations, textures, contexts — to be captured by a
> filter a human designs.
>
> That's the ceiling. Not that classical vision doesn't work, but that it only works for
> concepts simple enough for a person to specify.
>
> **Neural networks generalize the idea.** Same convolution, same sliding window, same
> multiply-and-sum. The network learns the kernel weights, and it learns them by
> backpropagation — which we'll get to shortly.

---

## F21 · Evolution into Neural Networks (build 2) — `main.tex:1973` · ~50 s · **5 clicks**

> Let me put that on one slide, with the scorecard.
>
> Checkmark: hand-crafted features worked. Cross: they need intense human intuition.
> Cross: they scale badly to complex objects and high intra-class variation.
>
> And look at that red arrow. It runs from the thing that made classical vision work
> straight down to the thing that killed it. **The same property — that a human designs
> the feature — is both why it succeeded and why it stalled.**
>
> [CLICK] Neural networks generalize the idea. [CLICK] [CLICK] **The network learns the
> kernels** — nobody writes down the nine numbers. [CLICK] **And the weights are optimized
> by backpropagation** — the network finds them by being told, repeatedly, how wrong it
> was.
>
> That's the whole conceptual jump. We stopped designing features and started designing
> the *process that discovers* features.

*(F20 and F21 are two builds of the same argument. If you want a tighter deck, record
only F21 — see the bottom of this document.)*

---

# Segment 1.3 — Neural Networks & Classification

## F22 · Hierarchical Feature Learning in CNNs — `main.tex:2047` · ~90 s

> Here's the payoff, and it's one of my favorite results in the field.
>
> This is from Zeiler and Fergus, 2014 — "Visualizing and Understanding Convolutional
> Networks." They took a trained CNN and asked a simple question: what does each filter
> in each layer actually respond to? Then they visualized the image patches that made
> each filter fire hardest.
>
> Nobody told this network about edges. Nobody told it about textures or object parts.
> It was trained on one task only — put a label on this photograph — and these features
> are what fell out.
>
> **When trained for classification, CNNs learn features in a hierarchy of increasing
> abstraction.** And when you look at what layer one learned — oriented edges and color
> gradients — you are looking at something extremely close to a Sobel filter. The network
> rediscovered, from data alone, the feature that a human wrote down by hand in 1968.
>
> It didn't stop there. It kept going, and built layers of abstraction on top that no
> human had successfully hand-designed.

---

## F23 · Hierarchical Feature Learning — the three levels — `main.tex:2077` · ~80 s

> Let's read the hierarchy properly, layer by layer.
>
> **Early layers**: simple edges and color gradients. Oriented bars, light-dark
> transitions. Sobel's descendants.
>
> **Middle layers**: textures, curves, and object parts. This is where it gets
> interesting — the network has started composing. It's taking combinations of edges and
> forming corners, circles, repeated patterns. Nobody specified this composition step;
> it's what you get when you stack convolutions.
>
> **Deep layers**: whole objects and scene-level concepts. Faces. Wheels. Dogs. Units
> that fire for a semantic category rather than a visual primitive.
>
> Now put that next to the classical pipeline from earlier — pixels, to edges, to shapes,
> to motion. **It's the same architecture.** Simple local features composed into
> increasingly abstract ones. The difference is that in classical vision a human designed
> each stage and wired them together, and here the features are learned end-to-end from
> data. The network was given only images and labels, and it built the pipeline itself.
>
> That's the thesis of deep learning, on one slide.

*(F24 at `main.tex:2138` is a third layout of this same figure. Skip it in the recording
— see the bottom of this document.)*

---

## F25 · A Simple Neural Network — `main.tex:2263` · ~2 min

> So how does a network actually learn anything? Let's strip everything away and look at
> the smallest network that has all the parts.
>
> Three layers. On the left, the **input layer** — two values, `i₁` and `i₂`. Think of
> them as two measurements. In the middle, the **hidden layer** — two units, `h₁` and
> `h₂`. On the right, the **output layer** — one unit, which produces our prediction.
>
> The interesting part isn't the circles, it's the boxes between them. Those are the
> **weights.** Every connection carries one number.
>
> Each input goes to both hidden units, and each connection has its own weight — `w₁`
> through `w₄`. So `h₁` is `i₁` times `w₁`, plus `i₂` times `w₂`. It's a weighted sum of
> the inputs. `h₂` is the same inputs with different weights — `w₃` and `w₄` — so it
> computes something different. Then `h₁` and `h₂` feed the output through `w₅` and `w₆`.
>
> Two things to notice.
>
> First, **the weights are the only thing we get to change.** The inputs come from the
> data and the structure is fixed. Training a neural network means finding good values for
> those six numbers, and nothing else. A modern network has billions of them, but it's
> the same statement.
>
> Second — and hold this thought, because we come back to it in about ten minutes — every
> operation on this slide is multiplication and addition. It's all linear. Which means
> this whole network, all three layers, could be collapsed into a single multiplication.
> That's a problem, and we'll fix it shortly.

---

## F26 · From Prediction to Learning — `main.tex:2499` · ~2.5 min · **7 clicks**

> Now let's make it learn. We'll unroll that network algebraically.
>
> The prediction is whatever comes out of the output unit. Call it `out`.
>
> [CLICK] But `out` isn't fundamental — it's built from the layer before. **Prediction
> equals `h₁` times `w₅`, plus `h₂` times `w₆`.**
>
> [CLICK] And the `h`'s aren't fundamental either. `h₁` is `i₁w₁ + i₂w₂`; `h₂` is
> `i₁w₃ + i₂w₄`. Substitute those in.
>
> [CLICK] And now everything is written in terms of what we actually have. **Prediction
> equals the quantity `i₁w₁ + i₂w₂`, times `w₅`, plus the quantity `i₁w₃ + i₂w₄`, times
> `w₆`.**
>
> Look at what's in that expression. The `i`'s — those are the data, fixed, given to us.
> And the `w`'s. That's all. There's nothing else.
>
> [CLICK] Which gives us the conclusion in the yellow box: **to change the prediction,
> we have to change the weights.** That's the only lever we have. Learning *is* adjusting
> weights.
>
> So the question becomes: adjust them which way?
>
> [CLICK] Here's the rule. **New weight equals old weight, minus alpha times the
> derivative of the error with respect to that weight.**
>
> Read it in pieces. The **error** is how wrong we were — we made a prediction, we know
> the true answer, we measure the gap.
>
> [CLICK] The **derivative of the error with respect to the weight** is the key quantity.
> It answers: if I nudge this one weight up a little, does the error go up or down, and
> by how much? It's a slope. If it's positive, increasing this weight makes things worse.
>
> So we subtract it — we move each weight in the direction that reduces the error. That's
> why there's a minus sign, and that's the whole idea of **gradient descent.**
>
> **Alpha** is the learning rate: how big a step to take. Too big and you overshoot and
> the training blows up. Too small and you'll be there all week. It's the first
> hyperparameter you'll ever tune, and in Module 2 you'll tune it for real.
>
> And computing that derivative efficiently, for every weight in a deep network, in one
> backward pass — that's **backpropagation.** It's the chain rule from calculus, applied
> systematically. PyTorch will do it for you in one line, but this is what that line is
> doing.
>
> That's the complete learning algorithm. Predict, measure the error, compute the slope,
> step downhill, repeat.

---

## Part divider · Classification — ~15 s

> We've got images and we've got a learning machine. Let's put them together on the first
> real task.

---

## F27 · Image Classification — `main.tex:2676` · ~70 s

> **Image classification.** You're given a fixed set of discrete labels in advance — and
> that "in advance" matters, so hold it — and the job is to learn a function `f` that
> takes an image and returns one of them.
>
> Here, three brain tumor types. Feed in this scan, out comes "Glioblastoma." This one,
> "Meningioma." This one, "Metastasis."
>
> Two things about that word *fixed*. First, the model can only ever produce a label from
> the list you gave it. If you show a three-class tumor model a healthy brain, it will
> confidently tell you it's one of the three tumors. It has no vocabulary for "none of
> these." That's not a bug in the training — it's a property of the problem as we've
> defined it, and it's a genuine safety issue in clinical deployment. Module 4's lecture
> on uncertainty is largely about that.
>
> Second, notice how much information we're throwing away. The scan is a hundred million
> voxels and we're compressing it to one word out of three. Segmentation and detection —
> Module 2 — are what you do when that's not enough.

---

## F28 · Image Classification Model — `main.tex:2734` · ~60 s

> Here's the architecture in outline, and you can now read every box on it.
>
> On the left, the input image. Then a stack of **convolutional layers** — those are the
> sliding kernels we spent the last twenty minutes on, except the weights are learned.
> Interleaved with them, **pooling layers**, which shrink the spatial dimensions so that
> later layers see a larger portion of the image through the same small kernel. That's
> how you get from local edges to global objects.
>
> All of that together is the **feature extractor**, and what comes out the other end is
> the learned hierarchy from the Zeiler and Fergus slide — a compact vector describing
> what's in the image.
>
> Then on the right, the **classifier head** — fully connected layers like the little
> network we just unrolled — which maps that vector to one score per class.
>
> Feature extractor, then classifier. Practically every classification model you meet has
> this shape, and in Module 2 you'll fine-tune exactly this by keeping a pretrained
> feature extractor and replacing the head.

---

## F29 · Activation Functions — `main.tex:2756` · ~2 min

> Two pieces missing. Here's the first, and it's the thought I asked you to hold.
>
> Every operation we've described — convolution, weighted sums — is linear. And stacking
> linear operations gets you nothing: a linear function of a linear function is still
> linear. A fifty-layer network of pure multiply-and-add is mathematically equivalent to
> a single layer. All that depth would be wasted.
>
> The fix is to insert a **nonlinearity** after each layer. It's a small thing that makes
> depth mean something.
>
> On the left, the **sigmoid.** One over one plus e to the minus x. Look at the shape: it
> squashes any input into the range zero to one, smoothly. Very large input goes to one,
> very negative goes to zero. Historically this was *the* activation function, partly
> because the output looks like a probability.
>
> On the right, **ReLU** — Rectified Linear Unit. `max(0, x)`. If the input is positive,
> pass it through unchanged. If it's negative, output zero. That's the entire function.
>
> It looks almost too simple to matter. The first time I saw it I assumed it couldn't
> possibly be enough. But that single kink at the origin is all the nonlinearity you need,
> and compositions of ReLUs are richly nonlinear.
>
> And it's cheap. Sigmoid needs an exponential for every activation. ReLU needs a
> comparison. Multiply that across billions of activations and it's a serious difference.
>
> ReLU is the default in essentially every modern architecture, and it's what you'll use
> in Module 2.

---

## F30 · Activation Functions Comparison — `main.tex:2907` · ~2 min

> Let's be precise about the trade-off, because there's one entry in this table that
> explains a decade of research.
>
> **Sigmoid.** Pros: smooth output between zero and one, natural for binary
> classification, and it's what everyone used historically.
>
> Cons, and here's the important one: the **vanishing gradient problem.** Look back at
> the sigmoid curve. Out at x equals five, the curve is essentially flat — the slope is
> nearly zero. Now remember backpropagation: the gradient at each layer gets multiplied
> by the gradient of that layer's activation. If that's a small number, and you multiply
> a chain of small numbers together across many layers, the gradient reaching the early
> layers becomes vanishingly small. Those layers stop learning. Effectively, **you can't
> train a deep sigmoid network.**
>
> That was a real wall. It's a large part of why neural networks stalled for years.
> Also, sigmoid isn't zero-centered, which biases the updates, and the exponential is
> expensive.
>
> **ReLU.** On the positive side the gradient is exactly one — not zero-point-something,
> *one*. Multiply a chain of ones and you get one. The gradient reaches the early layers
> intact. That, more than any other single change, is what made deep networks trainable.
> Plus it's computationally trivial and converges faster.
>
> Its cons: not zero-centered, and the **dying ReLU** problem. If a unit's input becomes
> negative for every training example, its output is always zero, so its gradient is
> always zero, so it never updates again. It's permanently dead. Variants like Leaky ReLU
> exist to fix that, but in practice plain ReLU is usually fine.
>
> The summary: ReLU in hidden layers, and sigmoid only when you specifically want a
> probability at the output.

---

## F31 · Softmax Classifier — `main.tex:3046` · ~2.5 min

> Last piece. Our network produces raw numbers, and we need probabilities and a training
> signal. Here's how you get both.
>
> Start on the left. We fed in this scan, and the network produced three raw scores —
> one per class. **Three point two for glioblastoma, five point one for meningioma, minus
> one point seven for metastasis.** These are called **logits**, or unnormalized
> log-probabilities.
>
> They're not probabilities. One of them is negative, and they don't sum to anything in
> particular. All we know is that bigger means the network favors that class.
>
> Two problems to fix, and one function fixes both.
>
> **Problem one: probabilities can't be negative.** So exponentiate. `e` to the power of
> each score. e-to-the-3.2 is about twenty-four and a half, e-to-the-5.1 is a hundred
> sixty-four, and e-to-the-minus-1.7 is about zero-point-one-eight. Everything is now
> positive, and notice that exponentiating *amplifies differences* — 5.1 was only 1.9
> above 3.2, but 164 is nearly seven times 24.5. The winner gets pushed ahead.
>
> **Problem two: probabilities have to sum to one.** So normalize — divide each by the
> total. That gives **0.13, 0.87, and effectively 0.00.**
>
> Exponentiate then normalize — that's the **softmax function**, and now we have a real
> probability distribution. The model is eighty-seven percent confident this is a
> meningioma.
>
> Now: how wrong is that? On the far right is the truth, as a one-hot vector — a one on
> the correct class, zeros everywhere else. We compare our distribution to that one, and
> the comparison is **cross-entropy loss**: negative log of the probability the model
> assigned to the correct class.
>
> Work through the two cases and you'll see why it's the right shape. If the model gives
> the true class 0.87, the loss is minus log of 0.87 — about **0.14**. Small; barely any
> correction. But if the model had assigned the true class only 0.13, the loss would be
> minus log of 0.13 — about **two point oh four.** Fifteen times larger. And as the
> assigned probability heads toward zero, the loss heads to infinity.
>
> **Confident and right costs almost nothing. Confident and wrong is punished savagely.**
> That's exactly the incentive you want.
>
> And that single number is the "Error" from the learning slide. It's what we take the
> derivative of. It's what flows backward through the network and adjusts every weight.
>
> Which means we've now closed the loop.

> ⚠️ **This narration assumes the softmax slide is fixed** — the thumbnail is a
> meningioma, so the one-hot vector must put 1.00 on **Meningioma**, not Glioblastoma.
> As the slide currently stands, the numbers say the model was 87% confident and *wrong*,
> which contradicts the story. See the bottom of this document. If you record before
> fixing it, swap the two loss values in the paragraph above.

---

## Closing — ~60 s

> Let's put the whole module in one paragraph.
>
> Medical imaging produces more data than experts can read, in modalities with different
> physics, and the data is scarce, shifted, subjectively labeled, and legally locked
> down. Classical computer vision attacked images with hand-designed filters — and the
> central one, the Sobel edge detector, is a three-by-three grid of numbers you slide
> across an image and multiply. That approach worked beautifully for concepts a human
> could write down, and hit a hard ceiling on everything else. Neural networks kept the
> operation and threw away the hand-design: the kernels become learnable weights, the
> weights are adjusted by gradient descent against a loss, and what emerges is a
> hierarchy of features that looks remarkably like the pipeline we used to build by hand
> — edges, then parts, then objects.
>
> You now have every concept you need to train a real model.
>
> That's Module 2. Three lectures, three architectures, three Colab notebooks you'll run
> yourself: ResNet for classification, U-Net for segmentation, and VAEs and GANs for
> synthesis. The recipe is the same one from today's learning slide, every time. All
> that changes is the model, the loss, and the shape of the output.
>
> See you there.

---

# Pre-recording fixes

Three real problems and three cosmetic ones. The first is the only one that breaks the
script.

### 1. Softmax slide contradicts itself — `main.tex:3074–3110` ⚠️ **blocking**

The thumbnail is `Figures/menigioma.png` and the model's probabilities peak on
**Meningioma** (0.87) — but the "Correct probs" column puts `1.00` on the
**Glioblastoma** row. As drawn, the model is confidently wrong, which is the opposite of
the point being made. Fix by moving the one-hot down a row:

```latex
\node[text=white] at (5.45,1.1)  {\Large 0.00};   % Glioblastoma
\node[text=white] at (5.45,0)    {\Large 1.00};   % Meningioma  ← truth
\node[text=white] at (5.45,-1.1) {\Large 0.00};   % Metastasis
```

Also on this frame: `normaize` → `normalize` (`main.tex:3117`).

### 2. Frame title says "Slide Right" but the window slides down — `main.tex:1597`

The code comments say `SHIFTED DOWN` and the values confirm it — the window moves one
row down, not one column right. Retitle to `Sobel Filter with Convolution (Slide Down + 1)`.

*(The arithmetic on both frames is correct: −8 and −9 both check out against the Gy
kernel. Only the title is wrong.)*

### 3. Institute typo — `main.tex:80`

`University of Wasghinton` → `University of Washington`. It's on the title slide.

Related: `main.tex:82` is `\date{\today}`, which stamps the build date on the title slide.
For an open course, set a fixed term instead.

### 4. Three near-duplicate frames to cut before the recording build

| Keep | Cut | Why |
|---|---|---|
| F10 `main.tex:814` (animated) | F9 `main.tex:702` | Identical content; F9 has no shroud overlay |
| F21 `main.tex:1973` | F20 `main.tex:1939` | Two builds of the same argument; F21 has the red arrow |
| F23 `main.tex:2077` | F22 `main.tex:2047`, F24 `main.tex:2138` | Three layouts of the Zeiler & Fergus figure |

Cutting all four saves ~4 minutes and removes the most obvious "why am I seeing this
again" moments. The script above is written so you can record either way — the frames
marked SKIP have no narration of their own.

If you keep F22 *and* F23, note that F22 uses `Overall.pdf` (28 MB, cropped) while F23
uses `zeiler_fig2.pdf` (25 MB) — that's ~53 MB of the final PDF's size for one figure
shown twice.

### 5. Cosmetic: swapped accent colors on the tasks pipeline — `main.tex:764–806`, `881–922`

Columns 2 and 3 have their border colors crossed relative to their backgrounds and
arrows: column 2 sits on `bgGreen` but its boxes are drawn in `accAmber`, and column 3
sits on `bgAmber` with `accGreen` boxes. Columns 1 and 4 are consistent. Swap
`draw=accAmber` and `draw=accGreen` on the four `phasebox` nodes to make it match.

Also on that frame: `main.tex:863–871` redefines the eight `acc*`/`bg*` colors that are
already in the preamble. Harmless, but dead code.

### 6. Delivery notes from `Lecture Outline.pdf`

- Lightboard studio, recorded with Lily Astone (`lileaa@uw.edu`) and Jake Simpson
  (`jsamson2@uw.edu`)
- Look at the camera, not the board
- Solid color shirt, no fine patterns (they moiré on camera); black shirt needs a visible
  belt and lapel
