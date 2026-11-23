# Skill Lab

Tools for measuring what an agent skill (`SKILL.md`) teaches beyond what the model already knows.

Each line of a skill falls into one of four outcomes:

| Without skill | With skill | Outcome | Action |
|---|---|---|---|
| right | right | `redundant` | Cut it to save tokens |
| right | wrong | `degraded` | Fix it first |
| wrong | right | `added` | Protect it |
| wrong | wrong | `still_missing` | Rephrase it |

## What's here

| Path | What it is |
|---|---|
| `skill-novelty/SKILL.md` | A Claude Code skill that runs the novelty check for you: splits a skill into atoms, probes each with and without the skill, grades strictly, and writes a report |
| `novelty-lab.html` | A standalone web page for scoring a skill by hand. Open it in a browser |
| `acme-tickets/SKILL.md` | A toy skill for a made-up company, used as the worked example |

## Use the skill

1. Copy `skill-novelty/` into `~/.claude/skills/`.
2. In Claude Code, ask: `check the novelty of my <skill name> skill`.
3. Add "rigorous" to also run the repeat (noise) and placebo (extra-context) arms.

The report is saved as `novelty-report.md` next to the skill you checked. Confirm any cut by re-running your task evals.

## Use the lab page

Open `novelty-lab.html` in a browser. Paste a skill, choose **Split into atoms**, and mark each atom Right or Wrong with and without the skill. It opens with the Acme example, pre-scored with predictions (not measured results).

## Background

Based on a conference talk on skill novelty: models already know most general concepts but miss exact details, so skills earn their tokens with specifics.
