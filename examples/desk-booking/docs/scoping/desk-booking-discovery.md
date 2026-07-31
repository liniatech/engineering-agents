# desk-booking — Discovery

Raw material from the kickoff interview. The user's words, not yet formalized.
`product-manager` reads this to write `PRD-desk-booking-001`.

## The problem

Since the company went hybrid, employees arrive at the office with no
guaranteed desk. The current lease has fewer desks (70) than staff (120), so
people either can't find a seat or several claim the same one. It is daily
friction, felt most acutely first thing in the morning.

## Who has it

~120 hybrid employees across two floors. The sharpest pain is on the office
managers, who field the complaints when seating breaks down.

## Why now

A new, smaller lease (70 desks for 120 people) starts in ~6 weeks. Desk
overcommit becomes a guaranteed daily problem the moment it does, not a
hypothetical one.

## Today's alternative

A shared Google Sheet with no conflict checking. Nobody trusts it, so people
stopped keeping it current — it now makes the problem worse, not better.

## Success looks like

- Zero double-bookings for a given desk/day.
- >80% of in-office employees reserving through the tool within one month of
  launch.

## Constraints

- Company Google SSO for auth (no separate accounts).
- Internal-only; not exposed to the public internet.
- Live before the new lease starts — ~6 weeks.
- Low scale: ~120 users, well under 120 reservations/day.
- No PII beyond employee name and work email.

## Non-goals

- Meeting-room booking.
- A visual floor-plan editor.
- A native mobile app (web is fine for v1).

## Open questions

1. [NEEDS INPUT] Can a desk be booked for a partial day, or full-day only?
2. [NEEDS INPUT] How far in advance may someone book (rolling window)?

## Last verified
2026-07-31
