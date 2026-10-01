# PROJECT:ZED // ANOMALY

This is the site for my ongoing fanfiction novel, *PROJECT:ZED // ANOMALY*, set in the PROJECT
universe from League of Legends. The novel is still being written.

Rather than a plain reading page, the whole thing is built as an in-world archive. You're browsing
PROJECT Corporation's own files on asset ZED and his mission records, diagnostics, redacted internal
mail, and the chapters are filed as case records you open from an index. Something that isn't
supposed to be in those files keeps annotating them in amber.

That something is TRIXE, my OC and the centre of the story. She started as a behavioural stabilizer
installed during Zed's augmentation. Somewhere along the way she started choosing
her own words. The corporation calls that an emotional artifact and schedules its removal. The novel
never settles whether what she feels is real or a very good routine that learned care improves
performance retention.

Live at **trixelle.net**

---

## What this repo is

The site runs on [Carrd](https://carrd.co), which handles hosting, the page skeleton and the text I
edit most often, which, since this is a novel in progress, mostly means chapters. Everything
interactive is dropped in through Carrd's Embed element as raw HTML/CSS/JS.

This repo is just where I'd like to keep track of this whole project.

---

## The blocks

**A1-system** holds the design tokens, the fonts, the HUD markup, the section nav and the boot gate.
CSS and HTML only. Everything else on the site reads its colour variables.

**A2-system-script** is the engine. One shared animation scheduler that every other block registers
with instead of opening its own loop, the cursor telemetry readout, the neural load meter, and
TRIXE's voice channel in the bottom right. Also the boot gate logic.

**B-hero** is the masthead. Parallax on the wordmark, with a red ghost copy that separates as you
scroll.

**C-projects** is the chapter index. Each row expands into a dossier with the case code, support
assets, disposition, and a line from her.

**D-archive** is the CEDD document reader. Five documents in a tabbed pane. Redactions reveal on
hover. The documents are deliberately limited to what CEDD could actually know from telemetry,
checksums and debrief answers — they don't know TRIXE exists, so her amber lines read as intrusions
rather than authorial asides.

**T-theatre** is the faction and threat board. The entries from later acts stay blurred behind a
clearance gate, so the shape of the whole story is visible without spoiling it. Clicking a sealed one
denies you and gets a comment.

**U-audit-flag** is optional. One designated redaction in the archive trips a CEDD audit banner,
which TRIXE then withdraws a couple of seconds later.

**E1-identity / E2-identity-script** are the platform schematic. Scrolling the section zooms from 1×
to 190× through five nested layers — chassis, lattice, neural bus, module housing, and a fifth layer
the schematic doesn't list, drawn in amber.

**G-descent-gauge** replaces the scrollbar. Scrolling down the page is descending the City, from the
Bastion at 1240m to Lower Central at 12m, with clickable tier markers.

**H-cursors** are the custom cursors, base64'd directly into the CSS so there's nothing to host.

**J-cursor-audio** is the cursor follower and the ambient sound. Cool thing is that there are no audio files, every sound including the UI ticks and TRIXE's chime are all synthesised in the browser.

**K-purge** is an easter egg. Type `purge` anywhere.

**L-sigil** draws the PROJECT mark as four filled SVG paths and injects it into the headings, the
boot gate and TRIXE's dossier card. It inherits `currentColor`, so the same shape is corporate red in
one place and amber in another.

**M-power-on** is the CRT turn-on that plays when the boot gate clears.

**S-chapter-modal** is the chapter reader.

**F1-comms-top / N-comms-terminal / F2-comms-footer** are the comms section. There's no contact form
and the relay isn't bonded to an inbox, so anything you type is parsed and discarded.

---

## How the chapter reader works

Chapters sit as plain text in ordinary Carrd Text elements, and
`S-chapter-modal` finds them by a marker line, hides the source, and re-renders them into a popup
with full formatting.

The formatting is worked out at runtime. The reader reads each paragraph and decides which voice it's
in:

- `[SYNAPTIC LATENCY: 0ms]` — telemetry, grey mono
- `[You are stable.]` — TRIXE, amber with a left rule
- `CASE FILE: CL-CE/19-447B` — mission report, mono on a plate with red corner brackets
- `UPGRADE TO ASCEND` — doctrine billboard, bold, one line per row
- anything else — body prose

The rule that separates telemetry from her: **telemetry never punctuates.** `[END OF LOG]` is the
system. `[YOU ARE STABLE.]` is her, even though the source has it in caps.

It ended up working this way because Carrd's code editor has a size ceiling somewhere between 14k and
19k characters, and my chapters run 19k–29k. Pasting them as HTML meant splitting every chapter
across two or three embeds, which was miserable to maintain. Text elements have no such limit.

`chapters.py` is the converter that turns my source PDFs into those text files. It rebuilds
paragraphs from the PDF's hard line wrapping, so what's on the site is character-identical to the
manuscript and I never retype anything.

---

## Things that broke

Writing these down because every one of them cost hours and none of them were obvious from the
symptom.

**1.** One embed lost its closing `</script>` and the
entire bottom half of the site went dead. The HTML parser doesn't respect embed boundaries, so it
swallowed the rest of the document as JavaScript. 

**2.** Carrd puts transforms on sections for its
entrance animations, and any ancestor with a transform, filter or `contain` makes `fixed` position
against that element instead of the viewport. Every fixed layer on the site now moves itself to
`<body>` on load to get out from under it.

**3.** A `z-index: 9999` overlay nested in a
transformed section still loses to the next section down. It stayed perfectly visible and stopped
receiving clicks, which looks identical to a dead button.

**4.** A banner pushed off-screen with
`translateY(-100%)` is still in the layout, still painted and still eating clicks. That one broke the
archive tabs on narrow screens for a looooooooong time.

**5.** `.rd` was the archive's redaction class and I reused it for
the chapter reader, which repainted every chapter as a solid white bar...

**6.** The two biggest performance wins were deleting a `mix-blend-mode`
on a full-screen overlay and a `backdrop-filter` on the modal scrim. Both force the browser to
recomposite the whole page every frame forever, regardless if anything moved or not. Reading a chapter
went from 46fps to 100fps once those were gone and the background animations paused behind the modal.

**7.** During one refactor I left two `count()` functions
in the same scope. The later one wins in JavaScript, so the live code was the broken one I thought
I'd replaced, throwing on every click. Worth checking after any large edit.

---

## Assets

`assets/` has the processed art — the PROJECT sigil as transparent PNGs, a cutout of the jump splash
with the white background and the speed beam keyed out, and the wireframe recoloured to red on
transparency. `paste-into-carrd/` has the chapter text files.

I didn't use any official Riot art, not because I couldn't get it working with carrd embeds or anything...

---

## Credits

*PROJECT:ZED // ANOMALY* is non-commercial fan fiction. League of Legends and PROJECT: Zed belong to
Riot Games, and this isn't endorsed by or affiliated with Riot.

TRIXE, and the story built around her, are mine. Everything she does on that site is unauthorised :>)