# Macula fork of `rustls-pki-types` 1.14.1

Vendored fork of [rustls/pki-types](https://github.com/rustls/pki-types)
at version `1.14.1`, mechanically widened so the `feature = "std"`
surface activates on `target_os = "none"` via the [`macula-std`] shim.

Used transitively by [macula-rustls](https://github.com/macula-io/macula-rustls);
quinn-proto reaches `UnixTime::now()` for ticket-rotation and
`ServerName::to_str()` for SNI logging, both of which are upstream
gated behind `feature = "std"`.

[`macula-std`]: https://github.com/macula-io/macula-std

## The patches

Same pattern as the rustls fork:

1. `src/lib.rs` — conditional `no_std` + extern crate alias:
   ```diff
   -#![cfg_attr(not(feature = "std"), no_std)]
   +#![cfg_attr(any(not(feature = "std"), target_os = "none"), no_std)]

   +#[cfg(target_os = "none")]
   +extern crate macula_std as std;
   ```
2. Bulk widen every `cfg(feature = "std")` to
   `cfg(any(feature = "std", target_os = "none"))`, plus the
   multi-arg variants surrounding `SystemTime` and `UnixTime::now()`.
3. `Cargo.toml` — add `macula-std` as a target-cfg dep on `target_os = "none"`.

## Versioning

Track upstream 1.14.x. Re-roll with the same sed pipeline as macula-rustls
plus a manual touch to the `cfg_attr(... no_std)` line at lib.rs:62.

## License

Inherited from upstream rustls-pki-types: Apache-2.0 OR MIT. No new code
beyond cfg widening + extern-crate alias.
