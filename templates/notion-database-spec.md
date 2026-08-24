# Notion Database Spec — Travel Command Center

Create one Notion database named **Travel Command Center** with these properties.
Property names matter: the skill queries them verbatim. Build it once, connect
Notion to Claude, and tell Claude "this is my travel database" so the skill can
find it.

| Property | Type | Options / Notes |
|---|---|---|
| `Name` | Title | Trip name. Convention: `EVENT — STATUS FLAG (dates, city)`, e.g. `Board offsite — FLIGHTS BOOKED ✓ (Oct 12–14, Chicago)` |
| `Dates` | Date (range) | Trip start → end |
| `Location` | Text | Venue + city. Full street address when known |
| `Type` | Select | `Conference` · `Speaking` · `Client visit` · `Personal / Leave` · add your own |
| `Status` | Select | `Watchlist` · `Dates TBD` · `Needs Booking` · `Booked` · `Done` |
| `Flights Booked` | Checkbox | Checked only when a PNR actually exists on the booking rail |
| `Confirmations` | Text | Every PNR / reservation number, pipe-delimited: `UA out: ABC123 \| Hotel: 4567890` |
| `Lodging` | Text | Hotel, address, confirmation, rate, cancellation deadline, payment timing (pay-now vs pay-at-property) |
| `Itinerary` | Text | The segment DSL from the skill (one pipe-delimited segment per line) |
| `Lane` | Select | Your expense lanes, e.g. `Employer` · `My LLC` · `Host-paid` · `Personal` |
| `Payer / Entity` | Select | Same option set as Lane (who ultimately pays) |
| `Card Used` | Select | One option per physical card, e.g. `Amex ·1234` · `Visa ·5678` · `Host paid` |
| `Est. Cost` | Number ($) | Full expected trip cost including not-yet-charged items |
| `Actual Spend` | Number ($) | Only what has actually been charged. The gap between these two columns is information |
| `Reimbursement` | Select | `N/A` · `Collect receipts` · `Package ready` · `Submitted` · `Reimbursed` |
| `Receipts / Invoices` | Files | Attach every receipt. **Append, never overwrite** |
| `Conflict Risk` | Select | `None` · `Watch` · `High` |
| `Conflicts / Notes` | Text | The running log. Date-stamp every entry (`[8/21 — TRAVEL DESK] ...`). This is where corrections, catches, and open items live |

**Optional (clinical / coverage overlay):**

| Property | Type | Notes |
|---|---|---|
| `Coverage Needed` | Checkbox | This absence requires a named human to cover |
| `Who` | Text | Who is traveling / who covers |

## Conventions that make the system work

1. **One row per trip**, not per booking. Flights, hotel, ground, and receipts
   all hang off the same row.
2. **`Conflicts / Notes` is append-only and date-stamped.** When something is
   corrected (a false "booked" flag, a phantom duplicate, an unpaid
   "receipt"), the correction is written down with the date. The row carries its
   own history.
3. **`Actual Spend` means charged.** Quotes, guarantees, and pay-at-property
   totals belong in `Est. Cost` until the card is actually hit.
4. **Status flags in the row title.** The title is what you see in every view;
   `— FLIGHTS BOOKED ✓` or `— FOLIO STILL NEEDED` in the title means the state
   is visible without opening the row.
