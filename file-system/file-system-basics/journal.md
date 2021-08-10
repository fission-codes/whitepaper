---
description: Journal / events / logs
---

# Events

A variant on a typical filesystem journal, each vnode includes a description of the state transition that got it into the current state from its previous version. Depending on your background, this can also be viewed as an event source or an audit log. Unlike most common file system journals, the entries are permanent, and form a linked list across vnodes.

While the file system itself only logs events about its own structure, the program that makes the request may additionally include information about what it did with richer semantics than ”file changed”.

WNFS features delegated write access. The user instance that performed the update must include their instance DID, and a hard link to their UCAN.

The exact format of the event is still under development, but will look something along these lines:

{% tabs %}
{% tab title="Haskell" %}
```haskell
data Event appAction = Event
  { writer    :: DID
  , action    :: FSAction
  , signature :: Signature
  , appAction :: appAction
  }
  
type FSEvent = Event ()

data FSAction
  = FileUpdate
  | MetadataAction
  | ChildAction
  
data ChildAction
  = Add    LinkName
  | Remove LinkName
  | Update LinkName
  
data MetadataAction = MetadataAction Key Value
```
{% endtab %}
{% endtabs %}

This stream of data can also be consumed as input to other programs. For instance, a user may want to watch another file system for changes to a particular directory. For example:

* An audio directory being used the source of a podcast \(”which is the new item?”\)
* A user writes to the a private subtree, but another part of the tree with different permissions needs to be kept in sync

