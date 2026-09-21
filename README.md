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
- **Sort** by last name (the default), first name, or most packages. The print and offline copies follow whatever order is on screen.
- **Print / save PDF** produces a clean checklist: checkbox, name, phone, package count, and a "Picked up by" column for initials.
- **Save offline copy** downloads the chosen date as a single self-contained file — see below.

## Working the pickup with no signal

`Save offline copy` downloads one HTML file (around 15 KB) holding just the selected date's list. Open it from your phone's downloads at the pickup table; it makes **no network requests of any kind**, so it works in airplane mode or a dead-zone building.

It keeps the parts you need while handing packages out:

- a search box, so you can find a cadet by name or phone without scrolling 130 rows;
- a tap-to-tick checkbox per cadet, with a running "X of Y packages handed out" counter;
- tick marks saved on the device, so a locked screen, a reload or a dead battery doesn't lose your place;
- tap-to-call phone numbers for cadets who don't show;
- a Print button, if you'd rather work from paper.

Download it while you still have wifi. A PDF can't do live check-off on a phone, which is why this is an HTML file rather than a PDF — print it, or use your browser's print-to-PDF, if you want paper.

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
