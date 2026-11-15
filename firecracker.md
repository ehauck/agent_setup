# Firecracker Codex VM

Small helper toolkit for managing a persistent Firecracker development VM and reusable snapshots.

## Commands

### `fc-fresh`

Create a new VM environment from scratch.

- installs/updates Firecracker
- downloads kernel and Ubuntu rootfs
- creates writable `rootfs.ext4`
- creates an SSH key
- replaces the current persistent VM disk

Use for initial setup or a full rebuild.

### `fc-run`

Boot the current persistent VM from:

```text
~/.firecracker-codex/images/rootfs.ext4
```

Use for normal persistent development.

### `fc-ssh`

SSH into the currently running VM.

Typical use:

```bash
fc-run
fc-ssh
```

### `fc-snapshot [name]`

Create a checkpoint of the currently running VM.

Default:

```bash
fc-snapshot
# same as:
fc-snapshot base
```

Stores:

```text
snapshots/<name>/
├── rootfs.ext4
├── memory.snap
└── vmstate.snap
```

Examples:

```bash
fc-snapshot clean
fc-snapshot codex-ready
fc-snapshot before-upgrade
```

### `fc-restore [name]`

Replace the current persistent VM with a previously created snapshot and resume it.

Default:

```bash
fc-restore
# restores "base"
```

Example:

```bash
fc-restore clean
fc-ssh
```

Changes made after the snapshot are discarded.

### `fc-stop`

Stop the running Firecracker process and remove the TAP device.

---

## Typical Workflows

### Initial Setup

```bash
fc-fresh
fc-run
fc-ssh
```

Configure the guest:

```bash
apt update
apt install ...
```

Install Codex, compilers, SDKs, and other development tools.

Then create a clean checkpoint:

```bash
exit
fc-snapshot clean
```

### Persistent Development

```bash
fc-run
fc-ssh
```

Changes remain in `rootfs.ext4` across VM restarts.

Stop:

```bash
fc-stop
```

Restart later:

```bash
fc-run
fc-ssh
```

### Create a Checkpoint

```bash
fc-snapshot working
```

Continue modifying the VM normally afterward.

### Roll Back

```bash
fc-stop
fc-restore working
fc-ssh
```

The persistent disk is replaced with the state captured by `working`.

### Reset to Clean Codex Environment

```bash
fc-stop
fc-restore clean
fc-ssh
```

### Rebuild Everything

```bash
fc-stop
fc-fresh
fc-run
fc-ssh
```

Then create a new baseline:

```bash
fc-snapshot clean
```

---

## Layout

```text
~/.firecracker-codex/
├── bin/
│   ├── fc-env
│   ├── fc-fresh
│   ├── fc-net
│   ├── fc-run
│   ├── fc-ssh
│   ├── fc-snapshot
│   ├── fc-restore
│   └── fc-stop
│
├── images/
│   ├── vmlinux
│   ├── rootfs.ext4
│   ├── id_ed25519
│   └── id_ed25519.pub
│
├── snapshots/
│   ├── clean/
│   │   ├── rootfs.ext4
│   │   ├── memory.snap
│   │   └── vmstate.snap
│   └── ...
│
└── run/
    ├── firecracker.sock
    ├── firecracker.pid
    └── firecracker.log
```

## Recommended Lifecycle

```text
fc-fresh
   ↓
fc-run
   ↓
configure VM
   ↓
fc-snapshot clean
   ↓
develop
   ↓
fc-snapshot checkpoint
   ↓
experiment
   ↓
fc-restore clean
```

Use `clean` as the known-good Codex/toolchain environment and create additional named snapshots before experimental changes.
