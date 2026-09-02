# kimilio-links

Two static pages that catch the links in KIMILIO's emails.

**This repository is public on purpose, and it contains nothing sensitive.**
It exists only because GitHub Pages will not serve a private repository on a
free account, and the KIMILIO source is — and stays — private. Rather than make
an entire codebase public to host two HTML files, those two files live here.

## What the pages do

KIMILIO's email confirmation and password-reset links used to point straight at
`kimilio://auth-callback`, the app's own URL scheme. A phone with the app
installed handles that. A computer does not: macOS Safari answers *"Safari
cannot open the page because the address is invalid"*. Most people read email on
a computer, so most people met an error in the middle of getting into their
account, with nothing on screen to explain it.

Supabase can only redirect to a URL, so the emails now point here instead.
`auth/index.html` looks at the device and either hands off to the app (phone) or
explains what to do (anything else). It never fires the app scheme on a desktop,
because doing that is what produces the Safari error in the first place.

`connect/index.html` does the same job for connection invites, which had the
identical defect and were fixed later. That one matters more than it looks: an
invite goes to somebody who does *not* have the app yet, and it is the first
thing KIMILIO ever shows them.

**They are two separate pages on purpose.** One page taking a "which screen
should I open?" parameter would be an open redirect into the app's own URL
scheme — anyone could email a link that opened any route in the app. Each page
hardcodes the single path it forwards to. The duplicated shell is the price of
that, and of the no-external-requests rule below, which rules out a shared
script file.

## What these pages must never do

The URL it receives carries a single-use authorization code. The page therefore
makes **no request to anywhere** — no analytics, no fonts, no images, no
scripts. Anything added here would be handed that code by the browser as a
referrer. The code is useless on its own (it can only be exchanged by the app
holding the matching PKCE verifier, which never leaves the device), and that
property is the only reason routing through a page is acceptable at all.

## Editing them

Don't edit them here. The canonical copies live in the private KIMILIO
repository at `docs/auth/index.html` and `docs/connect/index.html`, where they
are covered by tests that check the URL handling and assert the pages contain no
external references. Change them there, run the tests, then copy the files
across.

A scheduled job in that repository fetches both live pages every day and fails
if either one 404s or differs by a single byte from its tested copy — so a
forgotten copy across is noisy rather than silent.

## Replacing them

The better long-term answer is Universal Links on a real kimilio domain, which
would open the app directly with no interstitial and would not put a github.io
address in a security email. When that domain exists, this repository can go
away: one constant in the app's `src/lib/auth/config.ts` and two entries in the
Supabase redirect allowlist are all that point at it. (The invite link needs no
allowlist entry — it goes in an email KIMILIO sends itself, not a Supabase
redirect.)
