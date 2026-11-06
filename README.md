# AI Solo Dev Kit

Twenty-one production failures I actually hit while shipping AI products alone — each written as **symptom → why → fix → how to prove the fix worked** — plus the 20 checks and the deploy tooling that now stop them.

**[Get the kit — $10](https://leela.kodevex.dev/kit/buy)**

---

## Why this exists

I shipped a SaaS, a content pipeline, a mobile data product and a local video stack by myself. Every entry below cost me something real — a paid user, a night, a support email that started with "why did it say done?"

None of these are best practices. They are things that looked fine until they didn't. Most of them fail *silently*, which is why reading code never caught them.

I wrote them down the way I wish someone had written them down for me: not "use a health check," but *"here is the health check that says OK while the worker is dead, and here is what to check instead."*

---

## Three of the twenty-one, in full

### Your proxy silently strips images

**Symptom.** You paste a screenshot. The model answers "I can't see an image" — or the request arrives as `[Unsupported Image]`. Text-only prompts work perfectly.

**Why.** Some routing layers and cheaper model tiers drop non-text content parts before they reach the model. Nothing errors. The request succeeds.

**Fix.** Treat vision as a separate capability, not a property of "the model." Route image traffic to a model you have verified end-to-end, and keep the route explicit in config rather than relying on a default.

**Verify.** Send an image of a random 6-digit number and ask for the number. If it can't read it, the image never arrived — no point tuning the prompt.

---

### Gateways kill long requests at a hard timeout

**Symptom.** A 60-second task works locally, returns a 504 in production after ~15s. Retries don't help; the same request always dies at the same wall.

**Why.** A reverse proxy or tunnel enforces a request timeout. It's not your server. Your logs may show the request never finished because the connection was cut mid-flight.

**Fix.** A **fast lane + task state machine**: accept the request, create a task ID, return in under 2 seconds; run the work server-side, decoupled from the HTTP connection; let the client poll an event endpoint until `done`.

This also fixes "user closed the tab and the task died."

**Verify.** Time a genuinely 60-second task **through the public URL**, not localhost. Localhost bypasses the proxy and will lie to you.

---

### Redeploys kill in-flight user tasks

Default `systemctl restart` gives processes ~10 seconds. A user mid-task loses everything, and you'll hear about it.

**Fix.** Graceful drain: stop accepting new work, wait for in-flight tasks up to a hard cap, then exit. **600 seconds** is the value that stopped the complaints. Pair it with the task state machine from the gateway entry so a restarted process can resume.

---

## What's in the download

| Folder | What it is |
|---|---|
| `01-pitfalls/` | All 21 traps, ~6,000 words, same four-part structure |
| `02-checklist/` | 20 pre-launch checks, each with an **observable** pass condition |
| `03-deploy/` | A systemd unit that drains instead of killing work; `sd_notify` watchdog; an end-to-end probe that asserts accept-time < 2s |
| `04-skills/` | 7 Claude Code skills I use daily, drop into `~/.claude/skills/` |
| `05-tools/` | `preflight_lint.py` — 6 checks (committed `.env`, secret literals, static server exposing the app root, non-ASCII `print()` on Windows, silent clipping, constant health routes), exit 1 on findings |

The traps, the checklist and the tooling are the same set: every check in `preflight_lint.py` exists because one of the 21 traps bit me first.

**$10, one-time. No subscription, no account, no upsell.**

**[Get the kit →](https://leela.kodevex.dev/kit/buy)**

---

## Who this is not for

- You want a course. This is a ZIP of documents and scripts.
- You want a framework. There is no framework here — it's stdlib Python, a shell script, and markdown.
- You have a team that reviews each other's deploys. Most of these traps are solo-dev traps: they happen because nobody else looks.

## License

Personal and commercial use by one person or one team. No redistribution or resale. One folder (`04-skills/talking-head-cleanup/`) is derived from a GPL-3.0 skill and stays GPL-3.0; the rest is original.

Questions before buying: 2528128812@qq.com
