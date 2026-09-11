# GymOps — Project Plan

Living record of what's been researched, decided, and built, plus the next few steps.
Not a roadmap to a finished product — we plan a few steps ahead, not the whole future.

## Context

A young entrepreneur running a gym (mostly kids' classes — Judo and similar — with
parents as the paying party, plus some adult members) needs a system to manage
subscriptions, direct debit billing, parent/child account relationships, coach class
management, and automated invoicing.

## Research

- `gym-saas-market-landscape.md` — third-party AI-generated market research on gym SaaS
  platforms (global players, European/Spanish compliance landscape, regional
  competitors). Captured via browser, **flagged unverified** — pricing, regulatory
  deadlines, and fine amounts in it have not been independently checked.

## Data model

`entity-relationships.md`, now at v5, built incrementally through discussion rather than
designed upfront. Each revision is documented in the file itself; the throughline:

1. **v1–v2**: `PERSON` as one base identity for member/guardian/staff, `ACCOUNT` as the
   billing unit (not the person), `GROUP`/`CLASS_SESSION` scheduling, `WAIVER` with
   separate subject/signer references.
2. **v3**: split `COURSE` (the activity, e.g. Judo) from `GROUP` (a fixed cadence of that
   course) from `CLASS_SESSION` (one dated occurrence) — matching how the client actually
   thinks about scheduling. Added `SPACE`, `EQUIPMENT`, `COURSE_REQUIREMENT`.
3. **v4**: resource management became temporal bookings (`SPACE_BOOKING`,
   `EQUIPMENT_BOOKING`) instead of static foreign keys, so overlaps and over-allocation
   are detectable. Decided double-booking prevention is a **database-layer invariant**
   (exclusion constraints), not application logic — and confirmed with the client that
   **no override exists**: a conflict must be resolved by cancelling or reassigning, never
   forced through.
4. **v5**: added `STAFF_BOOKING` (coaches can't be double-booked either, same mechanism
   as rooms), and `WAITLIST_ENTRY` / `SIGNUP_INVITATION` for walk-ins and prospects. The
   client clarified the waitlist is a **staff working set** ("sandbox") for negotiating
   room/time/coach/equipment fit across several pending people at once — not an automated
   FIFO notify queue.

## Use cases

- `use-cases.md` — every use case traced to specific entities/fields in the ERD, by role
  (Prospect/Walk-in, Account Holder, Member, Coach, Admin, Owner, System). Items that
  depend on the still-undecided coach-vs-admin permission split are marked `[TBD]` rather
  than guessed at.
- `use-cases.es.md` — Spanish translation (entity/field identifiers left in English for
  traceability to the schema).

## UX approach

- `ux-design-principles.md` — OOUX (Object-Oriented UX) as the translation method from
  entities/use-cases into screen structure, plus a Material Design 3 reference.
- Decided **against** Balsamiq for prototyping: it needs a desktop app or paid Cloud
  account to view interactively, which is friction for the actual audience (parents,
  coaches) we want fast feedback from.
- Decided **for** web-based clickable low-fidelity mockups, published as shareable
  Artifacts (no install, no login for viewers) — deliberately kept visually ugly
  (grayscale, sketch-style) on purpose, so end users read them as "validate the flow,"
  not "review the design."

## Repository

Public repo: https://github.com/lorinc/GymOps — all docs above are committed there.

## Open questions (not yet answered by the client)

Carried from `entity-relationships.md` and `use-cases.md`, not repeated in full here:

- Coach vs. admin decision boundary (cancel own session, reassign coach/room, finalize a
  waitlist resolution) — blocks several `[TBD]` items in `use-cases.md`.
- Whether an ad-hoc swap/makeup for an existing member needs a full `SUBSCRIPTION`
  change, a lighter-weight standalone record, or belongs in `WAITLIST_ENTRY` at all.
- Waitlist/invitation mechanics: ordering vs. priority, what happens when a
  `SIGNUP_INVITATION` expires unused.

## Next steps

The agreed path forward, in order:

1. Build a complete low-fidelity clickable web mockup covering the full `use-cases.md`
   list — not just one flow. Published as a shareable Artifact, deliberately ugly, so it
   reads as flow validation rather than a design review.
2. Put the complete mockup and the use case list in front of the client together.
3. Hold a scoping session with the client off the back of that.
4. Produce an estimation.
5. Create a staged delivery plan.
6. Get the plan approved.
7. Execute it.

Each step feeds the next — no committing to delivery-plan or execution details before
the scoping session and estimation actually happen.
