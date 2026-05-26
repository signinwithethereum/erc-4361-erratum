---
title: Erratum commit series for ERC-4361
---

# Erratum commit series for ERC-4361

This file documents the 19 commits that landed on the [`review/4361`](https://github.com/signinwithethereum/ERCs/tree/review/4361) branch in [`signinwithethereum/ERCs#1`](https://github.com/signinwithethereum/ERCs/pull/1), one section per commit, in PR order. Each section shows the upstream diff hunk, the finding(s) addressed, and a short rationale and compatibility note.

## Conventions

- Diff hunks are reproduced verbatim from the upstream commits.
- Finding numbers reference [`inconsistencies.md`](./inconsistencies.md).
- Tags:
  - `editorial`: typos, stale names, prose wording. No normative effect.
  - `clarification`: no intended behavior change in conforming implementations.
  - `narrow`: grammar/prose becomes stricter. Previously-valid messages become invalid.
  - `widen`: grammar becomes looser. Previously-invalid messages become valid.
  - `judgment`: a direction call between competing readings, justified by compatibility evidence.

## Commit index

| # | SHA | Tag | Subject | Finding(s) |
| --- | --- | --- | --- | --- |
| 1 | [`a11d2af3`](https://github.com/signinwithethereum/ERCs/commit/a11d2af3) | editorial | Remove invalid space after `://` in informal template | 14 |
| 2 | [`e7a56390`](https://github.com/signinwithethereum/ERCs/commit/e7a56390) | editorial | Fix stale `request-uri` reference | 15 |
| 3 | [`ceac03e5`](https://github.com/signinwithethereum/ERCs/commit/ceac03e5) | editorial | Clarify RFC 3339 is a profile of ISO 8601 | 16 |
| 4 | [`1794b8e4`](https://github.com/signinwithethereum/ERCs/commit/1794b8e4) | editorial | Normalize EIP-55 to ERC-55 in ABNF comment | 17 |
| 5 | [`7aeadf14`](https://github.com/signinwithethereum/ERCs/commit/7aeadf14) | clarification | Assign ERC-191 prefixing to wallet/signing primitive | 4 |
| 6 | [`8ff67368`](https://github.com/signinwithethereum/ERCs/commit/8ff67368) | clarification | Harmonize omitted-`scheme` default via wallet `defaultScheme` | 7 |
| 7 | [`88f0fd19`](https://github.com/signinwithethereum/ERCs/commit/88f0fd19) | clarification | Drop redundant subdomain-mismatch rule | 8 |
| 8 | [`f62bb494`](https://github.com/signinwithethereum/ERCs/commit/f62bb494) | clarification | Formalize RFC 3986/5234/7405 grammar imports | 6 |
| 9 | [`ae9bcbb5`](https://github.com/signinwithethereum/ERCs/commit/ae9bcbb5) | clarification | Declare UTF-8 as the wire encoding | 3 |
| 10 | [`cfef9ef5`](https://github.com/signinwithethereum/ERCs/commit/cfef9ef5) | judgment | Pin `%s"0x"` and align ERC-55 toward SHOULD | 1 |
| 11 | [`d7f3ee3a`](https://github.com/signinwithethereum/ERCs/commit/d7f3ee3a) | narrow | Require non-empty domain host, exclude `userinfo` | 5, 13 |
| 12 | [`4b2cecff`](https://github.com/signinwithethereum/ERCs/commit/4b2cecff) | clarification | Empty optional field forms equate to omission | 10, 11, 12 |
| 13 | [`2e54091a`](https://github.com/signinwithethereum/ERCs/commit/2e54091a) | widen | Widen `statement` charset to printable ASCII excluding LF | 9 |
| 14 | [`ab3818b8`](https://github.com/signinwithethereum/ERCs/commit/ab3818b8) | clarification | Align reference implementation with cumulative grammar | 2 |
| 15 | [`3a7bc36f`](https://github.com/signinwithethereum/ERCs/commit/3a7bc36f) | clarification | Fix `statement-section` LF and emit `statement` in example | 2 |
| 16 | [`a1ec1a49`](https://github.com/signinwithethereum/ERCs/commit/a1ec1a49) | editorial | Normalize EIP/ERC link paths in the document | — |
| 17 | [`7578829d`](https://github.com/signinwithethereum/ERCs/commit/7578829d) | clarification | Restore browser `https` default to SHOULD | 7 |
| 18 | [`70447030`](https://github.com/signinwithethereum/ERCs/commit/70447030) | judgment | Restate ERC-55: mixed-case addresses MUST be checksum-valid | 1 |
| 19 | [`2663fff7`](https://github.com/signinwithethereum/ERCs/commit/2663fff7) | editorial | Sync reference grammar comment with ERC body | 1 |

Deferred from this erratum: Findings 18, 19, 20, 21.

---

## 1. `a11d2af3` — editorial: remove invalid space after `://`

**File:** `ERCS/erc-4361.md`. **Finding:** 14.

```diff
@@ -127,7 +127,7 @@ This specification defines the following SIWE Message fields that can be parsed
 A Bash-like informal template of the full SIWE Message is presented below for readability and ease of understanding, and it does not reflect the allowed optionality of the fields. Field descriptions are provided in the following section. A full ABNF description is provided in [ABNF Message Format](#abnf-message-format).

 ```
-${scheme}:// ${domain} wants you to sign in with your Ethereum account:
+${scheme}://${domain} wants you to sign in with your Ethereum account:
 ${address}

 ${statement}
```

**Rationale:** The ABNF is `[ scheme "://" ] domain` with no separator. The informal template should not contradict it.

**Compatibility:** Template-only. No producer or parser impact.

---

## 2. `e7a56390` — editorial: fix stale `request-uri` reference

**File:** `ERCS/erc-4361.md`. **Finding:** 15.

```diff
@@ -228,7 +228,7 @@ Resources:

 #### Verifying a signed Message

-- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `request-uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
+- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
```

**Rationale:** There is no `request-uri` field; the correct field is `uri`.

**Compatibility:** Prose-only. No impact.

---

## 3. `ceac03e5` — editorial: clarify RFC 3339 vs ISO 8601

**File:** `ERCS/erc-4361.md`. **Finding:** 16.

```diff
@@ -93,8 +93,9 @@ nonce = 8*( ALPHA / DIGIT )
 issued-at = date-time
 expiration-time = date-time
 not-before = date-time
-    ; See RFC 3339 (ISO 8601) for the
-    ; definition of "date-time".
+    ; See RFC 3339 for date-time. RFC 3339 is
+    ; a profile of ISO 8601; only RFC 3339
+    ; forms are permitted here.

 request-id = *pchar
     ; See RFC 3986 for the definition of "pchar".
```

**Rationale:** The parenthetical equivalence is imprecise. The field descriptions already cite RFC 3339 as the normative date-time form.

**Compatibility:** Comment-only. No impact.

---

## 4. `1794b8e4` — editorial: normalize `EIP-55` to `ERC-55`

**File:** `ERCS/erc-4361.md`. **Finding:** 17.

```diff
@@ -70,7 +70,7 @@ domain = authority

 address = "0x" 40*40HEXDIG
     ; Must also conform to capitalization
-    ; checksum encoding specified in EIP-55
+    ; checksum encoding specified in ERC-55
     ; where applicable (EOAs).

 statement = *( reserved / unreserved / " " )
```

**Rationale:** The Message Fields section says ERC-55. Use the current ERC name consistently.

**Compatibility:** Naming only. No impact.

---

## 5. `7aeadf14` — clarification: ERC-191 prefixing happens once

**File:** `ERCS/erc-4361.md`. **Finding:** 4.

```diff
@@ -29,8 +29,8 @@ The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "S

 Sign-In with Ethereum (SIWE) works as follows:

-1. The relying party generates a SIWE Message and prefixes the SIWE Message with `\x19Ethereum Signed Message:\n<length of message>` as defined in [ERC-191](./eip-191.md).
-2. The wallet presents the user with a structured plaintext message or equivalent interface for signing the SIWE Message with the [ERC-191](./eip-191.md) signed data format.
+1. The relying party generates a SIWE Message (the plaintext payload, without any prefix) and transmits it to the wallet.
+2. The wallet presents the user with a structured plaintext message or equivalent interface, and, upon user consent, signs the SIWE Message using the [ERC-191](./eip-191.md) signed data format. The ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) is applied by the signing primitive exactly once; the relying party MUST NOT pre-prefix the message.
 3. The signature is then presented to the relying party, which checks the signature's validity and SIWE Message content.
 4. The relying party might further fetch data associated with the Ethereum address, such as from the Ethereum blockchain (e.g., ENS, account balances, [ERC-20](./eip-20.md)/[ERC-721](./eip-721.md)/[ERC-1155](./eip-1155.md) asset ownership), or other data sources that might or might not be permissioned.
```

**Rationale:** If both the relying party and wallet/signing primitive apply the ERC-191 prefix, signatures are over a double-prefixed payload. Existing implementations prefix at signing time.

**Compatibility:** Aligns prose with current behavior. No wire change for conforming implementations.

---

## 6. `8ff67368` — clarification: route omitted-`scheme` through `defaultScheme`

**File:** `ERCS/erc-4361.md`. **Finding:** 7.

This commit introduces a browser-`https` MUST for `defaultScheme`. Commit 17 later walks that back to SHOULD; the rest of the harmonization survives.

```diff
@@ -110,7 +110,7 @@ resource = "- " URI
 This specification defines the following SIWE Message fields that can be parsed from a SIWE Message by following the rules in [ABNF Message Format](#abnf-message-format):

 - `scheme` OPTIONAL. The URI scheme of the origin of the request. Its value MUST be an RFC 3986 URI scheme.
-- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority. The authority includes an OPTIONAL port. If the port is not specified, the default port for the provided `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified, HTTPS is assumed by default.
+- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority. The authority includes an OPTIONAL port. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
 - `address` REQUIRED. The Ethereum address performing the signing. Its value SHOULD be conformant to mixed-case checksum address encoding specified in [ERC-55](./eip-55.md) where applicable.
 - `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
 - `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
@@ -262,7 +262,7 @@ The algorithm takes the following input variables:
 - fields from the SIWE message.
 - `origin` of the signing request - in the case of a browser wallet implementation - the origin of the page which requested the signin via the provider.
 - `allowedSchemes` - a list of schemes allowed by the Wallet.
-- `defaultScheme` - a scheme to assume when none was provided. Wallet implementers in the browser SHOULD use `https`.
+- `defaultScheme` - the scheme to assume when none was provided in the message. Wallet implementers in the browser MUST use `https`; non-browser wallet implementers MAY choose a different value appropriate to their transport.
 - developer mode indication - a setting deciding if certain risks should be a warning instead of rejection. Can be manually configured or derived from `origin` being localhost.

 The algorithm is described as follows:
```

**Rationale:** The Message Fields section reads like a protocol-wide HTTPS default, while the wallet algorithm uses an implementation input. The erratum routes the line 112 default through `defaultScheme` so both passages agree on a single resolution path.

**Compatibility:** No behavior change in conforming implementations.

---

## 7. `88f0fd19` — clarification: drop redundant subdomain rule

**File:** `ERCS/erc-4361.md`. **Finding:** 8.

```diff
@@ -270,8 +270,7 @@ The algorithm is described as follows:
 - If `scheme` was not provided, then assign `defaultScheme` as `scheme`.
 - If `scheme` is not contained in `allowedSchemes`, then the `scheme` is not expected and the Wallet MUST reject the request. Wallet implementers in the browser SHOULD limit the list of `allowedSchemes` to just `'https'` unless a developer mode is activated.
 - If `scheme` does not match the scheme of `origin`, the Wallet SHOULD reject the request. Wallet implementers MAY show a warning instead of rejecting the request if a developer mode is activated. In that case the Wallet continues processing the request.
-- If the `host` part of the `domain` and `origin` do not match, the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
-- If `domain` and `origin` have mismatching subdomains, the Wallet SHOULD reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
+- If the `host` subcomponent of `domain` and the host of `origin` do not match exactly, the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continue processing the request. Because `host` per RFC 3986 already includes the full hostname (all labels, including subdomains), a mismatched subdomain is a mismatched host and is covered by this rule.
 - Let `port` be the port component of `domain`, and if no port is contained in `domain`, assign `port` the default port specified for the `scheme`.
 - If `port` is not empty, then the Wallet SHOULD show a warning if the `port` does not match the port of `origin`.
 - If `port` is empty, then the Wallet MAY show a warning if `origin` contains a specific port. (Note 'https' has a default port of 443 so this only applies if `allowedSchemes` contain unusual schemes)
```

**Rationale:** Host equality already covers subdomains. Keeping a separate weaker subdomain rule invites inconsistent readings.

**Compatibility:** Any wallet already doing host equality behaves unchanged.

---

## 8. `f62bb494` — clarification: formalize RFC grammar imports

**File:** `ERCS/erc-4361.md`. **Finding:** 6.

```diff
@@ -38,7 +38,7 @@ Sign-In with Ethereum (SIWE) works as follows:

 #### ABNF Message Format

-A SIWE Message MUST conform with the following Augmented Backus–Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression (note that `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405)).
+A SIWE Message MUST conform with the following Augmented Backus–Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression. `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405). The grammar also normatively imports the following productions from [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986): `URI`, `authority`, `host`, `port`, `pchar`, `reserved`, and `unreserved`.

 ```abnf
 sign-in-with-ethereum =
@@ -58,6 +58,13 @@ sign-in-with-ethereum =
     [ LF %s"Resources:"
     resources ]

+; Imported from RFC 3986 Section 3:
+;     URI, authority, host, port, pchar, reserved, unreserved.
+; Imported from RFC 5234 Appendix B.1:
+;     ALPHA, DIGIT, HEXDIG, LF.
+; Imported from RFC 7405:
+;     %s (case-sensitive string literal prefix).
+
 scheme = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )
     ; See RFC 3986 for the fully contextualized
     ; definition of "scheme".
```

**Rationale:** The grammar already depends on RFC 3986 productions. The erratum makes that dependency explicit.

**Compatibility:** No intended wire change.

---

## 9. `ae9bcbb5` — clarification: declare UTF-8 as the wire encoding

**File:** `ERCS/erc-4361.md`. **Finding:** 3.

```diff
@@ -40,6 +40,8 @@ Sign-In with Ethereum (SIWE) works as follows:

 A SIWE Message MUST conform with the following Augmented Backus–Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression. `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405). The grammar also normatively imports the following productions from [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986): `URI`, `authority`, `host`, `port`, `pchar`, `reserved`, and `unreserved`.

+A SIWE Message is a sequence of Unicode code points encoded as UTF-8. All ABNF productions below describe the UTF-8 byte stream of the message. The `<length of message>` component of the ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) MUST be computed as the UTF-8 byte length of the SIWE Message.
+
 ```abnf
 sign-in-with-ethereum =
     [ scheme "://" ] domain %s" wants you to sign in with your Ethereum account:" LF
```

**Rationale:** ERC-191 length is byte length. Known implementations use UTF-8, and the spec should say so.

**Compatibility:** No known implementation uses another encoding.

---

## 10. `cfef9ef5` — judgment: pin `%s"0x"` and align ERC-55 toward SHOULD

**File:** `ERCS/erc-4361.md`. **Finding:** 1.

This commit makes two changes: it pins the `0x` prefix as case-sensitive (`%s"0x"`), and it rewords the ABNF comment to match the Message Fields section's SHOULD. Commit 18 later replaces the comment again with the final wording; the `%s"0x"` pin is permanent.

```diff
@@ -77,9 +77,9 @@ domain = authority
     ; See RFC 3986 for the fully contextualized
     ; definition of "authority".

-address = "0x" 40*40HEXDIG
-    ; Must also conform to capitalization
-    ; checksum encoding specified in ERC-55
+address = %s"0x" 40*40HEXDIG
+    ; SHOULD also conform to the mixed-case
+    ; capitalization checksum specified in ERC-55
     ; where applicable (EOAs).

 statement = *( reserved / unreserved / " " )
```

**Rationale:** The ABNF originally said the address "Must also conform to capitalization checksum encoding" — a stricter rule than the Message Fields section's SHOULD. This commit harmonizes the two while pinning the prefix as case-sensitive.

**Compatibility:** Pinning `%s"0x"` aligns the grammar with universal practice; no known producer emits non-lowercase `0X`.

---

## 11. `d7f3ee3a` — narrow: exclude `userinfo`, require non-empty host

**File:** `ERCS/erc-4361.md`. **Findings:** 5, 13.

The diff carries forward the browser-`https` MUST clause from commit 6; commit 17 later walks it back to SHOULD.

```diff
@@ -71,11 +71,11 @@ scheme = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )
     ; See RFC 3986 for the fully contextualized
     ; definition of "scheme".

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

 address = %s"0x" 40*40HEXDIG
     ; SHOULD also conform to the mixed-case
@@ -119,7 +119,7 @@ resource = "- " URI
 This specification defines the following SIWE Message fields that can be parsed from a SIWE Message by following the rules in [ABNF Message Format](#abnf-message-format):

 - `scheme` OPTIONAL. The URI scheme of the origin of the request. Its value MUST be an RFC 3986 URI scheme.
-- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority. The authority includes an OPTIONAL port. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
+- `domain` REQUIRED. The domain that is requesting the signing. Its value is an RFC 3986 `host` optionally followed by a `":"` and a port (the `userinfo` subcomponent of RFC 3986 `authority` is excluded). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
 - `address` REQUIRED. The Ethereum address performing the signing. Its value SHOULD be conformant to mixed-case checksum address encoding specified in [ERC-55](./eip-55.md) where applicable.
 - `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
 - `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
```

**Rationale:** `domain = authority` permits `userinfo@host`, but SIWE origin comparison only uses scheme, host, and port. Excluding `userinfo` aligns the grammar with the origin model. The non-empty-host rule makes the REQUIRED `domain` field actually identify an origin host.

**Compatibility:** Narrows. The concurrent test-vector PR moves two positive `userinfo@` grammar-completeness vectors to negative. No production relying party is known to emit `userinfo@host` in SIWE messages.

---

## 12. `4b2cecff` — clarification: empty optional field semantics

**File:** `ERCS/erc-4361.md`. **Findings:** 10, 11, 12.

```diff
@@ -121,16 +121,16 @@ This specification defines the following SIWE Message fields that can be parsed
 - `scheme` OPTIONAL. The URI scheme of the origin of the request. Its value MUST be an RFC 3986 URI scheme.
 - `domain` REQUIRED. The domain that is requesting the signing. Its value is an RFC 3986 `host` optionally followed by a `":"` and a port (the `userinfo` subcomponent of RFC 3986 `authority` is excluded). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
 - `address` REQUIRED. The Ethereum address performing the signing. Its value SHOULD be conformant to mixed-case checksum address encoding specified in [ERC-55](./eip-55.md) where applicable.
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
+- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present (producing a single blank-line-only sequence between the address and the URI line) rather than emitting an empty statement. Parsers MUST accept both forms. When emitted, an empty `statement` carries the same meaning as an omitted one.
 - `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
 - `version` REQUIRED. The current version of the SIWE Message, which MUST be `1` for this specification.
 - `chain-id` REQUIRED. The [EIP-155](./eip-155.md) Chain ID to which the session is bound, and the network where Contract Accounts MUST be resolved.
 - `nonce` REQUIRED. A random string typically chosen by the relying party and used to prevent replay attacks, at least 8 alphanumeric characters.
 - `issued-at` REQUIRED. The time when the message was generated, typically the current time. Its value MUST be an [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) datetime string.
 - `expiration-time` OPTIONAL. The time when the signed authentication message is no longer valid. Its value MUST be an [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) datetime string.
 - `not-before` OPTIONAL. The time when the signed authentication message will become valid. Its value MUST be an [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) datetime string.
-- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request.
-- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`.
+- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request. Producers SHOULD NOT emit an empty `request-id`; when no identifier is available, the `Request ID:` header is to be omitted entirely. Parsers MUST continue to accept messages that contain an empty `request-id` value, for backwards compatibility with existing message streams.
+- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`. Producers SHOULD NOT emit a bare `Resources:` header with no following resources; when no resources are present, the entire `[ LF %s"Resources:" resources ]` production is to be omitted. Parsers MUST accept a bare `Resources:` header as semantically equivalent to an omitted one, for backwards compatibility.

 #### Informal Message Template
```

**Rationale:** The canonical test vectors treat empty `statement`, empty `request-id`, and bare `Resources:` as valid. This commit preserves parser behavior and gives producer-side canonicalization guidance. Commit 13 rewrites the `statement` bullet again when it widens the character set.

**Compatibility:** No parser change. No message moves from valid to invalid.

---

## 13. `2e54091a` — widen: `statement` to printable ASCII

**File:** `ERCS/erc-4361.md`. **Finding:** 9.

```diff
@@ -82,10 +82,9 @@ address = %s"0x" 40*40HEXDIG
     ; capitalization checksum specified in ERC-55
     ; where applicable (EOAs).

-statement = *( reserved / unreserved / " " )
-    ; See RFC 3986 for the definition
-    ; of "reserved" and "unreserved".
-    ; The purpose is to exclude LF (line break).
+statement = *( %x20-7E )
+    ; Printable ASCII excluding LF (0x0A)
+    ; and other control characters.

 uri = URI
     ; See RFC 3986 for the definition of "URI".
@@ -121,7 +120,7 @@ This specification defines the following SIWE Message fields that can be parsed
 - `scheme` OPTIONAL. The URI scheme of the origin of the request. Its value MUST be an RFC 3986 URI scheme.
 - `domain` REQUIRED. The domain that is requesting the signing. Its value is an RFC 3986 `host` optionally followed by a `":"` and a port (the `userinfo` subcomponent of RFC 3986 `authority` is excluded). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
 - `address` REQUIRED. The Ethereum address performing the signing. Its value SHOULD be conformant to mixed-case checksum address encoding specified in [ERC-55](./eip-55.md) where applicable.
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present (producing a single blank-line-only sequence between the address and the URI line) rather than emitting an empty statement. Parsers MUST accept both forms. When emitted, an empty `statement` carries the same meaning as an omitted one.
+- `statement` OPTIONAL. A human-readable assertion that the user will sign. If emitted, the statement is a sequence of zero or more printable ASCII characters (bytes `0x20` through `0x7E`) and MUST NOT include `'\n'` (the byte `0x0a`) or any other control character. Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present rather than emitting an empty statement. Parsers MUST accept both empty and omitted statements. For authentication semantics, an empty statement is equivalent to an omitted statement.
 - `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
 - `version` REQUIRED. The current version of the SIWE Message, which MUST be `1` for this specification.
 - `chain-id` REQUIRED. The [EIP-155](./eip-155.md) Chain ID to which the session is bound, and the network where Contract Accounts MUST be resolved.
```

**Rationale:** The current grammar excludes common printable ASCII punctuation even though the prose describes an ASCII assertion and only forbids LF. Widening to printable ASCII makes the grammar match the prose while preserving empty-statement compatibility.

**Compatibility:** Widens. Producers using `"`, `%`, `<`, `>`, `{`, `}`, `|`, `\`, `^`, or backtick in statements become conforming. Parser updates should roll out before producers rely on the wider set.

---

## 14. `ab3818b8` — clarification: align reference implementation

**File:** `assets/erc-4361/example.js`. **Finding:** 2.

Commit 15 follows up with a small grammar fix and a `createMessage` correction; both commits are needed for the reference implementation to match the spec body.

```diff
@@ -1,15 +1,42 @@
 // To run this example, navigate to this directory and run `npm i && node example.js`
+//
+// ----------------------------------------------------------------------------
+// Regression tests - the following messages from the ERC-4361 specification
+// body (Examples section) MUST parse successfully against the grammar below.
+// If you edit the grammar, verify these still parse before committing.
+// ----------------------------------------------------------------------------
+//
+// Example 1 - implicit scheme:
+//
+//   example.com wants you to sign in with your Ethereum account:
+//   0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2
+//
+//   I accept the ExampleOrg Terms of Service: https://example.com/tos
+//
+//   URI: https://example.com/login
+//   Version: 1
+//   Chain ID: 1
+//   Nonce: 32891756
+//   Issued At: 2021-09-30T16:25:24Z
+//   Resources:
+//   - ipfs://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq/
+//   - https://example.com/my-web2-claim.json
+//
+// Example 2 - implicit scheme, explicit port (example.com:3388 in place of
+// example.com on the first line; otherwise identical to Example 1).
+//
+// Example 3 - explicit scheme (https://example.com in place of example.com
+// on the first line; otherwise identical to Example 1).

 const apgApi = require('apg-js/src/apg-api/api');
 const apgLib = require('apg-js/src/apg-lib/node-exports');

 const GRAMMAR = `
 sign-in-with-ethereum =
-    domain %s" wants you to sign in with your Ethereum account:" LF
+    [ scheme "://" ] domain %s" wants you to sign in with your Ethereum account:" LF
     address LF
     LF
-    [ statement LF ]
-    LF
+    statement-section
     %s"URI: " URI LF
     %s"Version: " version LF
     %s"Chain ID: " chain-id LF
@@ -21,15 +48,20 @@ sign-in-with-ethereum =
     [ LF %s"Resources:"
     resources ]

-domain = authority
+domain = host [ ":" port ]
+    ; userinfo subcomponent of RFC 3986 authority
+    ; is excluded; host MUST NOT be empty.

-address = "0x" 40*40HEXDIG
-    ; Must also conform to captilization
-    ; checksum encoding specified in EIP-55
+address = %s"0x" 40*40HEXDIG
+    ; SHOULD also conform to the mixed-case
+    ; capitalization checksum specified in ERC-55
     ; where applicable (EOAs).

-statement = 1*( reserved / unreserved / " " )
-    ; The purpose is to exclude LF (line breaks).
+statement = *( %x20-7E )
+    ; Printable ASCII excluding LF (0x0A)
+    ; and other control characters.
+
+statement-section = statement LF LF / LF

 version = "1"
```

Also rewrites the parser-callback table and extends `createMessage` to accept `scheme`, `expirationTime`, `notBefore`, `requestId`, and `resources` (see the upstream commit for the full helper rewrite).

**Rationale:** Bring the reference implementation in line with the spec body's grammar and the canonical libraries: accept the explicit-scheme example, exclude `userinfo`, pin `%s"0x"`, widen `statement`, and emit all optional fields from `createMessage`.

**Compatibility:** The reference implementation is non-normative. This aligns it with the spec body and canonical parser behavior.

---

## 15. `3a7bc36f` — clarification: fix `statement-section` and emit `statement`

**File:** `assets/erc-4361/example.js`. **Finding:** 2.

```diff
@@ -61,7 +61,7 @@ statement = *( %x20-7E )
     ; Printable ASCII excluding LF (0x0A)
     ; and other control characters.

-statement-section = statement LF LF / LF
+statement-section = statement LF LF / [ LF ]

 version = "1"

@@ -238,6 +238,7 @@ const createMessage = ({
   scheme,
   domain,
   address,
+  statement,
   uri,
   version,
   chainId,
@@ -249,7 +250,8 @@ const createMessage = ({
   resources,
 }) => {
   const prefix = scheme ? `${scheme}://${domain}` : domain;
-  const header = `${prefix} wants you to sign in with your Ethereum account:\n${address}\n\n\n`;
+  const header = `${prefix} wants you to sign in with your Ethereum account:\n${address}\n\n`;
+  const statementSection = statement ? `${statement}\n\n` : ``
   const requiredFields = [
     `URI: ${uri}\n`,
     `Version: ${version}\n`,
@@ -264,7 +266,7 @@ const createMessage = ({
   if (Array.isArray(resources) && resources.length >= 1) {
     optionalFields.push(`\nResources:\n- ${resources.join('\n- ')}`);
   }
-  return [header, ...requiredFields, ...optionalFields].join('');
+  return [header, statementSection, ...requiredFields, ...optionalFields].join('');
 }

 const message = createMessage({
```

**Rationale:** The previous `statement-section = statement LF LF / LF` over-counts a trailing newline when no statement is present. Making the bare branch `[ LF ]` matches the SIWE Message Format's optional `[ statement LF ]` followed by the separator. `createMessage` also needs to actually emit a `statement` line when one is provided.

**Compatibility:** Reference implementation fidelity only. Non-normative.

---

## 16. `a1ec1a49` — editorial: normalize EIP/ERC link paths

**File:** `ERCS/erc-4361.md`. General editorial; not tied to a specific finding.

```diff
@@ -30,9 +30,9 @@ The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "S
 Sign-In with Ethereum (SIWE) works as follows:

 1. The relying party generates a SIWE Message (the plaintext payload, without any prefix) and transmits it to the wallet.
-2. The wallet presents the user with a structured plaintext message or equivalent interface, and, upon user consent, signs the SIWE Message using the [ERC-191](./eip-191.md) signed data format. The ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) is applied by the signing primitive exactly once; the relying party MUST NOT pre-prefix the message.
+2. The wallet presents the user with a structured plaintext message or equivalent interface, and, upon user consent, signs the SIWE Message using the [ERC-191](./erc-191.md) signed data format. The ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) is applied by the signing primitive exactly once; the relying party MUST NOT pre-prefix the message.
 3. The signature is then presented to the relying party, which checks the signature's validity and SIWE Message content.
-4. The relying party might further fetch data associated with the Ethereum address, such as from the Ethereum blockchain (e.g., ENS, account balances, [ERC-20](./eip-20.md)/[ERC-721](./eip-721.md)/[ERC-1155](./eip-1155.md) asset ownership), or other data sources that might or might not be permissioned.
+4. The relying party might further fetch data associated with the Ethereum address, such as from the Ethereum blockchain (e.g., ENS, account balances, [ERC-20](./erc-20.md)/[ERC-721](./erc-721.md)/[ERC-1155](./erc-1155.md) asset ownership), or other data sources that might or might not be permissioned.
```

The full commit normalizes every intra-document Markdown link from `./eip-N.md` to `./erc-N.md` for ERCs that live in the ERCs repository (ERC-20, ERC-55, ERC-181, ERC-191, ERC-721, ERC-1155, ERC-1271, ERC-1328), and rewrites the lone Core EIP citation (EIP-712) to an absolute `https://eips.ethereum.org/EIPS/eip-712` URL. The `requires:` frontmatter is left untouched (resolver does not depend on file paths). The chain-id field also flips from `EIP-155` to `ERC-155`.

**Rationale:** Stale `./eip-N.md` paths are dead after the ERCs split. Normalizing makes every link in the document resolvable.

**Compatibility:** Link-targets only. No normative effect.

---

## 17. `7578829d` — clarification: restore browser `https` default to SHOULD

**File:** `ERCS/erc-4361.md`. **Finding:** 7.

Walks back the browser-`https` MUST language introduced in commits 6 and 11.

```diff
@@ -118,7 +118,7 @@ resource = "- " URI
 This specification defines the following SIWE Message fields that can be parsed from a SIWE Message by following the rules in [ABNF Message Format](#abnf-message-format):

 - `scheme` OPTIONAL. The URI scheme of the origin of the request. Its value MUST be an RFC 3986 URI scheme.
-- `domain` REQUIRED. The domain that is requesting the signing. Its value is an RFC 3986 `host` optionally followed by a `":"` and a port (the `userinfo` subcomponent of RFC 3986 `authority` is excluded). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes; browser wallets' `defaultScheme` MUST be `https`.
+- `domain` REQUIRED. The domain that is requesting the signing. Its value is an RFC 3986 `host` optionally followed by a `":"` and a port (the `userinfo` subcomponent of RFC 3986 `authority` is excluded). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` (see the Wallet Implementer Steps) is used as the effective scheme for origin-comparison purposes.
 - `address` REQUIRED. The Ethereum address performing the signing. Its value SHOULD be conformant to mixed-case checksum address encoding specified in [ERC-55](./erc-55.md) where applicable.
 - `statement` OPTIONAL. A human-readable assertion that the user will sign. If emitted, the statement is a sequence of zero or more printable ASCII characters (bytes `0x20` through `0x7E`) and MUST NOT include `'\n'` (the byte `0x0a`) or any other control character. Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present rather than emitting an empty statement. Parsers MUST accept both empty and omitted statements. For authentication semantics, an empty statement is equivalent to an omitted statement.
 - `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
@@ -270,7 +270,7 @@ The algorithm takes the following input variables:
 - fields from the SIWE message.
 - `origin` of the signing request - in the case of a browser wallet implementation - the origin of the page which requested the signin via the provider.
 - `allowedSchemes` - a list of schemes allowed by the Wallet.
-- `defaultScheme` - the scheme to assume when none was provided in the message. Wallet implementers in the browser MUST use `https`; non-browser wallet implementers MAY choose a different value appropriate to their transport.
+- `defaultScheme` - the scheme to assume when none was provided in the message. Wallet implementers in the browser SHOULD use `https`.
 - developer mode indication - a setting deciding if certain risks should be a warning instead of rejection. Can be manually configured or derived from `origin` being localhost.

 The algorithm is described as follows:
```

**Rationale:** The original SIWE text used SHOULD for the browser `https` default. The erratum preserves that.

**Compatibility:** Aligns with deployed wallets.

---

## 18. `70447030` — judgment: restate ERC-55 as mixed-case MUST

**File:** `ERCS/erc-4361.md`. **Finding:** 1.

Supersedes commit 10's ABNF comment with the final ERC-55 wording.

```diff
@@ -78,9 +78,8 @@ domain = host [ ":" port ]
     ; from SIWE domain values.

 address = %s"0x" 40*40HEXDIG
-    ; SHOULD also conform to the mixed-case
-    ; capitalization checksum specified in ERC-55
-    ; where applicable (EOAs).
+    ; If the address format is mixed-case, it
+    ; MUST conform to its ERC-55 checksum.

 statement = *( %x20-7E )
     ; Printable ASCII excluding LF (0x0A)
```

**Rationale:** The canonical libraries warn on all-lowercase and all-uppercase addresses and reject only wrong-checksum mixed-case addresses. The final ABNF comment encodes that behavior directly.

**Compatibility:** Existing producers emitting ERC-55 addresses remain conforming. Parsers that accept all-lowercase or all-uppercase addresses remain conforming. Parsers that previously accepted mixed-case addresses with an invalid checksum become non-conforming, matching the canonical libraries' existing reject behavior.

---

## 19. `2663fff7` — editorial: sync reference grammar comment

**File:** `assets/erc-4361/example.js`. **Finding:** 1.

```diff
@@ -53,9 +53,8 @@ domain = host [ ":" port ]
     ; is excluded; host MUST NOT be empty.

 address = %s"0x" 40*40HEXDIG
-    ; SHOULD also conform to the mixed-case
-    ; capitalization checksum specified in ERC-55
-    ; where applicable (EOAs).
+    ; If the address format is mixed-case, it
+    ; MUST conform to its ERC-55 checksum.

 statement = *( %x20-7E )
     ; Printable ASCII excluding LF (0x0A)
```

**Rationale:** Carries the wording from commit 18 over into the reference implementation grammar comment so the two artifacts stay in lockstep.

**Compatibility:** Reference implementation fidelity only.
