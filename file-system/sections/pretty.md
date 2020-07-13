---
description: A reduction cache of the pubic tree for better links
---

# Pretty

One aesthetic problem that arises from the multi-segment approach of the `public` section is that paths become cluttered with extraneous detail. Ideally we’d like to ignore the header section when giving relative paths to a file in web contexts. We achieve this by maintaining a reduction off the FS root.

## Reduction

This reduction only selects for userland paths, discarding headers, metadata, and so on. It encodes directly as protocol nodes to be legible to as many system as possible without needing to understand the FLOOFS schemata or application.

The leaves of the pretty section MUST match the leaves of the public section _at the protocol layer_. This is to say, we count the raw data index of a file, or the userland segment of the DAG as a leaf, but never a header segment or 

Being a reduction, this index cache can always be dropped and rebuilt deterministically.

## Versioning

Bcause the root of the file system can be selected as an absolute reference \(by CID\), FLOOFS still maintains versioning by adding one system-controlled directory at the top of the DAG. Paths loose the ability to seek by file version, and rather page back one full generation at a time. To draw an analogy from git, this is like asking for `HEAD^`.

## Layout

A pun in the layout of the `pretty` section is to view the prefix “pretty” as a function modifier on the enclosed path \(i.e. `/public/...`\)

```text
{root}
  |
  +—— pretty
        |
        +—— meta
        |     |
        |     +—— prev
        |
        +—— public
              |
             ...
```

### Examples

#### Relative / Mutable

As an example, a photo could be referenced at something like:

```bash
alice.fission.name/pretty/public/photos/vacation/beach.png

# Or as an absolute/immutable reference

QmRi3npTRwTp31gWPbi9uGrgmdBicBYhBHWFwRSDxTJmHN/pretty/public/photos/vacation/beach.png
```

This would be a reduction of the public path:

```text
alice.fission.name/public/dir/photos/dir/vacation/dir/beach.png/raw
```

As you can see, the pretty path is much more amenable to human memory and legibility, and just... looks less strange in an email. Browsers are also able to infer the MIME type based on the file extension in the pretty path.

History can be explored from this version backwards with more function-like modifiers. For instance, here are the 3 latest versions of the beach photo example above:

```bash
# Most recent
${root}/pretty/public/photos/vacation/beach.png

# Previous
${root}/pretty/prev/public/photos/vacation/beach.png

# Two previous
${root}/pretty/prev/prev/public/photos/vacation/beach.png
```

