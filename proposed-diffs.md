---
title: Proposed diffs for ERC-4361 remediation
---

# Proposed diffs for ERC-4361 remediation

Companion to [`inconsistencies.md`](./inconsistencies.md). This simplified plan removes changes that read as new hardening policy rather than errata:

- Finding #9 (percent-decoded URI display framing) is deferred from the ERC diff.
- Finding #10 (IDN / punycode policy) is deferred from the ERC diff.
- Finding #15 (empty explicit port) is not a standalone normative change.
- Finding #17 (leading-zero `chain-id`) is deferred from the ERC diff.

The remaining changes either fix direct contradictions, align the grammar/prose/reference implementation, or make explicitly classified behavior changes with conformance evidence.

**Compatibility tags:**

- `Editorial`: typos, stale names, prose wording. No normative effect.
- `Clarification`: no intended behavior change in conforming implementations.
- `Narrow`: grammar/prose becomes stricter. Previously-valid messages become invalid.
- `Widen`: grammar becomes looser. Previously-invalid messages become valid.
- `Judgment`: a direction call between competing readings, justified by compatibility evidence.

Conventions:

- Diffs are shown as unified-diff fragments.
- "ABNF" refers to the block at lines 43-105 of [`erc-4361-snapshot.md`](./erc-4361-snapshot.md).
- "Message Fields" refers to lines 109-123.
- "Wallet origin verification" refers to lines 267-277.

---

## Commit 1: Finding #18 (extra space after `://` in informal template)

**Tag:** Editorial.

**Replacement:** line 130.

```diff
-${scheme}:// ${domain} wants you to sign in with your Ethereum account:
+${scheme}://${domain} wants you to sign in with your Ethereum account:
```

**Rationale:** The ABNF is `[ scheme "://" ] domain` with no separator. The informal template should not contradict it.

**Compatibility:** Template-only. No producer or parser impact.

---

## Commit 2: Finding #19 (stale `request-uri` reference)

**Tag:** Editorial.

**Replacement:** line 231.

```diff
-- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `request-uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
+- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
```

**Rationale:** There is no `request-uri` field; the correct field is `uri`.

**Compatibility:** Prose-only. No impact.

---

## Commit 3: Finding #20 (RFC 3339 is a profile of ISO 8601)

**Tag:** Editorial.

**Replacement:** lines 96-97.

```diff
 issued-at = date-time
 expiration-time = date-time
 not-before = date-time
-    ; See RFC 3339 (ISO 8601) for the
-    ; definition of "date-time".
+    ; See RFC 3339 for date-time. RFC 3339 is
+    ; a profile of ISO 8601; only RFC 3339
+    ; forms are permitted here.
```

**Rationale:** The parenthetical equivalence is imprecise. The field descriptions already cite RFC 3339 as the normative date-time form.

**Compatibility:** Comment-only. No impact.

---

## Commit 4: Finding #21 (`EIP-55` vs. `ERC-55`)

**Tag:** Editorial.

**Replacement:** line 73.

```diff
 address = "0x" 40*40HEXDIG
     ; Must also conform to capitalization
-    ; checksum encoding specified in EIP-55
+    ; checksum encoding specified in ERC-55
     ; where applicable (EOAs).
```

**Rationale:** The Message Fields section says ERC-55. Use the current ERC name consistently.

**Compatibility:** Naming only. No impact.

---

## Commit 5: Finding #2 (ERC-191 prefixing assigned to two parties)

**Tag:** Clarification.

**Replacement:** lines 32-33.

```diff
-1. The relying party generates a SIWE Message and prefixes the SIWE Message with `\x19Ethereum Signed Message:\n<length of message>` as defined in [ERC-191](./eip-191.md).
-2. The wallet presents the user with a structured plaintext message or equivalent interface for signing the SIWE Message with the [ERC-191](./eip-191.md) signed data format.
+1. The relying party generates a SIWE Message (the plaintext payload, without any prefix) and transmits it to the wallet.
+2. The wallet presents the user with a structured plaintext message or equivalent interface, and, upon user consent, signs the SIWE Message using the [ERC-191](./eip-191.md) signed data format. The ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) is applied by the signing primitive exactly once; the relying party MUST NOT pre-prefix the message.
```

**Rationale:** If both the relying party and wallet/signing primitive apply the ERC-191 prefix, signatures are over a double-prefixed payload. Existing implementations prefix at signing time.

**Compatibility:** Aligns prose with current behavior. No wire change for conforming implementations.

---

## Commit 6: Finding #7 (omitted-`scheme` default disagreement)

**Tag:** Clarification.

**Replacement:** line 112 and line 264.

Line 112:

```diff
-- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority. The authority includes an OPTIONAL port. If the port is not specified, the default port for the provided `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified, HTTPS is assumed by default.
+- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority (see grammar below for restrictions on the `host` subcomponent). The authority includes an OPTIONAL port. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
```

Line 264:

```diff
-- `defaultScheme` - a scheme to assume when none was provided. Wallet implementers in the browser SHOULD use `https`.
+- `defaultScheme` - the scheme to assume when none was provided in the message. Wallet implementers in the browser MUST use `https`; non-browser wallet implementers MAY choose a different value appropriate to their transport.
```

**Rationale:** The Message Fields section reads like a protocol-wide HTTPS default, while the wallet algorithm uses an implementation input. This keeps `defaultScheme` but makes browser behavior match the prose.

**Compatibility:** Browser wallets already use `https`. No wire change.

---

## Commit 7: Finding #8 (redundant host/subdomain rule)

**Tag:** Clarification.

**Replacement:** lines 272-273.

```diff
-- If the `host` part of the `domain` and `origin` do not match, the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
-- If `domain` and `origin` have mismatching subdomains, the Wallet SHOULD reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
+- If the `host` subcomponent of `domain` and the host of `origin` do not match exactly, the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continue processing the request. Because `host` per RFC 3986 already includes the full hostname (all labels, including subdomains), a mismatched subdomain is a mismatched host and is covered by this rule.
```

**Rationale:** Host equality already covers subdomains. Keeping a separate weaker subdomain rule invites inconsistent readings.

**Compatibility:** Any wallet already doing host equality behaves unchanged.

---

## Commit 8: Finding #5 (ABNF not self-contained)

**Tag:** Clarification.

**Replacement:** insert after line 41 and add an imports comment inside the ABNF.

Before ABNF:

```diff
-A SIWE Message MUST conform with the following Augmented Backus-Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression (note that `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405)).
+A SIWE Message MUST conform with the following Augmented Backus-Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression. `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405). The grammar also normatively imports the following productions from [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986): `URI`, `authority`, `host`, `port`, `pchar`, `reserved`, and `unreserved`.
```

Inside ABNF:

```diff
 ```abnf
 sign-in-with-ethereum =
     [ scheme "://" ] domain %s" wants you to sign in with your Ethereum account:" LF
     ...
 
+; Imported from RFC 3986:
+;     URI, authority, host, port, pchar, reserved, unreserved.
+; Imported from RFC 5234 Appendix B.1:
+;     ALPHA, DIGIT, HEXDIG, LF.
+; Imported from RFC 7405:
+;     %s (case-sensitive string literal prefix).
+
 scheme = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )
 ```
```

**Rationale:** The grammar already depends on RFC 3986 productions. The erratum makes that dependency explicit.

**Compatibility:** No intended wire change.

---

## Commit 9: Finding #1 (character encoding of the wire format)

**Tag:** Clarification.

**Replacement:** insert after line 41, near the ABNF introduction.

```diff
 A SIWE Message MUST conform with the following Augmented Backus-Naur Form ...
+
+A SIWE Message is a sequence of Unicode code points encoded as UTF-8. All ABNF productions below describe the UTF-8 byte stream of the message. The `<length of message>` component of the ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) MUST be computed as the UTF-8 byte length of the SIWE Message.
```

**Rationale:** ERC-191 length is byte length. Known implementations use UTF-8, and the spec should say so.

**Compatibility:** No known implementation uses another encoding.

---

## Commit 10: Finding #3 (ERC-55 MUST vs. SHOULD)

**Tag:** Judgment. Harmonized down to SHOULD.

**Replacement:** lines 71-74.

```diff
-address = "0x" 40*40HEXDIG
-    ; Must also conform to capitalization
-    ; checksum encoding specified in ERC-55
+address = %s"0x" 40*40HEXDIG
+    ; SHOULD also conform to the mixed-case
+    ; checksum encoding specified in ERC-55
     ; where applicable (EOAs).
```

**Rationale:** The ABNF comment says mandatory checksum; the Message Fields section says SHOULD. The canonical vectors warn on all-lowercase/all-uppercase addresses and reject only wrong-checksum mixed-case addresses. This supports the SHOULD reading. The same hunk also pins the `0x` prefix as case-sensitive.

**Compatibility:** Existing producers emitting ERC-55 addresses remain conforming. Existing parsers that accept non-checksummed addresses remain conforming.

---

## Commit 11: Findings #6 and #16 (exclude `userinfo`, require non-empty host)

**Tag:** Narrow.

**Replacement:** line 65 and line 112.

ABNF:

```diff
-domain = authority
-    ; From RFC 3986:
-    ;     authority     = [ userinfo "@" ] host [ ":" port ]
-    ; See RFC 3986 for the fully contextualized
-    ; definition of "authority".
+domain = host [ ":" port ]
+    ; host and port are imported from RFC 3986.
+    ; The userinfo component of RFC 3986
+    ; authority is intentionally excluded
+    ; from SIWE domain values.
```

Message Fields, extending Commit 6:

```diff
- ... If `scheme` is not specified in the message, the wallet's `defaultScheme` ... browser wallets' `defaultScheme` MUST be `https`.
+ ... If `scheme` is not specified in the message, the wallet's `defaultScheme` ... browser wallets' `defaultScheme` MUST be `https`. The `host` subcomponent MUST be non-empty.
```

**Rationale:** `domain = authority` permits `userinfo@host`, but SIWE origin comparison only uses scheme, host, and port. Excluding `userinfo` aligns the grammar with the origin model. The non-empty-host rule makes the REQUIRED `domain` field actually identify an origin host.

**Compatibility:** Narrows. `@signinwithethereum/test-vectors` currently has two positive `userinfo@` grammar-completeness vectors. The concurrent test-vector PR moves them to negative. No production relying party is known to emit `userinfo@host` in SIWE messages.

---

## Commit 12: Findings #12, #13, and #14 (empty optional field semantics)

**Tag:** Clarification.

**Replacement:** prose only. ABNF unchanged.

Statement field:

```diff
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
+- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present rather than emitting an empty statement. Parsers MUST accept both empty and omitted statements. For authentication semantics, an empty statement is equivalent to an omitted statement.
```

Request ID field:

```diff
-- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request.
+- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request. Producers SHOULD NOT emit an empty `request-id`; when no identifier is available, the `Request ID:` header is to be omitted entirely. Parsers MUST continue to accept messages that contain an empty `request-id` value.
```

Resources field:

```diff
-- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`.
+- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`. Producers SHOULD NOT emit a bare `Resources:` header with no following resources; when no resources are present, the entire `[ LF %s"Resources:" resources ]` production is to be omitted. Parsers MUST accept a bare `Resources:` header as semantically equivalent to omission.
```

**Rationale:** The canonical test vectors treat empty `statement`, empty `request-id`, and bare `Resources:` as valid. The erratum preserves parser behavior and gives producer-side canonicalization guidance.

**Compatibility:** No parser change. No message moves from valid to invalid.

---

## Commit 13: Finding #11 (`statement` charset too narrow)

**Tag:** Widen.

**Replacement:** lines 76-79 and line 114.

ABNF:

```diff
-statement = *( reserved / unreserved / " " )
-    ; See RFC 3986 for the definition
-    ; of "reserved" and "unreserved".
-    ; The purpose is to exclude LF (line break).
+statement = *( %x20-7E )
+    ; Printable ASCII excluding LF (0x0A)
+    ; and other control characters.
```

Message Fields, replacing the statement text from Commit 12:

```diff
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). Producers SHOULD omit ...
+- `statement` OPTIONAL. A human-readable assertion that the user will sign. If emitted, the statement is a sequence of zero or more printable ASCII characters (bytes `0x20` through `0x7E`) and MUST NOT include `'\n'` (the byte `0x0a`) or any other control character. Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present rather than emitting an empty statement. Parsers MUST accept both empty and omitted statements. For authentication semantics, an empty statement is equivalent to an omitted statement.
```

**Rationale:** The current grammar excludes common printable ASCII punctuation even though the prose describes an ASCII assertion and only forbids LF. Widening to printable ASCII makes the grammar match the prose while preserving empty-statement compatibility.

**Compatibility:** Widens. Producers using `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, or backtick in statements become conforming. Parser updates should roll out before producers rely on the wider set.

---

## Commit 14: Finding #4 (reference implementation)

**Tag:** Clarification.

**Replacement:** edits to `assets/eip-4361/example.js`.

1. Add `[ scheme "://" ]` handling to the top-level parser so explicit-scheme examples parse.
2. Change the statement grammar to match Commit 13's `*( %x20-7E )`, with a helper production for the statement/blank-line separator so the reference parser accepts both no-statement and non-empty-statement layouts.
3. Extend `createMessage` to accept and emit `expirationTime`, `notBefore`, `requestId`, and non-empty `resources`.
4. Add the three ERC-4361 example messages as lightweight regression fixtures in comments.

**Rationale:** The reference implementation should match the cumulative ERC text, especially for the spec's own explicit-scheme example.

**Compatibility:** The reference implementation is non-normative. This aligns it with the spec body and canonical parser behavior.

---

## Summary table

| Commit | Finding(s) | Tag | Notes |
| --- | --- | --- | --- |
| 1 | #18 | Editorial | Template spacing |
| 2 | #19 | Editorial | `request-uri` -> `uri` |
| 3 | #20 | Editorial | RFC 3339 wording |
| 4 | #21 | Editorial | ERC-55 naming |
| 5 | #2 | Clarification | ERC-191 prefix once |
| 6 | #7 | Clarification | effective scheme/defaultScheme |
| 7 | #8 | Clarification | exact host comparison |
| 8 | #5 | Clarification | explicit grammar imports |
| 9 | #1 | Clarification | UTF-8 wire encoding |
| 10 | #3 | Judgment | ERC-55 SHOULD and `%s"0x"` |
| 11 | #6, #16 | Narrow | exclude userinfo; host non-empty |
| 12 | #12, #13, #14 | Clarification | empty optional-field semantics |
| 13 | #11 | Widen | printable ASCII statement |
| 14 | #4 | Clarification | reference implementation |

Deferred from this erratum: #9, #10, #15, #17.
