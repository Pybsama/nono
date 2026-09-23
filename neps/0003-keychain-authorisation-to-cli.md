---
nep: 0003
title: Move macOS keychain authorisation to `nono-cli`
authors:
  - Kurtis Charnock
status: draft
created: 2026-09-23
superseded-by:
---

# NEP-0003: Move macOS keychain authorisation to `nono-cli`

## Summary

Move the authorisation for macOS keychain access out of the `nono` library and
into the `nono-cli` crate.

## Motivation

See issue #1932.

Nono currently decides if a sandboxed process can access the macOS keychain by
inferring authorisation from a keychain file capability. A file grant can
therefore override `filesystem.deny` without an explicit
`filesystem.bypass_protection` entry, as reported in #1932. The CLI must make
that policy decision before the library generates the Seatbelt profile.

### Goals

- Fix the class of bugs leading to #1932

### Non-Goals

- No changes to the Landlock/Linux implementation

## Proposal

- Delete `has_explicit_keychain_db_access`.
- Move authorisation to `nono-cli::policy::apply_macos_keychain_db_exception`,
  distinguishing between the overarching denial and a bypass.

  API surface: No new library API.

  Platforms: macOS only

  Breaking change for library/FFI clients (nobody?). Still pre 1.0 so no problem
  IMO.

## Security Considerations

Required section — do not leave blank. Address explicitly, in terms of
the model in [security-model.mdx](../docs/cli/internals/security-model.mdx):

- Least privilege: reduces authority.
- Fail-secure behavior: changes from a fail-open bug to fail-denied.
- Path handling: N/A
- Library/CLI boundary: moves the policy decision into `nono-cli`
- Credential & secret secrecy: No new handling.

## Alternatives Considered

Add `bypass_protection` to the library - duplicates code across both crates,
including policy in `nono`.

## Open Questions

- N/A
