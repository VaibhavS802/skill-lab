---
name: acme-tickets
description: Triage and format support tickets for Acme Corp. Use when the user asks to create, classify, prioritize, or escalate an Acme ticket.
---

# Acme Ticket Triage

## Ticket ID format
- IDs look like `ACM-<TEAM>-<YYWW>-<NNN>`, e.g. `ACM-PAY-2647-012`.
- `<YYWW>` is the 2-digit year + ISO week number. `<NNN>` is a zero-padded counter that resets every week.
- Team codes: `PAY` (payments), `IDN` (identity/login), `SHP` (shipping), `GEN` (everything else).

## Priority codes
Acme uses colors, not P1–P4:
- `RED` — money is being lost or customers cannot log in. Respond within 15 minutes.
- `AMBER` — a feature is broken but a workaround exists. Respond within 4 hours.
- `GREEN` — cosmetic issues, questions, feature requests. Respond within 3 business days.
- Any ticket mentioning a customer with more than 500 seats is bumped up one color.

## Escalation
- `RED` tickets page the on-call lead via the `#acme-war-room` channel, never by email.
- If a `RED` ticket has no reply after 30 minutes, escalate to the Head of Support (role: "Duty Director").
- Tickets are never closed on a Friday after 15:00; set status to `PARKED` until Monday.

## Writing the ticket
- Be clear and concise.
- Describe the problem, steps to reproduce, and expected vs. actual behavior.
- Be polite and professional with customers.
- Summary line must start with the team code in brackets, e.g. `[PAY] Refunds stuck in pending`.
