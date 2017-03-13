---
layout: post
title:  "Device Information Service"
date:   2017-03-13 10:44:28 +0100
categories: ble, gatt
---

# Foreword & context

In Bluetooth, device information is used to expose the manufacturer and/or vendor
information about a device. It uses the UUID 0x180a [[2]](https://www.bluetooth.com/specifications/gatt/services).

The service has several characteristics defined, all of them optional. COIoT makes use of
some of them, making them __mandatory__.

# Device UI identification

The service MUST implement the following characteristics:

- Manufacturer Name String
- Model Number String
- Serial Number String

These caracteristics' values SHOULD NOT change during the life of the product.

These characteristics are used by COIoT's UI to have a simple unambiguous identifier to
display to a client.

## Example

The device manufactured by Foo that has a model number bar and a mac 1a:2b:3c:4d:5e:6f
may have the following characteristics:
- Manufacturer Name String: Foo
- Model Number: bar
- Serial Number String: 5e6f

COIoT's UI could then display the following name: "Foo's bar device (serial 56ef)" or in
short "bar #5e6f" and allow for sorting by eg manufacturer or model.

## Testing

- Device manufacturer, model or serial number missing: device is not recognized by COIoT
- Device long name contains manufacturer, model number and serial number

# Upgrade support

The device MAY implement the following characteristic:

- Software Revision String

When set, the format of this string May be of the form *M.n[/r]* where **M** is the major
version, **n** the minor version, **r** an implementation-dependent revision number.

- M: Major number, an integer
- n: Minor version, an arbitrary string not containing the symbol "/"
- r: Revision informations, an arbitrary string

When the major or minor changes, COIoT MUST try to update the device to the new
version.

COIoT UI MUST display a warning if a device is not up-to-date and if an update is available
with a major number that is higher than the current major number. The user MAY then refuse the
update, and COIoT MUST NOT ask again in the future for any update to this major version.

COIoT SHOULD ignore any change to the revision informations but SHOULD make it available to
the user (for debugging purposes).

This behaviour is intended so that any update can be deployed as-soon-as-possible to the
park without the user's intervention; while still letting the user in control of compatibility
issues for its device. This versionning scheme is also useful for having a separate beta and
production firmware deployed in parallel. The revision information can be used to mark builds
as, eg, manual or testing.

## Example

The device "bar #5e6f" has the following characteristic

- Software Revision String: "2.4/rev. 2e1f45 on forge.bar.com"

COIoT, detecting that an update is available to version "2.4.1/rev. 784a1a on forge.bar.com",
will update the device to this version.

If an update is available to version "3.1/rev. 93a4cc on forge2.bar.com", COIoT will first
prompt the user for an update. If the user refuses to update, COIoT will not ask again in the
future to switch to major 3, while still applying any updates that becomes available for the 
major 2.

## Testing

- Device without Software Revision String is still recognized by COIoT
- Update available with same major: update is done, no warning is displayed
- Update available with different major: a warning is displayed
- Refuse update to firmware M.n, M.n2 becomes available: update is not done, no warning is
displayed

# References

1. [Device Information Service v11r00](https://www.bluetooth.org/docman/handlers/downloaddoc.ashx?doc_id=244369)
2. [GATT services](https://www.bluetooth.com/specifications/gatt/services)
