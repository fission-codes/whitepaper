# Layout

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

