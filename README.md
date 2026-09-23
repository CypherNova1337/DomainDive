# DomainDive

A reconnaissance walkthrough for apex domains — subdomains deliberately left out.

![license](https://img.shields.io/badge/license-CC%20BY%204.0-blue?style=flat-square)
![type](https://img.shields.io/badge/type-methodology-lightgrey?style=flat-square)

## What this is

A written method, not a tool. It walks through reconnaissance against a single
apex domain — `example.com` itself — and treats subdomains as out of scope on
purpose.

That constraint is the whole idea. Almost every recon guide starts by
enumerating subdomains, and most of the attention goes there because it's where
the easy wins tend to be. So the apex gets a quick pass and then everyone moves
on, which is exactly why interesting things survive on it.

This is the pass that doesn't move on. It works one domain thoroughly: what it
runs, what it serves, what it used to serve, and what's still reachable that
nobody has looked at in years.

## Who it's for

Anyone who has already done the broad sweep and wants to go deeper on a single
host — bug bounty hunters working a target with a narrow scope, or testers
handed one domain and told to be thorough.

Useful if you've ever finished a recon run, seen nothing, and suspected you
skimmed rather than looked.

## How it's organised

Five phases, each feeding the next:

| Phase | What happens |
|---|---|
| **Target definition** | Fix the scope and keep it fixed |
| **1 — Probing and filtering** | What's alive, what it's running |
| **2 — URL discovery** | Current and historical URLs, and content pulled from them |
| **3 — Directory and file brute-forcing** | What exists but isn't linked |
| **4 — Vulnerability scanning** | Nuclei across what the earlier phases found |
| **5 — Specialised checks** | The target-specific things a checklist can't predict |

Three principles run through all of it:

- **Iterative** — anything you discover goes back in as input. Recon is a loop,
  not a list.
- **Layered** — several tools over the same ground, because each one misses
  different things.
- **Manual analysis** — automation finds candidates; you decide what matters.
  The phases are deliberately not a single script.

## Getting started

1. Read [install.md](install.md) and set up the tooling — `httpx`, `katana`,
   `gau`, `waybackurls`, `ffuf`, `nuclei`, `whatweb`, `nmap`, plus wordlists.
2. Work through the phases in [README](#how-its-organised) order against one
   domain.
3. Feed what you find back into earlier phases and go again.

`sort.py` is a small helper for trimming an oversized parameter wordlist down to
a workable size.

## Good to know

- **It assumes some familiarity.** It explains what to run and why, not how to
  read an HTTP response.
- **The tool list will drift.** Commands here reflect the versions current when
  they were written; check a tool's own help if a flag doesn't land.
- **Subdomains are out of scope by design.** If you want subdomain enumeration,
  this is the wrong guide — and running it *after* one is the intended order.

## Authorised use

Only run this against targets you own or that are in scope for an engagement or
bounty programme you're part of. Phases 3 and 4 are active and noisy.

## License

[CC BY 4.0](LICENSE) — use it, adapt it, credit it.
