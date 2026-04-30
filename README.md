# modlock

Disable automatic kernel module loading on Linux. Single-file Python 3 script, no dependencies.

## Why

Automatic module loading is a huge, mostly-ignored attack surface. Userspace can poke the kernel into loading ancient, rarely-audited module code on demand: an obvious recipe for disaster. [copy.fail](https://copy.fail/), for example, doesn't work if autoloading is off.

## How it works

1. Snapshot the currently-loaded modules (`lsmod`) into `/etc/modules` so they get loaded at boot.
2. Write `/bin/true` to `/proc/sys/kernel/modprobe` so the kernel can no longer satisfy userspace module requests.
3. Install a systemd unit that re-applies step 2 on every boot, immediately after `systemd-modules-load.service` processes `/etc/modules`.

## Commands

| | |
|---|---|
| `modlock --update`    | Snapshot loaded modules → `/etc/modules` (overwrites) |
| `modlock --lock`      | Set `/proc/sys/kernel/modprobe` = `/bin/true` |
| `modlock --unlock`    | Set `/proc/sys/kernel/modprobe` back to the real modprobe |
| `modlock --install`   | Copy script to `/usr/local/sbin/modlock`, install + enable `modlock.service` |
| `modlock --uninstall` | Remove the service (leaves the binary in place) |

All commands require root.

## Setup

Boot normally. Make sure every device you care about is up and every service you need has started. Modules not loaded at this moment will not be loaded after install.

```
./modlock --update
./modlock --install
./modlock --lock
```

Done.

## Adding hardware or software later

Anything that wants a new module will fail until you unlock:

```
 modlock --unlock     # autoload back on
# plug / install / configure the new thing
 modlock --update     # capture the expanded module set
 modlock --lock       # gate shut until reboot
```

`--install` only needs to run once. `--update` + `--lock` is enough thereafter.

## Requirements

Linux with systemd, Python 3, root.
