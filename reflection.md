# Reflection

## What I built and why

For this project I built **Stack Smash Lab**, an interactive visualization of a stack-frame buffer
overflow. I chose this topic because buffer overflows are often explained with a static diagram and a
"vague warning about `strcpy`" — I wanted something a learner could *poke at*: type a string, watch it
fill the buffer, and see the exact byte where things go wrong.

## How the project deepened my understanding

Building the simulation forced me to get several details right that I had only half-understood before:

- **Stack frame layout.** I had to model precisely what sits between `buffer[0]` and the return address
  (the saved frame pointer, and optionally a canary), and in which direction `strcpy` writes. The fact
  that the stack *grows down* but `strcpy` writes *up* toward the return address finally clicked for me —
  that mismatch is the entire reason the bug class exists.
- **The NUL terminator counts.** An "exact fit" of 15 characters plus the NUL byte is safe, but 16
  characters is an off-by-one overflow. Working through that arithmetic made off-by-one bugs feel real
  instead of theoretical.
- **The return address is only used at the end.** The overflow happens during the copy, but the damage
  happens at `return`. That insight shaped the app's two-phase flow (copy, then "Execute return") and its
  most satisfying moment: watching the CPU jump to a garbage address like `0x41414141`.
- **Mitigations are mechanical, not magic.** Implementing the canary toggle — a value checked before
  return that turns a hijack into a safe abort — made modern defenses (canaries, ASLR, NX) feel concrete.

## Workflow: building with AI assistance

I used an AI pair-programmer for the whole build, in an iterative loop:

1. **Design first.** I described the learning goals and the exact stack layout I wanted (addresses,
   regions, colors) and asked for a single-file HTML/JS app with no dependencies so it would run anywhere.
2. **Generate, then verify.** After each generation pass I ran the app in a browser and tested the four
   preset scenarios by hand: safe input, exact fit, off-by-one, and full overflow — plus the canary toggle
   and the return/crash path. Reading the generated code line-by-line caught small logic issues (for
   example, making sure the byte preview counts the NUL terminator, and that the layout rebuilds correctly
   when the canary is toggled).
3. **Refine for teaching.** My later prompts were less about code and more about pedagogy: adding the
   step-by-step execution log, the live hex monitors for EBP/return address, and the "exact fit is safe"
   preset, because those are the moments where the concept lands.

The biggest lesson about the workflow itself: AI is fast at producing a working first draft, but the
educational quality came from *me* defining what a learner should notice, and from testing every scenario
myself. Explaining the concept to the tool — precisely enough that it could encode it — turned out to be
the same skill as explaining it to another person, which is exactly what the app is for.
