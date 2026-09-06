---
name: pyprompt-website
description: Maintaining the PyPrompt marketing site at pyprompt.github.io — the App Store support and privacy policy pages. Use this whenever editing anything under the pyprompt.github.io repo, writing or revising the privacy policy, changing the support page, answering App Store Connect's App Privacy / Data Collection questionnaire, or deciding whether a new app capability needs to be disclosed. Also use it when a PyPrompt feature ships that moves data off the device (iCloud, networking, sharing, peer-to-peer), because that is when these pages silently go stale.
---

# PyPrompt website

The site backing PyPrompt's App Store listing. Two pages carry real weight:
`support/` is the App Store support URL, `privacy/` is the privacy policy URL.
Both are things Apple checks and users read, so an inaccuracy here is a
compliance problem rather than a typo.

## The repo

```
index.html        landing page
support/index.html    → https://pyprompt.github.io/support/
privacy/index.html    → https://pyprompt.github.io/privacy/
style.css         shared by all three
```

Plain static HTML, no build step, no dependencies. Directory-style paths keep the
App Store URLs clean. Default branch is `main`; GitHub Pages serves it from the
root. Contact address throughout is `support.pyprompt@icloud.com`.

Preview a page before publishing — these are read by strangers, and a broken
layout is invisible from the source:

```sh
/opt/pw-browsers/chromium --headless --disable-gpu --no-sandbox \
  --window-size=800,1400 --screenshot=/tmp/preview.png \
  "file:///home/user/pyprompt.github.io/privacy/index.html"
```

## Before publishing a privacy policy change

**Bump the `Last updated` line, and get the date from `date`.** A policy whose
date doesn't move when its content does is misleading about when terms changed.
Run `date "+%-d %B %Y"` rather than reasoning about what day it is — the answer
has been wrong before, and file timestamps in a fresh clone reflect the clone,
not today.

**Trace every factual claim to something you can point at.** The policy makes
falsifiable statements about where data goes. Each one needs a source:

| Claim | Verify against |
|---|---|
| What capabilities exist at all | `PyPrompt/CHANGELOG.md`, per version |
| iCloud containers, app groups, push | `PyPrompt/PyPrompt/PyPrompt.entitlements` |
| Local network / Bonjour services | `PyPrompt/PyPrompt/Info.plist` |
| What a Python module actually does | the module's source in its own repo |

**Reconcile the support page.** Its FAQ makes capability claims too — the
internet-connection answer especially. When the privacy policy gains a way for
data to leave the device, that FAQ usually contradicts it within the same commit.
Two pages disagreeing on a three-page site is conspicuous.

**Ask whether the App Store privacy label still holds.** Apple's definition of
*collect* is transmitting data off the device **in a way that lets the developer
access it**. The second half is what decides it:

- User's *private* iCloud database, iCloud Keychain — developer cannot read it,
  so not collection.
- Peer-to-peer over the local network — never reaches a server, not collection.
- The app's **public** iCloud database — sits in a container the developer
  operates and can read. This is collection, and "we choose not to look" is a
  policy promise, not a technical barrier. Apple asks what you *can* access.

So a feature that writes to the public database can flip the questionnaire from
"Data Not Collected" to declaring User Content, even though nothing about the
developer's intent changed.

## Where the truth lives, and where it doesn't

`PyPrompt/CHANGELOG.md` is the best single source for what shipped in which
version. Entitlements and `Info.plist` tell you what the app is *provisioned*
for, which is not the same as what it *uses* — a container can sit dormant for
releases before a feature lands on it.

The dependency repos (`SwiftPy`, `swiftpy-console`, `SwiftPyICloud`, and friends)
are usually **not on disk** in a fresh session; they're fetched as siblings by
`Scripts/bootstrap-dependencies.sh`. Nearly every interesting data path lives in
them, not in the app target, whose four Swift files are mostly wiring.

When a claim depends on code you cannot read, **ask or fetch it — don't infer
from a changelog line.** A privacy policy that guesses is worse than one with a
gap in it, because the gap is visible and the guess isn't. This is the single
most common way to get these pages wrong.

## Traps that have actually bitten

**Blanket "does not collect any data" headlines.** They read well and age badly:
one module that writes to a shared store makes the whole document false at its
most prominent sentence. Prefer describing the paths data can take — the reader
learns more, and it survives the next feature.

**Absolute reassurances that were only true of one subsystem.** "The developer
has no access and cannot read what it contains" is true of private iCloud sync
and false of the public database. When a section covers several mechanisms, split
it rather than averaging them.

**Describing capabilities in the wrong tense.** Wording that names a specific
library implies it exists today. Phrasing that describes what code can do —
"connections it opens" — stays accurate across releases and saves an edit.

**A stale clone.** Sessions get re-provisioned and the local checkout can silently
be behind, or on an old branch. `git fetch origin main && git status -sb` before
editing; committing onto a stale base quietly reverts work that was already
published.

## What can't be done from a remote session

The agent proxy blocks repository settings writes and ref deletions (`HTTP 403`),
so enabling GitHub Pages, changing the default branch, and deleting branches are
all things to hand to the user with exact steps. Normal pushes to `main` work
fine. The proxy also blocks fetching `pyprompt.github.io` itself, so a live URL
can't be verified from here — say so instead of implying it was checked.
