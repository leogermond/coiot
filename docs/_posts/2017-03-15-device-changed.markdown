---
layout: post
title: "Service Changed attribute"
date: 2017-03-15 16:00:00 +0100
categories: ble, gatt
---

# Context

In GATT each attribute (service, characteristic) is identified by its *attribute handle*. As
specified by spec, they must not change on their own for caching reasons; the way to update them
(either updating, adding, or deleting a handle) is to indicate it to other devices through the
*Service Changed* characteristic.
The cache can be retained accross connections if the client has a trusted relationship with
the server. (§2.5.2 of [[1]](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=421043))

# Service changed in COIoT

The issue there is with adding a *Service Changed* characteristic (UUID 0x2803) to an existing
product is a chicken-and-egg problem: to add it you need to give it an attribute handle
but you cannot erase the client's cache to indicate this change if you don't have the
*Service Changed* attribute.

Worse, as most IoT product are working using trusted connections, for eg privacy issues, the
cache could remain accross connection and a case where no amount of software workarounds can
clear it could arise.

This added to the fact that you can not foresee a change in attributes, makes that any device
that supports software updates MUST support the *Service Change* attribute. That in turn makes
the support for *Characteristic Value Indication* mandatory.
(§4.2 of GATT in [[1]](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=421043))

# Example

A device has an update to a new major version that gives access to its battery informations,
to offer this supports the updated firmware adds the Battery Service and associated characteristics.

# Testing

- A device has a *Software Revision Number* in its *Device Information Service*: it can therefore
be updated, and must have the *Service Changed* characteristic.

# References

1. [Bluetooth Core Specification v5.0](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=421043)
