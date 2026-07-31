# Product Requirements Document — Desk Booking (PRD-desk-booking-001)

Last verified: 2026-07-31
Status: Draft for architect handoff
Source of record: `docs/scoping/desk-booking-discovery.md` (kickoff interview, verified 2026-07-31)

## Problem

The company moved to hybrid work and is moving to a new lease with **70 desks for ~120 staff** in approximately six weeks. Employees arrive at the office with no guaranteed seat: some find nothing, and several may claim the same desk. This is daily friction, felt most sharply first thing in the morning.

The current workaround is a shared Google Sheet with no conflict checking. Because nobody trusts it, people stopped keeping it current, so it now makes the problem worse rather than better.

The moment the smaller lease begins, desk overcommit stops being hypothetical and becomes a guaranteed daily problem. A trustworthy reservation tool must be live before that date.

## Users

Ranked by pain:

1. **Office managers (primary).** They field every complaint when seating breaks down. They need a system that prevents conflicts so the complaints stop, and visibility into who is booked when. There are a small number of them (exact count `[NEEDS INPUT: how many office managers / admins?]`).
2. **In-office hybrid employees (~120 total, the population; a subset are in-office on any given day).** They need to know before or on arrival that they have a specific desk, without racing anyone for it.

All users are internal employees authenticated through the company's Google SSO. There are no external or public users.

## Goals

- Guarantee that a given desk can be held by at most one employee for a given day.
- Let an in-office employee reserve a desk quickly enough that they adopt it over the old sheet.
- Give office managers confidence that seating is under control without manual mediation.
- Be live and usable before the new lease begins (~6 weeks from project start).

## Success Metrics

| Metric | Baseline (today) | Target | Measurement window |
|---|---|---|---|
| Double-bookings for a desk/day | Occurs (uncontrolled sheet; exact rate unknown, `[NEEDS INPUT: current double-booking frequency, if measurable]`) | 0 | Ongoing from launch |
| Share of in-office employees booking through the tool | 0% (no trusted tool) | >80% | Within 1 month of launch |

Notes:
- "In-office employees" for the adoption metric means employees physically present on a given day. Measuring this requires a source of truth for who was actually in-office. `[NEEDS INPUT: how is "in-office" attendance determined for the denominator — badge data, self-report, or is the target measured against bookings only?]`

## Functional Requirements

**Authentication and access**
- FR-1: The system must allow a user to sign in only through the company Google SSO identity; no separate username/password may be created.
- FR-2: The system must reject any user who is not authenticated through the company SSO and must not expose booking data to them.
- FR-3: The system must record, for each reservation, the identity (name and work email) of the employee who made it.

**Viewing availability**
- FR-4: An authenticated employee must be able to see, for a chosen date, which desks are available and which are already reserved.
- FR-5: The system must show the current state of a date's availability; a reservation made by one user must be reflected to other users viewing that date within `[NEEDS INPUT: acceptable staleness — is real-time required, or is a refresh acceptable?]`.

**Making a reservation**
- FR-6: An authenticated employee must be able to reserve one available desk for a chosen date.
- FR-7: The reservation unit is a **full day** (working assumption; see Open Questions on partial-day and booking window).
- FR-8: The system must prevent a desk from being reserved by more than one employee for the same date. A second attempt on an already-taken desk/date must fail with a clear message and must not create a conflicting reservation.
- FR-9: The system must define and enforce whether one employee may hold more than one desk for the same date. `[NEEDS INPUT: may a single employee book more than one desk for the same day? Assumed no — one desk per employee per day — pending confirmation.]`
- FR-10: The system must reject a reservation for any date outside the allowed booking window. `[NEEDS INPUT: how far in advance may an employee book (rolling window)? Full-day booking is assumed; the window is unresolved.]`

**Managing a reservation**
- FR-11: An employee must be able to view their own upcoming reservations.
- FR-12: An employee must be able to cancel their own reservation, after which that desk/date becomes available to others.
- FR-13: The system must define who, other than the booking employee, may cancel or reassign a reservation. `[NEEDS INPUT: can an office manager cancel or reassign another employee's reservation?]`

**Office-manager visibility**
- FR-14: An office manager must be able to see all reservations for a given date across both floors, including which employee holds each desk.
- FR-15: The desks that exist and are bookable must be configurable so the inventory reflects the actual 70 desks across two floors. `[NEEDS INPUT: who maintains the desk inventory, and does a desk need attributes beyond an identifier and floor — e.g., zone, accessibility, equipment?]`

## Non Functional Requirements

- NFR-1 (Scale): The system must support ~120 registered users and up to 120 reservations per day without degradation, remaining correct at the busiest booking hour. `[NEEDS INPUT: expected concurrent users during the morning peak — proposed planning assumption: up to 120 within a 30-minute window.]`
- NFR-2 (Latency): Availability views and reservation confirmations must return in under 1 second at p95 under the load in NFR-1. (Proposed target; confirm acceptable.)
- NFR-3 (Availability): The tool must be available during business hours in all office time zones, targeting 99.5% monthly uptime; overnight maintenance is acceptable. `[NEEDS INPUT: office time zones / working hours to protect.]`
- NFR-4 (Correctness under concurrency): Under simultaneous reservation attempts for the same desk/date, exactly one must succeed and all others must fail cleanly. Zero double-bookings is a hard requirement, not a target.
- NFR-5 (Security / exposure): The tool must be internal-only and must not be reachable from the public internet.
- NFR-6 (Data minimization): The system must store no personal data beyond employee name and work email.
- NFR-7 (Timeline): The system must be production-ready before the new lease start date, approximately six weeks from project start. `[NEEDS INPUT: exact go-live date.]`
- NFR-8 (Data retention): Reservation history must be retained for `[NEEDS INPUT: retention period — proposed default: keep current and prior 90 days, then purge]`.

## Acceptance Criteria

- AC-1 (SSO required): Given a person not signed in through company Google SSO, When they open the tool, Then they are prompted to sign in and see no desk or reservation data until authenticated.
- AC-2 (View availability): Given an authenticated employee on the availability screen for a date, When the page loads, Then each desk is shown as available or reserved for that date.
- AC-3 (Successful booking): Given an authenticated employee viewing an available desk for an allowed date, When they reserve it, Then it is confirmed to them and shows reserved to all other users.
- AC-4 (Double-booking prevented): Given a desk already reserved for a date, When another employee attempts to reserve it, Then the attempt fails with a clear message and no second reservation is created.
- AC-5 (Concurrent booking, exactly one winner): Given two employees submitting for the same available desk/date simultaneously, When both are processed, Then exactly one succeeds and the other receives a clear "no longer available" message.
- AC-6 (Cancellation frees the desk): Given an employee with a reservation, When they cancel it, Then that desk/date becomes available to others.
- AC-7 (Booking window enforced): Given an employee attempting to book a date outside the allowed window, When they submit, Then it is rejected with a message stating the allowed window.
- AC-8 (Manager visibility): Given an office manager viewing a date, When they open the reservations view, Then they see every reserved desk and the employee holding each.
- AC-9 (Empty state): Given a date with no reservations, When any employee views it, Then all bookable desks show available and the day reads as fully open.
- AC-10 (Fully booked state): Given a date where all 70 desks are reserved, When an employee tries to book it, Then no desk is offered and the day reads as full.

## Risks

- **Adoption / trust (dominant).** The prior tool failed because people stopped trusting it. Slowness, confusion, or one wrong seat collapses adoption. Mitigation: guaranteed correctness (NFR-4) and low latency (NFR-2).
- **Timeline.** Six weeks to a hard external deadline. Late answers to open questions delay build.
- **Demand exceeds supply by design.** 70 desks for 120 people; complaints may shift from "someone took my desk" to "I couldn't get one." `[NEEDS INPUT: is any fairness/allocation policy in scope for v1, or is first-come-first-served acceptable?]`
- **Adoption-metric denominator.** If in-office attendance can't be measured, the >80% target isn't verifiable as written.
- **No-show hoarding.** `[NEEDS INPUT: is no-show handling in scope for v1?]`

## Out of Scope

- Meeting-room booking.
- A visual floor-plan editor.
- A native mobile app (responsive web is acceptable for v1).
- Partial-day / hourly bookings (working assumption is full-day).
- Any allocation/fairness algorithm beyond simple availability, unless later confirmed.
- Integration with badge/attendance/calendar/HR systems, unless required to measure the adoption denominator.

## Open questions

1. [NEEDS INPUT] Booking window — how far in advance may an employee book? (FR-10, AC-7)
2. [NEEDS INPUT] Partial-day vs full-day booking? PRD assumes full-day. (FR-7)
3. [NEEDS INPUT] May one employee hold more than one desk per day? Assumed no. (FR-9)
4. [NEEDS INPUT] How many office managers / admin users? (Users)
5. [NEEDS INPUT] Can an office manager cancel/reassign another employee's reservation? (FR-13)
6. [NEEDS INPUT] Who maintains desk inventory; do desks need attributes beyond id + floor? (FR-15)
7. [NEEDS INPUT] Acceptable staleness for availability updates? (FR-5)
8. [NEEDS INPUT] How is "in-office" attendance determined for the adoption denominator? (Success Metrics)
9. [NEEDS INPUT] Current double-booking frequency for a baseline? (Success Metrics)
10. [NEEDS INPUT] Expected concurrent users at morning peak? (NFR-1)
11. [NEEDS INPUT] Office time zones / working hours to protect? (NFR-3)
12. [NEEDS INPUT] Exact go-live date? (NFR-7)
13. [NEEDS INPUT] Reservation-history retention period? (NFR-8)
14. [NEEDS INPUT] Fairness/allocation policy in scope for v1? (Risks)
15. [NEEDS INPUT] No-show handling in scope for v1? (Risks)
16. Confirm proposed NFR defaults: p95 < 1s (NFR-2), 99.5% uptime (NFR-3).

## Last verified
2026-07-31
