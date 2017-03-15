# Description of the smarter coffee api

## Sources & Acknowledgement

* https://github.com/jkellerer/fhem-smarter-coffee
* https://github.com/AdenForshaw/smarter-coffee-api
* https://github.com/petermajor/SmartThingsSmarterCoffee
* https://github.com/evilsocket/coffee
* https://github.com/AdenForshaw/smarter-coffee-api
* https://github.com/matthanley/smarter-coffee-api
* https://github.com/Half-Shot/Smarter-Coffee-NET
* https://github.com/Tristan79/iBrew

## Basics

* Communication is via a simple TCP socket, port is **2081**
* Discovery is via UDP broadcast `\x64\x7e` (see info command), also on port **2081**
* Commands and Responses are binary, represented here as hex
* Each command starts with 1 byte, identifying the command / method
* Each command is terminated with `\x7e`
* Length of total command is therefore variable, but minimum 2 bytes
* The response codes seem to be mostly `requestcode` + `\x01`, e.g. `\x47` for `\x46`, or just `\x03`

## Command reference

| name | hex | # of args | args | example | response hex | # of response args | response args |
| ---- | --- | --------- | ---- | ------- | --- | --- | --- |
| get status | `\x00` | 0 | *none* | `\x00\x7e` | `\x32` | 5 | status,*unknown*+waterlevel,*unknown*,strength,*unknown*+cups |
| reset | `\x10` | 0 | *none* | `\x10\x7e` |
| brew | `\x37` | 0 | *none* | `\x37\x7e` |
| brew with settings | `\x33` | 4 | cups, strength, hotplate, grinder | `\x33\x01\x01\x01\x01\x7e` |
| set defaults | `\x38` | 4 | strength, cups, grinder, hotplate | `\x38\x01\x01\x01\x01\x7e` |
| get defaults | `\x48` | 0 | *none* | `\x48\x7e` | `\x47` | 4 | cups, strength, grinder, hotplate_on_for_minutes |
| stop | `\x34` | 0 | *none* | `\x34\x7e` |
| set strength | `\x35` | 1 | strength | `\x35\x01\x7e` |
| set cups | `\x36` | 1 | cups | `\x36\x01\x7e` |
| toggle grinder | `\x3c` | 0 | *none* | `\x3c\x7e` |
| hotplate on | `\x3e` | 1 | minutes | `\x3e\x7e` |
| hotplate off | `\x4a` | 0 | *none* | `\x4a\x7e` |
| carafe required | `\x4c` | 0 | *none* | `\x4c\x7e` |
| single cup mode status | `\x4f` | 0 | *none* | `\x4f\x7e` |
| info | `\x64` | 0 | *none* | `\x64\x7e` | `\x65` | 2 | *unkown*, fw-version |
| history | `\x46` | 0 | *none* | `\x46\x7e` |

### info

* command `\x64`
* response `\x65`

#### Response args

* `\x65`
* `\x02` ? maybe device identifier ?
* `\x16` == firmware version `22`
* `\x7e`

### get status

* command `\x00`
* response `\x32`

#### Response args

* Status bits (see below)
* 4 bits unknown (apparently always 1) + 4 bits waterlevel
* 8 bits unknown
* 4 bits unknown + 4 bits number of cups

## Status reference

Status is 1 byte, the bits meaning:

1. `00000001`: carafe present
2. `00000010`: grinder activated
3. `00000100`: machine ready
4. `00001000`: *unknown*
5. `00010000`: brewing / descaling
6. `00100000`: heating
7. `01000000`: hotplate on
8. `10000000`: *unknown*

## Water levels

* `\x00`: empty, `0.0`
* `\x01`: low, `0.25`
* `\x02`: half, `0.5`
* `\x03`: full, `1.0`

## Strength

* `\x00`: weak
* `\x01`: medium
* `\x02`: strong

## Number of cups

* `\x01`: `1`
* etc. until
* `\x0c`: `12`

## Grinder status

* `\x00`: disabled => pre-ground coffee in filter
* `\x01`: enabled
