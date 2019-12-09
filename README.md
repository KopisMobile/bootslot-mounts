# bootslot-mounts

This is a shell script that runs as a [systemd generator][systemd-generator]
very early in the boot process.  It is intended to be used on a device with an
A/B partition scheme, in coordination with [RAUC][rauc].  Because RAUC does not
handle mounting additional slots into the filesystem tree, this script's job is
to automatically select and mount additional partitions using generated
[systemd `.mount`][systemd-mount] files.

## Installation

Install `bootslot-mounts` to `/usr/lib/systemd/system-generators`.
Ensure it is executable.
Then, write a configuration file; see below.

## Configuration file

The configuration file is an INI-formatted file placed at `/etc/bootslot-mounts.conf`.
This has three sections, as follows.

- The `[mountpoints]` section specifies names for mountpoints referred to in
  `slot` sections.  The key of each entry is a "friendly name" of the
  mountpoint, and the value is the absolute path in the filesystem.  Ensure this
  path exists in the filesystem; this script does not currently create it.
- Each `[slot]` section has heading format `[slot_BOOTNAME]`, where `BOOTNAME`
  is the value of the `rauc.slot` string passed on the kernel command line.
  The key of each entry in these sections is the friendly name of each
  mountpoint in the `mountpoints` section, and the value is the name of the
  block device to mount.

Other `mount` functionality, such as manually specifying mount options, is not
supported at this time.

Specifying the root filesystem device is not supported (although this would not
be difficult).  But typically this is passed in by the bootloader.

### Example

The following configuration file defines two mountpoints, `boot` and `var`, and
three slots, `A`, `B`, and `recovery`.  The `boot` mountpoint has an A/B scheme
and is backed by either `/dev/mmcblk0p1` or `/dev/mmcblk0p2`, depending on the
active boot slot.  Likewise, `var` is mounted at `/dev/mmcblk0p5` or
`/dev/mmcblk0p6`.  The `recovery` partition only mounts its special `var` and
does not mount `boot` at all.

```
[mountpoints]
boot=/boot
var=/var

[slot_A]
boot=/dev/mmcblk0p1
var=/dev/mmcblk0p5

[slot_B]
boot=/dev/mmcblk0p2
var=/dev/mmcblk0p6

[slot_recovery]
var=/dev/mmcblk0p7
```

[systemd-generator]: https://www.freedesktop.org/software/systemd/man/systemd.generator.html
[systemd-mount]: https://www.freedesktop.org/software/systemd/man/systemd.mount.html
[rauc]: https://rauc.io/
