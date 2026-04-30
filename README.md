# ERC-4361 erratum

Evidence backing a pull request against [`ethereum/ERCs`](https://github.com/ethereum/ERCs) that remediates internal contradictions in ERC-4361 (Sign-In with Ethereum).

**The pull request:** [signinwithethereum/ERCs#1](https://github.com/signinwithethereum/ERCs/pull/1) (preview, pre-submission). When the PR moves to `ethereum/ERCs` at submission, this link will be updated.

The PR body is a rendering of the evidence in this repository, not a separate argument. Reading the PR and then this repository is redundant; use the PR for the summary and navigation, and drill into this repository when you want the primary sources behind any specific claim.

The remediation is filed as an erratum against version 1 of ERC-4361 by the current maintainer of the canonical [@signinwithethereum](https://github.com/signinwithethereum) SIWE implementations. Most changes are editorial corrections or clarifications that align the grammar, prose, and reference implementation. The few behavior-affecting changes are explicitly classified in [`proposed-diffs.md`](./proposed-diffs.md).

## What's in this repository

| Document | Purpose |
| --- | --- |
| [`inconsistencies.md`](./inconsistencies.md) | Catalogue of internal contradictions, ambiguities, mismatches, and deferred hardening ideas considered for the ERC-4361 erratum. Each active finding links to any relevant public discussion (GitHub issues, Ethereum Magicians threads). |
| [`proposed-diffs.md`](./proposed-diffs.md) | Replacement text, rationale, and backwards-compatibility classification for the simplified ERC-4361 erratum PR. |
| [`conformance-matrix.md`](./conformance-matrix.md) | Empirical audit of the proposed changes against the canonical `@signinwithethereum/test-vectors` corpus shared by the TypeScript, Python, Rust, and Go SIWE libraries. |
| [`erc-4361-snapshot.md`](./erc-4361-snapshot.md) | Frozen copy of `ERCS/erc-4361.md` at the revision audited (2026-04-23). Ensures line-number citations elsewhere in this repository remain valid even as upstream merges other PRs. |

## Reading order for reviewers

1. Start with the [PR description](https://github.com/signinwithethereum/ERCs/pull/1) for the erratum summary and per-commit classification table.
2. Drill into [`inconsistencies.md`](./inconsistencies.md) for the primary-source catalogue behind each finding.
3. Read [`proposed-diffs.md`](./proposed-diffs.md) for the per-commit replacement text and rationale.
4. Consult [`conformance-matrix.md`](./conformance-matrix.md) for the empirical backing on backwards compatibility.

## Scope boundaries

This repository is the **public evidence base** for the ERC-4361 erratum. It is scoped to technical evidence, proposed text, compatibility notes, and conformance data.

## Related pull requests

The erratum ships as a coordinated set of pull requests.

- **Main PR:** [signinwithethereum/ERCs#1](https://github.com/signinwithethereum/ERCs/pull/1) (preview, pre-submission). The simplified ERC-4361 erratum. Will move to `ethereum/ERCs` at submission.
- **Concurrent PR:** `signinwithethereum/test-vectors`, reclassifying the two `userinfo@` grammar-completeness vectors from positive to negative, and updating statement-character vectors for the proposed printable-ASCII widening. URL added when the PR opens.
- **Downstream PRs:** minor version bumps against `@signinwithethereum/siwe` (TS), `@signinwithethereum/siwe-py`, `@signinwithethereum/siwe-rs`, and the Go port, applying only the surviving parser changes. URLs added when the PRs open.

## License

Dedicated to the public domain under [CC0 1.0 Universal](./LICENSE), matching the Creative Commons Zero convention used by Ethereum ERCs.
