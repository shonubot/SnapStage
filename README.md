<h1 align="left">
  <img src="/icon.svg" alt="SnapStage Icon" height="52"> <sup>SnapStage</sup>
</h1>

SnapStage is a small, distro‑agnostic utility that lets you queue Snap installs or removals during ISO customization (e.g. inside a chroot) and automatically complete them on the first boot of the system.

It works by maintaining simple manifest files and prefetching Snap packages and their assertions for offline installs. On first boot, a systemd unit installs or removes the queued snaps, preferring cached files over network downloads.

---

## Features

* **Queue installs/removals** while customizing an ISO
* **Offline prefetch** of `.snap` and `.assert` files for fast first boot
* **Automatic first‑boot processing** via systemd
* **Safe in chroot** (doesn’t start snapd until the system is live)
* **Manifest‑based** simple text lists under `/etc/snapstage/`

## Usage

### Add or remove snaps

```bash
snapstagectl add <snap> [channel]
snapstagectl remove <snap>
```

Examples:

```bash
snapstagectl add firefox stable
snapstagectl add gnome-calculator edge
snapstagectl remove thunderbird
```

### Prefetch snaps for offline install

Prefetches all queued snaps (or a specific one) into `/var/local/snapstage/snaps`:

```bash
snapstagectl prefetch
snapstagectl prefetch firefox
```

### Manage queues

```bash
snapstagectl list        # List
snapstagectl unqueue foo # Remove 'foo' from install list
snapstagectl clear-cache # Delete cache files
```

### On first boot

* `snapstage-firstboot.service` waits for `snapd.service` to start
* Installs queued snaps (offline first, then online if missing)
* Removes queued snaps
* Marks completion with `/var/lib/snapstage/.done`

You can check its logs later with:

```bash
sudo journalctl -u snapstage-firstboot.service
```

---

## Install

Download the release from the releases page and install it via:
```bash
sudo apt install ./snapstage_1.0.deb
```

---

## Notes

* `snapstagectl` is designed to work even when snap is unavailable (e.g., in a chroot). In that case it still queues installs/removals and simply skips validation/prefetch with a friendly note.
* `snapstagectl` checks for `snap info` availability to warn if a snap name is invalid.
* Prefetching uses the host’s `snap download`, so run it on a system with snapd available.
* The service runs once and exits; to re‑run, delete `/var/lib/snapstage/.done` and restart it.
* Works across Debian, Ubuntu, and derivatives.

---

## **Host vs Chroot workflows**

**Option A: Full chroot-only (no snap available)**

Queue snaps in the chroot:
```bash
snapstagectl add firefox stable
snapstagectl add gnome-calculator
snapstagectl remove thunderbird
```

Do not prefetch (it will be skipped automatically).

On first boot, the service installs from the network.

**Option B: Fast offline installs (recommended)**

On a host/VM where snap is available, stage a cache:
```bash
snapstagectl add firefox stable
snapstagectl add gnome-calculator
snapstagectl prefetch           # downloads .snap and .assert to /var/local/snapstage
```

Copy `/var/local/snapstage/{snaps,assertions}` into the image’s `/var/local/snapstage/`.

**Build the ISO. On first boot, the service installs offline from the cache, falling back to online if a file is missing.**

## License

GPL‑3.0 or later — free to use, modify, and redistribute.
