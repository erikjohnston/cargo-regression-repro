To reproduce, use nightly (e.g. 2024-08-09) and run `cargo check`. This worked in versions older than nightly-2024-07-08 and on stable channel.

This will produce an error:

```
thread '<unnamed>' panicked at src/cargo/core/compiler/custom_build.rs:1266:39:ld), itertools, bytes, syn                       
build script output collision for anyhow v1.0.86/4f7f6ef5a5e5df20
old=BuildOutput { library_paths: [], library_links: [], linker_args: [], cfgs: ["std_backtrace", "error_generic_member_access"], check_cfgs: ["cfg(anyhow_nightly_testing)", "cfg(anyhow_no_fmt_arguments_as_str)", "cfg(anyhow_no_ptr_addr_of)", "cfg(anyhow_no_unsafe_op_in_unsafe_fn_lint)", "cfg(doc_cfg)", "cfg(error_generic_member_access)", "cfg(std_backtrace)"], env: [], metadata: [], rerun_if_changed: ["build/probe.rs"], rerun_if_env_changed: [], warnings: [] }
new=BuildOutput { library_paths: [], library_links: [], linker_args: [], cfgs: ["std_backtrace", "error_generic_member_access"], check_cfgs: ["cfg(anyhow_nightly_testing)", "cfg(anyhow_no_fmt_arguments_as_str)", "cfg(anyhow_no_ptr_addr_of)", "cfg(anyhow_no_unsafe_op_in_unsafe_fn_lint)", "cfg(doc_cfg)", "cfg(error_generic_member_access)", "cfg(std_backtrace)"], env: [], metadata: [], rerun_if_changed: ["build/probe.rs"], rerun_if_env_changed: [], warnings: [] }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
thread 'main' panicked at src/cargo/core/compiler/job_queue/mod.rs:971:64:
called `Result::unwrap()` on an `Err` value: PoisonError { .. }
```

Removing any of the following lines will result in it succesfully building:
- Cargo.toml
  - Any dependency
  - The `debug=0` flag in `profile.dev`
- .cargo/config.toml
  - Any line

The commit in rustc that regressed this: https://github.com/rust-lang/rust/commit/20ae37c18df95f9246c019b04957d23b4164bf7a
The PR https://github.com/rust-lang/cargo/pull/13900 fees like a likely candid for causing this issue.

`cargo bisect-rustc` reported the following:

```
********************************************************************************
Regression in 20ae37c18df95f9246c019b04957d23b4164bf7a
********************************************************************************

Attempting to search unrolled perf builds
ERROR: couldn't find perf build comment
==================================================================================
= Please file this regression report on the rust-lang/rust GitHub repository     =
=        New issue: https://github.com/rust-lang/rust/issues/new                 =
=     Known issues: https://github.com/rust-lang/rust/issues                     =
= Copy and paste the text below into the issue report thread.  Thanks!           =
==================================================================================

searched nightlies: from nightly-2024-07-05 to nightly-2024-08-10
regressed nightly: nightly-2024-07-08
searched commit range: https://github.com/rust-lang/rust/compare/ed7e35f3494045fa1194be29085fa73e2d6dab40...20ae37c18df95f9246c019b04957d23b4164bf7a
regressed commit: https://github.com/rust-lang/rust/commit/20ae37c18df95f9246c019b04957d23b4164bf7a
```

<details>
<summary>bisected with <a href='https://github.com/rust-lang/cargo-bisect-rustc'>cargo-bisect-rustc</a> v0.6.9</summary>


Host triple: x86_64-unknown-linux-gnu
Reproduce with:
```bash
cargo bisect-rustc --access=github 
```
</details>
