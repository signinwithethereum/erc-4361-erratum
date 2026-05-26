# ERC-4361 Inconsistencies

A catalogue of contradictions, ambiguities, and mismatches found in ERC-4361. Each finding cites the relevant location in [`erc-4361-snapshot.md`](./erc-4361-snapshot.md), so line numbers stay stable.

The erratum addresses every finding in the Critical, High, Medium, and Low sections; the Deferred section explains the issues we considered but chose not to ship as part of this erratum.

Severity reflects the practical impact on implementers:

- `Critical`: current text can produce incompatible conformance claims or contradict the reference implementation on valid examples.
- `High`: current text can break signatures, origin checks, or wallet/RP agreement if implemented literally.
- `Medium`: current text creates parser/producer ambiguity or mismatches grammar and prose.
- `Low`: editorial drift, stale naming, or documentation quality issues.
- `Deferred`: real concern, but not appropriate as a normative change in this erratum.

## Critical

### 1. Address checksum: MUST vs. SHOULD contradiction

**Where:** ABNF lines 71-74, field description line 113.

The ABNF comment says the address "Must" conform to the ERC-55 checksum, while the Message Fields section says the address value "SHOULD" be ERC-55 conformant where applicable.

One implementation can therefore reject lowercase or uppercase EOA addresses while another accepts them with warnings, and both can claim to be following the current document. The canonical `@signinwithethereum` test vectors split the difference: all-lowercase and all-uppercase addresses produce a warning, while wrong-checksum mixed-case addresses are rejected.

The erratum restates the ABNF comment to match that split — "if the address format is mixed-case, it MUST conform to its ERC-55 checksum" — and pins the address prefix as case-sensitive with `%s"0x"`.

**Public discussion:** No public discussion found that explicitly identifies the MUST vs. SHOULD split as of 2026-04-13.

### 2. The reference implementation does not match the published ABNF

**Where:** Reference Implementation link at line 350 -> `https://eips.ethereum.org/assets/eip-4361/example.js`.

The linked file diverges from the spec body in material ways:

- Its top-level grammar omits `[ scheme "://" ]`, so the spec's own explicit-scheme example fails against the reference implementation.
- Its statement production does not match the published grammar.
- Its `createMessage` helper cannot emit several optional fields, including `Resources:`.

Implementers who treat the linked file as authoritative can reject valid messages or produce incomplete SIWE messages while still thinking they are following ERC-4361.

**Public discussion:**

- `spruceid/siwe#30` reports reference implementation mismatches: <https://github.com/spruceid/siwe/issues/30>
- A later comment reports explicit-scheme parsing failure: <https://github.com/spruceid/siwe/issues/30#issuecomment-1666440625>
- `spruceid/siwe#195` adds optional-scheme support in the canonical TypeScript parser: <https://github.com/spruceid/siwe/pull/195>
- Ethereum Magicians follow-up: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>

## High

### 3. No specified character encoding for the wire format

**Where:** entire spec; statement field description line 114 mentions "ASCII" but no message-level encoding rule.

ERC-191 prefixes the signed payload with `\x19Ethereum Signed Message:\n<length of message>`, where length is byte length. ERC-4361 does not say which encoding determines those bytes.

Known implementations converge on UTF-8, so this is a documentation gap rather than observed ecosystem divergence. It is still load-bearing for interop because a different encoding would change the ERC-191 prefix length and therefore the signed payload.

**Public discussion:** No public discussion found in GitHub issues/PR comments, Ethereum Magicians, or general web/forum search as of 2026-04-13.

### 4. ERC-191 prefixing is assigned to two different parties

**Where:** Overview steps 1 and 2 (lines 31-33).

Step 1 says the relying party prefixes the SIWE message with the ERC-191 prefix. Step 2 says the wallet signs the SIWE message with the ERC-191 signed data format. If both happen literally, the message is double-prefixed and the signature covers a different payload.

In practice, the wallet or signing primitive applies the ERC-191 prefix exactly once. The erratum makes that explicit and says the relying party must not pre-prefix the plaintext SIWE message.

**Public discussion:**

- General EIP-191 vs EIP-712 discussion: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- Initial review discussion in the draft PR: <https://github.com/ethereum/EIPs/pull/4361>
- `go-ethereum` discussion about SIWE validation relative to `TextAndHash()`: <https://github.com/ethereum/go-ethereum/issues/24132#issuecomment-3971009338>

### 5. `domain = authority` admits a `userinfo@` component

**Where:** ABNF lines 65-69, prose lines 227 and 253.

The grammar permits authority values such as `trusted.com@evil.com:443`. The surrounding prose treats `domain` like a web origin, but a web origin is scheme + host + port and has no userinfo component. The wallet origin-verification algorithm compares host and port; it never says what to do with userinfo.

This is a real origin-display ambiguity and a plausible phishing surface. The erratum removes `userinfo` from the `domain` production. It is the only intentional grammar narrowing in the erratum, and it requires the two canonical userinfo grammar-completeness vectors to move from positive to negative.

**Public discussion:**

- Review comment questioning `authority` / `dnsauthority` for the domain field: <https://github.com/ethereum/EIPs/pull/4361#discussion_r748777532>
- The Magicians thread notes phishing-related wallet/security review: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- No public discussion found that explicitly demonstrates the `userinfo@` case as of 2026-04-13.

## Medium

### 6. The ABNF is not self-contained

**Where:** ABNF lines 81-82, 99-100, 65-69, and 104.

The grammar references `URI`, `pchar`, and `authority` as if they were terminals, but only RFC 5234 core rules and RFC 7405 `%s` are formally introduced. RFC 3986 productions appear only in comments.

The erratum explicitly imports the RFC 3986 productions that the grammar already depends on.

**Public discussion:** No public discussion found that explicitly calls out the non-self-contained ABNF problem as of 2026-04-13.

### 7. Omitted `scheme` gets two different defaults

**Where:** field description line 112, wallet algorithm lines 263-269.

The Message Fields section says HTTPS is assumed by default when `scheme` is omitted. The wallet origin-verification algorithm instead takes a `defaultScheme` input and only says browser wallets SHOULD use `https`.

The erratum resolves this by routing the line 112 default through the wallet's `defaultScheme`, so both passages agree on a single resolution path. The original SHOULD on browser `https` is preserved.

**Public discussion:** No public discussion found that explicitly identifies the conflicting omitted-scheme defaults as of 2026-04-13.

### 8. Wallet origin verification mixes full-host equality with weaker subdomain checks

**Where:** Wallet origin verification lines 272-273.

The recommended algorithm first says the wallet MUST reject if the host parts of `domain` and `origin` do not match. It then separately says the wallet SHOULD reject if subdomains mismatch.

Under RFC 3986/RFC 6454 terminology, host already includes all labels, including subdomains. The second rule is redundant or implies a weaker path that the first rule already forbids. The erratum keeps exact host comparison and drops the separate subdomain bullet.

**Public discussion:** No public discussion found that explicitly identifies the host-vs-subdomain contradiction as of 2026-04-13.

### 9. `statement` character set is narrower than "ASCII"

**Where:** ABNF lines 76-79, field description line 114.

The grammar limits `statement` to `reserved / unreserved / " "`, which excludes printable ASCII characters such as `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, and backtick. The field description instead calls the statement a "human-readable ASCII assertion" and only forbids LF.

The erratum widens the grammar to printable ASCII (`%x20-7E`) while preserving the current empty-statement compatibility rule.

**Public discussion:** No public discussion found that explicitly identifies the printable-ASCII vs. `reserved`/`unreserved` mismatch as of 2026-04-13.

### 10. Optional `statement` permits an empty form

**Where:** ABNF lines 46-50 and 76.

The grammar permits both an omitted statement and a present-but-empty statement. The canonical test vectors explicitly treat the empty form as valid. The erratum keeps parser acceptance, tells producers to omit the statement when no statement is present, and treats empty and omitted statements as equivalent for authentication semantics.

**Public discussion:**

- Holiman raised the optional/empty statement ambiguity on Ethereum Magicians: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- `spruceid/siwe#30` covers the implementation/spec mismatch: <https://github.com/spruceid/siwe/issues/30>
- `go-ethereum` implementation issue references the parser ambiguity: <https://github.com/ethereum/go-ethereum/issues/24132#issuecomment-1514897680>

### 11. `Resources:` header is allowed with zero resources

**Where:** ABNF lines 58-59 and 102-103.

`resources = *( LF resource )` permits a bare `Resources:` header. The canonical test vectors treat this as valid. The erratum keeps parser acceptance, tells producers not to emit a bare `Resources:` header, and treats it as semantically equivalent to omission.

**Public discussion:**

- `spruceid/siwe#30` calls out the resources mismatch: <https://github.com/spruceid/siwe/issues/30>
- Maintainer response on empty resources semantics: <https://github.com/spruceid/siwe/issues/30#issuecomment-1002164190>
- Magicians thread reference: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>

### 12. Empty `request-id` is grammatically valid

**Where:** ABNF lines 99-100.

`request-id = *pchar` permits `Request ID:` with an empty value. The canonical test vectors treat this as valid. The erratum keeps parser acceptance and adds producer guidance to omit the field when no identifier is available.

**Public discussion:** No public discussion found that explicitly identifies empty `request-id` permissiveness as of 2026-04-13.

### 13. Empty `domain` is grammatically valid

**Where:** ABNF lines 65-69, field description line 112.

Through RFC 3986, `host` may be a `reg-name`, and `reg-name = *(...)` permits the empty string. That permits a leading line with no domain at all:

```text
 wants you to sign in with your Ethereum account:
```

That conflicts with `domain` being REQUIRED and origin-identifying. The erratum adds a prose requirement that the host subcomponent must be non-empty.

**Public discussion:** No public discussion found that explicitly identifies the empty-domain parse path as of 2026-04-13.

## Low

### 14. The informal template inserts an invalid space after `://`

**Where:** Informal template line 130.

The template shows `${scheme}:// ${domain}` with a space after `://`, while the ABNF requires `[ scheme "://" ] domain`.

**Public discussion:** No public discussion found that explicitly identifies the extra-space bug as of 2026-04-13.

### 15. Verification prose refers to non-existent `request-uri`

**Where:** Verifying a signed Message line 231.

The SIWE Message defines a `uri` field, not `request-uri`.

**Public discussion:** No public discussion found that explicitly identifies the stale `request-uri` name as of 2026-04-13.

### 16. RFC 3339 is not identical to ISO 8601

**Where:** ABNF comment lines 96-97.

The comment says `RFC 3339 (ISO 8601)`. RFC 3339 is a strict profile of ISO 8601; only RFC 3339 date-time forms are permitted by ERC-4361's field descriptions.

**Public discussion:** No public discussion found that explicitly identifies the RFC 3339 / ISO 8601 wording issue as of 2026-04-13.

### 17. "EIP-55" vs. "ERC-55" naming drift

**Where:** ABNF comment line 73 ("EIP-55") vs. field description line 113 ("ERC-55").

The same standard is referenced under two names in adjacent paragraphs. The erratum uses ERC-55 consistently.

**Public discussion:** No public discussion found that explicitly identifies the naming drift as of 2026-04-13.

## Deferred

### 18. Percent-encoded LF in URIs and display framing

**Where:** ABNF lines 50, 81-82, 104, and wallet display rule line 281.

Percent-encoded line feeds in URI values can confuse a wallet UI that decodes and then visually treats decoded output as SIWE framing. No known parser re-frames decoded URIs today, and the proposed fix would be new wallet-display hardening rather than correction of a contradiction. This is deferred from the erratum.

### 19. IDN / punycode handling for `domain`

**Where:** ABNF line 65, Wallet origin verification lines 268-277.

The current spec is silent on IDNA versions, A-labels, U-labels, and Unicode case-folding. Choosing IDNA2008 A-label-only wire form would be new policy rather than erratum. This is deferred to follow-up work.

### 20. Empty explicit ports

**Where:** ABNF lines 65-69, Wallet origin verification lines 274-276.

`example.com:` is under-specified, but the current wallet algorithm has an explicit branch for empty ports. The erratum does not spend a standalone normative change on this pattern.

### 21. Leading-zero `chain-id`

**Where:** ABNF lines 86-87.

Leading-zero chain IDs are non-canonical, but forbidding them is grammar canonicalization rather than correction of a direct contradiction. This is deferred from the erratum.
