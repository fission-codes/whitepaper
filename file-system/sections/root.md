# Root

The WNFS root is a specialized structure that keeps the disparate segments together.

```haskell
data RootNode = RootNode
  { pretty  :: BareIPFS
  , public  :: PublicNode
  , private :: MMPT
  }
```

Since this layer is completely controlled by WebNative, we don't need to worry about metadata elements conflicting with userland.

## Pretty

The pretty tree \(public reduction\) is represented as simply `/p/` to be unobtrusive in URLs. If this becomes corrupted, it can always be deleted and reconstructed from the source of truth: the public DAG.

