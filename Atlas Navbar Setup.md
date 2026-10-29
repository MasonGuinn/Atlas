---
title: "Atlas Navbar Setup"
type: reference
---

# Atlas Navbar Setup

The global navbar is **not** in your notes anymore. The fake callout navbar is gone — it caused the scrollbar, the wrapping, and the pill that never looked right.

The real navbar is the **Note Toolbar** plugin, with a toolbar called **Atlas Navbar**. It renders at the top of every note, sticky, as buttons — not as note content, so it cannot add bullets, scrollbars, or extra lines to your notes.

## One-time setup (2 minutes)

1. Pull this vault in Obsidian Git.
2. Settings → Community Plugins → Browse → search **Note Toolbar** → Install → Enable.
   - If Obsidian says the plugin is already there after pulling, just Enable it.
3. Reload Obsidian (or close and reopen the vault) so Note Toolbar reads its config.
4. Open any note. The navbar should be at the top: Dashboard, Learning, Networking, Systems, Programming, Cybersecurity, School.
   - The current page's button highlights, like tabs on a website.

The toolbar config is tracked in Git at `.obsidian/plugins/note-toolbar/data.json`, with **Default toolbar = Atlas Navbar** and position **Top** on desktop, tablet, and mobile, styled as buttons, centered, sticky, with a border.

## If the navbar does not appear

Do not edit notes to add a navbar back. Check, in order:

1. Settings → Community Plugins → Note Toolbar is **Enabled**.
2. Settings → Note Toolbar → **Default toolbar** is set to **Atlas Navbar**.
3. In that toolbar's settings: Position = **Top**, Styles include **Button, Center, Sticky, Border**.
4. If the toolbar list is empty (for example, installing from the store overwrote the config), recreate it — it takes 2 minutes:
   - Settings → Note Toolbar → New toolbar, name it **Atlas Navbar**.
   - Add 7 items, type **File**, icon + label + file as below, in this order:
     - home — Dashboard — `Dashboard.md`
     - book-open — Learning — `Learning/Learning.md`
     - globe — Networking — `Learning/Networking/Networking.md`
     - monitor — Systems — `Learning/Systems/Systems.md`
     - code — Programming — `Learning/Programming/Programming.md`
     - shield — Cybersecurity — `Learning/Cybersecurity/Cybersecurity.md`
     - graduation-cap — School — `School.md`
   - Set that toolbar as the **Default toolbar**, Position **Top**, Styles **Button, Center, Sticky, Border**.
   - Reload Obsidian.

## Why the Dashboard looks different now

The Tracks section is no longer a bullet list at all — each track is a heading, styled as a card. A heading cannot render a bullet dot, so that bug cannot come back, regardless of theme or snippet quirks. The Continue / Next up / Progress sections are still Dataview cards, with the second column styled as a pill (status, track, or progress count).
