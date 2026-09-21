# attest

Check what your AI support vendor billed you, against criteria fixed in advance.

Intercom bills Fin at **$0.99 per outcome**, and counts a resolution when the customer confirms
the answer worked (*confirmed*) **or leaves without asking for more help** (*assumed*) — both at
the same price ([Fin pricing: Outcomes](https://fin.ai/help/en/articles/13975800-fin-pricing-outcomes),
retrieved 2026-09-20). Zendesk bills automated resolutions from a **resolution allowance** under
the tier model it introduced on 18 May 2026, and publishes no single per-resolution list price
([About automated resolution tiers](https://support.zendesk.com/hc/en-us/articles/9570369117338-About-automated-resolution-tiers),
retrieved 2026-09-20).

In both cases the vendor's own system decides what counts. Zendesk has a language model read the
transcript to confirm the request was resolved — a real check, run on the conversation. Neither
vendor's published definition takes account of a refund three days later, or of the same customer
opening a new ticket about the same thing. That is the gap `attest` measures, from records you already hold.

**Download a binary from the [releases page](../../releases/latest). Run one closed month. See the number before you talk to anyone.**

## Install

Download the binary for your platform from the [latest release](../../releases/latest), then:

```bash
# macOS (Apple silicon)
curl -fsSLO https://github.com/BrijStream-Technologies/attest-cli/releases/latest/download/attest-macos-arm64
curl -fsSLO https://github.com/BrijStream-Technologies/attest-cli/releases/latest/download/SHA256SUMS
shasum -a 256 -c SHA256SUMS --ignore-missing     # detects a corrupted download
chmod +x attest-macos-arm64 && mv attest-macos-arm64 /usr/local/bin/attest
```

On macOS the first run may be blocked by Gatekeeper — `xattr -d com.apple.quarantine
/usr/local/bin/attest` clears it.

**Platforms.** macOS (Apple silicon and Intel), Linux x86_64 (static, musl — runs on whatever
your company standardised on) and Windows x86_64. Release binaries are built by an automated
workflow that refuses uncommitted changes; `attest --version` names the commit. The binaries are
not yet signed, so macOS and Windows will warn on first run. Check the
[releases page](../../releases/latest) for what is actually there rather than taking this list on
trust.

On Windows, SmartScreen may block the first run; choose "More info", then "Run anyway".

## Your first month

You need an export of a **billing period that has already closed**, and the vendor's
per-resolution usage export for that period — the itemised list of what was billed, which the
invoice total alone does not give you. No contract and no integration.

```bash
attest normalise --vendor intercom --source ./exports --period 2026-08
attest report    --claims claims.jsonl
```

That is the whole first run. It prints what was billed, what fails which criterion, and what is at
stake — computed **entirely on your machine**. Nothing has been sent anywhere at this point.

To show what the output looks like — not what to expect, since no pilot against a real buyer's
exports has been run: on a synthetic month of 12,000 conversations with
9,909 resolutions billed at $0.99, a ruleset derived from the standard set (with engagement-specific staff domains and bot ids) flags **1,459 claims worth $1,444.41** of
**$9,809.91** invoiced. Repeat contact in a *new* ticket accounts for most of it, because Intercom's
reversal rule follows the original conversation, and the example assumes customers often open a
new one instead.

## What leaves your network

**No message body is parsed, stored, or sent to Kyvryn.** Every check runs on metadata and state
transitions — who opened a ticket, when it changed status, who replied, whether a refund followed.

Said precisely, because the stronger version is not true of every helpdesk. Salesforce is queried
field by field and never returns a comment body. Zendesk and Intercom cannot be narrowed that way:
Zendesk's ticket export includes each ticket's first comment and its audits carry comment bodies on
the very events we must read to count turns, and Intercom's conversation endpoint has no field
projection at all. Those bytes arrive **inside your own network**, where the connector runs, and are
discarded without being parsed — the types declare no body field. No message content reaches
Kyvryn or appears in any artifact. The source bundle does carry requester ids and email addresses,
which are needed to classify staff and test accounts, so treat it as personal data if you share it.

You can check the second half of that yourself and should: run the published derivation against the
bundle `attest export-bundle` writes, and every field the measurement depends on is visible. What
the connectors request is a property of closed code, so take that part as something we are
accountable for rather than something the published crate can prove.

Your helpdesk credentials are used only to call **your own** helpdesk and are never sent to Kyvryn.
On the offline path (`--source`, `--bundle`) no credential is needed and nothing leaves the machine.

When claims are adjudicated, what crosses the wire is the derived claim record: the vendor's claim
and ticket ids, the period, the resolution type and timestamp the vendor assigned, the vendor's own
intent tag, a public-turn count, the amount billed, the derived flags, and two digests. No names, no
email addresses, no message content. The full field list is `ClaimRecord` in the published crate.

## What your vendor can check

A measurement the measured party cannot reproduce is an assertion, and an assertion is what an
account manager is paid to dismiss. So the part that does the measuring is **open source**:

- **[resolution-normalise](https://github.com/BrijStream-Technologies/resolution-normalise)** —
  the derivation, Apache-2.0. Hand your vendor the source bundle (`attest export-bundle`) and the
  ruleset; they re-run it against their own copy of the records and recompute every claim byte for
  byte, including the digests each claim carries.
- **Versioned rulesets** with published digests, so which rules applied is a lookup rather than a
  claim.
- **Signed verdicts.** `attest verify` checks each signature against the key the attestation names,
  offline, with no account and no network. That proves the file is **internally consistent** — the
  decision, criteria and digests match the key the file names. It does **not** prove the file is
  unaltered, because an altered verdict re-signed with a different key names that key and passes.
  To establish who signed it, compare the named key against the Judge's published key at
  [`/.well-known/judge-keys.json`](https://a2a-orchestrator-production.up.railway.app/.well-known/judge-keys.json) —
  served to anyone, with no credential, so you never have to ask us for it.

This repository holds the binaries, which are **proprietary** ([LICENSE](LICENSE)) — free to download
and run for your own business, not open source. The connectors and report packaging are closed too.
None of it is needed to reproduce a result, which is why keeping it closed costs the trust model
nothing: the part that has to be checkable is the part that is published.

## What it is not

- **Not a judgement of answer quality.** Every criterion is a fact with a record behind it. Whether
  the agent's reply was any *good* is never assessed — substituting our model's opinion for the
  vendor's would repeat the error being pointed at.
- **Not an audit.** It is a comparison against criteria fixed before the run — agreed with your vendor where an
  engagement allows, and stated in every pack either way. Kyvryn is not a
  registered audit firm, performs no attest engagement, and expresses no opinion or assurance on
  any financial statement.
- **Not contingency-priced.** Never a share of what you recover. A measurer paid on what it finds
  has a stake in finding fault, and nobody should accept its figures for that reason.
- **Not sold to your vendor.** They are the party being measured.

## Pricing

Indicative, and settled in the order form: metered per claim adjudicated, **$0.0200**
pay-as-you-go, with the **first 1,000 claims each calendar month free**. The fee does not vary with the amount disputed or recovered, and Kyvryn is
never paid a share of what you recover — that term belongs in the order form, not only on this page.
Deriving and reporting on your own machine — the whole first run above — costs nothing and requires
no account.

## Commands

| Command | What it does |
|---|---|
| `attest normalise` | Derive claim records from a helpdesk export or a source bundle |
| `attest report` | Print the period's counts and what is at stake |
| `attest export-bundle` | Write the vendor-neutral records a run was derived from, to hand over |
| `attest identity` | Create this deployment's attester identity, or enrol it |
| `attest submit` | Send claims for adjudication and collect signed verdicts |
| `attest pack` | Build the evidence pack: report, dispute file, verdicts |
| `attest verify` | Check verdict signatures offline, with no credentials |
| `attest reconcile` | Compare periods: flagged against credited, and the trend |

`attest <command> --help` explains each one.

---

Zendesk, Intercom, Fin, Salesforce and Agentforce are trademarks of their respective owners. Kyvryn
and BrijStream Technologies are not affiliated with, endorsed by, or sponsored by any of them.
Vendor pricing and definitions here are those vendors' own published statements, retrieved
2026-09-20, with the source linked beside each claim. Vendors change pricing — check the linked
source before relying on a figure on this page.
