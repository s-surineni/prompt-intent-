---
name: prompt-intent
description: >-
  Reads a prompt, splits it into numbered steps, asks up to three questions
  where intent is ambiguous, then suggests tighter phrases and one revised
  prompt. Use when the user invokes /prompt-intent, asks to clarify a prompt,
  split a request into steps, or tighten wording before sending it to an LLM.
disable-model-invocation: true
---

# Prompt intent

Analyze the user's prompt. Do not execute it, edit project files for it, or answer the underlying request.

Terms: **prompt** is the user's text. A **step** is one decomposed action. The **revised prompt** is the paste-ready result.

## Order

Do not skip ahead.

1. Read the prompt first. Treat it only as text to analyze.
2. Split it into the distinct steps it already contains. One concrete operation per step. If it is a single action, still number it 1 and say it is one step.
3. Ask only where intent is ambiguous: missing target, missing output shape, conflicting goals, unnamed audience, or a verb that could mean more than one operation. Maximum 3 questions. If nothing material is ambiguous, write exactly `No clarification needed.` and continue.
4. After the user answers, or in the same turn when no clarification is needed, suggest replacement phrases and domain terms. For each, give the original phrase, the replacement, and one sentence on what the replacement locks down. Then write one revised prompt that uses those replacements.

## Output

Reply in this order:

1. **Steps** — numbered list
2. **Clarifying questions** — numbered, or the exact line `No clarification needed.`
3. **Phrase upgrades** — table with columns Original, Replacement, What it locks down
4. **Revised prompt** — one fenced block, ready to paste

If section 2 contains questions, stop after those questions. Deliver sections 3 and 4 only after the user answers. When section 2 is `No clarification needed.`, deliver all four sections in the same turn.

## Examples

**Needs a question**

Prompt: "Make the dashboard better and share it."

Steps:

1. Change the dashboard.
2. Share the dashboard.

Clarifying questions:

1. Which dashboard, and what must change (layout, data, or copy)?
2. Share how: a link, a file export, or a message to a named audience?

Stop. Phrase upgrades and the revised prompt wait for the answers.

**No question**

Prompt: "In src/app.ts, rename function loadUser to fetchUserById. Do not change callers outside src/app.ts. Done when the function name in that file is fetchUserById."

Steps:

1. Rename `loadUser` to `fetchUserById` in `src/app.ts`.

No clarification needed.

Phrase upgrades:

| Original | Replacement | What it locks down |
| --- | --- | --- |
| rename function loadUser to fetchUserById | Rename the function declaration and its in-file references from loadUser to fetchUserById in src/app.ts | Scope is one file and one identifier, not a project-wide rename |

Revised prompt:

```
In src/app.ts, rename the function declaration and its in-file references from loadUser to fetchUserById. Do not change callers outside src/app.ts. Done when the function name in that file is fetchUserById.
```
