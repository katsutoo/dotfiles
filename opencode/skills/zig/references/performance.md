# Zig performance

Read this reference when profiling, optimizing, or making a performance claim
about Zig code. Derive workloads and budgets from the deployed artifact.

- Think from data layout, access patterns, allocation, copying, branch behavior,
  and the slowest relevant resource. Estimate network, disk, memory, and CPU
  before optimizing.
- Batch fixed-cost I/O, allocation, synchronization, and foreign calls. Keep hot
  loops simple and separate control-plane branching when measurement supports it.
- Load the `benchmark` skill for performance claims. Benchmark a correct production-like
  artifact in the intended optimization and safety mode, and report compiler
  version, target CPU, allocator, linking, LTO, and relevant build options.
- Do not benchmark Debug unless Debug performance is the question. Compare
  ReleaseSafe and ReleaseFast only when their different safety policies are
  acceptable and explicitly reported.
- Measure allocations, peak memory, binary size, startup, and tail behavior in
  addition to throughput when those resources matter.
- Do not assume zero-copy, packed layout, custom allocation, comptime generation,
  or unchecked access is faster. Include lifecycle and maintenance cost, then
  prove the effect with representative measurements.
