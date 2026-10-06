# Fork notes

This fork starts from upstream commit
[`177bd66b5338ab66f4e9ce8f3cca418b9463d809`](https://github.com/mapbox/earcut.hpp/commit/177bd66b5338ab66f4e9ce8f3cca418b9463d809),
based on earcut 3.2.4. It retains upstream history, tests, benchmarks and the ISC
license. The implementation lives in `include/mapbox/earcut.hpp`.

- Ear checks use a uniform vertex grid with approximately four vertices per cell.
  Each candidate triangle searches cells overlapping its bounding box instead
  of a potentially broad Morton interval. Intrusive cell lists support constant-time
  removal. Splitting a polygon rebuilds the grid for each resulting ring.
- Hole bridges keep upstream's block pruning and candidate order. A block's live
  linked-list range can grow far beyond its original 16 edges as holes are inserted.
  An implicit treap stores live edges in ring order, with subtree bounds and counts.
  Queries visit the same cyclic range while skipping subtrees outside the ray or
  bridge triangle. Splices and filtering update the tree; the overlapping local
  filter windows can revisit an already-removed bridge, which must not remove it twice.
  Deterministic pseudorandom priorities balance the tree independently of geometry.
- The internal `mapbox::detail::Earcut<N>` object clears node storage, queued holes
  and output indices at the start of each call, including calls that return early
  for empty or degenerate input. Callers that retain this internal object can reuse
  its scratch capacity; input containers are not retained. The public
  `mapbox::earcut<N>(polygon)` function constructs a fresh object for each call and
  returns an owned index vector. It does not retain a thread-local scratch cache.

Clipping order, geometric predicates, bridge tie-breaking and the public API are
preserved. Upstream's linear path for small rings, filtering, local-intersection
repair and diagonal splitting remain to handle small or degenerate input. The
optional `mapbox::refine` post-pass is unchanged.

For usage and build instructions, see the [upstream README](https://github.com/mapbox/earcut.hpp/blob/177bd66b5338ab66f4e9ce8f3cca418b9463d809/README.md).
