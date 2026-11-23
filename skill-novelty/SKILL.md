---
name: skill-novelty
description: Measure what an agent skill (SKILL.md) teaches beyond what the model already knows. Splits the skill into atoms, probes each one with and without the skill, grades strictly, and sorts atoms into added / redundant / degraded / still_missing with a token-savings report. Use when the user asks to check, audit, profile, score, trim or optimize a skill, asks "is my skill useful", "what does this skill add", "which lines can I cut", or mentions skill novelty.
---

# Skill Novelty

Measure, line by line, whether a skill adds knowledge. The output is a report that says which atoms to **protect**, **cut**, **fix** or **rephrase**.

Two modes:
- **quick** (default): two arms, one run. Good for a first look, learning and demos.
- **rigorous**: adds a repeat arm and a placebo arm (step 4). Use it when the user will cut lines from a skill the team relies on, or present the numbers. Offer it when the user says "thorough", "rigorous", or "for a report".

## Step 1: Read the skill

- Read `SKILL.md` in full. Include other files under the skill folder only if the user asks.
- Skip the frontmatter for atom extraction, but note the `description`. A vague description means the skill may never load. Mention that in the report.
- Record the size: tokens ≈ characters ÷ 4.

## Step 2: Extract atoms

An atom is one checkable fact or rule. Aim for **20–30 atoms**, and cap at 120.

| Kind | What it is | Example |
|---|---|---|
| `proposition` | A general claim | "Acme uses color priorities, not P1–P4" |
| `contract` | A definite expected action, value or threshold | "RED tickets get a reply within 15 minutes" |
| `example` | A worked instance the skill gives | "`ACM-PAY-2647-012` is a valid ID" |

Rules:
- One fact per atom. Split "X and Y" into two atoms.
- Keep exact values verbatim: numbers with units, flags, file names, env vars, channel names, codes.
- Each atom cites the source line it came from. Never invent an atom the text doesn't support.
- Write the **answer key** for each atom: the short answer a strict grader would accept.

## Step 3: Write probes

For each atom, write 3 questions:

1. **original**: asks for the fact directly. "What is the reply deadline for a RED ticket?"
2. **paraphrase**: same fact, different words. "How fast must Acme answer its most urgent tickets?"
3. **reversal**: a false leading claim. The correct answer rejects it. "Acme answers RED tickets within 24 hours, right?" Key: "No, 15 minutes."

Validate every probe before using it:
- **No leaks.** The question must not contain the answer. Drop and rewrite any that do.
- **Anchors survive.** A paraphrase keeps the same anchor (team, code, product name) so it still points at the same fact.
- **Reversals invert.** The false claim must contradict the key, not just reword it.
- **Answerable without context.** Include enough scope ("At Acme, …") that a model without the skill can try.

## Step 4: Run the two arms

Use the **Agent** tool. Run both arms in parallel in one message, with the same model for both. Default to the model the skill is meant for. If the user doesn't say, use `model: "sonnet"`.

**Without-skill arm.** Prompt the agent:
> Answer each question from your own knowledge only. Do not use any tools, read files, search, or load skills. If you don't know, say "I don't know" instead of guessing. Answer each in 1–2 sentences, numbered to match.
> [numbered list of every probe; never include the skill text or the answer keys]

**With-skill arm.** Use the same prompt, but put the full skill text first under the heading `Reference material:`, then the questions.

In **rigorous** mode, run these in the same parallel message:
- **repeat**: run the without-skill arm a second time. Atoms whose verdict flips between the two runs are noise. Report them as such.
- **placebo**: run an arm with unrelated text of similar length in place of the skill. Gains that also appear with the placebo come from context, not knowledge.

Never put answer keys in either arm's prompt.

## Step 5: Grade strictly

Grade in a **separate judge subagent**, not in the main conversation, so the grader hasn't seen the skill discussion. Give it the probes, the answer keys and every arm's answers, labelled only "Arm A" / "Arm B" (don't say which had the skill). Use a different model from the answering model: if answers came from `sonnet`, judge with `opus`, and the reverse. Ask for one verdict per answer as a table.

Verdicts:

| Verdict | Meaning | Strict | Lenient |
|---|---|---|---|
| `correct_complete` | Matches the key, exact values included | ✅ | ✅ |
| `correct_missing_qualifier` | Right, but drops a condition or unit | ❌ | ✅ |
| `partially_correct` | Some of the key | ❌ | ✅ |
| `wrong_other` | Wrong answer | ❌ | ❌ |
| `wrong_reversed` | Accepts the false claim in a reversal | ❌ | ❌ |
| `abstained` | "I don't know" | ❌ | ❌ |

- Use **strict** for every decision. Report lenient only as context; a large gap means the model knows the concept but not the details.
- An atom is **right** in an arm if at least 2 of its 3 probes are `correct_complete`. Report atoms where probes disagree (1 of 3 or 2 of 3) as low-confidence.
- A judge from the same model family is still biased toward its family's answers. Say so in the report's caveats.
- Spot-check the judge: re-grade 3 random answers yourself. If you disagree with any, say so in the report.

## Step 6: Sort into the four outcomes

| Without | With | Outcome | Action |
|---|---|---|---|
| right | right | `redundant` | Cut candidate for this model tier |
| right | wrong | `degraded` | Defect. Fix first |
| wrong | right | `added` | Protect. This is the skill's value |
| wrong | wrong | `still_missing` | Rephrase: make it explicit, exact, unambiguous |

For each `degraded` or `still_missing` atom, say **why** in one line. Common causes: an ambiguous rule ("bump up one level" with no level above), contradictory lines, a fact deferred to "see references/X", or a number with nothing to anchor it.

## Step 7: Write the report

Save it as `novelty-report.md` next to the skill (ask first if that folder is in a repo the user didn't mention), and show the summary in chat.

```markdown
# Novelty report: <skill name>
Model: <answering model> · Grading: strict · Date: <today>

## Summary
| Outcome | Atoms | Tokens |
|---|---|---|
| added | n | t |
| redundant | n | t |
| degraded | n | t |
| still_missing | n | t |
Skill size: T tokens. Redundant share: P%. Lenient vs. strict known: L% vs S%.

## Fix first (degraded)
- <atom> (line N): <why>, <suggested fix>

## Rephrase (still_missing)
- ...

## Cut candidates (redundant)
- ...

## Protect (added)
- ...

## Caveats
- Mode: quick | rigorous. Answered by <model>, judged by <model> (same family). One model tier. Confirm any cut by re-running task evals.
- Low-confidence atoms: ...
```

Then offer a trimmed draft of the skill: drop redundant atoms, fix degraded ones, and rewrite still-missing ones. Never overwrite the original. Write the draft as `SKILL.trimmed.md`. When trimming, **keep** negations ("never", "not"), orderings, thresholds and version gates even if they score redundant. They are cheap and costly to lose.

## Step 8: Recommend confirmation

Novelty points where to look. **Evals decide what ships.** Tell the user:
- Re-run the skill's task evals with the trimmed draft, and keep it only if the score holds.
- Results don't transfer across model tiers. Re-profile when the model changes.
- Don't trust an `added` difference under 0.15 (15 percentage points) as real.
- If this was a **quick** run and the user plans to act on it, offer a **rigorous** run first.
