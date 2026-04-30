---
title: Conformance matrix for ERC-4361 remediation
---

# Conformance matrix

This file tabulates the message patterns that matter for the simplified ERC-4361 erratum.

Columns:

1. **Pre-spec**: current ERC-4361 as of [`erc-4361-snapshot.md`](./erc-4361-snapshot.md), audited 2026-04-23.
2. **Post-spec**: ERC-4361 after the simplified commits in [`proposed-diffs.md`](./proposed-diffs.md).
3. **Ref-impl**: linked reference implementation before the erratum.

Rows that change between Pre-spec and Post-spec are the compatibility story. Rows that only differ in Ref-impl support the reference implementation fix.

---

## 1. Methodology

**Corpus construction.**

- **Tier A: spec examples.** The three examples in `erc-4361-snapshot.md` lines 152-204.
- **Tier B: canonical test vectors.** The shared [`@signinwithethereum/test-vectors`](https://github.com/signinwithethereum/test-vectors) corpus consumed by the maintained TypeScript, Python, Rust, and Go SIWE libraries.

**Scope change from the earlier draft.** The simplified erratum no longer proposes normative changes for IDN/punycode handling, percent-decoded URI display framing, leading-zero chain IDs, or empty explicit ports. Those patterns are therefore not treated as ERC diff requirements here.

---

## 2. Tier A: spec examples

| Source | Pre-spec | Post-spec | Ref-impl | Interpretation |
| --- | --- | --- | --- | --- |
| Example 1: implicit scheme | pass | pass | pass | No change. |
| Example 2: implicit scheme and explicit port | pass | pass | pass | No change. |
| Example 3: explicit scheme | pass | pass | fail | Direct evidence for the reference implementation fix. The ABNF allows `[ scheme "://" ]`; the linked reference parser omits it. |

Zero spec examples move between valid and invalid under the simplified erratum.

---

## 3. Tier B: canonical test vectors

Scan date: 2026-04-23.

### 3.1 Behavior-changing or compatibility-relevant patterns

| Finding | Pattern | Positive hits | Negative hits | Simplified PR status |
| --- | --- | --- | --- | --- |
| #6 | `userinfo@` in `domain` | 2 (`parsing/parsing_positive.json:domain is RFC 3986 authority with userinfo{, and port}`) | 0 | Kept as the only intentional narrowing. Concurrent test-vector PR moves these to negative. |
| #11 | statement with `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, or backtick | 0 | 6 in `grammar/invalid_chars.json` (backtick, `<`, `>`, `{`, `|`, `}`) | Kept as explicit widening. Parser updates must precede producer reliance. |
| #12 | empty `statement` | 1 (`grammar/valid_specification.json:statement empty`) | 0 | Kept valid. Producer guidance prefers omission. |
| #13 | bare `Resources:` header | 1 (`grammar/valid_specification.json:resources empty`) | 0 | Kept valid. Producer guidance prefers omission. |
| #14 | empty `Request ID:` | 1 (`grammar/valid_specification.json:request-id empty`) | 0 | Kept valid. Producer guidance prefers omission. |
| #15 | empty explicit port (`example.com:`) | 0 | 0 | No standalone change in simplified erratum. |
| #17 | leading-zero `chain-id` | 0 | 0 | Deferred from simplified erratum. |

### 3.2 Related checks

| Finding | Pattern in corpus | Interpretation |
| --- | --- | --- |
| #3 | All-lowercase and all-uppercase addresses are warnings; wrong-checksum mixed-case addresses are negative. | Supports harmonizing ERC-55 down to SHOULD. |
| #2 | Verification vectors apply a single ERC-191 prefix at signing time. | Supports the prefix-once clarification. |
| #4 | Explicit-scheme messages are accepted by canonical parsers but rejected by the linked reference implementation. | Supports updating `assets/eip-4361/example.js`. |

---

## 4. Summary matrix

| Source | Pre-spec | Post-spec | Ref-impl pre-fix | Finding(s) |
| --- | --- | --- | --- | --- |
| Spec example with explicit scheme | pass | pass | fail | #4 |
| `userinfo@` domain vectors | pass | fail | pass | #6 |
| Empty `request-id` | pass | pass | pass | #14 |
| Empty `statement` | pass | pass | pass | #12 |
| Bare `Resources:` | pass | pass | pass | #13 |
| Statement punctuation currently negative | fail | pass | fail | #11 |
| Non-EIP-55 all-lower/all-uppercase address | warning | warning | warning | #3 |

The simplified erratum has one retained narrowing (`userinfo@` in `domain`) and one retained widening (`statement` printable ASCII). Everything else is editorial, clarification, or producer guidance that preserves parser acceptance.
