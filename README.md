# kimilio-links

A single static page that catches the links in KIMILIO's account emails.

**This repository is public on purpose, and it contains nothing sensitive.**
It exists only because GitHub Pages will not serve a private repository on a
free account, and the KIMILIO source is — and stays — private. Rather than make
an entire codebase public to host one HTML file, that one file lives here.

## What the page does

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

## What this page must never do

The URL it receives carries a single-use authorization code. The page therefore
makes **no request to anywhere** — no analytics, no fonts, no images, no
scripts. Anything added here would be handed that code by the browser as a
referrer. The code is useless on its own (it can only be exchanged by the app
holding the matching PKCE verifier, which never leaves the device), and that
property is the only reason routing through a page is acceptable at all.

## Editing it

Don't edit it here. The canonical copy lives in the private KIMILIO repository at
`docs/auth/index.html`, where it is covered by tests that check the URL
handling and assert the page contains no external references. Change it there,
run the tests, then copy the file across.

## Replacing it

The better long-term answer is Universal Links on a real kimilio domain, which
would open the app directly with no interstitial and would not put a github.io
address in a security email. When that domain exists, this repository can go
away: one constant in the app's `src/lib/auth/config.ts` and two entries in the
Supabase redirect allowlist are all that point at it.
