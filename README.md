# Care Package Pickup List

A single-page tool that turns Jotform care-package CSV exports into a printable pickup checklist for one delivery date.

## Hosting it on GitHub Pages

1. Create a repo (public) and drop `index.html` in the root.
2. Settings → Pages → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`.
3. Wait a minute; the tool is live at `https://<username>.github.io/<repo>/`.

There is no build step and no dependencies to install. The only external request is the Google Fonts stylesheet; everything else is inline.

## Using it

- Drop in one or more CSV exports. Multiple forms (Prep School, Arnold Hall, …) can be loaded at once — each file becomes a "pickup location" you can filter by.
- Pick the delivery date. The number on each date chip is the total packages ordered for that date across every loaded file.
- The checklist merges each cadet into a single line, adding up packages from all of their orders, and flags any line built from more than one order.
  Identity is the **cadet's own email address**, not their phone. A parent buying for two different cadets frequently enters the same contact number on both
  orders, so keying on phone welds two unrelated people into one row — and one of them then never appears on the list at all. Email was present on every row
  of both sample exports and produced no false merges. If a future export has a blank email, that row falls back to phone, then to name.
- **Sort** by last name (the default), first name, or most packages. Sorting by last name also displays names as `Last, First`, the way a roster reads. The printout follows whatever order is on screen.
- **Print / save PDF** produces a clean checklist: checkbox, name, phone and package count, at a size you can read across a table. Print it before the pickup — paper needs no signal.

## Reminders

Two contact tools, both zero-infrastructure — no API key, no server, nothing to keep running or pay for.

**Email this list…** builds the reminder for the selected date as **two separate sends**, because a Bcc blast carries one message and cadets and purchasers need different wording:

- **Cadets** — "Your care package is ready to collect tonight", written to the person who has to walk over and get it.
- **Parents** — "The care package you ordered is ready for your cadet to collect", noting that the cadet has been emailed directly too.

Switch audiences with the Cadets / Parents chips; each has its own address list, subject and body, all copy-ready.

### Pickup window

The forms only ever state a deadline ("must be picked up ... by 7:10 PM"), which is fine for a message sent at the door but not for one sent over breakfast — a cadet would reasonably turn up at lunchtime. So the messages carry the whole window: **Prep School 6:10–7:10 PM, Arnold Hall 6:30–7:30 PM**.

The opening time defaults to one hour before the cut-off, which matches both locations today, but that is an assumption rather than something the export states. It is therefore shown as an editable field per location in the reminder panel and saved in your browser, so a changed window is a two-second correction rather than a code change. Clearing the field restores the derived default.

Both messages are written as reminders and phrase the timing relative to the day you actually send: the same button produces "ready to collect **today**" on the morning of, "**tomorrow**" the day before, "**this Thursday**" earlier in the week, and the full date beyond that. Subject lines follow (`Your care package is ready today` / `Your care package — Thursday, September 24`). Nothing needs editing when the send time moves. Neither message can name an individual, since both go to the whole list at once.

A purchaser who used their cadet's own address is counted in the cadet send only, so nobody receives both letters. The day, building and cut-off time are read out of each form's own terms text ("...picked up at Arnold Hall by 7:30 PM"), so each location gets its correct details automatically.

The panel also warns about: addresses with a mistyped domain that will bounce (checked against an exact list of common misspellings — `gamil.com`, `yahoo.con` and the like, so real domains such as `frontier.com` are never flagged), orders naming a purchaser with no email, cadets who ordered for themselves and so have no purchaser, and lists large enough to hit your mail provider's per-message recipient cap.

**Tap-to-text** — texting is **cadet-only**; parents are emailed, not messaged. Each cadet's phone number on the checklist is a link that opens your own phone's Messages app with the number and the message prefilled. Nothing sends automatically. This is for chasing the handful who have not collected by the cut-off, one at a time.

The text wording is an editable template under the **Text message** tab, saved in your browser:

| Token | Becomes |
|---|---|
| `{first}` | the cadet's first name |
| `{day}` | `today`, `tomorrow`, `this Thursday`, or `on Thursday, September 24` — worked out from the delivery date against the day you send |
| `{window}` | `6:30–7:30 PM`, the full pickup window for that cadet's location |
| `{from}` | just the opening time |
| `{place}` | the pickup point for their location |
| `{time}` | that location's cut-off |
| `{packages}` | ` You have 3 packages.` — and nothing at all when they only have one |
| `{qty}` | the bare number |
| `{name}` | the full name as entered |

A live preview shows the message as a real cadet on the current list would receive it, with a character and segment count. Editing the template updates every phone link immediately.

### Why there is no bulk-send button

Quo (formerly OpenPhone) states plainly that *"mass texting or bulk texting is not available"* — its group texting caps at 10 people in a single shared thread, where every recipient sees the others. Sending individually through the Quo API is technically possible, but their own deliverability guidance warns against *"identical messages to large contact groups"*, it needs completed US carrier registration (A2P 10DLC) plus prepaid credits, and any fallout lands on the phone number the restaurants actually run on. It would also need a server-side proxy, since an API key cannot live in a public page.

Email reaches every cadet and purchaser for free, with none of that. So email carries the bulk reminder and texting stays a manual, one-at-a-time tool for no-shows.

### Tracking

Each cadet carries three marks for each delivery date — **picked up**, **emailed**, **texted** — with a running tally above the table. These are stored in the browser's local storage, keyed by date, so they survive a reload and a closed laptop. They live on one device in one browser: they do not sync to your phone or to a colleague, and clearing site data erases them. `Clear marks` resets all three for the selected date only.

## How the package count is worked out

The `Delivery Dates` field holds the whole payment block, and one order can list several dates:

```
September 24th, 2026 (Amount: 25.00 USD, Quantity: 1)
October 15th, 2026 (Amount: 25.00 USD, Quantity: 1)
 Subtotal: 50.00 USD
 Tax: 4.66 USD
Total: $54.66
```

The tool reads that field line by line, keeps only lines that begin with a real month name and day, ignores the Subtotal / Tax / Total / Transaction ID lines, and reads the `Quantity:` on each date line. A cadet's package count for a chosen date is the sum of the quantities on that date across all of their orders.

## Built-in checks

Anything the tool is not fully sure about is listed in an amber "needs a look" panel rather than silently folded into the total:

- an order where no delivery date could be read;
- a date line with no `Quantity:` (counted as 1, and flagged);
- a date line with no year (taken from the submission date, and flagged);
- an order whose line items don't add up to its own `Subtotal` — the strongest check, since it catches a payment block the parser misread;
- the same file loaded twice (refused, so nothing double-counts);
- a cadet whose name is spelled differently across merged orders.

## Column detection

Headers are matched loosely, so the tool survives small form edits — `Delivery Dates` and `Delivery Dates:` both work, and extra columns like `Workflow: …` are ignored. If no delivery column is recognised by name, it falls back to whichever column most often contains `Quantity:`.

## Privacy

Everything runs in the browser. No CSV is uploaded anywhere.
