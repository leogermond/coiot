# coiot.github.io
COIoT Technical documentation

# FAQ

## What does COIoT stand for

COIoT Opens (the) Internet of Things

## What is COIoT

COIoT is an implementation based on standard BLE + GATT to give access to BLE devices through 
a gateway that acts as an entry point to the BLE network from the outside world (ie the
Internet). COIoT runs both on the peripheral BLE-only device as well as on the gateway, it
does not aim to interface with existing IoT products but rather to live as 
"yet another standard" ([XKCD reference, amirite?](https://xkcd.com/927/)) alongside them.

The main idea behind COIoT is to give the IoT community a FOSS technological brick to build
higher level applications on as well as a reference implementation in the form of various
peripherals and a gateway, all based on easily available embedded hardware.

## What are the advantages of COIoT

There are at least two main advantages to a FOSS IoT solution:

1. security-through-public-scrutiny
2. long-term interroperability for real, instead of relying on a proprietary solution that 
will become deprecated at some unknown point in the future.

## What is the status of COIoT

COIoT is in a exploratory development status, it is not ready to build a proof of concept on
at the moment. Until it reaches the demo-ready status, the code probably won't be released.
