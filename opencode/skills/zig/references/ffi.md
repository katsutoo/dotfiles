# Zig foreign interoperability

Read this reference when changing foreign signatures, bindings, callbacks,
exported symbols, or ownership across an ABI. Verify the pinned toolchain and
each affected target ABI.

- Match the foreign ABI exactly: calling convention, symbol, integer width and
  signedness, layout, alignment, nullability, sentinel, ownership, and callback
  lifetime.
- Prefer C ABI types and narrow checked wrappers around raw declarations. Convert
  pointer-plus-length inputs to slices only after validating the pair.
- Keep C pointers such as `[*c]T` at translated or ABI boundaries rather than in
  ordinary handwritten Zig APIs.
- Specify who allocates and frees every foreign object. Keep allocation and
  deallocation on the same side unless the foreign API provides a matching
  release function.
- Keep callback state alive until the foreign library cannot call it again. Catch
  and translate recoverable Zig errors before returning through a C ABI. Do not
  rely on recovering from a Zig panic: the default panic path aborts or traps
  rather than unwinding. Catch foreign exceptions on the foreign side so they do
  not unwind through Zig frames.
- For C translation choices and migration details, read
  [toolchain-changes.md](toolchain-changes.md#c-translation).
- Verify bindings on every supported target ABI. Cross-compilation is a Zig
  strength, not evidence that an untested target-specific ABI is correct.
