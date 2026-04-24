# ERC-4361 Inconsistencies

A catalogue of internal contradictions, ambiguities, and grammar/prose mismatches found in [`erc-4361-snapshot.md`](./erc-4361-snapshot.md). Line numbers refer to the local copy.

The items below are ordered by implementation priority rather than discovery order:

- `Critical`: can directly break signatures, parsing interoperability, or conformance claims.
- `High`: materially increases security risk or leaves core validation behavior under-specified.
- `Medium`: creates ambiguous parsing, non-canonical encodings, or producer/parser mismatch.
- `Low`: editorial drift, stale naming, or documentation quality issues.

## Critical

### 1. No specified character encoding for the wire format

**Where:** entire spec; statement field description line 114 mentions "ASCII" but no field-level rule.

The spec never says that the byte stream which gets ERC-191-prefixed and signed must be UTF-8, ASCII, or anything else. The ERC-191 prefix is `\x19Ethereum Signed Message:\n<length of message>` where `<length of message>` is the **byte** length of the payload, but byte length is encoding-dependent. Two wallets that disagree on whether the SIWE payload is UTF-8 vs. UTF-16 vs. Latin-1 will compute different prefix lengths and produce different signatures over the same logical message.

This is arguably the most consequential interop omission in the document.

**Public discussion:** No public discussion found in GitHub issues/PR comments, Ethereum Magicians, or general web/forum search as of 2026-04-13.

### 2. ERC-191 prefixing is assigned to two different parties

**Where:** Overview steps 1 and 2 (lines 31-33).

> 1. The relying party generates a SIWE Message **and prefixes** the SIWE Message with `\x19Ethereum Signed Message:\n<length of message>` as defined in ERC-191.
> 2. The wallet presents the user with a structured plaintext message or equivalent interface for **signing the SIWE Message with the ERC-191 signed data format**.

If the relying party prefixes the message and the wallet then signs it as ERC-191 data, which prefixes again, the result is a double prefix and therefore a different signed payload than expected. In practice, only the wallet/signing primitive applies the ERC-191 prefix; step 1's wording is incorrect.

**Public discussion:**

- General EIP-191 vs EIP-712 discussion in the Ethereum Magicians thread: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- Initial review discussion in the draft PR: <https://github.com/ethereum/EIPs/pull/4361>
- `go-ethereum` implementation discussion about where SIWE validation must happen relative to `TextAndHash()` / ERC-191 prefixing: <https://github.com/ethereum/go-ethereum/issues/24132#issuecomment-3971009338>
- No public discussion found that explicitly calls out the two-party/double-prefix contradiction.

### 3. Address checksum: MUST vs. SHOULD contradiction, plus latent `"0x"` case lenience

**Where:** ABNF lines 71-74, field description line 113.

```abnf
address = "0x" 40*40HEXDIG
    ; Must also conform to capitalization
    ; checksum encoding specified in EIP-55
    ; where applicable (EOAs).
```

The ABNF comment says the address "**Must** also conform to ... EIP-55." The Message Fields description at line 113 downgrades the same requirement to:

> Its value **SHOULD** be conformant to mixed-case checksum address encoding specified in ERC-55 where applicable.

One reads as mandatory, the other as recommended. An implementation that takes the ABNF comment as normative will reject a non-checksummed EOA address; one following the Message Fields section will accept it. Because conforming parsers and producers can therefore disagree on what counts as a valid SIWE message, this is a conformance-claim split rather than a grammar bug.

#### 3a. Note on case-sensitivity of the ABNF itself

It is tempting to read the production as structurally unable to express an EIP-55-encoded (mixed-case) address, because [RFC 5234 Appendix B.1](https://www.rfc-editor.org/rfc/rfc5234#appendix-B.1) defines `HEXDIG` with uppercase letters only. In fact, [RFC 5234 §2.3](https://www.rfc-editor.org/rfc/rfc5234#section-2.3) makes ABNF string literals case-**in**sensitive by default, and [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405)'s `%s` prefix is the opt-in mechanism for case-sensitive matching. This spec uses `%s` for its field labels (`%s"URI: "`, `%s"Version: "`, …) but does **not** use it on `HEXDIG` or on the `"0x"` prefix. Consequently:

- Mixed-case hex digits (and therefore EIP-55-encoded addresses) are structurally valid against the grammar — no change to `HEXDIG` is needed to admit them.
- As a side effect, the literal `"0x"` is also case-insensitive, so `0X…` would parse. Neither EIP-55 nor the field description expects that; an explicit `%s"0x"` or a `%x30 %x78` equivalent would close that gap.

**Public discussion:** No public discussion found that explicitly identifies the MUST vs. SHOULD split, or the `"0x"` / `0X` case-lenience in the ABNF, as of 2026-04-13.

### 4. The reference implementation does not match the published ABNF

**Where:** Reference Implementation link at line 350 -> `https://eips.ethereum.org/assets/eip-4361/example.js`.

The linked file diverges from the spec body in several material ways:

- Its top-level grammar omits the `[ scheme "://" ]` production, so the spec's own explicit-scheme examples (lines 188-204) will not parse against it.
- It defines `statement = 1*( reserved / unreserved / " " )` (one-or-more) instead of the spec's `*( ... )` (zero-or-more), inverting the optionality semantics for a zero-length statement.
- Its `createMessage` function does not accept `expirationTime`, `notBefore`, `requestId`, or `resources`; there is no `Resources:` emission code path at all.

Implementers who treat the linked file as authoritative will produce parsers that reject conforming `https://example.com ...` messages and producers that cannot emit `Resources:` blocks while still claiming SIWE conformance.

**Public discussion:**

- `spruceid/siwe#30` explicitly reports that the reference implementation is not in line with the spec, covering the `statement` and `resources` mismatches: <https://github.com/spruceid/siwe/issues/30>
- A later comment on the same issue reports that valid spec-conforming messages with an explicit scheme failed to parse: <https://github.com/spruceid/siwe/issues/30#issuecomment-1666440625>
- `spruceid/siwe#195` was later merged specifically to add support for the optional scheme in the message header: <https://github.com/spruceid/siwe/pull/195>
- Ethereum Magicians follow-up linking to that issue: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>

### 5. The ABNF is not self-contained

**Where:** ABNF lines 81-82, 99-100, 65-69 and 104.

The grammar references `URI`, `pchar`, and `authority` as if they were terminals, but only RFC 5234 core rules (`ALPHA`, `DIGIT`, `HEXDIG`, `LF`) and RFC 7405 (`%s`) are formally pulled in. RFC 3986's productions appear only in code comments, and the reference implementation has to inline about 80 lines of RFC 3986 grammar to make the spec parseable.

Two implementers using two different in-house copies of RFC 3986, or different revisions of BCP 13 / RFC 3987 IRI productions, can disagree about what is conforming, particularly for IPv6 hosts, IPvFuture, and percent-encoded `pchar`.

**Public discussion:** No public discussion found that explicitly calls out the non-self-contained ABNF problem as of 2026-04-13.

## High

### 6. `domain = authority` admits a `userinfo@` component

**Where:** ABNF lines 65-69, prose lines 227 and 253.

```abnf
domain = authority
    ; authority = [ userinfo "@" ] host [ ":" port ]
```

The grammar permits `domain` values like `admin@example.com:443`. The spec, however, treats `domain` as equivalent to a web origin (RFC 6454: scheme + host + port, no `userinfo`). The Wallet Implementer algorithm (lines 268-277) compares only `host`, subdomains, and `port` against the request `origin`; it never describes what to do if `domain` contains a `userinfo` component.

This mismatch is also a phishing surface: a message starting with

```text
trusted.com@evil.com wants you to sign in with your Ethereum account:
```

is structurally valid SIWE, displays a recognizable prefix, but is sourced from `evil.com`.

**Public discussion:**

- Review comment questioning the use of `authority`/`dnsauthority` for the `domain` field: <https://github.com/ethereum/EIPs/pull/4361#discussion_r748777532>
- The Magicians thread notes that SamWilsn contributed phishing-related wallet/security improvements during spec review: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- No public discussion found that explicitly demonstrates the `userinfo@` phishing case.

### 7. Omitted `scheme` gets two different defaults

**Where:** field description line 112, wallet algorithm lines 263-269.

The Message Fields section says:

> If `scheme` is not specified, HTTPS is assumed by default.

But the wallet origin-verification algorithm takes `defaultScheme` as an input variable and only says:

- browser wallets SHOULD use `https`
- if `scheme` was not provided, assign `defaultScheme`

Those are not equivalent. One text reads like a protocol-wide default of HTTPS; the other leaves the default scheme implementation-defined outside browser wallets. A relying party or parser that assumes omitted `scheme` means `https` can therefore disagree with a wallet following a different `defaultScheme`.

**Public discussion:** No public discussion found that explicitly identifies the conflicting omitted-`scheme` defaults as of 2026-04-13.

### 8. Wallet origin verification mixes full-host equality with weaker subdomain checks

**Where:** Wallet origin verification lines 272-273.

The RECOMMENDED algorithm says:

- If the `host` part of `domain` and `origin` do not match, the wallet MUST reject.
- If `domain` and `origin` have mismatching subdomains, the wallet SHOULD reject.

In RFC 3986 / RFC 6454 terms, the `host` already includes the full hostname, including subdomains. If the subdomains mismatch, then the hosts do not match, which triggers the stronger prior MUST-reject rule. The weaker subdomain rule is therefore redundant or unreachable unless "host" is being used in some non-standard sense that the specification never defines.

**Public discussion:** No public discussion found that explicitly identifies the host-vs-subdomain contradiction as of 2026-04-13.

### 9. Percent-encoded LF in URIs can break line-based framing

**Where:** ABNF lines 50, 81-82, 104, vs. wallet display rule line 281.

`uri = URI` and `resource = "- " URI`. RFC 3986's `URI` production permits percent-encoded octets in the path/query/fragment, so a URI value containing `%0A` is conformant. Wallets are told to render `resources` as a list to the user, and many will percent-decode for display.

A producer that emits

```text
URI: https://evil.example/foo%0AVersion:%202
```

and a parser/UI that decodes before display can show what looks like a second `Version:` field. The grammar is line-oriented (`resources = *( LF resource )`), but the values it carries are not, and the spec never tells implementers when, or whether, to percent-decode.

**Public discussion:** No public discussion found that explicitly identifies percent-decoded line-break/framing risk as of 2026-04-13.

### 10. IDN/punycode handling for `domain` is undefined

**Where:** ABNF line 65, Wallet origin verification lines 268-277.

`domain = authority` with `host = IP-literal / IPv4address / reg-name`. RFC 3986's `reg-name` is bytes; RFC 3987 IRIs are not invoked. The Wallet algorithm's bullet "If the `host` part of the `domain` and `origin` do not match" (line 272) does not define "match" for IDN values: U-label vs. A-label, IDNA2003 vs. IDNA2008, or case-folding rules.

**Public discussion:** No public discussion found that explicitly identifies IDN/punycode matching ambiguity as of 2026-04-13.

### 11. `statement` character set is far narrower than "ASCII"

**Where:** ABNF lines 76-79, field description line 114.

```abnf
statement = *( reserved / unreserved / " " )
    ; The purpose is to exclude LF (line break).
```

Per RFC 3986, `reserved` union `unreserved` union `" "` covers only:

```text
A-Z a-z 0-9 - . _ ~ : / ? # [ ] @ ! $ & ' ( ) * + , ; = SP
```

The following printable ASCII characters are **excluded**: `"` `%` `<` `>` `{` `}` `|` `\` `^` `` ` ``.

Statements such as `I accept the "Terms of Service"` or `Balance < 100` are therefore invalid per the grammar, despite the field description (line 114) calling it "a human-readable ASCII assertion ... which MUST NOT include `'\n'`." The stated purpose ("exclude LF") does not match the actual restriction.

**Public discussion:** No public discussion found that explicitly identifies the printable-ASCII vs `reserved`/`unreserved` mismatch as of 2026-04-13.

## Medium

### 12. Optional `statement` produces ambiguous blank-line layout

**Where:** ABNF lines 46-50.

```abnf
address LF
LF
[ statement LF ]
LF
%s"URI: " uri LF
```

- With a statement: `address \n \n statement \n \n URI:...`
- Without a statement: `address \n \n \n URI:...` (two consecutive blank lines)

Combined with inconsistency #11, `statement = *( ... )` allows a zero-length statement, so "no statement" and "empty statement" can collapse to identical byte sequences and a parser cannot reliably tell whether a statement was intentionally omitted.

**Public discussion:**

- `holiman` explicitly raised the optional/empty `statement` ambiguity on Ethereum Magicians, with examples and a proposed fix to `1*(...)`: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>
- `spruceid/siwe#30` also covers the implementation/spec mismatch for empty-vs-present `statement`: <https://github.com/spruceid/siwe/issues/30>
- `go-ethereum` implementation issue where Holiman points contributors back to these spec issues while working on a parser: <https://github.com/ethereum/go-ethereum/issues/24132#issuecomment-1514897680>

### 13. `Resources:` header is allowed with zero resources

**Where:** ABNF lines 58-59 and 102-103.

```abnf
[ LF %s"Resources:"
resources ]
```

```abnf
resources = *( LF resource )
```

`*( LF resource )` permits zero items, so a message may legitimately end with a bare `Resources:` line. The spec never distinguishes "Resources omitted" from "Resources present but empty," and the field description (line 123) only describes the non-empty case. `1*( LF resource )` would have been the intended grammar.

**Public discussion:**

- `spruceid/siwe#30` explicitly calls out the `resources` zero-or-more vs one-or-more mismatch between spec and implementation: <https://github.com/spruceid/siwe/issues/30>
- Maintainer response discussing whether an explicitly empty `Resources:` section should remain distinguishable from omission: <https://github.com/spruceid/siwe/issues/30#issuecomment-1002164190>
- The same issue is referenced from the Magicians thread: <https://ethereum-magicians.org/t/eip-4361-sign-in-with-ethereum/7263>

### 14. Empty `request-id` is grammatically valid

**Where:** ABNF lines 99-100.

```abnf
request-id = *pchar
```

`*pchar` allows the empty string, so `Request ID: \n` parses successfully. The field description (line 122) describes `request-id` as "a system-specific identifier"; an empty identifier is unlikely to be intended but is not forbidden.

**Public discussion:** No public discussion found that explicitly identifies empty `request-id` permissiveness as of 2026-04-13.

### 15. Empty explicit ports are allowed and under-specified

**Where:** ABNF lines 65-69, Wallet origin verification lines 274-276.

Because `domain = authority` and RFC 3986 defines `port = *DIGIT`, values like `example.com:` are grammatically valid. The recommended wallet algorithm distinguishes between:

- a present `port` that does not match `origin`
- an absent `port`, in which case the default port is assumed

but it does not say how an explicit empty port should be treated. It is neither a normal concrete port value nor a clean omission of the port component.

There is also a small internal logic problem in the RECOMMENDED algorithm: step 274 assigns a default port when the port is absent, so step 276 (`If port is empty ...`) is only reachable if an explicit empty port such as `example.com:` survived parsing.

**Public discussion:** No public discussion found that explicitly identifies empty-port handling ambiguity as of 2026-04-13.

### 16. Empty `domain` is grammatically valid

**Where:** ABNF lines 65-69, field description line 112.

```abnf
domain = authority
    ; authority = [ userinfo "@" ] host [ ":" port ]
```

Through RFC 3986, `host` may be a `reg-name`, and `reg-name = *(...)` permits the empty string. As a result, the leading line of the message can begin with no domain at all:

```text
 wants you to sign in with your Ethereum account:
```

That is difficult to reconcile with `domain` being described as a REQUIRED field identifying the requesting domain.

**Public discussion:** No public discussion found that explicitly identifies the empty-`domain` parse path as of 2026-04-13.

### 17. `chain-id = 1*DIGIT` permits leading zeros

**Where:** ABNF lines 86-87.

```abnf
chain-id = 1*DIGIT
    ; See EIP-155 for valid CHAIN_IDs.
```

Strings like `01`, `001`, and `1` all parse, with no rule for whether they denote the same chain. EIP-155 chain IDs are integers, so leading zeros are at best non-canonical; the spec does not say whether they should be rejected, normalized, or treated as distinct.

**Public discussion:** No public discussion found that explicitly identifies leading-zero `chain-id` ambiguity as of 2026-04-13.

## Low

### 18. The informal template inserts an invalid space after `://`

**Where:** Informal template line 130.

```text
${scheme}:// ${domain} wants you to sign in with your Ethereum account:
```

The ABNF requires the optional scheme prefix to be `[ scheme "://" ] domain` with no intervening space. The template therefore renders an example like `https:// example.com ...`, which does not conform to the grammar it is supposed to illustrate.

**Public discussion:** No public discussion found that explicitly identifies the extra-space bug in the informal template as of 2026-04-13.

### 19. Verification prose refers to non-existent `request-uri`

**Where:** Verifying a signed Message line 231.

> checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `request-uri` etc.)

The SIWE Message defines a `uri` field, not `request-uri`. This looks like a stale name from an earlier draft and creates avoidable ambiguity in the verification guidance.

**Public discussion:** No public discussion found that explicitly identifies the stale `request-uri` name as of 2026-04-13.

### 20. RFC 3339 != ISO 8601

**Where:** ABNF comment lines 96-97.

```text
; See RFC 3339 (ISO 8601) for the
; definition of "date-time".
```

RFC 3339 is a *profile* of ISO 8601, not a synonym. ISO 8601 permits forms such as `2021-09-30T16:25:24+00`, basic format `20210930T162524Z`, and ordinal dates `2021-273` that RFC 3339 does not. The parenthetical equivalence is misleading; the field descriptions correctly cite only RFC 3339.

**Public discussion:** No public discussion found that explicitly identifies the RFC 3339/ISO 8601 wording issue as of 2026-04-13.

### 21. "EIP-55" vs. "ERC-55" naming drift

**Where:** ABNF comment line 73 ("EIP-55") vs. field description line 113 ("ERC-55").

The same standard is referenced under two names in adjacent paragraphs. Minor, but illustrative of the spec's general drift between the grammar block and the prose that explains it.

**Public discussion:** No public discussion found that explicitly identifies the `EIP-55` vs `ERC-55` naming drift as of 2026-04-13.
