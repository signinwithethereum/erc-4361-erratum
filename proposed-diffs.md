---
title: Proposed diffs for ERC-4361 remediation
---

# Proposed diffs for ERC-4361 remediation

Companion to [`inconsistencies.md`](./inconsistencies.md). For each finding we give:

- **Replacement text**: minimal edit to `erc-4361.md`. Line numbers refer to the frozen copy at [`erc-4361-snapshot.md`](./erc-4361-snapshot.md) as of 2026-04-23.
- **Rationale**: why the edit is correct and why this phrasing.
- **Compatibility**: what happens to messages/implementations that were valid under the current spec.
- **Compat tag**: one of `Editorial`, `Clarification`, `Narrow`, `Widen`, `Judgment`. Categories are defined immediately below.

Ordering matches the order in which commits are applied on the PR branch (low-controversy first), not the inconsistencies.md numbering.

**Compatibility tags:**

- `Editorial`: typos, stale names, prose wording. No normative effect.
- `Clarification`: no wire change, no behavior change in any correct implementation. Aligns prose or reference implementation with what the spec already meant.
- `Narrow`: grammar becomes stricter. Previously-valid messages become invalid.
- `Widen`: grammar becomes looser. Previously-invalid messages become valid. Producers adopt lazily until parsers have rolled through.
- `Judgment`: a genuine direction call between two competing readings with real-world breakage risk depending on which way the fix goes.

Conventions:

- Diffs are shown as unified-diff fragments. `-` lines are removed, `+` lines added, lines with neither are surrounding context only.
- "ABNF" refers to the block at lines 43-105.
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

**Rationale:** The ABNF (line 45) is `[ scheme "://" ] domain` with no separator. The informal template is meant to illustrate the grammar, not contradict it.

**Compatibility:** Template is non-normative. No impact on producers or parsers.

---

## Commit 2: Finding #19 (stale `request-uri` reference)

**Tag:** Editorial.

**Replacement:** line 231.

```diff
-- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `request-uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
+- The SIWE Message MUST be checked for conformance to the ABNF Message Format in the previous sections, checked against expected values after parsing (e.g., `expiration-time`, `nonce`, `uri` etc.), and its signature MUST be checked as defined in [Signing and Verifying Messages with Ethereum Accounts](#signing-and-verifying-messages-with-ethereum-accounts).
```

**Rationale:** There is no `request-uri` field; the correct field name is `uri` (line 115).

**Compatibility:** Prose-only fix. No impact.

---

## Commit 3: Finding #20 (RFC 3339 ≠ ISO 8601)

**Tag:** Editorial.

**Replacement:** lines 96-97.

```diff
 issued-at = date-time
 expiration-time = date-time
 not-before = date-time
-    ; See RFC 3339 (ISO 8601) for the
-    ; definition of "date-time".
+    ; See RFC 3339 for the definition of
+    ; "date-time". RFC 3339 is a profile of
+    ; ISO 8601 and is the normative reference
+    ; here; forms valid under ISO 8601 but
+    ; not RFC 3339 are not permitted.
```

**Rationale:** The parenthetical equivalence is misleading: RFC 3339 excludes ordinal dates, basic format, and two-digit offsets that ISO 8601 permits. The field descriptions already cite only RFC 3339; this brings the ABNF comment in line.

**Compatibility:** Comment-only. No impact on conforming implementations (which already read the Message Fields section as authoritative on date-time form).

---

## Commit 4: Finding #21 (`EIP-55` vs `ERC-55` naming drift)

**Tag:** Editorial.

**Replacement:** line 73.

```diff
 address = "0x" 40*40HEXDIG
     ; Must also conform to capitalization
-    ; checksum encoding specified in EIP-55
+    ; checksum encoding specified in ERC-55
     ; where applicable (EOAs).
```

(The `Must` is addressed in Commit 10 / Finding #3; this commit only normalizes the name.)

**Rationale:** The Message Fields section at line 113 says "ERC-55"; the ABNF comment says "EIP-55". The standard has been renamed ERC-55. One name in adjacent paragraphs.

**Compatibility:** Naming drift only. No impact.

---

## Commit 5: Finding #2 (ERC-191 prefixing assigned to two parties)

**Tag:** Clarification.

**Replacement:** lines 32-33 (Overview steps 1-2).

```diff
-1. The relying party generates a SIWE Message and prefixes the SIWE Message with `\x19Ethereum Signed Message:\n<length of message>` as defined in [ERC-191](./eip-191.md).
-2. The wallet presents the user with a structured plaintext message or equivalent interface for signing the SIWE Message with the [ERC-191](./eip-191.md) signed data format.
+1. The relying party generates a SIWE Message (the plaintext payload, without any prefix) and transmits it to the wallet.
+2. The wallet presents the user with a structured plaintext message or equivalent interface, and, upon user consent, signs the SIWE Message using the [ERC-191](./eip-191.md) signed data format. The ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) is applied by the signing primitive exactly once; the relying party MUST NOT pre-prefix the message.
```

**Rationale:** Only one party can apply the ERC-191 prefix without producing a double-prefixed payload with a different hash. In every conforming implementation the wallet (or the signing primitive it delegates to) is the party that prefixes. Step 1 as written is incorrect; the corrected text also adds an explicit MUST NOT for the RP to close the ambiguity.

**Compatibility:** No conforming implementation today double-prefixes; this edit aligns the prose with observed behavior. No byte-level change for any correct producer/wallet.

---

## Commit 6: Finding #7 (omitted-`scheme` default disagreement)

**Tag:** Clarification.

**Replacement:** line 112 (Message Fields) and lines 264, 269 (Wallet algorithm).

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

**Rationale:** The Message Fields text reads like a protocol-wide HTTPS default; the Wallet algorithm reads like an implementation-defined default. The resolution most faithful to both readings is: the effective default is `defaultScheme`, and browser wallets are required (not merely recommended) to set it to `https`. This collapses the two readings into one without breaking non-browser deployments.

**Compatibility:** No existing browser wallet sets `defaultScheme` to anything other than `https`; this makes the observed practice normative. No wire change.

---

## Commit 7: Finding #8 (redundant host/subdomain rule)

**Tag:** Clarification.

**Replacement:** lines 272-273.

```diff
-- If the `host` part of the `domain` and `origin` do not match, the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
-- If `domain` and `origin` have mismatching subdomains, the Wallet SHOULD reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continues processing the request.
+- If the `host` subcomponent of `domain` and the host of `origin` do not match exactly (byte-for-byte after IDN normalization; see [IDN handling](#idn-and-punycode)), the Wallet MUST reject the request unless the Wallet is in developer mode. In developer mode the Wallet MAY show a warning instead and continue processing the request. Because `host` per RFC 3986 already includes the full hostname (all labels, including subdomains), a mismatched subdomain is a mismatched host and is covered by this rule.
```

(The subdomain bullet is dropped entirely.)

**Rationale:** Under RFC 3986, `host` is the full hostname including all labels. If two hosts have mismatching subdomains, their `host` values do not match, so the prior MUST-reject rule already fires. The weaker subdomain SHOULD-reject rule is redundant or, worse, invites an implementation to think it licenses accepting subdomain mismatches after already-mismatched hosts passed, which the spec never meant.

**Compatibility:** Any implementation already doing the stronger host-equality check behaves unchanged. Any implementation that read the two rules as distinct (and was therefore accepting some subdomain mismatches under SHOULD) now behaves more strictly; but such behavior is not known to exist in a production wallet.

---

## Commit 8: Finding #5 (ABNF not self-contained)

**Tag:** Clarification.

**Replacement:** insert after line 41 (before the ABNF block), and add explicit imports inside the ABNF.

Before ABNF block:

```diff
-A SIWE Message MUST conform with the following Augmented Backus-Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression (note that `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405)).
+A SIWE Message MUST conform with the following Augmented Backus-Naur Form (ABNF, [RFC 5234](https://www.rfc-editor.org/rfc/rfc5234)) expression. `%s` denotes case sensitivity for a string term, as per [RFC 7405](https://www.rfc-editor.org/rfc/rfc7405). The grammar also normatively imports the following productions from [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986): `URI`, `authority`, `host`, `port`, `pchar`, `reserved`, `unreserved`. Where the spec's grammar and RFC 3986 disagree on a production, the definition in RFC 3986 §3 is authoritative.
```

Inside ABNF, add a comment block at the top of the imports area (replacing the scattered "See RFC 3986" comments is optional; the explicit-import block above is the normative change):

```diff
 ```abnf
 sign-in-with-ethereum =
     [ scheme "://" ] domain %s" wants you to sign in with your Ethereum account:" LF
     address LF
     LF
     [ statement LF ]
     LF
     %s"URI: " uri LF
     ...
 
+; Imported from RFC 3986 §3:
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

**Rationale:** Today the grammar refers to `URI`, `authority`, `pchar` etc. only via code comments, forcing the reference implementation to re-inline ~80 lines of RFC 3986. Implementers can disagree about which revision of BCP 13 / RFC 3987 / errata apply to IPv6 hosts, IPvFuture, and percent-encoded pchar. Naming RFC 3986 §3 as the authoritative source closes the ambiguity without rewriting the grammar.

**Compatibility:** Every conforming implementation already uses RFC 3986 (implicitly, via the comments). This makes the dependency explicit. No wire change.

---

## Commit 9: Finding #1 (character encoding of the wire format)

**Tag:** Clarification (with new normative guidance).

**Replacement:** insert a new paragraph immediately after line 41 (before the ABNF), and reinforce in the ERC-191-signing section.

Before ABNF:

```diff
 A SIWE Message MUST conform with the following Augmented Backus-Naur Form ...
+
+A SIWE Message is a sequence of Unicode code points encoded as UTF-8. All ABNF productions below describe the UTF-8 byte stream of the message. The `<length of message>` component of the ERC-191 prefix (`\x19Ethereum Signed Message:\n<length of message>`) MUST be computed as the UTF-8 byte length of the SIWE Message. Implementations MUST NOT encode SIWE Messages in UTF-16, Latin-1, or any other encoding.
```

**Rationale:** ERC-191's prefix embeds the byte length of the payload. Byte length is encoding-dependent: two wallets that disagree on UTF-8 vs UTF-16 produce different signed prefixes over the same logical text. Every known production implementation treats SIWE messages as UTF-8 (because web transport and `TextEncoder` default to UTF-8), so this is a documentation gap, not a real-world divergence; but the gap is load-bearing for interop claims.

**Compatibility:** No known implementation treats SIWE as anything other than UTF-8. This edit makes the observed unanimity normative. No wire change for any correct implementation.

---

## Commit 10: Finding #9 (percent-encoded LF in URIs can break framing)

**Tag:** Clarification (with new normative guidance).

**Replacement:** add a new bullet in the Wallet Implementer Steps, after line 281 (Creating Sign-In with Ethereum Interfaces).

```diff
 - Wallet implementers MUST display to the user the following fields from the SIWE Message request by default ...
+- Wallet implementers MUST NOT percent-decode `uri` or `resources` values before rendering them as SIWE message framing (i.e., the decoded octets MUST NOT be re-interpreted as additional SIWE fields). Wallets MAY percent-decode a URI for display alongside the raw form for user readability, but percent-decoded forms MUST be presented as URI values, not as pseudo-fields of the SIWE message.
```

**Rationale:** A URI like `https://evil.example/foo%0AVersion:%202` is grammatically valid (`URI` per RFC 3986 permits percent-encoded octets). If a wallet percent-decodes for display and re-parses, the decoded `\n` can appear to introduce a new SIWE field. The grammar is line-oriented but URI values are not; the resolution is to forbid decoding that crosses the framing boundary.

**Compatibility:** No parser re-frames on decoded URIs today (they tokenize on the raw byte stream). This codifies current behavior. No wire change.

---

## Commit 11: Finding #10 (IDN/punycode handling for `domain`)

**Tag:** Clarification (with new normative guidance).

**Replacement:** add a new subsection under "Wallet Implementer Steps", after the "Verifying the Request Origin" block, titled "IDN and Punycode".

```diff
+#### IDN and Punycode
+
+- The `host` subcomponent of `domain` MUST be an RFC 3986 `reg-name` (A-label form). Internationalized domain names MUST be expressed in ASCII-compatible-encoding (ACE, "xn--…") form, per [RFC 5891](https://www.rfc-editor.org/rfc/rfc5891) (IDNA2008), before being placed in a SIWE Message. Wallets MUST compare `host` to `origin` after applying the same ACE conversion to `origin`.
+- U-label (Unicode) forms of `host` MUST NOT appear in a SIWE Message. A wallet encountering a `host` containing non-ASCII octets MUST reject the message.
+- Case-folding of `host` MUST follow ASCII lowercase comparison (A-Z folded to a-z); no Unicode case-folding is applied, because U-labels are forbidden.
```

**Rationale:** RFC 3986's `reg-name` is opaque bytes; the spec never invoked RFC 3987 IRIs. Leaving this undefined means two wallets can disagree on whether `münchen.de` and `xn--mnchen-3ya.de` are equal, whether IDNA2003 or IDNA2008 applies, and how case-folding interacts. Requiring A-label form at the wire level (and rejecting U-labels) mirrors browser origin-comparison behavior and is the least surprising reading.

**Compatibility:** This narrows behavior for any wallet that was accepting U-label hosts; but no production wallet is known to do so, since `new URL()` in browsers and Go/Rust/Node URL parsers all normalize to ACE on origin access.

---

## Commit 12: Finding #3 (MUST vs SHOULD on EIP-55, plus `"0x"` case lenience)

**Tag:** Judgment. Harmonized DOWN to SHOULD (rationale in the Compatibility section below).

**Replacement:** lines 71-74 (ABNF) and line 113 (Message Fields; already "SHOULD"; no change needed there).

ABNF:

```diff
-address = "0x" 40*40HEXDIG
-    ; Must also conform to capitalization
-    ; checksum encoding specified in EIP-55
+address = %s"0x" 40*40HEXDIG
+    ; SHOULD also conform to the mixed-case
+    ; capitalization checksum specified in ERC-55
     ; where applicable (EOAs).
```

Two separate changes in one hunk:

1. `%s"0x"` instead of `"0x"`; closes the latent case-insensitivity of the literal so `0X…` no longer parses. Per RFC 5234 §2.3, unprefixed ABNF string literals are case-insensitive; `%s` opts into case-sensitive matching (RFC 7405), which is what every known implementation actually does.
2. `Must` → `SHOULD` in the comment; harmonizes with line 113's `SHOULD`.

**Rationale:** The current text is self-contradictory: ABNF comment says `Must`, Message Fields says `SHOULD`. Harmonizing UP would newly reject lowercase-address messages that production wallets and libraries accept today (a real breakage). Harmonizing DOWN is lossless; any producer emitting EIP-55 addresses remains conforming. Separately, every library uses `%s` for field labels but forgot to apply it to the `"0x"` literal; the spec's own example `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` (line 156) requires case-sensitive match of the lowercase `0x` anyway.

**Compatibility:**
- The `Must` → `SHOULD` half: lossless; any implementation already accepting lowercase addresses remains conforming, any implementation rejecting them remains conforming (SHOULD permits it).
- The `%s"0x"` half: narrows. A message starting with `0X…` was technically valid under the current ABNF and is invalid after. No production implementation is known to emit or accept `0X` prefixes (case-sensitive `0x` is de facto universal), so the breakage is theoretical.

---

## Commit 13: Finding #14 (empty `request-id`)

**Tag:** Clarification (softened from Narrow; see revision note).

**Replacement:** line 122 (Message Fields), prose only. ABNF unchanged.

```diff
-- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request.
+- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request. Producers SHOULD NOT emit an empty `request-id`; when no identifier is available, the `Request ID:` header is to be omitted entirely. Parsers MUST continue to accept messages that contain an empty `request-id` value, for backwards compatibility with existing message streams.
```

**Rationale:** `*pchar` permits the empty string, so `Request ID: \n` parses. The field description describes `request-id` as "a system-specific identifier"; a zero-length identifier is unlikely to be intended. However, the canonical test-vectors (`@signinwithethereum/test-vectors`, `vectors/grammar/valid_specification.json:request-id empty`) explicitly document the "present but empty" form as valid, with `requestId: ""` as the expected parsed result; the distinction between "missing" and "present but empty" is intentional. We therefore keep the grammar permissive and add producer-side guidance without breaking existing parsers.

**Compatibility:** No wire change; no parser change. Producers get new SHOULD-NOT guidance. No messages move between conforming and non-conforming.

**Revision note:** This commit was originally specified as an ABNF narrowing (`request-id = 1*pchar`). The conformance scan of `@signinwithethereum/test-vectors` surfaced a positive vector requiring the empty form to parse (see `conformance-matrix.md` §3.1). The commit was softened to prose-only guidance to preserve the intentional "present but empty" semantic.

---

## Commit 14: Finding #15 (empty explicit port)

**Tag:** Narrow.

**Replacement:** since `port` comes from RFC 3986 (where it is `*DIGIT`), the narrowing has to be stated in prose at the `domain` field description (line 112) and reinforced in the Wallet algorithm (line 276). No ABNF change, because we should not unilaterally redefine an imported production.

Line 112 (extending the Commit 6 edit):

```diff
-- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority (see grammar below for restrictions on the `host` subcomponent). The authority includes an OPTIONAL port. If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` ... browser wallets' `defaultScheme` MUST be `https`.
+- `domain` REQUIRED. ... The authority includes an OPTIONAL port; if the `":"` port delimiter is present, the port component MUST be non-empty (i.e., `example.com:` is invalid). If the port is not specified, the default port for the effective `scheme` is assumed ...
```

Line 276 (remove the unreachable branch):

```diff
-- If `port` is empty, then the Wallet MAY show a warning if `origin` contains a specific port. (Note 'https' has a default port of 443 so this only applies if `allowedSchemes` contain unusual schemes)
```

**Rationale:** `example.com:` survives the grammar via RFC 3986's `port = *DIGIT` but has no defined semantics in the wallet algorithm: step 274 assigns a default when port is absent, making step 276 only reachable for the explicit-empty case. Rejecting explicit-empty ports at the `domain` layer eliminates an under-specified edge case that no producer intentionally emits.

**Compatibility:** Narrows. Any producer emitting `example.com:` becomes non-conforming. No such producer is known.

---

## Commit 15: Finding #16 (empty `domain`)

**Tag:** Narrow.

**Replacement:** line 65 (ABNF).

```diff
-domain = authority
-    ; From RFC 3986:
-    ;     authority     = [ userinfo "@" ] host [ ":" port ]
-    ; See RFC 3986 for the fully contextualized
-    ; definition of "authority".
+domain = host [ ":" 1*DIGIT ]
+    ; host is imported from RFC 3986 §3.2.2.
+    ; The userinfo component of RFC 3986
+    ; authority is intentionally excluded
+    ; (see Verifying the Request Origin).
+    ; host MUST NOT be empty; see Message
+    ; Fields for the port non-emptiness rule.
```

This also substantially implements Finding #6 (the `userinfo@` exclusion); see Commit 17 for why it is split out as its own commit even though the mechanical edit overlaps.

For the host-non-empty rule, a further prose addition at line 112 (already being edited for Commits 6/14):

```diff
- ... i.e., `example.com:` is invalid). If the port is not specified, the default port for the effective `scheme` is assumed (e.g., 443 for HTTPS). If `scheme` is not specified in the message, the wallet's `defaultScheme` ...
+ ... i.e., `example.com:` is invalid). The `host` subcomponent MUST be non-empty. If the port is not specified, the default port for the effective `scheme` is assumed ...
```

**Rationale:** RFC 3986's `reg-name = *(...)` admits the empty string, so today the message can start with ` wants you to sign in ...`. That is difficult to reconcile with `domain` being a REQUIRED, origin-identifying field. Since `host` is imported, the non-emptiness rule is stated in prose.

**Compatibility:** Narrows. No producer is known to emit messages with an empty leading domain.

---

## Commit 16: Finding #17 (leading-zero `chain-id`)

**Tag:** Narrow.

**Replacement:** lines 86-87.

```diff
-chain-id = 1*DIGIT
-    ; See EIP-155 for valid CHAIN_IDs.
+chain-id = "0" / ( NONZERO-DIGIT *DIGIT )
+    ; See EIP-155 for valid CHAIN_IDs.
+    ; Canonical decimal form: no leading zeros.
+
+NONZERO-DIGIT = %x31-39
```

**Rationale:** `1*DIGIT` admits `01`, `001`, `1` as distinct strings with no rule for whether they denote the same chain. EIP-155 chain IDs are integers; leading zeros are non-canonical. Enforcing canonical decimal form at the grammar level removes the ambiguity, and the cost is zero because no wallet or library emits chain IDs with leading zeros today.

**Compatibility:** Narrows. No known producer emits leading zeros.

---

## Commit 17: Finding #6 (remove `userinfo@` from `domain`)

**Tag:** Narrow. Primary fix is grammatical; prose fallback is ready if editors object. Requires coordinated corpus migration; see release plan below.

**Replacement:** already folded into Commit 15's ABNF edit (`domain = host [ ":" 1*DIGIT ]` excludes `userinfo`). If editors object to grammar narrowings, fall back to a prose-only fix: keep `domain = authority` in the grammar and add the following to the Wallet "Verifying the Request Origin" block (around line 253):

```diff
+- If `domain` contains a `userinfo` component (an `@` preceding the `host`), the Wallet MUST reject the message. The `userinfo` production is permitted by RFC 3986 `authority` but is out of scope for SIWE origin comparison.
```

The grammar-first path is preferred and is what Commit 15 implements. This commit exists primarily so reviewers can see the phishing argument and rationale in one place, and so the commit log tells the right story.

**Rationale:** `domain = authority` permits `trusted.com@evil.com:443`. The wallet algorithm compares only `host`, which equals `evil.com` here, but the plaintext displayed to the user prominently shows `trusted.com` at the head of the message. This is a real phishing surface. Excluding `userinfo` at the grammar level matches the spec's conceptual model (per RFC 6454, a web origin is scheme + host + port, no userinfo).

**Compatibility:** Narrows. `@signinwithethereum/test-vectors` currently includes two positive vectors in `vectors/parsing/parsing_positive.json`; `domain is RFC 3986 authority with userinfo` and `domain is RFC 3986 authority with userinfo and port`; which this commit would move out of conformance. These vectors are RFC-3986 grammar-completeness tests (domain = `test@127.0.0.1`), not real SIWE-usage samples. No production SIWE relying party is known to emit `userinfo@host` in a SIWE message; the phishing case depends on attacker-controlled messages.

### Coordinated release plan (unique to this commit)

This is the only surviving narrowing where the canonical `@signinwithethereum/test-vectors` corpus must change in lockstep. The sequencing matters:

1. **Pre-submission.** Open a PR against `@signinwithethereum/test-vectors` that moves the two userinfo vectors from `parsing/parsing_positive.json` into `parsing/parsing_negative.json` (or a new `parsing_deprecated_positive.json` file, if we want to preserve the history). Do not merge yet. The test-vectors PR title should explicitly reference the upcoming ERC-4361 erratum, e.g. `test-vectors: reclassify userinfo@ domain vectors (ERC-4361 erratum)`.
2. **Pre-submission.** Open draft PRs against `@signinwithethereum/siwe`, `@signinwithethereum/siwe-py`, `@signinwithethereum/siwe-rs`, and `the Go port` that update their parser grammar to reject `userinfo@` in `domain`. Each draft PR imports the updated test-vectors revision as a dependency.
3. **Submission day.** Open the ERCs PR. In its body, link to the four downstream draft PRs and the test-vectors PR as "concurrent release plan evidence." Editors and reviewers see a complete, coordinated rollout, not a spec change with a dangling implementation gap.
4. **Post-merge.** Merge the test-vectors PR. Cut coordinated minor releases of all four libraries. Close the downstream draft PRs as merged.
5. **Fallback.** If the ERCs editor asks to drop the narrowing, close the four downstream draft PRs and the test-vectors PR with a reference to the editor decision. No wasted work shipped.

This sequencing keeps the "erratum filed by the current canonical maintainer" frame coherent: we're not asking the ecosystem to change, we're showing the ecosystem changing in sync with the spec.

**Residual risk:** a downstream user of `@signinwithethereum/test-vectors` who pins an older version and emits `userinfo@` messages will be surprised by the parser update. Mitigated by: (a) the vectors are grammar-completeness tests with no documented real-world provenance; (b) release notes call this out explicitly; (c) the minor-version bump is a semver signal the change happened.

---

## Commit 18: Findings #12 (`statement`) and #13 (`Resources:` empty)

**Tag:** Clarification (softened from Narrow; see revision note). Coupled because the same "present but empty" semantic applies to both, and the resolution is the same.

**Replacement:** prose notes at lines 114 and 123. ABNF at lines 76 and 102 is unchanged.

Message Fields, line 114 (note: Commit 19's widening will replace the charset language; this commit only adds the presence/empty guidance. See the "Invariants for reviewers" section at the end of this document for the sequencing.):

```diff
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
+- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). Producers SHOULD omit the `[ statement LF ]` production entirely when no statement is present (producing a single blank-line-only sequence between the address and the URI line) rather than emitting an empty statement. Parsers MUST accept both forms. When emitted, an empty `statement` carries the same meaning as an omitted one.
```

Message Fields, line 123:

```diff
-- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`.
+- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`. Producers SHOULD NOT emit a bare `Resources:` header with no following resources; when no resources are present, the entire `[ LF %s"Resources:" resources ]` production is to be omitted. Parsers MUST accept a bare `Resources:` header as semantically equivalent to an omitted one, for backwards compatibility.
```

**Rationale:** `*(...)` in both productions admits a zero-length body. Holiman's proposal on Ethereum Magicians and earlier drafts of this plan called for `1*(...)`. However, the canonical `@signinwithethereum/test-vectors` (`vectors/grammar/valid_specification.json`) contains explicit positive vectors named `"statement empty"` and `"resources empty"` that document the "present but empty" form as a parseable message with distinct parsed shape (`statement: ""`, `resources: []`). The maintained `@signinwithethereum` libraries (TS, Py, Rust) implement the corpus faithfully. The narrowing would therefore require coordinated releases of the test-vectors package and all four SIWE libraries, and would break the intentional "missing vs empty" distinction that the corpus documents. The softened form achieves the interop goal; equate the two forms semantically and steer producers toward the canonical omission; without breaking any existing parser or message.

**Compatibility:** No wire change; no parser change. Producers get new SHOULD/SHOULD-NOT guidance. Any parser already distinguishing "missing" from "present but empty" at the object level may continue to do so, or may collapse them per the new prose; the spec explicitly permits either.

**Revision note:** This commit was originally specified as an ABNF narrowing for both findings (`statement = 1*(...)` and `resources = 1*( LF resource )`). The conformance scan of `@signinwithethereum/test-vectors` surfaced positive vectors requiring both empty forms to parse (see `conformance-matrix.md` §3.1). The commit was softened to prose-only guidance to preserve the intentional "present but empty" semantic that the canonical corpus documents.

---

## Commit 19: Finding #11 (`statement` charset too narrow)

**Tag:** Widen. Last among normative commits because this is the only direction that newly admits previously-rejected messages; parser rollout precedes producer rollout.

**Replacement:** lines 76-79 (ABNF) and line 114 (Message Fields; extending the Commit 18 edit).

ABNF:

```diff
-statement = 1*( reserved / unreserved / " " )
-    ; See RFC 3986 for the definition
-    ; of "reserved" and "unreserved".
-    ; The purpose is to exclude LF (line break).
+statement = 1*( %x20-7E )
+    ; Printable ASCII excluding LF (0x0A)
+    ; and other control characters. The
+    ; purpose is to exclude LF (line break)
+    ; while admitting natural-language
+    ; punctuation such as " % < > { } | \ ^ `.
```

Message Fields, line 114:

```diff
-- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`). When present, the `statement` MUST be non-empty; ...
+- `statement` OPTIONAL. A human-readable assertion that the user will sign. When present, the `statement` MUST be a non-empty sequence of printable ASCII characters (bytes `0x20` through `0x7E`), and MUST NOT include `'\n'` (the byte `0x0a`) or any other control character. To indicate the absence of a statement, omit the `[ statement LF ]` production entirely, which produces a single blank-line-only sequence between the address and the URI line.
```

**Rationale:** The current `reserved / unreserved / " "` charset excludes `"` `%` `<` `>` `{` `}` `|` `\` `^` `` ` ``. This makes `I accept the "Terms of Service"`, `Balance < 100`, and any statement mentioning `100%` invalid per the grammar; despite the field description calling `statement` "a human-readable ASCII assertion" whose stated purpose is only "exclude LF". The mismatch was never intentional; no production RP treats these characters as forbidden. Widening to printable-ASCII-minus-LF makes the grammar match the description.

**Compatibility:** Widens. Producers that already emit such characters (in violation of the current grammar but consistently with the field description) become conforming. A strict parser can keep accepting the old, narrower set without being non-conforming because the old set is a subset of the new one; but producers SHOULD NOT rely on parsers accepting the wider set until rollout has occurred. The commit message will explicitly note a parser-first, producer-second rollout ordering.

---

## Commit 20: Finding #4 (reference implementation)

**Tag:** Clarification (no wire change; brings `assets/eip-4361/example.js` into alignment with the cumulative grammar).

**Replacement:** edits to `assets/eip-4361/example.js` (not `erc-4361.md`). Four specific corrections:

1. Add `[ scheme "://" ]` handling to the top-level parser so `https://example.com wants you to sign in ...` (spec example at line 191) parses.
2. Change `statement = 1*( reserved / unreserved / " " )` to match Commit 19's `1*( %x20-7E )`. (The reference file's prior `1*(...)` matched neither the spec's former `*(...)` nor the new widened form.)
3. Extend `createMessage` to accept `expirationTime`, `notBefore`, `requestId`, and `resources` and to emit the corresponding headers, including a `Resources:` block when `resources.length >= 1` (not when merely present/defined; matches Commit 18's `1*( LF resource )`).
4. Add the three example messages from `erc-4361.md` lines 152-204 to the file header as a comment block labeled "regression tests", so any future edit can manually verify the examples still parse.

No npm dependencies. The ~80 lines of inlined RFC 3986 grammar remain; Commit 8 made those imports normative rather than optional.

**Rationale:** Reviewers reading the reference implementation will not be able to tell which production it is supposed to conform to until the cumulative grammar is settled. Landing this as the final commit means the reference matches the grammar as it exists after the PR merges, not some intermediate state. Custody is with the PR maintainer (the current maintainer of the canonical `@signinwithethereum` libraries), so the reference implementation and the production libraries are in the same custody chain.

**Compatibility:** The reference implementation is non-normative. Implementers who treated it as authoritative will see their reference move; implementers who treated `erc-4361.md` as authoritative see no change. siwe#30 (the original bug report) is resolved.

---

## Summary table

| Commit | Finding | Tag | Files touched |
| --- | --- | --- | --- |
| 1 | #18 | Editorial | `erc-4361.md` |
| 2 | #19 | Editorial | `erc-4361.md` |
| 3 | #20 | Editorial | `erc-4361.md` |
| 4 | #21 | Editorial | `erc-4361.md` |
| 5 | #2 | Clarification | `erc-4361.md` |
| 6 | #7 | Clarification | `erc-4361.md` |
| 7 | #8 | Clarification | `erc-4361.md` |
| 8 | #5 | Clarification | `erc-4361.md` |
| 9 | #1 | Clarification | `erc-4361.md` |
| 10 | #9 | Clarification | `erc-4361.md` |
| 11 | #10 | Clarification | `erc-4361.md` |
| 12 | #3 | Judgment (down to SHOULD) | `erc-4361.md` |
| 13 | #14 | Clarification (softened) | `erc-4361.md` (prose only) |
| 14 | #15 | Narrow | `erc-4361.md` |
| 15 | #16 | Narrow | `erc-4361.md` |
| 16 | #17 | Narrow | `erc-4361.md` |
| 17 | #6 | Narrow (+ corpus migration) | `erc-4361.md` (ABNF via Commit 15) + `@signinwithethereum/test-vectors` concurrent PR |
| 18 | #12, #13 | Clarification (softened, coupled) | `erc-4361.md` (prose only) |
| 19 | #11 | Widen | `erc-4361.md` |
| 20 | #4 | Clarification | `assets/eip-4361/example.js` |

20 commits covering all 21 findings (#6 is implemented by Commit 15's ABNF edit and historicized as its own commit; #12 and #13 share Commit 18).

---

## Invariants for reviewers

- Line numbers cited in this document refer to [`erc-4361-snapshot.md`](./erc-4361-snapshot.md), the frozen copy of the spec as audited on 2026-04-23. Upstream renumbering caused by commits 1-4 (which land first) shifts later line citations; reviewers tracing a specific commit against the live upstream file should account for those shifts.
- Commits 6, 14, 15 all edit the `domain` field description on the same line. They are applied in that order; each rebases cleanly onto its predecessor and the text grows with each edit.
- Commits 18 and 19 both edit the `statement` field description on the same line. Commit 18 lands first (prose-only, adds producer SHOULD-omit guidance), Commit 19 second (widens the charset). The resulting final ABNF line for `statement` is `statement = 1*( %x20-7E )` (Commit 19 alone; Commit 18 does not touch the ABNF).
- Commit 17 (#6 userinfo) is the only commit whose rollout depends on an out-of-tree PR: the concurrent `@signinwithethereum/test-vectors` reclassification that moves the two `userinfo@` grammar-completeness vectors from positive to negative. The downstream library updates (`@signinwithethereum/siwe`, `-py`, `-rs`, and the Go port) ship minor bumps in lockstep.
- Commit 20 (reference implementation in `assets/erc-4361/example.js`) depends on the cumulative normative state produced by Commits 1-19, not on any specific line numbering. It is scheduled last so it reflects that cumulative state without rework.
