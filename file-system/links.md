# Links

Just as on POSIX, links come in two flavours: hard links and soft links. Neither is superior; they are suited to different use cases. Each may contain the other; the major difference is if the reference is mutable or immutable.

## Hard Links

Hard links are "merely" CIDs. They point directly to the exact bits that the CID represents. If these bits originate on a different account, they are copied into the linking file system.

## Soft \(Symbolic\) Links

Soft links are URLs. They are relative to a mutable pointer, such as a DNSlink. They have the advantage of updating along with versions, but are more liable to break.

## Comparison

|  | Hard Links | Soft Links |
| ---: | :--- | :--- |
| Type | CID | URL |
| Mechanism | Copy | Reference |
| Mutability | Immutable | Mutable |
| Availability Guarantee | ❌ | ❌ |
| Self-validating | ✅ | ❌ |
| Subpaths | ✅ | ✅ |
| Cycles | ❌ | ✅ |
| Auto-update | ❌ | ✅ |
| Web 2 Compatible | ❌ | ✅ |

