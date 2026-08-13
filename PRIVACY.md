# Privacy policy — PickMyCard Chrome extension

_Last updated: August 2026_

This is the privacy policy for the **PickMyCard Chrome extension**. It lives
in this small, dedicated public repository — separate from the extension's
own source, which stays private — for one reason: a Chrome Web Store listing
needs a privacy policy URL that stays reachable by anyone, with no login, and
a page that depends on a deployment staying up is one that can quietly break.

## The short version

The extension sends nothing anywhere. There is no server, no account, no
analytics, and no network request of any kind. Everything it knows stays in
your own browser.

## What it stores

Only which cards you told it you carry, two display preferences (dollars vs
points, light vs dark), your own point valuations if you set any — those
override the published defaults for how much a currency like Chase Ultimate
Rewards is worth to you — and any category corrections you make. All of it
lives in `chrome.storage.local` on your own machine.

A category correction is the one entry that names a website. If the
extension guesses that `example.com` is a restaurant and you correct it to
groceries, that single pair — hostname and category — is saved so you do
not have to correct it on every visit. Two things are worth being precise
about:

- **Nothing is written by visiting a page.** An entry appears only when you
  yourself pick a different category, in the popup or the overlay. A site
  you never corrected is never recorded, so this is not a browsing history
  and cannot be used as one.
- **It still goes nowhere.** Like everything else here it stays in local
  storage on your machine. You can drop any entry with the back arrow that
  appears on a corrected site, and removing the extension deletes the lot.

It never asks for, and has no way to learn, a card number, a CVC, an expiry
date, or a billing address.

## Permissions, and what each is for

| Permission | What it does |
| --- | --- |
| `storage` | Remembers your card list and preferences, locally. |
| `activeTab` | Lets the popup read the address of the current tab — only at the moment you click the toolbar icon — to work out which store you are on. |
| Content script on all sites | Runs the on-page checkout overlay. This is what Chrome describes at install as "read and change all your data on all websites." |

That last permission is the broad one, so here is exactly what the script
does with it.

## What the content script reads

To decide whether a page is a checkout page, it looks at:

- The `autocomplete`, `name`, `id`, `placeholder`, `aria-label`, and
  `data-testid`/`data-qa`/`data-test` attributes of form fields that were
  already present in the page's HTML — the last few catch checkout forms
  built with a component library that skips `name`/`id` in favor of a
  visible label or a test hook. Still markup metadata in every case, never
  a typed value.
- The `src` of iframes, checked against a list of known payment processors
  (Stripe, Braintree, Adyen and similar), since those render the real card
  field in a cross-origin frame the extension cannot see into at all.
- The page's own hostname, to work out which store you are on.

To work out **what kind of business** a site is when the hostname does not
say — `momofuku.com` gives nothing away, and no keyword list will ever tell
you it is a restaurant — it also looks at the description the page publishes
about itself for search engines and link previews:

- The `@type` labels inside the page's schema.org `application/ld+json`
  blocks (`Restaurant`, `Hotel`, `Pharmacy`, and similar). **Only the type
  labels.** Everything else in the block is walked past and discarded, which
  matters on a checkout page, where such a block can also hold an order
  total or a name.
- The page's Open Graph `og:type`, the same one-word label that decides how
  a link to the page looks when it is shared.
- The hostnames — not the contents — of the scripts, frames and stylesheets
  the page loads from elsewhere, checked against a short list of restaurant
  ordering and hotel booking platforms (Toast, ChowNow, OpenTable, Resy and
  similar). A restaurant with a plain domain name almost always embeds one.

All three are metadata a page publishes deliberately, for anyone to read.
None of it is the page's visible content, none of it is stored, and like
everything else here, none of it leaves your browser.

## What it does not read

- **It never reads a field's value.** Not the card number, not the CVC, not
  anything else you type into any field on any page. It reads attributes,
  which are part of the page's markup, not its contents.
- **It attaches exactly one event listener**, for `focus`, and only so it
  knows when to show the overlay — the same moment your browser's own autofill
  suggestion would appear. A focus event carries no information about what
  gets typed afterwards. It is never `input` or `keydown`.
- **It makes no network requests.** The card database and the ranking engine
  are bundled into the extension itself, so it works fully offline. There is
  nowhere for your data to go.
- **It does not read page content**, cookies, passwords, or anything you
  type. The metadata described above is the page's own machine-readable
  description of itself, published for search engines — not the text,
  images, prices or order details a person sees on the page.
- **It cannot change what you pay with.** It shows a recommendation; choosing
  the card is always yours.

## The card recommendation itself

Computed entirely in your browser from the card list bundled in the extension.
Your wallet is never transmitted in order to rank it.

## Changes and questions

Material changes will update this file, and its history is public in this
repository's git log — you can see exactly what changed and when.

Questions or concerns: open an issue at
<https://github.com/arushiandankur-cloud/pickmycard-privacy/issues>.
