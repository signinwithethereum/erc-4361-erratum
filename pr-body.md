## Summary

This PR corrects internal contradictions in ERC-4361 that caused independent implementations (the @signinwithethereum TypeScript/Python/Rust libraries and the Go port, the historical spruceid/siwe series they descend from, go-ethereum, and the linked reference implementation) to diverge from the spec and from each other. It does not change the intended wire format of a version-1 SIWE message; it aligns the grammar, the prose, and the reference implementation to a single reading of what version-1 has always meant.

This PR is filed by the current maintainer of the canonical `@signinwithethereum` SIWE implementations. It is intended as an erratum to v1, not a version bump: the PR's backwards-compatibility posture is documented below and the supporting evidence is quantitative, drawn from the shared test-vectors corpus consumed by all four maintained SIWE libraries.

## What changes

20 commits against `ERCS/erc-4361.md` and `assets/erc-4361/example.js`, one commit per finding (two couple-findings share a commit), low-controversy first. Classification key:

- `editorial`: typos, stale names, wording. No normative effect.
- `clarification`: no wire change, no behavior change in any correct implementation. Aligns prose or reference implementation with what the spec already meant.
- `narrow`: grammar becomes stricter. No real implementation produces the affected patterns today. Three narrowings ship with zero corpus impact; one (finding #6) requires a coordinated `@signinwithethereum/test-vectors` release.
- `widen`: grammar becomes looser. Parser rollout is recommended before producer rollout.
- `judgment`: a genuine direction call between two competing readings; resolution documented in the commit body.

| # | Commit subject | Tag | Finding |
| --- | --- | --- | --- |
| 1 | ERC-4361 editorial: remove invalid space after :// in informal template (#18) | editorial | #18 |
| 2 | ERC-4361 editorial: fix stale request-uri reference (#19) | editorial | #19 |
| 3 | ERC-4361 editorial: clarify RFC 3339 is a profile of ISO 8601, not equivalent (#20) | editorial | #20 |
| 4 | ERC-4361 editorial: normalize EIP-55 to ERC-55 in ABNF comment (#21) | editorial | #21 |
| 5 | ERC-4361 clarification: assign ERC-191 prefixing to wallet/signing primitive only (#2) | clarification | #2 |
| 6 | ERC-4361 clarification: harmonize omitted-scheme default via wallet defaultScheme (#7) | clarification | #7 |
| 7 | ERC-4361 clarification: drop redundant subdomain-mismatch rule, host already covers it (#8) | clarification | #8 |
| 8 | ERC-4361 clarification: formalize RFC 3986/5234/7405 grammar imports (#5) | clarification | #5 |
| 9 | ERC-4361 clarification: declare UTF-8 as the wire encoding (#1) | clarification | #1 |
| 10 | ERC-4361 clarification: forbid percent-decoded URI values from re-framing the message (#9) | clarification | #9 |
| 11 | ERC-4361 clarification: define IDN/punycode handling for domain matching (#10) | clarification | #10 |
| 12 | ERC-4361 judgment: harmonize ERC-55 to SHOULD and pin "0x" to case-sensitive (#3) | judgment | #3 |
| 13 | ERC-4361 clarification: discourage empty request-id in producers, parsers keep accepting (#14) | clarification | #14 |
| 14 | ERC-4361 narrow: reject empty explicit port and remove unreachable wallet branch (#15) | narrow | #15 |
| 15 | ERC-4361 narrow: require non-empty domain host and exclude userinfo subcomponent (#16) | narrow | #6, #16 |
| 16 | ERC-4361 narrow: require canonical chain-id with no leading zeros (#17) | narrow | #17 |
| 17 | ERC-4361 narrow: document phishing rationale for domain userinfo exclusion (#6) | narrow | #6 |
| 18 | ERC-4361 clarification: empty statement and bare Resources header equate to omission (#12, #13) | clarification | #12, #13 |
| 19 | ERC-4361 widen: statement charset to printable ASCII excluding LF (#11) | widen | #11 |
| 20 | ERC-4361 clarification: align reference implementation with cumulative grammar (#4) | clarification | #4 |

Finding numbers refer to the catalogue in the evidence document linked below.

## Backwards compatibility

All changes were audited against the canonical shared test-vector corpus `@signinwithethereum/test-vectors` (`vectors/*.json`), which is consumed by the four maintained SIWE libraries (TS, Python, Rust, Go). Observed effects on the corpus:

- 15 of 20 commits are editorial or clarifications with no wire change.
- 3 narrowings (findings #15, #16, #17) affect zero positive vectors in the corpus.
- 1 narrowing (finding #6, userinfo exclusion) affects 2 positive vectors that are RFC-3986 grammar-completeness tests, not real SIWE usage samples. A concurrent release plan moves these vectors from positive to negative and ships coordinated minor releases of the four libraries. See the "Concurrent release plan" section below.
- 1 widening (finding #11) allows characters the current grammar forbade but which no production relying party avoided. Parser updates in the four libraries roll out first, producer updates second.
- 3 patterns that previous drafts of this PR would have narrowed (findings #12, #13, #14, empty `statement` / empty `Resources:` / empty `request-id`) are explicitly documented in the canonical corpus as parseable with distinct parsed shape. Those commits were consequently softened to prose-only "producers SHOULD NOT emit, parsers MUST accept" guidance, preserving the intentional "present but empty" semantic. No wire change.
- Finding #3 (EIP-55 MUST vs SHOULD) is harmonized DOWN to SHOULD. The maintained parsers already treat non-checksum addresses as warnings rather than rejections, so this aligns the spec with implemented behavior.

The per-finding corpus classification is in the evidence document.

## Concurrent release plan (finding #6 only)

The narrowing for finding #6 (userinfo exclusion in `domain`) is the only commit whose ecosystem rollout must be visible at merge time. A parallel PR against `signinwithethereum/test-vectors` moves the two userinfo grammar-completeness vectors from `parsing_positive.json` to `parsing_negative.json`; draft PRs against `signinwithethereum/siwe` (TS), `signinwithethereum/siwe-py`, `signinwithethereum/siwe-rs`, and the Go port update their parsers in lockstep. Merging any subset of this PR that excludes commit 15 or 17 removes the dependency on the concurrent release. Links to the parallel PRs appear below once this PR opens.

## Why we believe no CFI is required

- The change is internal to v1: no producer or parser that correctly implements the current spec is moved out of conformance by any editorial or clarification commit.
- The narrowings are grounded in an explicit corpus audit, not "no known implementation." The corpus is maintained by the same party filing this PR, and the audit artefacts are linked below.
- The widening (finding #11) moves in the direction producers are already taking in practice, codifying what the field description already described.
- The judgment call (finding #3) resolves a self-contradiction in the current spec in the direction that implementations already treat as normative.

If editors disagree on any specific commit, naming the SHA will trigger a cherry-pick-out of that commit; the rest of the PR can merge while a CFI is scoped to whatever remains in contention.

## Evidence and references

All supporting documents are published at `signinwithethereum/erc-4361-erratum`:

- Findings catalogue with public-discussion links: [`inconsistencies.md`](https://github.com/signinwithethereum/erc-4361-erratum/blob/main/inconsistencies.md)
- Conformance audit against the canonical test-vectors corpus: [`conformance-matrix.md`](https://github.com/signinwithethereum/erc-4361-erratum/blob/main/conformance-matrix.md)
- Per-commit replacement text and rationale: [`proposed-diffs.md`](https://github.com/signinwithethereum/erc-4361-erratum/blob/main/proposed-diffs.md)
- Frozen snapshot of the spec as audited (for line-number citation stability): [`erc-4361-snapshot.md`](https://github.com/signinwithethereum/erc-4361-erratum/blob/main/erc-4361-snapshot.md)

The per-commit compatibility classification table in this PR body is the authoritative summary; the evidence documents back it up with primary sources.

## Suggested reviewers

From the author list:
- @wyc (Wayne Chang)
- @obstropolos (Gregory Rocco)
- @brantlymillegan (Brantly Millegan)
- @Arachnid (Nick Johnson)
- @awoie (Oliver Terbu)

## How to request changes to specific commits

If you object to a specific commit, please name the SHA and the objection. We will cherry-pick the commit off the branch, force-push, and note the drop in this thread with our reasoning. Every commit is atomic and independently revertable; the PR is structured so that partial acceptance is a clean rebase, not a rewrite.

---

Filed as an erratum against version 1 of ERC-4361. The status of the spec remains Final; no version bump is proposed. Future work on cross-chain, EIP-712 mode, SIOPv2, or DIDs (per the spec's Forwards Compatibility section) remains the natural home for any successor version.
