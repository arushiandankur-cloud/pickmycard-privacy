# Privacy policy — Cardful Chrome extension

_Last updated: September 2026_

This is the privacy policy for the **Chrome extension**. It lives in a
repository rather than on a website on purpose: a Chrome Web Store listing
needs a privacy policy URL that stays reachable, and one that depends on a
deployment staying up is one that can quietly break.

## The short version

The extension sends nothing anywhere unless you switch on sync or the daily
rate index, and both are off until you turn them on. There are no analytics,
no tracking, and no third parties.

If you switch on **sync**, your offers and card list are copied to your other
devices **through a server that cannot read them**. Everything is encrypted on
your own machine first, with a key that is generated there and never sent — so
what the server holds is bytes, a record id, and a timestamp. That is a real
change from "nothing leaves", and it is why it is off by default and described
in full below.

If you switch on the **daily rate index**, the extension downloads one file of
shopping-portal rates from `cardful.xyz`, once a day. It is worth being exact
about what that is and is not. The file is the same bytes for every user and
covers every portal and every store at once; it is downloaded **whole** and
searched on your machine. The extension never asks for a particular store,
because a request naming one would tell the server what you were about to buy
— which is the fact this whole design exists not to learn. The request carries
no cookie, no credential, no card and nothing that identifies you. What the
server can see is that some address downloaded a public file, which is what a
server sees when anyone loads any web page.

## What it stores

All of it lives in `chrome.storage.local` on your own machine. In full:

- which cards you told it you carry, and which bonus categories you have
  selected on the ones that let you choose
- two display preferences: dollars vs points, light vs dark
- your own point valuations if you set any — those override the published
  defaults for how much a currency like Chase Ultimate Rewards is worth to you
- your rewards balances and your typical monthly spend per category, if you
  enter them, which are used to rank cards and nothing else
- any category corrections you make
- the bank offers you have added or scanned, and a receipt per bank saying when
  it was last scanned and how that went. These are cleaned up rather than kept
  forever: once a scan finds that your bank has stopped listing an offer, it
  leaves your lists straight away, and the record itself is deleted a month
  later — long enough that an offer which comes back is simply restored. An
  offer you typed in yourself is never swept, because no scan speaks for it;
  that one is yours to delete
- travel-portal rates read from a portal page, each with the time it was read
- a short-lived note that you asked to activate a card's bonus categories,
  which authorises one click on that issuer's own site and then expires
- with sync on: your email address, this device's sync cursor, and the vault key

There is no analytics, no error reporting, and no identifier of any kind that
is not listed above.

A category correction is the one entry that names a website. If the
extension guesses that `example.com` is a restaurant and you correct it to
groceries, that single pair — hostname and category — is saved so you do
not have to correct it on every visit. Two things are worth being precise
about:

- **Nothing is written by visiting a page.** An entry appears only when you
  yourself pick a different category, in the popup or the overlay. A site
  you never corrected is never recorded, so this is not a browsing history
  and cannot be used as one.
- **It still goes nowhere — including with sync on.** Your site corrections are
  deliberately left out of what syncs, along with your light/dark preference.
  They are keyed by the websites you visit, and that is not something we will
  copy to a server even encrypted. You can drop any entry with the back arrow
  that appears on a corrected site, and removing the extension deletes the lot.

**Your Pro licence key, if you have one.** A key is a signed string carrying a
plan, an expiry date and a random id — and deliberately nothing else. There is
no email, no name and no customer id inside it, so the extension cannot learn
who paid for it even in principle.

- **Activating a key contacts nothing.** The key is checked on your own machine
  against a public key built into the extension. There is no licence server, no
  activation call and no account to sign in to, which is also why Pro keeps
  working with no connection. The one server this extension can reach is the
  sync vault described below, and the licence never touches it.
- **The date your trial started** is stored beside it, as a plain calendar day.
- **It does not sync.** You paste your key on each computer you use, from the
  email it arrived in.

Buying is handled entirely by Stripe, and the receipt email by our fulfillment
worker. Neither is the extension, and nothing connects the two: the extension
has no idea a purchase happened, only that it is holding a key that verifies.

It never asks for, and has no way to learn, a card number, a CVC, an expiry
date, or a billing address.

## Permissions, and what each is for

| Permission | What it does |
| --- | --- |
| `storage` | Remembers your card list and preferences, locally. |
| `activeTab` | Lets the popup read the address of the current tab — only at the moment you click the toolbar icon — to work out which store you are on. If that tab is your own bank's offers page, it is also what lets the scan below reach it, for that one tab only. |
| `scripting` | Puts the offer scanner onto the bank offers page you are looking at, when you press the button that starts a scan. |
| `alarms` | Wakes the extension on a schedule to check whether an offer you added is about to expire. It is a timer and nothing else; it carries no data and reaches no network. |
| `notifications` | Shows that expiry reminder. Only ever about offers already in your own vault, and only if you leave reminders switched on. |
| Content script on all sites | Runs the on-page checkout overlay. This is what Chrome describes at install as "read and change all your data on all websites." |

Four bank sites, four shopping portals, this project's own site and the sync
endpoint appear as *optional* host permissions, which Chrome asks about
individually and only when you turn something on that needs one. None is
requested at install, and declining one costs exactly the feature that asked:

| Optional access | Asked for when |
| --- | --- |
| `global.americanexpress.com`, `secure.chase.com`, `secure.bankofamerica.com`, `online.citi.com` | You ask Cardful to walk your banks' offer pages unattended. Scanning the page you are already looking at needs none of these — that is what `activeTab` above is for. |
| `card.discover.com` | You ask it to activate a rotating card's quarterly categories for you. |
| `www.rakuten.com`, `capitaloneshopping.com` | You ask it to read a cash-back shopping portal's current rates. It reads the rates on screen and presses nothing. On a long list it scrolls down to reach the rest of it — never if you scroll, tap or type first, stopping the moment you do, and putting the page back where it was. |
| `www.aadvantageeshopping.com`, `www.skymilesshopping.com` | You ask it to read an airline shopping portal's current rates. The same reading and the same restraint as the row above; what differs is the payout, which is quoted in miles rather than cash. |
| The sync endpoint, `vbawbduxhvyfpfomulrh.supabase.co` | You switch on sync. What it sends is ciphertext, and the key that would open it never leaves your machine. |
| `cardful.xyz` | You switch on the daily rate index. One GET of one fixed file, once a day, with no cookies and no query — the same bytes everyone else downloads. It is the only address this extension contacts that is not the sync endpoint, and unlike that one it receives nothing from you at all. |

The last permission is the broad one, so here is exactly what the script
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

Once it has decided a page *is* a checkout, it also looks at:

- The `alt`, `title`, `aria-label` and `src` attributes of images whose
  attributes mention a card network, to see which networks the checkout says
  it accepts. This is how it can tell you a store does not take your Discover
  before you get to the card entry screen.
- **The order total the page is already displaying**, so it can tell you what
  a card earns on this cart in dollars instead of as a rate, and whether the
  cart clears the minimum spend on an offer you hold. It matches text like
  "Order total $87.40" against the visible text of the row containing it.

  This is the one place the extension reads text rather than markup, and it
  is worth being exact about what that means. It reads the number the page is
  showing you on screen. It does not read what you type, it keeps no record
  of the total, and the figure never leaves the page — it is used to render
  one line in the overlay and then discarded when the overlay re-renders.

## On your bank's offers page

Nothing happens on a bank's site unless you ask for it, on that visit. There
is no "connected bank" state, no timer and no background scan.

When you are signed in to your bank, on its own offers page, you can click the
toolbar icon and press "Scan this page for offers". That, and only that, puts
the scanner onto that one tab. Chrome's `activeTab` grant covers the tab you
invoked the extension on and ends when that tab goes somewhere else, so the
extension has no way back onto your bank's site afterwards without you doing
the same thing again.

What the scanner reads is the offer tiles the page is displaying: the store
name, what the offer pays, any minimum spend, the expiry date, and whether the
offer is already on your card. It also reads the card's product name with the
digits stripped out, so offers from two cards can be told apart. It reads no
account numbers, no balances and no transactions. It clicks only buttons whose
own visible name is on a short allowlist ("Add to card", "Activate" and a few
more), which is checked separately from how the button was found — so a
mismatch costs a missed offer rather than doing something else.

All of it is written to `chrome.storage.local` on your machine, and sent
nowhere unless you have switched on sync.

Separately, you can grant one of these banks standing access from the offers
page, which lets the extension open several banks in turn in a window you
watch — and, at a bank that keeps the offers already on your card on a second
page, both of that bank's offer pages rather than one. That is optional, never
requested at install, revocable from the same row, and the only thing it
changes is how the scanner gets to the page. Both pages are inside the access
you granted and neither is anything but a list of offers.

## What it does not read

- **It never reads a field's value.** Not the card number, not the CVC, not
  anything else you type into any field on any page. It reads attributes,
  which are part of the page's markup, not its contents.
- **It attaches exactly one event listener**, for `focus`, and only so it
  knows when to show the overlay — the same moment your browser's own autofill
  suggestion would appear. A focus event carries no information about what
  gets typed afterwards. It is never `input` or `keydown`.
- **It makes no network requests unless you turn on sync or the rate index.**
  The card database and the ranking engine are bundled into the extension
  itself, so it works fully offline. With both off, there is nowhere for your
  data to go. With sync on, it contacts the sync server and the only thing it
  sends is ciphertext. With the rate index on, it downloads one public file
  from `cardful.xyz` once a day and sends nothing at all — no store, no card,
  no identifier. Neither request happens on a page you are visiting, and
  neither carries anything read from one.
- **It does not read cookies, passwords, or anything you type.** The only page
  text it reads at all is the displayed order total described above, and only
  on pages it has already identified as checkouts. It reads no other text, no
  product names, no addresses, and no order history.
- **It cannot change what you pay with.** It shows a recommendation; choosing
  the card is always yours.

## The card recommendation itself

Computed entirely in your browser from the card list bundled in the extension.
Your wallet is never transmitted in order to rank it, with or without sync.

## Sync, in detail

Off by default. Turning it on asks for permission to reach the sync server, and
for an email address so your devices can find each other's data.

- **The server cannot read anything.** A 32-byte key is created on your device
  the first time you turn sync on, and is never sent anywhere. Each record is
  sealed with XChaCha20-Poly1305 before it leaves. A second device gets that key
  from a pairing code you reveal and carry across yourself — the browser and the
  phone can each show one and each read one.
- **Not even the names of the records.** Each record's name is replaced on the
  wire by a keyed hash of it, so the server stores rows it can neither read nor
  identify. Without the vault key, `wallet:card:amex-gold` and `rakuten:costco`
  are 22 characters of noise.
- **What the server can see** is your email address, how many records you have,
  which of four broad kinds each is, and when each one changed. Not what any of
  them says and not which record any of them is.
- **What is excluded even when sync is on**: your site corrections and your
  theme, for the reasons above.
- **Nobody can recover it for you.** If you lose every device you have signed
  in on, the copy on the server stays encrypted forever — we have no key, and
  there is no reset. The pairing code is the only backup, which is why the
  extension offers to copy it somewhere safe.
- **The wallet page on the website** is a third device by the same rules, not a
  way around them. It signs in to the same account and asks for the same
  pairing code, because signing in returns ciphertext and only the key opens
  it. It cannot change a record or add one: it holds no clock and no
  randomness, so it cannot seal anything to send. The one thing it can do
  besides read is delete — "Delete my data on the server" is offered there as
  well, and it removes every record the account holds. The key it is given is
  kept in that browser's local storage so you do not paste it every visit,
  which is the same trade the extension and the phone already make — and the
  reason the page carries a "Forget this browser" button, for a machine that is
  not yours.
- **Deleting it.** Three separate actions, because they do three different
  things and one of them cannot be undone.
  - *Sign out* ends the session on that device and stops it syncing. It keeps
    the vault key, so you can sign back in and carry on.
  - *Forget this vault* deletes the key from that device. If you have not saved
    the pairing code and no other device holds it, everything already synced
    becomes permanently unreadable — by you and by us alike.
  - *Delete my data on the server* removes every record this account has stored,
    and signs that device out so it does not upload them again. What is on your
    devices stays on them; it was yours before sync existed.

## This website

This policy is about the extension, which has no analytics. The website at
`cardful.xyz` is a separate thing, and it does count visits: it uses Vercel
Web Analytics, which records the page viewed, the referring site, and the
visitor's country and device type, sets no cookie, and keeps no identifier
that lasts past the day. It is not loaded on the wallet page (`/vault`), and
nothing the extension does ever reaches it.

## Changes and questions

Material changes will update this file, and its history is public in the
repository's git log — you can see exactly what changed and when.

Questions or concerns: email <keys@cardful.xyz>.
