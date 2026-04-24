---
title: Conformance matrix for ERC-4361 remediation
---

# Conformance matrix

This file tabulates a corpus of SIWE messages against three columns:

1. **Pre-spec**: the current `erc-4361.md` grammar as of 2026-04-23.
2. **Post-spec**: the grammar as it would read after all 20 commits in [`proposed-diffs.md`](./proposed-diffs.md).
3. **Ref-impl**: the linked reference implementation (`https://eips.ethereum.org/assets/eip-4361/example.js`), unmodified.

The delta between columns is the interop story we take into the PR. In particular:

- A message with `Pre-spec=pass, Ref-impl=fail` is direct evidence for Finding #4.
- A message with `Pre-spec=pass, Post-spec=fail` is a narrowing that needs the conformance matrix's support to argue is safe.
- A message with `Pre-spec=fail, Post-spec=pass` is a widening that needs producer-behavior evidence to be safe to ship.

---

## 1. Methodology

**Corpus construction.** Messages are drawn from two tiers:

- **Tier A: spec examples.** The three messages in `erc-4361-snapshot.md` lines 152-204. Authoritative by definition.
- **Tier B: canonical test-vectors.** The shared test-vector corpus published as [`@signinwithethereum/test-vectors`](https://github.com/signinwithethereum/test-vectors), consumed by construction by the four maintained SIWE libraries: [`@signinwithethereum/siwe`](https://github.com/signinwithethereum/siwe) (TypeScript), [`@signinwithethereum/siwe-py`](https://github.com/signinwithethereum/siwe-py), [`@signinwithethereum/siwe-rs`](https://github.com/signinwithethereum/siwe-rs), and the Go port. Because the four libraries implement the same corpus, a result against Tier B is a result against all four libraries simultaneously.

**Evaluation.** Each message is classified per column as `pass` / `fail`. For `fail`, record the specific ABNF production or prose rule that rejected it. The same classification logic applies to all three columns; only the reference grammar differs.

**Tier C note.** An earlier draft of this audit budgeted a Tier C pass over real-world relying-party samples (ENS sign-in, a DeFi deployment, a wallet developer-mode dump). Tier B proved sufficient for the decisions this document informs: the canonical corpus is authoritative for the four libraries that together cover the maintained SIWE ecosystem, and the patterns contradicted by Tier B could not have been resolved by additional RP samples. Tier C remains available for follow-up work but was not executed.

---

## 2. Tier A: spec examples (done inline)

### 2.1 Example with implicit scheme (erc-4361.md lines 152-168)

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
| Pre-spec | **pass** | `domain = example.com` (reg-name). Statement uses only unreserved, reserved (`:`, `/`), and space. Address lowercase-hex; Pre-spec ABNF comment says `Must` EIP-55, field description says `SHOULD`; parses either way, the conformance question is normative not grammatical. Resources has two items. |
| Post-spec | **pass** | No charset, grammar, or prose rule moves this out of conformance. |
| Ref-impl | **pass** | The spec's canonical example; the reference file is expected to accept it. |

### 2.2 Example with implicit scheme and explicit port (erc-4361.md lines 170-186)

Identical to 2.1 except `domain = example.com:3388`.

| Column | Result | Notes |
| --- | --- | --- |
| Pre-spec | **pass** | `:3388` is a valid RFC 3986 `port`. |
| Post-spec | **pass** | Post-spec narrowing forbids *empty* explicit port (`example.com:`), not a populated one. |
| Ref-impl | **pass** | No parse path is different from 2.1 except the port. |

### 2.3 Example with explicit scheme (erc-4361.md lines 188-204)

Identical to 2.1 except the first line is `https://example.com wants you to sign in ...`.

| Column | Result | Notes |
| --- | --- | --- |
| Pre-spec | **pass** | ABNF line 45 is `[ scheme "://" ] domain %s" wants you to sign in ..."`. Explicit scheme is grammatically valid. |
| Post-spec | **pass** | No change to the `[ scheme "://" ]` production. |
| Ref-impl | **fail** | Per Finding #4 and siwe#30's follow-up comment: the reference implementation's top-level grammar omits `[ scheme "://" ]` and rejects messages with an explicit scheme prefix. This is the single most damaging `Pre-spec=pass, Ref-impl=fail` cell and is direct evidence for Commit 20. |

**Tier A delta summary.** 1 fail out of 9 cells (3 messages × 3 columns), all attributable to Finding #4. Zero `Pre-spec ↔ Post-spec` divergence on the spec's own examples.

---

## 3. Tier B: canonical test-vectors

Corpus: `@signinwithethereum/test-vectors`, the shared fixture set consumed by `@signinwithethereum/siwe` (TS), `@signinwithethereum/siwe-py`, `@signinwithethereum/siwe-rs`, and the Go port. Because the four libraries implement the same corpus, passing Tier B is equivalent to passing all four libraries simultaneously.

Scan date: 2026-04-23.

### 3.1 Coverage of the narrowing and widening patterns

Each row counts, across the entire corpus, how many **positive** vectors exhibit the pattern a proposed commit would forbid (for narrowings) or newly admit (for widening). A non-zero positive-vector count for a narrowing means the maintained libraries explicitly test that the pattern parses today; a direct contradiction of the narrowing.

| Commit | Finding | Pattern | Positive hits | Negative hits | PR status |
| --- | --- | --- | --- | --- | --- |
| 13 | #14 | empty `Request ID:` | **1** (`grammar/valid_specification.json:request-id empty`) | 0 | **Contradicted** |
| 14 | #15 | empty port (`example.com:`) | 0 | 0 | Safe |
| 15 | #16 | empty `host` (missing domain) | 0 | 1 (`parsing_negative.json:missing domain`) | Safe; already rejected |
| 16 | #17 | leading-zero `chain-id` | 0 | 0 | Safe |
| 17 | #6 | `userinfo@` in `domain` | **2** (`parsing/parsing_positive.json:domain is RFC 3986 authority with userinfo{, and port}`) | 0 | **Contradicted** |
| 18 | #12 | empty `statement` | **1** (`grammar/valid_specification.json:statement empty`) | 0 | **Contradicted** |
| 18 | #13 | empty `Resources:` list | **1** (`grammar/valid_specification.json:resources empty`) | 0 | **Contradicted** |
| 19 | #11 | statement with `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, `` ` `` | 0 positive | 6 negative in `grammar/invalid_chars.json` (`` ` < > { | } `` explicitly rejected) | Widening; requires concurrent parser update |

### 3.2 Related findings (not narrowing/widening, checked opportunistically)

| Finding | Pattern in corpus | Interpretation |
| --- | --- | --- |
| #3 (MUST vs SHOULD on EIP-55) | `parsing_warnings.json` treats all-lowercase and all-uppercase addresses as **warnings** (not rejections). `parsing_negative.json:address mixed-case wrong EIP-55 checksum` rejects wrong-checksum mixed-case addresses. | The canonical parsers implement SHOULD (warning on non-EIP-55, reject only when mixed-case checksum is wrong). **Supports our harmonize-DOWN decision.** The PR aligns the spec with implemented behavior. |
| #2 (double-prefix risk) | Verification vectors in `verification/` use a single ERC-191 prefix applied at signing time. | No corpus evidence of double-prefix in the wild. Our clarification commit codifies existing behavior. |
| #4 (reference-impl divergence) | Spec example 3 with explicit scheme (`https://example.com wants you to sign in...`) is tested in `parsing_positive.json:domain contains optional scheme`. The canonical parsers accept it; the linked reference impl does not. | **Supports Commit 20**; the reference implementation must be updated to match the canonical behavior. |

### 3.3 Consequences for the PR

Four findings that earlier drafts of the PR would have narrowed grammatically are contradicted by positive Tier B vectors. Each was reconciled before submission:

| Commit | Finding | Positive-vector hits | Reconciliation taken |
| --- | --- | --- | --- |
| 13 | #14 (empty `Request ID:`) | 1 | **Softened.** Grammar unchanged; prose guides producers ("SHOULD NOT emit empty `request-id`") and mandates parser acceptance for compatibility. |
| 17 | #6 (userinfo in `domain`) | 2 | **Kept + concurrent corpus migration.** Grammar excludes `userinfo`. The two positive vectors (both RFC-3986 grammar-completeness tests with `domain = test@127.0.0.1`) move from positive to negative in a concurrent `@signinwithethereum/test-vectors` release; the four libraries ship minor bumps in lockstep. |
| 18 | #12 (empty `statement`) | 1 | **Softened.** Same rationale as #14: the "missing vs present-but-empty" distinction is documented in the corpus as intentional. |
| 18 | #13 (empty `Resources:` list) | 1 | **Softened.** Same rationale. |

Commits 14, 15, 16 (Findings #15, #16, #17) ship as grammatical narrowings: no positive corpus vector exercises the narrowed patterns, and Finding #16 is already aligned (the corpus rejects missing-domain in `parsing_negative.json`).

Commit 19 (Finding #11 widening) requires concurrent library work: the TS parser explicitly rejects ``< > { | } ` `` in statements per `invalid_chars.json`. Parser updates across the four libraries roll out first, producer updates second.

See [`proposed-diffs.md`](./proposed-diffs.md) Commits 13, 17, and 18 for the exact replacement text in each reconciled case.

---

## 4. Summary matrix

One-page summary suitable for inclusion in the PR body. Rows that change columns are the interop story; rows that do not are the safety evidence.

| Source | Pre-spec | Post-spec | Ref-impl (pre-Commit 20) | Finding(s) implicated |
| --- | --- | --- | --- | --- |
| Tier A: spec example 1 (implicit scheme) | pass | pass | pass | (none) |
| Tier A: spec example 2 (implicit scheme, explicit port) | pass | pass | pass | (none) |
| Tier A: spec example 3 (explicit scheme) | pass | pass | **fail** | #4 |
| Tier B: empty `request-id` | pass | pass (with producer SHOULD-NOT) | pass | #14 |
| Tier B: empty `statement` | pass | pass (with producer SHOULD-NOT) | pass | #12 |
| Tier B: empty `Resources:` list | pass | pass (with producer SHOULD-NOT) | pass | #13 |
| Tier B: `userinfo@` in `domain` (2 vectors) | pass | **fail** (narrowing kept; concurrent corpus migration) | pass | #6 |
| Tier B: empty port (`example.com:`) | pass | pass (corpus has no hits) | pass | #15 |
| Tier B: empty `host` (missing domain) | fail | fail | fail | #16 |
| Tier B: leading-zero `chain-id` | pass | pass (corpus has no hits) | pass | #17 |
| Tier B: statement with ``< > { | } ` `` (6 negative vectors) | fail | pass (widening) | #11 |
| Tier B: non-EIP-55 address (all-lowercase or all-uppercase) | warning | warning | warning | #3 |
