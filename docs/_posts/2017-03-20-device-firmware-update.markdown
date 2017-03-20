---
layout: post
title: "Device Firmware Update (DFU)"
date: 2017-03-15 16:00:00 +0100
categories: ble, http
---

# Context

Most IoT product embed a firmware update feature, allowing them to be kept up-to-date with
features and security issues. No standard exists in bluetooth to offer such a feature, all
solutions are entirely proprietary. 

Software update is probably the most critical feature an IoT device can have, as it can be
necessary to update the device to keep it working with latest mobile OS releases, or to
protect it from a previously unknown security flaw.

It is critical for a second reason: it can be botched and brick the device. It can also be
counterfeit and pushed to the device as a way to compromise the users privacy or to use the
device model as a botnet. As such, any software update mechanism should offer some security 
guarantees, such as checking authenticity, being resilient to power supply failures or
spurious reset events.

In COIoT, the gateway acts as a trusted party and is the one in charge of checking firmware
authenticity; it does not mean no check must be performed on the device-side (the gateway
could itself be compromised), but it acts as a basic security layer. The device is only in
charge of applying the update with the guarantee that it will not be bricked.

The general architecture of a COIoT setup is as follows:

![The gateway is connected to the Internet by HTTP, and to a network of devices by BLE]({{site.url}}/img/typical_setup.png)

The HTTP and the BLE protocols used by the gateway when a device update is performed.

Only the HTTP protocol is used when a gateway update is performed.

# HTTP Firmware transfer

The firmware is packaged so that a software update corresponds to a single package, with a
unique version string.

## Firmware packaging

COIoT packaging in its draft version has the following format:

`V | Signature | Firmware version | Changelog | Format | Firmware`

- **V**: Version of the COIoT packaging method, 1 byte, set to 0 for this version
- **Signature**: Signature of the package, OpenPGP Message Format([rfc4880]).
- **Version**: *Firmware Revision String* [dis] of the firmware contained in the package
- **Changelog**: Firmware changelog base URL
- **Firmware**: Binary of the firmware

### Firmware HTTP signature

The signature is calculated from the hash of the concatenated fields following it

`Sign(Hash(Firmware version | Changelog | Format | Firmware))`

When depackaging the firmware, the gateway MUST verify the signature, log as error and
reject any firmware for which verification of the signature failed.

The gateway MUST reject unsupported signature or hash algorithms.

The following signature algorithms are supported ([www.keylength.com]):
- RSA with a public factoring modulus size over-or-equal 3072 bits
- ECDSA with a key size over-or-equal 384 bits

The following hash algorithms are supported:
- SHA384
- SHA512
- SHA3-384
- SHA3-512

### Firmware changelog

The firmware changelog is a 0-terminated encoded URL that can be used by the gateway to
display the firmware changes to the user; the gateway modifies the called URL, adding
the following query arguments, if they are not already set:
- **cfw**: current firmware version of the device
- **nfw**: next firmware version of the device (the one it has been updated to)
- **lang**: language to display the changelog in, in ISO format ([ISO-639-1])

The cfw an nfw parameters allow for a user to get a complete changelog if the device
updated several times in a row.

### Firmware format

The firmware format is a 0-terminated ASCII-encoded string that contains a list of
comma-separated MIME types describing the formats of the data that follows,that is,
the firmware.
The gateway is in charge of handling the firmware type in order to get from this format
to a list of files to transfer to the device for the update. The special value "" means
the firmware is in raw format, eg it consists of a single file that is not compressed.

The packaging format must be handled right-to-left, that is `application/x-tar,application/x-xz`
represents the classical .tar.xz archive file-format.

The gateway MUST support the following MIME formats:
- application/x-tar
- application/x-bzip2
- application/x-xz
- application/zip

## Firmware transfer

The firmware itself is accessed from the gateway using HTTP, via a GET to a known URL.
The connection through which the firmware is downloaded MUST be secured using HTTPS and a
certificate trusted by the gateway.

The security of the gateway certificates, especially in regard to CRL and PKI, is outside
the scope of this document.

### HTTP/2 support

The gateway MUST support HTTP/2. The server delivering the firmware SHOULD support HTTP/2
connections. [http/2]

# BLE firmware transfer

The protocol used for firmware transfer manages several aspects that could lead to a
firmware corruption (ie bricked device):
- packets integrity
- stream integrity in a mesh network
- support for writing delays on the device-side
- support for the device being unavailable during an update
- support for cancelling an update of the device *mid-flight*

## 

# References

1. [Bluetooth Core Specification v5.0](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=421043)
2. [RFC-7696 Guidelines for Cryptographic Algorithm Agility and Selecting Mandatory-to-Implement Algorithms](https://tools.ietf.org/html/rfc7696)
