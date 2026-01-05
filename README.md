# quicue-proxmox

Proxmox VE provider for [quicue](https://github.com/quicue/quicue). Implements `quicue.ca/vocab` action interfaces with `qm`, `pct`, and `pvecm` commands.

**Security note:** Command strings use direct interpolation. Do not pass untrusted input as parameters.

## Install

### From Registry (when published)

```bash
export CUE_REGISTRY='quicue.ca=ghcr.io/quicue'
cue mod tidy
```

### Local Development

```bash
mkdir -p cue.mod/pkg/quicue.ca
ln -sf /path/to/quicue/vocab cue.mod/pkg/quicue.ca/vocab
ln -sf /path/to/quicue/patterns cue.mod/pkg/quicue.ca/patterns
```

## Usage

```cue
import "quicue.ca/proxmox/patterns"

// VM with all action types
_web: {VMID: 100, NODE: "pve1"}

web_server: {
    vm:        patterns.#VMActions & _web
    lifecycle: patterns.#VMLifecycle & _web
    snapshots: patterns.#VMSnapshots & _web
}

// LXC container
dns: patterns.#ContainerActions & {CTID: 101, NODE: "pve1"}

// Hypervisor node
pve1: patterns.#HypervisorActions & {NODE: "pve1"}
```

## Patterns

### Action Patterns (proxmox.cue)

Direct implementations of vocab interfaces:

| Pattern | Implements | Actions |
|---------|------------|---------|
| `#VMActions` | `vocab.#VMActions` | status, console, config |
| `#VMLifecycle` | `vocab.#LifecycleActions` | start, stop, restart, suspend, resume, reset |
| `#VMSnapshots` | `vocab.#SnapshotActions` | list, create, revert, remove |
| `#ContainerActions` | `vocab.#ContainerActions` | status, console, logs, config |
| `#ContainerLifecycle` | `vocab.#LifecycleActions` | start, stop, restart |
| `#ContainerSnapshots` | `vocab.#SnapshotActions` | list, create, revert, remove |
| `#HypervisorActions` | `vocab.#HypervisorActions` | list_vms, list_containers, cluster_status |
| `#ConnectivityActions` | `vocab.#ConnectivityActions` | ping, ssh, mtr |
| `#GuestAgent` | `vocab.#GuestActions` | exec, upload, download, info |
| `#BackupActions` | - | backup, list_backups |

### Action Templates (templates.cue)

Standalone templates with UPPERCASE parameters for flexible composition:

```cue
import "quicue.ca/proxmox/patterns"

_T: patterns.#ActionTemplates

// Build actions from templates
actions: {
    ping: _T.ping & {IP: "10.0.1.5"}
    ssh:  _T.ssh & {IP: "10.0.1.5", USER: "root"}
    pct_status: _T.pct_status & {CTID: 101, NODE: "pve1", NODE_HOST: "10.0.0.1"}
}
```

Available templates use UPPERCASE parameters: `ping` (IP), `ssh` (IP, USER), `info` (NAME), `pct_status` (CTID, NODE, NODE_HOST), `pct_console`, `pct_start`, `pct_stop`, `pct_config`, `qm_status` (VMID, NODE, NODE_HOST), `qm_config`, `qm_console`, `qm_start`, `qm_stop`, `list_vms` (IP, USER), `list_containers`, `cluster_status`, `storage_status`, `check_dns`, `query_zones`, `verify_resolution`, `proxy_health`, `reload_config`, `test_config`, `gpu_info` (IP, USER), `docker_ps`, `disk_usage`, `list_active_sessions`, `check_auth_log`, `check_vault`, `git_health`, `list_repos`, `prometheus_targets`

### Auto-Generation (graph.cue)

Generate actions automatically based on resource fields and `@type`:

```cue
import "quicue.ca/proxmox/patterns"

_resources: {
    "dns": {
        "@type": ["DNSServer", "LXCContainer"]
        ip: "10.0.1.10"
        ctid: 100
        node: "pve1"
    }
}

_nodes: {
    "pve1": {ip: "10.0.0.1"}
}

infraGraph: patterns.#InfraGraph & {
    Templates: patterns.#ActionTemplates
    Nodes: _nodes
    Resources: _resources
}

// infraGraph.Output["dns"].actions now has:
// - ping (from ip field)
// - pct_status, pct_console (from ctid + node)
// - check_dns, query_zones, verify_resolution (from @type: DNSServer)
```

## Export Commands

```bash
# Get specific command
cue export ./examples -e web_server.vm.status.command --out text
# -> ssh pve1 'qm status 100'

# Export all actions as YAML
cue export ./examples -e web_server.vm --out yaml
```

## Run Tests

```bash
cue vet -c ./...
```

## Publishing

```bash
# Authenticate to ghcr.io
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Publish (requires quicue.ca to be published first)
CUE_REGISTRY='quicue.ca=ghcr.io/quicue' cue mod publish v0.1.0
```

## License

MIT
