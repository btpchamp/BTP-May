### Promise Methods Summary

| Method | What It Does | Resolves When | Rejects When |
|--------|-------------|--------------|-------------|
| `Promise.all([p1, p2, p3])` | Run all in parallel | ALL succeed | ANY one fails |
| `Promise.race([p1, p2, p3])` | Race — first result wins | First one settles | First one rejects |
| `Promise.allSettled([...])` | Wait for all (success or failure) | All settle | Never rejects |
| `Promise.any([p1, p2, p3])` | First SUCCESS wins | First one resolves | ALL fail |
