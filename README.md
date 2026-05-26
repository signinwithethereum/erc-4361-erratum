# ERC-4361 erratum

Evidence backing the ERC-4361 (Sign-In with Ethereum) erratum at [`signinwithethereum/ERCs#1`](https://github.com/signinwithethereum/ERCs/pull/1), filed against version 1 of ERC-4361 by the maintainer of the canonical [@signinwithethereum](https://github.com/signinwithethereum) SIWE implementations.

The PR body is the summary and per-commit table. This repository is the evidence base behind it: catalogue of findings, the 19-commit series with diff hunks, and the conformance audit against the canonical test vectors.

Most commits are editorial or clarifications. The behavior-affecting changes are narrow: `userinfo@` is excluded from the SIWE `domain` field, and `statement` is widened to printable ASCII.

## What's in this repository

| Document | Purpose |
| --- | --- |
| [`inconsistencies.md`](./inconsistencies.md) | Catalogue of contradictions, ambiguities, and mismatches in ERC-4361, with public-discussion pointers and severity ratings. |
| [`proposed-diffs.md`](./proposed-diffs.md) | One section per PR commit, in PR order, with the upstream diff hunk, tag, finding(s) addressed, and compatibility note. Reflects the 19 commits actually on the [`review/4361`](https://github.com/signinwithethereum/ERCs/tree/review/4361) branch. |
| [`conformance-matrix.md`](./conformance-matrix.md) | Empirical audit of the proposed changes against the canonical [`@signinwithethereum/test-vectors`](https://github.com/signinwithethereum/test-vectors) corpus shared by the TypeScript, Python, Rust, and Go SIWE libraries. |
| [`erc-4361-snapshot.md`](./erc-4361-snapshot.md) | Frozen copy of `ERCS/erc-4361.md` at the revision audited (2026-04-23). Keeps line-number citations stable as upstream evolves. |

## Reading order

1. The [PR description](https://github.com/signinwithethereum/ERCs/pull/1) for the summary and per-commit table.
2. [`inconsistencies.md`](./inconsistencies.md) for the primary-source catalogue.
3. [`proposed-diffs.md`](./proposed-diffs.md) for the actual commit-by-commit text.
4. [`conformance-matrix.md`](./conformance-matrix.md) for backwards-compatibility evidence.

## Related work

- **Main PR:** [`signinwithethereum/ERCs#1`](https://github.com/signinwithethereum/ERCs/pull/1) — open on the `review/4361` branch.
- **Concurrent test-vector PR:** [`signinwithethereum/test-vectors#1`](https://github.com/signinwithethereum/test-vectors/pull/1) — moves the two `userinfo@` domain vectors from positive to negative, and updates statement-character vectors for the printable-ASCII widening.
- **Downstream library PRs:** minor version bumps against the maintained TypeScript, Python, Rust, and Go SIWE libraries, applying only the surviving parser changes. URLs to be added when those PRs open.

## License

Dedicated to the public domain under [CC0 1.0 Universal](./LICENSE), matching the Creative Commons Zero convention used by Ethereum ERCs.
