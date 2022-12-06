### Unsafe Traits

Traits can be marked unsafe to indicate that impling the trait might require unsafe code. Both `Send` and `Sync` are marked `unsafe` because if they aren't automatically implemented for a type that means it must contains some non-`Send` or non-`Sync` member and we have to take extra care as the implementers to make sure there are no data races if we want to manually mark the type as `Send` and `Sync`.

```rust
// SomeType is not Send or Sync
struct SomeType {
    not_send_or_sync: *const (),
}
// but if we're confident that our impl doesn't have any data
// races we can explicitly mark it as Send and Sync using unsafe
unsafe impl Send for SomeType {}
unsafe impl Sync for SomeType {}
```
