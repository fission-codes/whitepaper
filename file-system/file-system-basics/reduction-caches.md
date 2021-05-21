# Reduction Caches

A copy of a larger structure, with some data removed. This process is inherently lossy by definition, and never introduces new information to the reduction.

### Example

`pretty` is a DAG reduction of the `public` section. It's a reduction index because it contains precisely the same files and paths, but with extra detail removed. This is held directly in the DAG to facilitate human readable URLs.

