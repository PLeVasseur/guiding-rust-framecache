# framecache: Specification

## framecache v0.1: signal decode and latest-value cache

A reusable crate for tooling and simulation targets (std environment).
Two capabilities: decode and cache.

### Decode

A `SignalDb` describes messages and signals (a simplified DBC: message id
to signals; each signal has a name, start bit, length, scale, offset, and
unit). It's loaded from a TOML file at startup. Given a raw frame (id
plus up to 8 payload bytes), decoding yields the signal values of that
message in engineering units.

### Cache

The cache ingests decoded frames from one ingest thread and serves the
latest value per signal to multiple reader threads.

### Requirements (from the stakeholder workshop)

1. (Platform) Frame payloads must be processed zero-copy: decoding
   reads bytes where the receive buffer placed them, and a cache entry
   keeps using those bytes in place, including after the receive-buffer
   slot they occupy is reused for new frames.
2. (Tooling) A cache read returns the signal's latest value at any time,
   independent of what the ingest thread is doing, and the returned value
   remains valid for as long as the reader holds it.
3. (Platform) The receive buffer is a fixed ring. Buffer slots are
   recycled as new frames arrive.
4. (Diagnostics) Readers may subscribe to a signal and iterate values as
   they arrive.
5. (All) The public API must be documented, and a downstream team must be
   able to mock the cache in their tests.

### Deliverables

The `framecache` crate, and an `examples/` binary wiring a synthetic
frame source through decode into the cache with two reader threads.

## Working the fill

Even with the public surface frozen, fill it in slices, one ask at a
time: data types and validation first, then core behavior, then
subscriptions and concurrency, then tests. Review between slices. A
single everything-at-once commission removes the review points this
course is designed to create.
