# GymOps — Use Cases by Role

Derived directly from `entity-relationships.md` (v5). Each use case names the entities
it reads or writes, so this doc stays traceable to the data model rather than drifting
into aspirational feature-speak.

A single real person (`PERSON`) can hold multiple roles at once — e.g. a self-paying
adult member who is also a coach, or an owner who also coaches. The roles below are
*capabilities*, not job titles: assign as many as apply to a given person.

**Universal rule, no exceptions:** nobody, at any role, can force a double-booking.
`SPACE_BOOKING`, `EQUIPMENT_BOOKING`, and `STAFF_BOOKING` conflicts are rejected by the
database itself. The only paths forward are cancelling a `CLASS_SESSION` or
reassigning/removing the conflicting booking — there is no admin bypass.

**Marked `[TBD]`:** items that depend on the coach-vs-admin permission split the client
hasn't decided yet. These are listed under the role that's the *likely* candidate based
on the entities involved, not a firm assignment — don't build access control against
them until confirmed.

---

## 0. Prospect / Walk-in (no account yet)

*Someone with no `ACCOUNT`, and often no `PERSON` record at all until front desk creates
one. Not a system user in any login sense — represented purely through what staff enter
on their behalf.*

- Give contact details and a desired activity/schedule to front desk, becoming a
  `WAITLIST_ENTRY` (`subject_person_id`/`contact_person_id` created as bare `PERSON`
  rows if they weren't already in the system, `desired_course_id`, optional
  `desired_group_id`, free-text `preferred_schedule_notes`)
- Receive a `SIGNUP_INVITATION` link from staff once a spot is worked out, and complete
  registration themselves — this is what actually creates their `ACCOUNT` (if they
  don't have one), `ACCOUNT_MEMBER` link, and resulting `SUBSCRIPTION`
- Alternatively, be directly enrolled by staff without ever touching a link
  (`resolution_method = direct_assignment`) — e.g. staff sets up billing over the phone

Cannot: do anything else in the system — no login, no self-service, until/unless they
convert into an Account Holder.

---

## 1. Account Holder / Payer

*A `PERSON` who owns an `ACCOUNT` — either a self-paying adult member or a parent/guardian
paying for others.*

- Register and create their own `ACCOUNT`
- Link another `PERSON` to their `ACCOUNT` via `ACCOUNT_MEMBER` (e.g. add a child)
- Add, update, or remove a `PAYMENT_METHOD` on their `ACCOUNT`
- View `INVOICE`s and `PAYMENT`s billed to their `ACCOUNT`
- Create a `SUBSCRIPTION` (enroll) for any `PERSON` linked to their `ACCOUNT`, to a `GROUP`
- Cancel or request a plan change on an existing `SUBSCRIPTION`
- Sign a `WAIVER` — as `signed_by_person_id`, for themselves or for a linked minor
  (`subject_person_id`)
- Browse `COURSE` / `GROUP` / `CLASS_SESSION` listings (read-only) to decide what to enroll in
- **Request** an ad-hoc swap or makeup ("Juan can't come Tuesday, can he join another
  Judo group this week?") — this is a request routed to Admin, not a self-service write;
  the account holder cannot directly modify `SUBSCRIPTION`/`ATTENDANCE` for someone else's
  session slot. *(Open question, see `entity-relationships.md`: whether this ends up as
  a `WAITLIST_ENTRY`, sharing machinery with new-prospect intake, or as a separate,
  lighter-weight one-off record — a temporary accommodation isn't quite the same shape
  as a new enrollment.)*
- Receive automated billing notifications (system-generated; read-only)

Cannot: touch anyone else's `ACCOUNT`, `SUBSCRIPTION`, or `PAYMENT_METHOD`; create or
edit `GROUP`/`COURSE`/`SUBSCRIPTION_PLAN`; make or alter any `*_BOOKING`.

---

## 2. Member (subject of a subscription)

*A `PERSON` enrolled via `SUBSCRIPTION`. May or may not be the account holder — a minor
member typically has no login at all and is represented purely as data (`ATTENDANCE`
subject, `WAIVER` subject).*

- If self-paying adult: identical capabilities to Account Holder, for themselves
- If minor: no direct system actions — coach and admin act on their behalf
- **Optional, if the product ever grants a limited portal to older kids/teens:**
  - View own `SUBSCRIPTION`s and resulting schedule (`GROUP` → `CLASS_SESSION`)
  - View own `ATTENDANCE` history and any coach-logged progress notes

Cannot: manage billing, sign their own `WAIVER` if a minor, alter anyone's booking.

---

## 3. Coach

*A `PERSON` with a `STAFF` record where `role = coach`. Scope is generally limited to the
`GROUP`s/`CLASS_SESSION`s they're actually assigned to via `STAFF_BOOKING`.*

- View their own schedule: `CLASS_SESSION`s where they hold a `STAFF_BOOKING`
- View the roster (`SUBSCRIPTION` list) for a `GROUP` they coach
  - Whether they see billing status (`INVOICE`/`SUBSCRIPTION.status` with amounts) or
    only an eligibility flag (active/frozen, no financial detail) is controlled per-group
    by `GROUP_VISIBILITY_SETTING.coach_sees_billing_status`, set by Admin
- Take and edit `ATTENDANCE` for a `CLASS_SESSION` they're booked for
- Create an ad-hoc `EQUIPMENT_BOOKING` for their own `CLASS_SESSION`
  ("tomorrow we need elastic bands and kettlebells") — subject to the same
  `EQUIPMENT.total_quantity` conflict rule as everyone else; no override
- Log grading/progress notes on a `PERSON` in their roster, if the product supports it
- **`[TBD]`** Cancel their own `CLASS_SESSION` (e.g. sudden illness)
- **`[TBD]`** Request/assign a substitute coach (modify their own `STAFF_BOOKING` or
  create one for a colleague)
- **`[TBD]`** Change the `SPACE_BOOKING` for their own session (ad-hoc room change)
- **Waitlist triage (confirmed shared with Admin, per client):** review open
  `WAITLIST_ENTRY` records for courses they coach, and help work out whether a new
  `GROUP`/`CLASS_SESSION` or an existing one with room can absorb them — checking
  `SPACE_BOOKING`/`STAFF_BOOKING`/`EQUIPMENT_BOOKING` availability in the process. A
  coach can propose a resolution; whether they can *finalize* it (create the
  `SUBSCRIPTION` or send the `SIGNUP_INVITATION` themselves) vs. handing it to Admin to
  execute is part of the same still-open coach/admin boundary as the `[TBD]` items above

Cannot: create or edit `COURSE`, `GROUP`, `SUBSCRIPTION_PLAN`, or pricing; view rosters or
billing for groups they don't coach; touch `ACCOUNT`, `PAYMENT_METHOD`, or `INVOICE`
directly; override a booking conflict (no one can).

---

## 4. Admin (front desk / administrative staff)

*A `PERSON` with a `STAFF` record where `role = admin`.*

- Full CRUD on `PERSON`, `ACCOUNT`, `ACCOUNT_MEMBER` (create records, merge duplicates,
  relink a person to a different account)
- Create and manage `COURSE` and its `COURSE_REQUIREMENT` (default material needs)
- Create and manage `GROUP`: cadence, default coach, capacity, age bracket
- Configure `GROUP_VISIBILITY_SETTING` per group (what coaches assigned to it can see)
- Create and manage `CLASS_SESSION`s: schedule, cancel, adjust `capacity_override`
- Create and manage `SPACE` and `EQUIPMENT` catalogs, including `EQUIPMENT.total_quantity`
- Create, reassign, or cancel `SPACE_BOOKING`, `EQUIPMENT_BOOKING`, and `STAFF_BOOKING` —
  this is how a booking conflict actually gets resolved (cancel one side, or reassign)
- Create, manage, and price `SUBSCRIPTION_PLAN`s
- Create, cancel, or freeze a `SUBSCRIPTION` on behalf of **any** person — this is the
  actual execution path for an account holder's swap/makeup request
  ("move Juan to Thursday's group this week")
- Answer availability questions by querying `GROUP`s of the same `course_id` and their
  `CLASS_SESSION` capacity vs. current bookings — the direct answer to "can he join
  another group this week?"
- Trigger/monitor billing runs; view and retry failed `PAYMENT`s; process a manual
  `PAYMENT`; issue a refund (up to whatever threshold Owner sets)
- Manage `WAIVER` records: upload/verify `document_ref`, chase missing signatures
- Run cross-entity reports: revenue, occupancy, delinquency, churn
- Create a `WAITLIST_ENTRY` from a walk-in or phone inquiry — creating bare `PERSON`
  records for subject/contact if they aren't already in the system
- **Waitlist triage (the "sandbox"):** review all open `WAITLIST_ENTRY` records for a
  `COURSE`, and negotiate the fit — this is inherently a planning exercise, not a single
  CRUD action: cross-reference desired schedules against `SPACE`/`STAFF`/`EQUIPMENT`
  availability, and against each other, since several waitlisted people might only
  collectively justify opening a new `GROUP`/`CLASS_SESSION`
- Resolve a `WAITLIST_ENTRY` by either:
  - **Direct assignment** — create the `SUBSCRIPTION` themselves, set
    `resolution_method = direct_assignment`, `resolved_by_staff_id`, `resolved_group_id`
  - **Sending a signup link** — create a `SIGNUP_INVITATION` against a proposed `GROUP`,
    letting the contact complete their own registration and `SUBSCRIPTION`
- Track and chase expiring/unused `SIGNUP_INVITATION`s

Cannot: force a booking conflict through (no one can); by default, create/edit other
`STAFF` accounts or approve refunds above threshold — those are Owner-level, per the
role split below, unless the client says otherwise.

---

## 5. Owner

*A `PERSON` with a `STAFF` record where `role = owner`. Superset of Admin.*

- Everything Admin can do, plus:
- Create, edit, and deactivate any `STAFF` account, including other Admins and Owners
- Approve refunds/exceptions above the threshold Admin can self-approve
- View business-level financial and compliance reports across the whole gym
- Configure gym-wide settings (e.g. multi-location, if the business ever expands)

**Note on modeling gap:** the current ERD only has a coarse `STAFF.role` enum
(`coach`/`admin`/`owner`) — there's no `PERMISSION` entity giving per-action granularity.
Everything above is enforced in application logic against that one field, not in the
data model itself. That's fine for a single-gym v1, but won't scale to something like
"this admin can issue refunds, that one can't" without adding a real permissions table.

---

## 6. System (automated)

*No `PERSON` behind these — background jobs acting under a system identity.*

- Generate `INVOICE`s on each `SUBSCRIPTION_PLAN.billing_cycle`, based on active
  `SUBSCRIPTION`s tied to an `ACCOUNT`
- Attempt `PAYMENT` via the `ACCOUNT`'s `PAYMENT_METHOD` (SEPA direct debit, card, Bizum)
- Retry a failed `PAYMENT` per dunning policy; flag the `SUBSCRIPTION`/`ACCOUNT` as
  delinquent if retries are exhausted
  *(note: `SUBSCRIPTION.status` currently only has `active/frozen/cancelled` — a
  `delinquent` or `payment_failed` state is likely needed and isn't in the model yet)*
- Prefill a new `CLASS_SESSION`'s default `EQUIPMENT_BOOKING`/`SPACE_BOOKING` from
  `COURSE_REQUIREMENT` and `GROUP` defaults
- Enforce booking exclusion constraints at write time (rejects overlapping
  `SPACE_BOOKING`/`STAFF_BOOKING`, or `EQUIPMENT_BOOKING` exceeding `total_quantity`) —
  this is the mechanism behind "no one can override a double-booking," not a role
- Send automated notifications to `ACCOUNT.billing_email` (new invoice, failed payment,
  renewal reminder)
- Deliver a `SIGNUP_INVITATION` link to the `contact_person_id`'s email/phone once staff
  creates it, and mark it `expired` if `expires_at` passes unused — what happens to the
  parent `WAITLIST_ENTRY` at that point is still an open question (see below)

---

## Open items carried over from the ER model

These use cases inherit the same unresolved questions noted in `entity-relationships.md`:

- The `[TBD]` coach actions above (cancel own session, reassign coach/room) depend on
  the client's still-pending answer on the coach/admin decision boundary.
- Whether an ad-hoc attendance/makeup requires a full `SUBSCRIPTION` change or can stand
  alone on `ATTENDANCE` affects whether "Account Holder requests a swap" ends in a new
  `SUBSCRIPTION` or a lighter-weight one-off record.
- No `PERMISSION` entity exists yet, so Admin/Owner boundaries above are a proposed
  default, not something enforceable at the data layer today.
- Whether a coach can finalize a waitlist resolution (create the `SUBSCRIPTION` /
  send the `SIGNUP_INVITATION`) or can only propose one for Admin to execute is the
  same open coach/admin boundary question, now extended to waitlist triage.
- What happens to a `WAITLIST_ENTRY` when its `SIGNUP_INVITATION` expires unused —
  auto-revert to `open` for re-triage, or does it require a human to notice and act?
- Whether the existing-member "one-off swap" scenario belongs in `WAITLIST_ENTRY` at
  all, or needs its own lighter-weight entity, per the note under Account Holder above.
