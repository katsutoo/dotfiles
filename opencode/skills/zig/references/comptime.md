# Zig comptime

Read this reference when changing comptime evaluation, reflection, generic APIs,
or compile-time performance. Use the pinned compiler and matching documentation.

- Use `comptime` to require compile-time values, generate types or tables, select
  platform implementations, and make unsupported states fail compilation.
- Prefer ordinary functions and data when they work at both compile time and
  runtime. Do not use comptime as a blanket optimization or abstraction system.
- Keep reflection focused and readable. Exhaustive tagged-union transformations
  and generated bindings can justify it; replacing straightforward code usually
  does not.
- Derive type-creation and reflection builtins from the selected compiler; these
  APIs have changed across Zig releases.
- Use `inline` only when semantics require inline iteration or measurement proves
  the trade. Do not raise the evaluation branch quota before examining the
  algorithm and generated work.
- Audit compile time with the pinned compiler's supported timing facilities and
  inspect build-runner work, linking, generated code, binary size, and cache
  behavior separately. Do not assume `comptime` or `inline` is free.
- Zig analyzes declarations lazily. CI should instantiate affected generic APIs,
  reference optional branches, and force representative field-dependent use for
  the affected supported targets and feature combinations. Declaration-reference
  helpers do not replace behavior tests or representative instantiation.
