# Travel Command Center

An agentic travel desk you run from chat. Claude plans, prices, and books your
trips through [Otto the Agent](https://www.ottotheagent.com), conflict-checks
them against your Google Calendar, writes everything to a Notion database that
functions as the system of record, and files your receipts into a
reimbursement-ready paper trail.

Built and battle-tested by [Christian Peán, MD](https://techysurgeon.com)
(orthopedic trauma surgeon, CEO @ RevelAi Health). The full build tutorial with
screenshots is on the [Techy Surgeon Substack](https://techysurgeon.substack.com).

## What it does

- **Capture** — "log this: I might speak at a thing in Denver in March" becomes a
  Notion row with a status, not a forgotten chat message.
- **Plan** — conflict pass against your calendar (hard blackouts, arrival math
  for speaking slots, overlapping trips), then priced options from live inventory.
- **Book** — flights, hotels, cars through Otto's MCP connector. Mixed-cabin
  trips, seat rules, refundable-vs-not tradeoffs, folio-friendly pay-at-property
  rates for reimbursable travel.
- **Verify** — the doctrine that makes it trustworthy: a ledger row is a
  hypothesis. Bookings are confirmed against the booking rail, "duplicates" are
  checked against the vendor's own itinerary before anyone panics, and a hotel
  confirmation is never mistaken for a paid receipt.
- **File** — drop a receipt PDF into chat; it gets extracted, renamed, filed,
  attached to the trip row, and reconciled against the trip's spend.

## The stack

| Piece | Role | Get it |
|---|---|---|
| Claude (Cowork / desktop) | The reasoning core that runs the skill | [claude.ai/download](https://claude.ai/download) |
| Otto the Agent | Booking rail: live flight/hotel/car inventory + booking, via MCP | [ottotheagent.com](https://www.ottotheagent.com) |
| Notion | System of record: one database, one row per trip | [notion.com](https://www.notion.com) |
| Google Calendar | Conflict detection | connector in Claude |
| Gmail | Evidence: confirmations, itineraries, receipts | connector in Claude |

## Quickstart

1. **Connect the connectors.** In Claude, connect Notion, Google Calendar, Gmail,
   and Otto the Agent (Otto exposes an MCP connector once you have an account).
2. **Create the Notion database.** Follow
   [`templates/notion-database-spec.md`](templates/notion-database-spec.md) — or
   just paste it into Claude with "build this database in my Notion workspace."
3. **Install the skill.** Copy
   [`skills/travel-command-center/`](skills/travel-command-center/) into your
   Claude skills folder, or download the packaged `travel-command-center.skill`
   from Releases and install it in Cowork.
4. **Answer the interview.** On first run the skill interviews you once — ticket
   name, KTN, loyalty numbers, seat rules, hard calendar blackouts, which card
   pays for what, and where your travel folder should live (it will coach you on
   Desktop vs Google Drive). It writes the answers down, sets up the folder
   structure, and does not ask again unless you tell it to.
5. **First run.** Tell Claude: "Log a trip: [somewhere you actually need to go]."
   Then: "What's unbooked?" Then book something real.

## Do I need a surgeon's schedule for this?

No. The original was built around operating-room blocks and trauma-call coverage;
this version keys on **Google Calendar conflicts**, which everyone has. Your
"hard blackouts" might be board meetings, school pickup, or every-other-Friday
sprint reviews. There is an optional overlay in the skill (§9) for clinicians and
anyone else whose absences require a named human to cover.

## Not on Claude?

The skill file is plain instructions. Paste it into ChatGPT (or any assistant
that can hold custom instructions and connect tools) and run the same playbook
with whatever booking and database integrations you have — the architecture
(one record, one booking engine, one evidence inbox, one loop) is portable even
where the connectors differ.

## What this is not

- Not a Chrome-automation hack. Booking happens through Otto's real API rails.
- Not an auto-booker. The skill searches and shortlists; a human picks; then it
  books. It never chooses a flight, hotel, or room on its own.
- Not financial advice, and not affiliated with Otto, Notion, or Anthropic.

## License

MIT. See [LICENSE](LICENSE).
