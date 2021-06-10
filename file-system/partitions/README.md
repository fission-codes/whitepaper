---
description: WNFS required top-level links
---

# Partitions

## Base Layout

```text
${username}.fission.codes
  |
  +——public
  |
  +——p[retty]
  |
  +——private
```

* `/public`
  * Unencrypted data, visible by anyone
  * History encoded directly in the hash-linked structure
* `/p`
  * A cached reduction of the `/public` tree
  * Exists to make short, friendly _relative_ links
    * Relative is relative to some root
    * Permalinked if the root is a CID
    * Latest link if the root is a mutable pointer such as DNSLink
* `/private`
  * Encrypted data, with contents fully encrypted
  * Includes:
    * Private files
    * Key sharing

## Common Structure

The public and private subtrees have an identical API:

```yaml
root
  - tags
    - important
      - Kansai Tip # Directory pointer
      - Harlem Shake # File pointer
    - todo
      - Workout 2020
  - collections
    - playlists
      - Workout 2020
        - Born To Be Wild
        - Harlem Shake
        - You Shook Me All Night Long
    - albums
      - Kansai Trip
        - IMG_1234.png
        - IMG_5678.png
        - IMG_9ABC.png
  - formats # indices primarily for lookup
    - music
      - 3gp
      - aac
      - mp3
      # and so on
    - images
      - tiff
      - jpg
      - gif
      - raw
      - png
  - event_streams
    - filesystem
      - [FFS events]
  - app
    - diffuse
      - preferences
  - workspace # unstructured userland
    - inbox # AKA saved or downloads
      - foo.pdf
      - bar.png
    - work
      - My Cool Project
        - README.md
        - app.js
```

Standard WNFS are named in the plural and snake case. All names MUST be [URL safe](https://www.ietf.org/rfc/rfc3986.txt)\(!\)

Files \(leaves\) do not have a canonical path. This is a DAG, not a tree. They DO have a canonical hash \(content addressing\).

## Internal Structure

```haskell
-- Fission FileSystem Rough Schema

newtype Raw = Raw ByteString
newtype FileName = FileName Text

data Version = Version
  { current  :: Natural -- Just an increasing counter of chain length... maybe just calculate at runtime?
  , comment  :: Text
  , previous :: Maybe File
  }

type File = File
  { raw      :: Raw
  , metadata :: Map Text Text
  , version  :: Version
  }

Type Directory = Directory
  { files    :: Map Text Content
  , metadata :: Map Symbol Text
  , version  :: Version
  , events   :: [Event]
  , public   :: Maybe Directory
  , tags     :: Tags -- Aggregates child tags. What to do in case of conflict? I guess fully qualify them?
  }

type Content =
  = FileContent  (Maybe AES) File
  | Subdirectory (Maybe AES) Directory

type MetaData
  = OrderedList MetaData
  | Set MetaData
  | Map Symbol MetaTree
  | Text
  | Natural
  | Integer
  | Float

data MetadataEvent
  = AddMeta    { key   :: Symbol
               , value :: Text
               }
  | RemoveMeta { key :: Symbol }

data FileEvent
  = AttachFile { name    :: Text
               , content :: CID
               }
  | DetachFile        { target :: Text }
  | NewFileVersion    { from     :: Text
                      , parent   :: CID
                      , new_data :: CID }
  | RevertFileVersion { from           :: Text
                      , tagret_version :: Natural
                      } -- Or should this just be a new pointer to the old file? Does “revert” need a special place in the ontology?

data DirectoryEvent
  = AttachDirectory   { name    :: Text
                      , content :: CID
                      }
  | DetachDirectory   { target :: Text }
  | UpdateDirContents { target  :: Text
                      , new_cid :: CID
                      , update  :: Event
                      }
  | RenameLink        { from :: Text
                      , to   :: Text
                      }

data TagEvent
  = AddTag    { tag :: Symbol, content :: Text }
  | RemoveTag { tag :: Symbol, from    :: Text }

data CollectionEvent
  = AddToCollection      { collection :: Symbol, content :: Text }
  | RemoveFromCollection { collection :: Symbol, content :: Text }

data Event
  = Metadata   MetadataEvent
  | File       FileEvent
  | Directory  DirectoryEvent
  | Tag        TagEvent
  | Collection CollectionEvent

type TagData = Map Symbol (File | Dir | Tags | Collections)

type Tag = TagsSymbol TagData

type CollectionData = [Map FileName (File | Dir | Tag | Collection)]

type Collection = Collection Symbol CollectionData

```

