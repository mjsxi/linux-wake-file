# Linux USB Wake from Suspend

udev rules to let specific USB devices wake the machine from suspend.

Based on the [Arch Wiki: Bluetooth § Wake from suspend](https://wiki.archlinux.org/title/Bluetooth#Wake_from_suspend).

## How it works

Most USB devices aren't allowed to wake the system from suspend by default. These udev rules set `power/wakeup` to `enabled` for matched devices whenever they're connected, so a wake event on that device brings the machine out of S3.

## Find your device IDs

```sh
lsusb
```

Output looks like:

```
Bus 001 Device 005: ID 28de:1304 Valve Software Steam Controller
Bus 001 Device 004: ID 7392:c611 Edimax Technology Co., Ltd
```

The `vendor:product` pair (`28de:1304`, `7392:c611`) is what you'll plug into the rule.

## Install

1. Create a rules file in `/etc/udev/rules.d/`, e.g. `99-wakeup.rules`.
2. Add one line per device (see below), substituting your own IDs.
3. Reload:

   ```sh
   sudo udevadm control --reload-rules && sudo udevadm trigger
   ```

## Rules

```
# usb bluetooth dongle
SUBSYSTEM=="usb", ATTRS{idVendor}=="7392", ATTRS{idProduct}=="c611" RUN+="/bin/sh -c 'echo enabled > /sys$env{DEVPATH}/../power/wakeup;'"

# steam controller (2026) dongle
SUBSYSTEM=="usb", ATTRS{idVendor}=="28de", ATTRS{idProduct}=="1304" RUN+="/bin/sh -c 'echo enabled > /sys$env{DEVPATH}/../power/wakeup;'"
```

## Verify

After reloading, check that the device is allowed to wake:

```sh
grep . /sys/bus/usb/devices/*/power/wakeup
```

You should see `enabled` for the target device's path. Then suspend with `systemctl suspend` and try waking it with the device.

## Notes

- Rule files are loaded in lexical order; the `99-` prefix is just a convention to put user rules after most defaults.
- If a device shows up under a USB hub, `$env{DEVPATH}/../power/wakeup` still resolves to the right node because the rule fires on the device itself.
- Some BIOSes also need USB wake enabled at the firmware level — check there if rules look correct but nothing wakes.
