# quicue-proxmox

Proxmox VE provider for quicue. Implements action interfaces with qm, pct, and pvecm commands.

**Status:** [Active]
## Installation

```cue
import "quicue.ca/proxmox@v0"
```
## Schemas
| `#VMActions` | VM operations via qm command |
| `#VMLifecycle` | VM lifecycle: start, stop, shutdown, reboot |
| `#VMSnapshots` | VM snapshot management |
| `#ContainerActions` | LXC container operations via pct |
| `#HypervisorActions` | Proxmox host operations |
| `#GuestAgent` | QEMU guest agent operations |

## Patterns
- `3-layer`
- `adapter`

---

Part of the [quicue](https://quicue.ca) ecosystem.
