# KaleidOS fork

`main` starts at upstream `mango` commit
`75d5f51cf918ba3333f8a5a8098fc70192aebdbc`. The upstream `dragonos`
default branch is a separate integration branch; it was not used as the
feature baseline for this fork.

The KaleidOS adapter uses only `load_read_only_checked`, `getattr`, `lookup`,
`listdir`, and offset-based `read`. It refuses writes and reliable flush,
and therefore never activates journal recovery or writable transactions.
The upstream write/journal implementation is retained for later work.

The read-path audit found disk buffers were interpreted with aligned
`ptr::read` and immutable extent references. The fork checks slice length,
uses `read_unaligned` and decodes immutable extent fields by value. Native
struct layouts and integer fields are retained; the supported KaleidOS
platforms are little-endian RV32/RV64. Leaf directory validation rejects
non-UTF8 names before the existing String API can interpret them, preserving
ENOTSUP instead of turning it into EIO. Superblock block-size computation
uses a checked shift for untrusted geometry.

These changes do not constitute an audit of writable extent views, xattr
mutation, arbitrary public AsBytes callers, crash recovery, or the entire
library. Do not infer safe writable mounts or reliable storage barriers
from the read-only tests.

Upstream mango's direct-range unit test still expected a one-block write
to be rejected after its actual minimum was reduced to one block. That
failure was reproduced on the pristine upstream revision. Only the stale
test expectation was updated; production write behavior was preserved.
