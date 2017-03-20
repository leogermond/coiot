---
layout: post
title: "Device Firmware Update: Part 1: HTTP"
date: 2017-03-20 12:00:00 +0100
categories: http, security
---

# Context

Most IoT product embed a firmware update feature, allowing them to be kept up-to-date with
features and security issues. No standard exists in bluetooth to offer such a feature, all
solutions are entirely proprietary. 

Software update is probably the most critical feature an IoT device can have, as it can be
necessary to update the device to keep it working with latest mobile OS releases, or to
protect it from a previously unknown security flaw and at the same time a badly secured
device update is the gateway to most IoT attacks: the update firmware can be
counterfeit and pushed to the device as a way to compromise the users privacy or to use the
device model as a botnet.

It is critical for another reason: it can be botched and brick the device. As such, any software update mechanism should offer some security 
guarantees, such as checking authenticity, being resilient to power supply failures or
spurious reset events.

In COIoT, the gateway acts as a trusted party and is the one in charge of checking firmware
authenticity; it does not mean no check must be performed on the device-side (the gateway
could itself be compromised), but it acts as a basic security net. The device is in
charge of applying the update with the guarantee that it will not become bricked.

The general architecture of a COIoT setup is as follows:

![The gateway is connected to the Internet by HTTP, and to a network of devices by BLE]({{site.url}}/img/typical_setup.png)

The HTTP and the BLE protocols are used by the gateway to update a device.

When a gateway updates, it uses the HTTP protocol.

# Firmware packaging

The firmware is packaged so that a software update corresponds to a single package, with a
unique version string.

In version 0 of the protocol, packages have the following format:

`V | Signature | Firmware version | Changelog | Format | Firmware`

- **V**: Version of the COIoT packaging method, 1 byte, set to 0 for this version
- **Signature**: Signature of the package, OpenPGP Message Format [[1]](https://tools.ietf.org/html/rfc4880).
- **Version**: *Firmware Revision String* [[2]]({{ site.baseurl }}{% post_url 2017-03-13-device-information %}) of the firmware contained in the package
- **Changelog**: Firmware changelog base URL
- **Firmware**: Binary of the firmware

## Firmware HTTP signature

The signature is calculated from the hash of the concatenated fields following it

`Sign(Hash(Firmware version | Changelog | Format | Firmware))`

When depackaging the firmware, the gateway MUST verify the signature, log as error and
reject any firmware for which verification of the signature failed.

The gateway MUST reject unsupported signature or hash algorithms.

The following signature algorithms are supported [[3]](http://www.keylength.com):
- RSA with a modulus size over-or-equal 3072 bits
- ECDSA with a key size over-or-equal 384 bits

The following hash algorithms are supported:
- SHA384
- SHA512
- SHA3-384
- SHA3-512

## Firmware changelog

The firmware changelog is a 0-terminated encoded URL that can be used by the gateway to
display the firmware changes to the user; the gateway modifies the called URL, adding
the following query arguments, if they are not already set:
- **cfw**: current firmware version of the device
- **nfw**: next firmware version of the device (the one it has been updated to)
- **lang**: language to display the changelog in, in ISO format [[4]](https://www.loc.gov/standards/iso639-2/php/code_list.php)

The cfw an nfw parameters allow for a user to get a complete changelog if the device
updated several times in a row.

## Firmware format

The firmware format is a 0-terminated ASCII-encoded string that contains a list of
comma-separated MIME types describing the formats of the data that follows,that is,
the firmware.
The gateway is in charge of handling the firmware type in order to get from this format
to a list of files to transfer to the device for the update. The special value ""
(empty string) means the firmware is in raw format, eg it is not an archive or compressed.

The packaging format must be handled right-to-left.

eg: `application/x-tar,application/x-xz` is an `.tar.xz` archive file-format.

The gateway MUST support the following MIME formats:
- `application/x-tar`
- `application/x-bzip2`
- `application/x-xz`
- `application/zip`

# Firmware transfer

The firmware itself is accessed from the gateway using HTTP, via a GET to a known URL.
The connection through which the firmware is downloaded MUST be secured using HTTPS and a
certificate trusted by the gateway.

The security of the gateway certificates, especially in regard to CRL and PKI, is outside
the scope of this document.

## HTTP/2 support

The gateway MUST support HTTP/2. The server delivering the firmware SHOULD support HTTP/2
[[5]](https://http2.github.io/) connections. 

# References

1. [RFC-4880:  OpenPGP Message Format](https://tools.ietf.org/html/rfc4880)
2. [COIoT Device Information Service]({{ site.baseurl }}{% post_url 2017-03-13-device-information %})
3. [BlueKrypt Cryptographic Key Length Recommendation](http://www.keylength.com)
4. [ISO-639-1: Codes for the Representation of Names of Languages](https://www.loc.gov/standards/iso639-2/php/code_list.php)
5. [HTTP/2](https://http2.github.io/)
6. [RFC-7696: Guidelines for Cryptographic Algorithm Agility and Selecting Mandatory-to-Implement Algorithms](https://tools.ietf.org/html/rfc7696)

