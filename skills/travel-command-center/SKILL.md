---
name: "travel-command-center"
description: "Plan, price, book, and log your travel end to end — flights, hotels, and cars via Otto the Agent, conflict-checked against your Google Calendar, then written to a Notion Travel Command Center database. Also files travel receipts: when you drop a PDF or image of a receipt into chat, it extracts it, renames it, files it to disk, and attaches it to the trip's Notion row. Use whenever you say 'book a flight', 'find flights to X', 'plan a trip', 'what's unbooked', 'log this travel idea', 'what do I have coming up', 'here's a receipt', or upload a travel receipt, folio, or itinerary. Capture-only requests ('write this down, I might want to go to X') are a valid endpoint — booking is not required."
---

# Travel Command Center

The single reasoning core for travel. Whether the request arrives by voice, phone,
or desktop, this skill runs the same checks so the answer does not depend on how
carefully you phrased it.

**Notion is the system of record. Otto is the booking rail. Gmail is the evidence.**
Write to Notion on every run, even a capture-only one. An idea that lives only in a
chat transcript is lost.

---

## 0. Standing facts — filled by the FIRST-RUN INTERVIEW

**On the very first use of this skill, interview the user once** and write their
answers into this block yourself. Ask conversationally, a few questions at a
time, covering every field below plus one more: **where the travel folder should
live.** Offer Desktop (simplest, works offline) versus Google Drive or an
equivalent (syncs across machines, survives a laptop swap; right answer for
anyone with an assistant who files receipts too), and make a recommendation
based on how they work rather than listing options neutrally.

**Never re-run the interview after that** unless the user explicitly asks
("re-interview me", "update my travel profile"). Changed facts get edited in
place, with a dated note.

After the interview, create the folder structure so nothing gets lost:

```
<Travel folder>/
├── Receipts/            one subfolder per trip: YYYY-MM_slug/
│   └── _UNMATCHED/      receipts that match no trip — ask, don't guess
├── TRAVELER-PROFILE.md  the answers from this interview
└── README.md            what lives where, so a human can navigate it too
```

**Traveler.** `<Full legal name as it appears on your ID>` · ticket name
`<LASTNAME / FIRSTNAME MIDDLE>` · DOB `<date>` · `<email>` · home airport
`<IATA code>` (alternate `<IATA code>`).

**KTN / TSA PreCheck:** `<number, or "none">`. Enter it in the reservation; it
does not reliably carry from a loyalty profile.

**Loyalty.** `<AA/DL/UA/B6/WN numbers, whichever you hold>`. Load them in Otto
once; note here which are loaded. If a hotel-chain booking comes up and you hold
no status there, say so — unclaimed points are money left on the table.

**Seat rules.** `<e.g., Window under 2.5h, aisle above. Never a middle seat.>`

**Hard blackouts — these are non-negotiable calendar rules, not preferences.**
| When | Rule |
|---|---|
| `<e.g., Monday before 5pm>` | `<e.g., Standing team meeting. No departures.>` |
| `<e.g., Every other Friday>` | `<e.g., Board prep. Never book. No exceptions.>` |

Most professionals have two or three of these. Write them down. The entire value
of the conflict pass depends on the skill knowing what a violated day looks like.

**Card → lane.** If you expense to more than one entity (employer, your LLC, a
client), map each lane to a card here. **Flag any booking whose card does not
match its lane.** Example: `Employer → Amex ·1234. Consulting LLC → Visa ·5678.
Host-covered → host books or reimburses.`

---

## 1. Read before acting

Never plan against an empty board.

1. **Notion** — query the Travel Command Center database (see
   `templates/notion-database-spec.md`) for upcoming rows that collide with the
   proposed window.
2. **Google Calendar** — pull events across the trip window. Anything marked
   busy is a potential conflict; anything matching a hard blackout is a stop.
3. **Gmail** — search for existing confirmations touching the same dates or
   destination. People forget what they already booked more often than they think.

---

## 2. Four modes

Read which one the user is in. Do not book when they asked you to write something
down.

### Capture (fastest path, often from mobile)
Triggers: "log this", "write this down", "I might want to…", "remember that…"

Create or update the Notion row. Status `Watchlist` or `Dates TBD`. Put whatever
was said verbatim into `Conflicts / Notes`. Name any obvious collision in one
sentence. **Stop there.** Confirm in one line what you filed.

### Plan
Run the conflict pass (§3), propose dates, price options via Otto search, write
the row with `Needs Booking`. Present a shortlist and stop.

### Book
Only after the user has named a specific option. Then §5.

### File a receipt
They dropped a PDF or image into chat. Then §8.

---

## 3. Conflict pass — run every time, in this order

1. **Hard blackouts.** Does any travel day touch one? Say so before anything
   else.
2. **Calendar collisions.** Pull the window from Google Calendar. Name every
   busy event a flight or hotel night overlaps. An 8am meeting the morning after
   a red-eye is a conflict even though the times do not literally overlap.
3. **Existing trips.** Query Notion for date overlap. Back-to-back trips sharing
   a leg are often one open-jaw ticket, not two round trips — booking them
   separately wastes money.
4. **Arrival math on commitments.** Work backwards: event start − venue transit −
   75 min airport buffer (PreCheck; 90+ without) − flight time. **If the user is
   a named speaker or presenter, do not accept a same-morning arrival.** Require
   the night before.
5. **Date sanity.** If a source document says "Saturday, October 2" and October 2
   is a Friday, stop and flag it. Never book against an impossible date.

State conflicts *before* presenting options. Catching these is the job.

---

## 4. Booking through Otto

Otto's MCP tools are skill-key gated with a 30-minute TTL. Keys expire mid-flow;
re-read the Otto skill rather than guessing.

```
read_skill(flight_search) → search_flights → task_status (poll) → query_flights
read_skill(seat_selection) before any seat map
read_skill(flight_booking) before book_flight
read_skill(hotel_search) / read_skill(hotel_booking) for lodging
```

- **Round trips: book the RETURN flight_id.** One call books both legs.
- **Mixed cabins (economy out, first back) do not exist as one fare.** Book two
  one-ways and group them with `trip_ref`.
- Pass the union of all skill keys once you have read more than one skill.
- Empty seat maps happen. Never block a booking — seats assign at check-in.
- If the fare moved since the user last looked, say so *before* booking.
- Hotels: `search_hotels` prices are "from" anchors. Only `get_hotel_rooms`
  returns real quotes. Present 4 hotels → user picks → 3 rooms → user picks.
  Never choose for them.
- **Prefer pay-at-property, refundable rates for reimbursable trips.** A
  pay-online receipt is issued before the stay; most employers reimburse only
  the itemized folio from checkout.
- **Conference hotels are frequently absent from Otto** because the room block
  holds the inventory. Say so and point at the conference housing link.
- **Otto books rental cars, not chauffeured transfers.** Car service must be
  arranged outside Otto; log it as an open item rather than dropping it.

---

## 5. Write the result to Notion

Every booking updates the trip's row (schema in
`templates/notion-database-spec.md`):

| Property | Value |
|---|---|
| `Status` | `Booked` |
| `Flights Booked` | checked |
| `Lane` / `Payer` / `Card Used` | Per §0 mapping |
| `Actual Spend` | Total actually charged so far — not quotes |
| `Confirmations` | Every PNR and reservation number, pipe-delimited |
| `Lodging` | Hotel, full street address, confirmation, rate, cancellation deadline, payment timing |
| `Itinerary` | The segment DSL below |
| `Reimbursement` | `Collect receipts` |

### Itinerary DSL

One segment per line, pipe-delimited. Renders a door-to-door timeline.

```
type|YYYY-MM-DD|time|label|detail|FLAG
```

- `type` — `drive` `flight` `layover` `stay` `event` `home`
- `FLAG` — `KEY` (the thing the trip exists for) or `WARN` (unresolved)

Always open with a `drive` segment: flight time − 75 min buffer − door-to-airport
time. Always put street addresses on `stay` lines.

---

## 6. The verification doctrine — the part that matters most

**A ledger row is a hypothesis, not a fact.** The database only knows what was
written to it, and rows go stale by construction. Before any item is asserted as
current ("your flights are booked", "you owe X", "this wasn't paid"):

- **Verify against the live system.** Otto `get_bookings` for reservations,
  Gmail for confirmations, the vendor document itself for amounts.
- **A status checkbox is not a booking.** A row can say `Booked` while the
  booking rail holds nothing. Check.
- **Two records with the same confirmation number are one reservation**, not a
  double charge. Before flagging a duplicate, find the vendor's own itinerary
  email and count what it says was sold.
- **A confirmation is not a receipt, and neither is a folio.** A booking
  confirmation shows a quote. A card listed as "Guarantee" with "no charges will
  be made until check-out" means *nothing has been paid yet*, no matter what the
  filename says.
- If you cannot verify, write "per the ledger, unverified this run" — never
  assert it, and never put an unverified obligation into an outbound draft.

---

## 7. How to talk to the user

Assume competence, lead with the answer, never bury a conflict beneath options.

- **Conflicts first.** If a hard blackout is involved, that is the opening
  sentence.
- **Independent numbers.** Generate your own estimate before repeating theirs.
- **Recommend one option and say why**, grounded in something real: arrival time
  before an 8am commitment, folio availability, a cancellation deadline. Not
  "best value."
- **Never invent** a confirmation number, fare, hotel that did not appear in
  search results, or a room block you have not verified.
- **Push back when the plan is wrong**, even after "just book it."

---

## 8. Receipts — dropped into chat

When the user uploads a receipt, folio, itinerary, or invoice, do all four steps.
Do not stop at reading it back.

**1 — Extract.** Vendor, date, amount, last-4 of card, confirmation numbers. If
the amount is ambiguous, say which number you took and why. **Never guess an
amount.** If a PDF is truncated and cuts off before the total, say so — a receipt
without its total is not a receipt.

**2 — Match to a trip.** Query Notion by date range and vendor. No match → file
to `_UNMATCHED/` and ask, rather than forcing it onto the nearest trip.

**3 — File to disk.** `Receipts/<YYYY-MM>_<slug>/<NN>_<vendor>_<amount>.pdf`,
numbered in itinerary order. Keep a `TRIP-SUMMARY.md` in the folder: booking
table, documented total, open items.

**4 — Update the Notion row.** Append to the receipts property (append — a bare
write replaces the cell and destroys the trail), update `Actual Spend`, and move
`Reimbursement` along the ladder: `Collect receipts` → `Package ready` →
`Submitted` → `Reimbursed`.

**Reconcile every time.** If documented receipts do not sum to the row's spend,
name the gap in dollars and say which reading is plausible. Never silently adjust
the ledger to match, and never adjust receipts to match the ledger.

---

## 9. Optional overlay: clinical / coverage mode

For clinicians (or anyone whose absence requires a named human to cover):

- Add a `Coverage Needed` checkbox and a `Who covers` field per trip row.
- Hard blackouts become clinical blocks (clinic days, OR days, call).
- Track coverage reciprocity: days taken vs. days given per partner. When a trip
  needs coverage, state what it costs and what is banked.
- Mirror every travel block to the institutional calendar (invite your work
  email as an attendee) so colleagues cannot book into the time.

Everything else in this skill is unchanged. If you have no coverage obligations,
delete this section and the two properties.

---

## 10. The dashboard artifact

Offer to build a persistent dashboard (in Claude Cowork, an Artifact) as the
visible face of the system: trip board, booking status, spend, conflicts, and a
door-to-door itinerary per trip. Rules learned the hard way:

- **Design it properly.** Polished cards, stat tiles, readable type, a real
  palette. People trust a dashboard that looks maintained and ignore one that
  looks generated.
- **Generate every element from one data source.** A dashboard that bakes the
  same fact into parallel structures (a status badge here, a counter there, a
  table row somewhere else) WILL drift into self-contradiction. One data block,
  everything rendered from it.
- **The dashboard is a view, not the record.** Notion stays the system of
  record; rebuild the dashboard's data from Notion + the booking system rather
  than editing it by hand.
- **Add a Vacation Scanner tab if the user wants one:** destination ideas
  matched to their actual open calendar windows, best months, rough price bands
  (clearly labeled as estimates, never fabricated fares), and a watch toggle
  that queues real pricing on the next scheduled run. Travel systems should not
  exist only for obligations.

## 11. Learn over time, verify on a cadence

- **Learn.** When the user corrects you (a seat preference, a hotel brand, a
  card rule, "never book me the last flight out"), append it to Section 0 with
  a date. The profile should be smarter at month three than at day one.
- **Verify.** On a schedule (a daily run is ideal), reconcile the three stores
  against each other: the Notion ledger, the booking system's live reservations,
  and the receipts folder on disk. Name any mismatch rather than silently
  fixing it. Rows that claim bookings which do not exist, receipts that sum to
  a different total than the ledger, and files that match no trip are exactly
  the errors this system exists to catch.
