# method — how spec-recovery reads a reference

This is the reasoning procedure behind the skill. It reads a reference in **two passes** — the first
to understand it faithfully, the second to interrogate it for gaps — and turns what it finds into a
specification with an open-questions ledger. Everything below is that one procedure, in the order it
is carried out.

## Working mode

The procedure drives a model's reasoning; it does not prescribe the shape of the output. A skill
cannot give a model reasoning it does not have, so the goal is not that every model produce the same
words, but that every model produce a result that is **true and reasoned**, degrading to honest open
questions where it cannot. That is enforced by obligations of accountability, never by rules of
style:

- **Every statement is traceable** — to a line of the reference, a named related file, or a stated
  definition. Nothing is invented.
- **Every claim is reasoned, not guessed** — the two passes and the triage below are reasoning
  moves, not a template to pad out.
- **What cannot be settled is flagged, not fabricated** — marked `UNCONFIRMED` or put to the
  designer. A model that reasons more says more; a model that reasons less says less; neither says
  something false.

There are no rules here on length or wording. The section structure exists so the result reads
clearly and hands forward consistently — the anchor is the reasoning, not a house style.

---

## Pass A — Understand (state it faithfully, judge nothing yet)

Describe, in plain terms, the three things that make up any program, exactly as the reference
presents them. If Pass A is wrong, everything built on it is wrong, so keep it free of assumptions.

**1. Inputs and outputs.** The declared arguments and returns, and the **hidden** ones the signature
omits — files it loads, global or persistent state, constant tables baked into the body, values it
computes but drops. These are the most commonly missed and the most important to surface (the
per-language patterns that reveal them are tabulated at the end). Describe each by what it *means* —
what it represents and its role — not by its representation: do not pin down bit ordering, packing,
or interface form here, even where the code implies one. Those are deferred to a later stage, and
keeping to meaning holds the spec at the right level.

**2. Data structures.** Every value that holds state — scalars, arrays, tables, buffers. For each,
capture what will matter later: its element type, its size or shape, how it is indexed, and its
lifetime — does it live within one call (transient) or carry across calls (persistent state)? These
distinctions are what later decide register, memory, or wire, so record them now, by what the
structure holds and why, not by a concrete width.

**3. Algorithm flow.** The steps from inputs to outputs — the phases, the loops and what bounds
them, the branches and their conditions — and the **numeric semantics** inside the steps: sign
handling, saturation or wraparound, rounding, fixed-point scaling, and any value that depends on the
result of a previous call. These are the details most easily lost between a reference and its
hardware, so state them precisely.

**The file's place in a larger system.** A reference file is usually one part of a whole: a decoder
has an encoder somewhere, a stage has the stage that feeds it, a table has the file that generates
it. Note these counterparts and dependencies as you meet them — you are specifying one file, but
knowing what it is paired with is what keeps the next pass honest.

---

## Pass B — Interrogate (find the gaps, then close or flag them)

Go back over the same three essentials and ask whether each is complete enough to build from. Inputs
and outputs must reach *confirmed* — they are the block's boundary. Data structures and algorithm
flow may carry open items, but every one must be made visible, never silently resolved.

When something is uncertain or undefined, do not guess and do not jump straight to marking it open.
First ask **where its answer would live**, in this order:

1. **A related file?** The answer may be in another part of the system — the encoder behind this
   decoder, the file that generates a table it loads. If so, the point is a candidate to confirm
   there, not a fact here: name the file that would settle it, and read it or ask for it. A
   correctness finding you can only see half of is a candidate, never a proven defect.
2. **A known definition?** The answer may be fixed by a standard, a specification, or a domain
   convention (what a category-zero symbol means in JPEG). Resolve it from that — and where the
   definition and the code disagree, surface the conflict rather than trust either silently.
3. **Genuinely undefined?** Only if neither settles it is the point truly open. Put it to the
   designer as a single convergent question with a recommended answer, or mark it `UNCONFIRMED`,
   stating what is known and what is missing.

### Tag every open question with where it is closed

Whatever stays open goes into the specification's ledger, and each entry is tagged with the stage
that will close it — so it travels to the right place instead of being rediscovered later:

- **behavior** — what the design should do; settled with the designer now (settling it changes the
  spec);
- **reference-defect** — a fault in the source model; fixed or knowingly accepted;
- **hardware-binding** — a representational choice deliberately deferred to a later hardware stage (a
  width, an interface form, a table encoding);
- **must-define-for-hardware** — behavior left undefined that hardware cannot leave undefined (an
  illegal input, an error path, an out-of-range value).

Give each entry a status too — `open`, `resolved`, or `deferred` with a reason — so the ledger
carries its own history and a downstream stage can inherit exactly the entries tagged for it.

The ledger runs in both directions (the `pipeline` skill defines the rules): a stage may close only the
entries tagged for it, and a downstream stage that uncovers a *behavioral* question routes it back
into this spec's ledger rather than deciding it. When that happens, this stage owns the entry:
resolve it with the designer, update the spec (logging the change in the Revision log), and the
downstream stage re-reads what it depends on.

### Resolving with the designer

The first time through, the spec is *complete in structure but still carries open questions*. Do not
leave them as a silent list. Work through them with the designer in a **convergent conversation** —
one question at a time, each phrased plainly and each carrying a recommended answer drawn from the
code or the domain — and as each is answered, move it out of the ledger and into the body. The
designer reaches a trustworthy spec through a short series of small, clear decisions, never a wall of
choices at once.

---

## Working across a whole design

A reference is often not one file but a whole design — many files, each a block, forming a pipeline.
Every block deserves the same specification a single one gets, so the hardware can be built up from
the blocks, bottom-up, against specs that are already clear and connected.

For a whole design this skill **declares a schedule** and leaves how it runs to the library's
coordination contract — the `pipeline` skill — stepwise in one context, or concurrently across agents,
whichever the harness supports; the outputs are identical either way. The schedule is:

1. **Discover** — read the entry point (the driver or top function) to see what the whole design
   does, identify the blocks it is made of and how they connect, and sketch a first-version
   top-level spec. These are the software's *own* blocks (its files and functions), not a hardware
   partition invented here. This task's output determines the fan-out that follows.
2. **Specify each block** *(independent tasks — one per block)* — run this same procedure, Pass A
   then Pass B, on that one file, producing that block's spec. Because every task follows the same
   procedure, the specs come back consistent with one another whether they ran in turn or at once.
3. **Converge** *(barrier — needs every block spec)* — reconcile the specs where the blocks meet:
   does one block's output match, in meaning, the next block's input? Here the *related-file* route
   pays off — a question left open in one block is often answered by a sibling's spec, and a
   disagreement between two is a conflict to surface. Then fold the confirmed understanding back
   into the top-level spec. What is still open afterward is a genuine system-level question.
4. **Write back** *(independent tasks — one per block)* — push the settled results down into each
   block spec: status lines, closed ledger entries, cross-references. A block is never left "draft,
   pending" while the system has moved on; the whole set tells one consistent story.

If the harness cannot iterate over files at all, do not pretend: specify the one block in hand,
state that it is part of a larger design, and say which schedule tasks remain (see the `pipeline` skill on
honest degradation).

---

## Updating an existing spec

When a spec already exists and something changes — the reference is edited, a decision is settled, a
block or file is removed — do not re-derive the whole document and do not scatter change-notes
through the prose. Find the delta; rewrite only the sections it touches, so each again reads as clean
prose describing the current truth (no "(previously X)" scars); add one newest-first line to the
**Revision log** at the foot of the document saying what changed and why; and leave everything else
exactly as it was. This keeps the body always-current while the history stays in one field. It is
distinct from the ledger: the ledger tracks the state of design *decisions*, the Revision log tracks
changes to the *document*.

---

## Reference — hidden input / output patterns, by language

The signature is never the whole story. These are where real inputs, outputs, and state hide outside
it.

| Language | Where hidden I/O and state hide |
|---|---|
| MATLAB | `readtable` / `load` / `readmatrix`; `global` / `persistent`; `evalin`, `getappdata`; values computed then never returned |
| C / C++ | `extern` variables; file I/O; global lookup tables; `#define` parameters; `static` state |
| Python | `open()` / `pd.read_*` / `np.load`; module-level constants and dicts; `os.environ`; tables pulled in by import |
| Any language | anything read from state that was not passed in; implicit default values; a table treated as a constant; state that survives between calls |

## Reference — numeric semantics worth recovering precisely

State each of these as a concrete rule when it appears; they are the details most often lost between
a reference model and its hardware:

- sign reconstruction or sign extension,
- saturation versus wraparound on overflow,
- rounding mode and fixed-point scaling,
- any value that depends on the result of a previous call (cross-call prediction).
