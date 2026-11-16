# go-ge: the design

Research into GE Appliances' connected appliances, the SmartHQ platform, done on
2026-11-15 from public sources, to decide what a Go module for them would be. The
appliance in view is not yet named, so this is the platform as a whole. Nothing is built
and nothing here was run against an appliance. The decisions at the end are open.

In short: nothing local is known on an appliance's Wi-Fi. The local route is a wire: the
appliance's own serial bus, on a service port, which GE documents and for which GE's
maker arm sells an adapter. Short of hardware, GE has an official cloud API an owner can
sign up for.

## The device

- A connected appliance carries a Wi-Fi module that is set up from the SmartHQ app and
  then talks to GE's cloud. What it sends there is not documented, and nothing found
  shows the app talking to the appliance on the LAN.
- An appliance's state is a set of ERDs: properties with a 16-bit id and a typed value.
  GE publishes the dictionary
  ([public-appliance-api-documentation](https://github.com/geappliances/public-appliance-api-documentation)).
- The boards of an appliance talk over a serial bus, GEA3 on newer appliances and GEA2
  on older ones, and the bus is brought out on an RJ45 service port. The Wi-Fi module
  sits on that same bus.

## Ways in

### The serial bus (local, with hardware)

- GE's maker arm, FirstBuild, sells an adapter: a small ESP32 board on an RJ45 carrier
  that plugs into the service port and publishes the appliance's ERDs to MQTT
  ([its firmware](https://github.com/geappliances/home-assistant-adapter)).
- [esphome-gea](https://github.com/mguaylam/esphome-gea) is an ESPHome component for
  the same port: GEA3 at 230,400 baud, GEA2 at 19,200, reading and writing, with
  configurations for dishwashers, a washer, dryers, water heaters, a refrigerator and a
  wall oven.
- GE publishes C++ libraries for the bus itself
  ([tiny-gea-api](https://github.com/geappliances/tiny-gea-api)).
- What can be written, as against read, depends on the appliance's own firmware.

### GE's cloud API (official)

The SmartHQ developer program ([developer.smarthq.com](https://developer.smarthq.com/)):
REST with a WebSocket for changes, OAuth2 by authorization code, an appliance modelled as
services. GE's [own Home Assistant integration](https://github.com/geappliances/geappliances-smarthq-integration)
is built on it, and says an owner registers an application to get a client id and secret.

### GE's cloud, as the app uses it (unofficial)

The app's older API, by user name and password with a code sent by mail, reading and
writing ERDs over a WebSocket: [gehome](https://github.com/simbaja/gehome). Its own
Home Assistant integration now points its users to GE's.

### Matter

None found on a GE appliance.

## Two ways in, compared

| | The serial bus | The cloud API |
|---|---|---|
| Supported by GE | the port and dictionary are documented; the adapter is FirstBuild's | yes |
| Local at run time | yes | no |
| Needs | a board on the appliance's service port | a SmartHQ account, a registered application |
| Gives | every ERD the appliance publishes; writes where its firmware allows | the services GE models |
| Risk | the port may be behind a panel; writing to a mains appliance | a cloud in the path |

## What this module would be

Not decided. The options, most local first:

1. **A reader of an adapter on the bus**: the Go side speaks MQTT or ESPHome's native
   API to the board and decodes ERDs from GE's published dictionary. A larger variant
   speaks GEA3 itself over a serial adapter.
2. **A client of the official cloud API**: OAuth once in a browser, then REST and a
   WebSocket.

No Go implementation of either was found.

## Open questions

- Which appliance, and its model number: the type decides the ERDs, the model year
  decides GEA3 or GEA2.
- Whether its service port can be reached without removing panels, and whether a board
  left on it is acceptable.

## Not verified

- Nothing above was observed on an appliance.
- The bus's framing and addresses come from one set of reverse-engineering notes, not
  from GE's documents.
- The official API's terms, limits and price: its documentation site refused to be read.
- Whether the appliance in view lets its controls be written from the bus.

## Sources

- [public-appliance-api-documentation](https://github.com/geappliances/public-appliance-api-documentation):
  GE's ERD dictionary.
- [home-assistant-adapter](https://github.com/geappliances/home-assistant-adapter): the
  firmware of FirstBuild's adapter.
- [esphome-gea](https://github.com/mguaylam/esphome-gea): the bus from ESPHome, both
  protocols, the appliances tried.
- [tiny-gea-api](https://github.com/geappliances/tiny-gea-api): GE's library for the bus.
- [developer.smarthq.com](https://developer.smarthq.com/) and
  [geappliances-smarthq-integration](https://github.com/geappliances/geappliances-smarthq-integration):
  the official cloud API and its use.
- [gehome](https://github.com/simbaja/gehome): the app's API.
