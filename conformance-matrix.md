---
title: Conformance matrix for ERC-4361 remediation
status: seeded (Stage 0.3 of action-plan.md). Library corpus gathering is a separate decision point — see §5.
---

# Conformance matrix

Per `action-plan.md` (maintainer-internal, not published) §0.3, this file tabulates a corpus of SIWE messages against three columns:

1. **Pre-spec**: the current `erc-4361.md` grammar as of 2026-04-23.
2. **Post-spec**: the grammar as it would read after all 20 commits in [`proposed-diffs.md`](./proposed-diffs.md).
3. **Ref-impl**: the linked reference implementation (`https://eips.ethereum.org/assets/eip-4361/example.js`), unmodified.

The delta between columns is the interop story we take into the PR. In particular:

- A message with `Pre-spec=pass, Ref-impl=fail` is direct evidence for Finding #4.
- A message with `Pre-spec=pass, Post-spec=fail` is a narrowing that needs the conformance matrix's support to argue is safe.
- A message with `Pre-spec=fail, Post-spec=pass` is a widening that needs producer-behavior evidence to be safe to ship.

---

## 1. Methodology

**Corpus construction.** Messages are drawn from three tiers:

- **Tier A — spec examples.** The three messages in `erc-4361.md` lines 152–204. Authoritative by definition.
- **Tier B — library test vectors.** Positive and negative test cases shipped in the following libraries' test suites:
  - [`spruceid/siwe`](https://github.com/spruceid/siwe) — TypeScript, de facto reference library.
  - [`spruceid/siwe-rs`](https://github.com/spruceid/siwe-rs) — Rust port.
  - [`spruceid/siwe-py`](https://github.com/spruceid/siwe-py) — Python port.
  - [`ethereum/go-ethereum`](https://github.com/ethereum/go-ethereum) — Go, plus the parser work tracked in go-ethereum#24132.
- **Tier C — real-world samples.** Captured SIWE messages from production RPs where publicly observable: ENS sign-in, an ENS-name resolver deployment, any one wallet developer-mode dump. Anonymize nonces but preserve grammar-relevant structure.

**Evaluation.** Each message is classified per column as `pass` / `fail`. For `fail`, record the specific ABNF production or prose rule that rejected it. The same classification logic applies to all three columns — only the reference grammar differs.

**Automation.** The evaluation is a hand exercise for Tier A (done below); scripted for Tiers B and C once the corpus is gathered. The script should consume the three grammars as ABNF text and produce a CSV; writing it is a Stage 0.3 implementation task, not part of this document.

---

## 2. Tier A — spec examples (done inline)

### 2.1 Example with implicit scheme (erc-4361.md lines 152–168)

```text
example.com wants you to sign in with your Ethereum account:
0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2

I accept the ExampleOrg Terms of Service: https://example.com/tos

URI: https://example.com/login
Version: 1
Chain ID: 1
Nonce: 32891756
Issued At: 2021-09-30T16:25:24Z
Resources:
- ipfs://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq/
- https://example.com/my-web2-claim.json
```

| Column | Result | Notes |
| --- | --- | --- |
| Pre-spec | **pass** | `domain = example.com` (reg-name). Statement uses only unreserved, reserved (`:`, `/`), and space. Address lowercase-hex; Pre-spec ABNF comment says `Must` EIP-55, field description says `SHOULD` — parses either way, the conformance question is normative not grammatical. Resources has two items. |
| Post-spec | **pass** | No charset, grammar, or prose rule moves this out of conformance. |
| Ref-impl | **pass** | The spec's canonical example; the reference file is expected to accept it. |

### 2.2 Example with implicit scheme and explicit port (erc-4361.md lines 170–186)

Identical to 2.1 except `domain = example.com:3388`.

| Column | Result | Notes |
| --- | --- | --- |
| Pre-spec | **pass** | `:3388` is a valid RFC 3986 `port`. |
| Post-spec | **pass** | Post-spec narrowing forbids *empty* explicit port (`example.com:`), not a populated one. |
| Ref-impl | **pass** | No parse path is different from 2.1 except the port. |

### 2.3 Example with explicit scheme (erc-4361.md lines 188–204)

Identical to 2.1 except the first line is `https://example.com wants you to sign in ...`.

| Column | Result | Notes |
| --- | --- | --- |
| Pre-spec | **pass** | ABNF line 45 is `[ scheme "://" ] domain %s" wants you to sign in ..."`. Explicit scheme is grammatically valid. |
| Post-spec | **pass** | No change to the `[ scheme "://" ]` production. |
| Ref-impl | **fail** | Per Finding #4 and siwe#30's follow-up comment: the reference implementation's top-level grammar omits `[ scheme "://" ]` and rejects messages with an explicit scheme prefix. This is the single most damaging `Pre-spec=pass, Ref-impl=fail` cell and is direct evidence for Commit 20. |

**Tier A delta summary.** 1 fail out of 9 cells (3 messages × 3 columns), all attributable to Finding #4. Zero `Pre-spec ↔ Post-spec` divergence on the spec's own examples.

---

## 3. Tier B — canonical test-vectors (@signinwithethereum)

Corpus: `libs/test-vectors/vectors/` — consumed by `@signinwithethereum/siwe` (TS), `@signinwithethereum/siwe-py`, `@signinwithethereum/siwe-rs`, and the Go port at `libs/go`. Since the three maintained libraries share this corpus by construction, passing the shared corpus is equivalent to passing all three libraries simultaneously.

Scan date: 2026-04-23.

### 3.1 Coverage of the narrowing and widening patterns

Each row counts, across the entire corpus, how many **positive** vectors exhibit the pattern a proposed commit would forbid (for narrowings) or newly admit (for widening). A non-zero positive-vector count for a narrowing means the maintained libraries explicitly test that the pattern parses today — a direct contradiction of the narrowing.

| Commit | Finding | Pattern | Positive hits | Negative hits | PR status |
| --- | --- | --- | --- | --- | --- |
| 13 | #14 | empty `Request ID:` | **1** (`grammar/valid_specification.json:request-id empty`) | 0 | **Contradicted** |
| 14 | #15 | empty port (`example.com:`) | 0 | 0 | Safe |
| 15 | #16 | empty `host` (missing domain) | 0 | 1 (`parsing_negative.json:missing domain`) | Safe — already rejected |
| 16 | #17 | leading-zero `chain-id` | 0 | 0 | Safe |
| 17 | #6 | `userinfo@` in `domain` | **2** (`parsing/parsing_positive.json:domain is RFC 3986 authority with userinfo{, and port}`) | 0 | **Contradicted** |
| 18 | #12 | empty `statement` | **1** (`grammar/valid_specification.json:statement empty`) | 0 | **Contradicted** |
| 18 | #13 | empty `Resources:` list | **1** (`grammar/valid_specification.json:resources empty`) | 0 | **Contradicted** |
| 19 | #11 | statement with `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, `` ` `` | 0 positive | 6 negative in `grammar/invalid_chars.json` (`` ` < > { | } `` explicitly rejected) | Widening — requires concurrent parser update |

### 3.2 Related findings (not narrowing/widening, checked opportunistically)

| Finding | Pattern in corpus | Interpretation |
| --- | --- | --- |
| #3 (MUST vs SHOULD on EIP-55) | `parsing_warnings.json` treats all-lowercase and all-uppercase addresses as **warnings** (not rejections). `parsing_negative.json:address mixed-case wrong EIP-55 checksum` rejects wrong-checksum mixed-case addresses. | The canonical parsers implement SHOULD (warning on non-EIP-55, reject only when mixed-case checksum is wrong). **Supports our harmonize-DOWN decision.** The PR aligns the spec with implemented behavior. |
| #2 (double-prefix risk) | Verification vectors in `verification/` use a single ERC-191 prefix applied at signing time. | No corpus evidence of double-prefix in the wild. Our clarification commit codifies existing behavior. |
| #4 (reference-impl divergence) | Spec example 3 with explicit scheme (`https://example.com wants you to sign in...`) is tested in `parsing_positive.json:domain contains optional scheme`. The canonical parsers accept it; the linked reference impl does not. | **Supports Commit 20** — the reference implementation must be updated to match the canonical behavior. |

### 3.3 Implications — four narrowings are active R3 hits

- **Commits 13 (empty `Request ID:`), 17 (userinfo in domain), and 18 (empty statement + empty Resources)** require reconciliation before the PR is submitted. The canonical test-vectors explicitly document these patterns as valid SIWE messages with documented parsed shapes. Filing a PR that narrows the spec against the canonical test-vectors — from the very maintainer who owns those test-vectors — is self-contradictory.

- For each contradicted commit, there are three paths:

  1. **Drop the commit.** Keep the grammar permissive. Add prose to the field descriptions clarifying the "present but empty" semantic is intentional. Lowest ecosystem cost; concedes the narrowing.
  2. **Keep the commit and update the corpus concurrently.** Move the affected vectors from positive → negative in the same release cycle. Ship coordinated releases of `libs/test-vectors`, `libs/ts`, `libs/py`, `libs/rs`, `libs/go`. High coordination cost but the narrowing lands.
  3. **Soften the commit.** Replace the grammatical narrowing with prose-level guidance ("producers SHOULD NOT emit an empty `statement`; parsers MUST continue to accept one for backwards compatibility"). Middle ground — no ecosystem break, clearer guidance.

- **Recommended per-commit direction** (for discussion before Stage 2 drafting):

  | Commit | Finding | Recommendation | Rationale |
  | --- | --- | --- | --- |
  | 13 | #14 (empty request-id) | **Option 1 (drop)** or **Option 3 (soften)** | "Present but empty" semantic is intentional per test-vector; no safety argument for breaking it. |
  | 17 | #6 (userinfo) | **Option 2 (keep + update corpus)** | Phishing argument is real, and `test@127.0.0.1` as a positive SIWE-level vector looks like an RFC-3986-completeness test, not a real usage pattern. Moving the 2 vectors positive → negative is a defensible release note. |
  | 18 (#12) | empty statement | **Option 3 (soften)** | Same as #14 — semantic is intentional. |
  | 18 (#13) | empty Resources | **Option 3 (soften)** | Same. |

- **Commits 14, 15, 16 (Findings #15, #16, #17)** are safe: no positive corpus vector exercises the narrowed patterns, and Finding #16 is already aligned (the corpus rejects missing-domain via `parsing_negative.json`).

- **Commit 19 (Finding #11 widening)** requires concurrent library work: the TS parser explicitly rejects ``< > { | } ` `` in statements per `invalid_chars.json`. This is expected for a widening — the commit message already specifies parser-first rollout; we now know the parser work is non-trivial but well-scoped.

### 3.4 Corpus coverage caveats

- Vectors like `domain is RFC 3986 authority with userinfo` look closer to RFC-3986 grammar completeness tests than real SIWE-usage samples. The *intent* of preserving them in the corpus should be confirmed with a co-author or the historical commit log before defaulting to "the corpus says it must parse, therefore the narrowing dies."
- The corpus does not sample real-world RPs (Tier C). Tier C remains nice-to-have but is not load-bearing given how explicit Tier B has become.

---

## 4. Tier C — real-world RP samples (pending)

- [ ] ENS sign-in (ens.domains / related properties). One sample.
- [ ] One SIWE-using DeFi property. Candidate list: the Ethereum Foundation's own internal tooling if any is public, 1inch, Gnosis.
- [ ] One wallet's developer-mode message dump (MetaMask, Rainbow, or Frame).

Tier C evidence strengthens the "real producers emit canonical messages" argument but is nice-to-have; Tier B is sufficient on its own if Tier C proves hard to collect.

---

## 5. Decision point — how to reconcile four contradicted narrowings

Tier B is done and it produced an unambiguous signal: four narrowings (Commits 13, 17, 18) contradict the canonical test-vectors. See §3.3 for per-commit recommendations.

Before Stage 2 drafting, the maintainer should pick one path per contradicted commit (drop / keep-and-migrate-corpus / soften). This is a strategic call, not an editorial one, and it materially reshapes the PR.

`proposed-diffs.md` will need to be revised to reflect whichever paths are chosen. The revisions are localized to Commits 13, 17, 18 (and possibly a new prose-only commit if Option 3 is chosen for any of them).

---

## 6. Delta columns expected in the final matrix

Once Tiers B and C are filled in, this section becomes a one-page summary table suitable for inclusion in the PR body. Expected schema:

| Message ID | Source | Pre-spec | Post-spec | Ref-impl | Finding(s) implicated |
| --- | --- | --- | --- | --- | --- |
| A.1 | erc-4361.md:152 | pass | pass | pass | — |
| A.2 | erc-4361.md:170 | pass | pass | pass | — |
| A.3 | erc-4361.md:188 | pass | pass | **fail** | #4 |
| B.1.* | spruceid/siwe | … | … | … | … |
| B.2.* | siwe-rs | … | … | … | … |
| B.3.* | siwe-py | … | … | … | … |
| B.4.* | go-ethereum | … | … | … | … |
| C.1 | ENS sign-in | … | … | … | … |
| C.2 | DeFi RP sample | … | … | … | … |
| C.3 | Wallet devmode dump | … | … | … | … |

Rows that change columns are the interop story. Rows that don't are the safety evidence.
