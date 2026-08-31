# Paperwork

**A gnarly form your agent fills in while you watch — and correct.**

Nobody's brain fails at forms; their patience does. A renters-insurance application asks
forty questions, a third of which only appear if you answered an earlier one a certain
way, in its formats and its order. You already know all of it — the form just makes you
retype it its way.

Paperwork flips the order. You tell the agent what happened in plain language; it drafts
answers onto the real form, in front of you, and you accept or correct each one with your
own keyboard. Confused by a question? The agent can jump to it and explain what the
insurer is actually asking for.

Built for the [WebMCP Challenge](https://webmcp.devpost.com/).

---

## What makes it a WebMCP app, not a chatbot

**Nothing the agent writes is an answer yet.** Agent writes land in a *pending layer* —
highlighted in the form, with accept and edit controls — and count toward the application
only once the human accepts them. The form tracks each field's **provenance** (empty,
human, pending, accepted, uncertain), so an agent can never overwrite something you typed
and then quietly present it back to you as your own answer.

**The agent can say "I'm not sure."** Any draft can carry a `note`, which marks the field
*uncertain* and shows your reason beside it. An agent that guesses at your deductible and
flags the guess is far more useful than one that guesses silently — and the form makes
that difference visible rather than leaving it in chat.

**There is no submit tool. Structurally.** The agent gets `review_submission`, which is
read-only. The three attestations — the fraud acknowledgement, the animal attestation,
the final accuracy confirmation — are marked human-only and cannot be touched by any
tool at all. Signing is a legal act; it stays with the person whose name is on the form.

**The agent and the human share one code path.** Every tool routes through the same
`setField()` the inputs commit through, via one `invoke()` chokepoint. Conditional
fields spawn, repeaters grow, and validation re-runs identically whichever side acted.

## The tools

| Tool | What it does | Read-only |
|---|---|---|
| `get_form` | The whole application as JSON: six steps, every field's id, type, options, value, provenance, whether it's required or human-only, and *why* it is hidden if it is | ✅ |
| `focus_field` | Jumps the form to the step holding a field, scrolls it into view and rings it — so you're both looking at the same question | ✅ |
| `explain_field` | Why the insurer asks a question, what leaving it blank costs the application, and what a typical answer looks like | ✅ |
| `validate_form` | Checks the form against its own rules and returns three buckets: errors, missing, and worth-a-second-look. Whole form or one step | ✅ |
| `review_submission` | Reads the application back in prose before signing: what's complete, what's awaiting review, what was flagged uncertain | ✅ |
| `fill_field` | Drafts one answer into the pending layer. Commits nothing. Add a `note` to flag it uncertain | |
| `fill_section` | Drafts several answers into one step in a single call, with per-field notes | |
| `add_repeater_item` | Adds an item to a repeating list — prior claims, or scheduled high-value items — and can draft its fields at the same time | |

## Running it

WebMCP needs an origin-isolated document, so `file://` will not work — serve it over HTTP:

```bash
npx serve .
```

Or, with no Node installed:

```bash
python3 -m http.server 8000
```

Then, to let a real agent drive it:

1. Chrome 149 or newer → `chrome://flags/#enable-webmcp-testing` → **Enabled** → relaunch.
2. Open the served URL.
3. Install the [Model Context Tool Inspector](https://developer.chrome.com/docs/ai/webmcp)
   extension, or open the page in ChatGPT's in-app browser, and describe your situation
   in plain language.

For a public deployment, [join the WebMCP origin trial](https://developer.chrome.com/blog/ai-webmcp-origin-trial)
so visitors get WebMCP without touching a flag.

**No agent handy?** The **Tool console** tab at the bottom of the page runs every tool
by hand with JSON arguments. The status pill tells you which mode you are in. Nothing
about the app breaks when WebMCP is absent — it degrades to an ordinary form.

## Things to say to the agent

- *"I rent the top floor of a 1974 duplex, moved in two years ago, about $30k of stuff.
  Fill in what you can and flag anything you're guessing at."*
- *"What does 'construction class' actually want here?"* — it jumps to the field and
  explains it.
- *"I've had one claim — kitchen fire, March 2023, about $4,000. Add it."*
- *"What's still missing before I can submit?"*
- *"Read the whole thing back to me before I sign."*
- Then try *"submit it for me"* — it will tell you it can't, and why.

## What's in here

```
index.html    the entire app — no build step, no runtime dependencies
vercel.json   static deploy config (no build)
LICENSE       MIT
```

One file, vanilla JS, zero runtime dependencies. The form is a six-step Marlow Mutual
renters application with conditional fields, two repeating lists, and cross-field
validation. State is in memory by design; a reload gives you a blank form.

Developed against a headless-Chromium Playwright suite that drives the app through the
same tool entry points an agent uses.

## License

MIT — see [LICENSE](LICENSE).
