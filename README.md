# The MIDIMonster

Named for its scary math, the MIDIMonster is a universal translation
tool between multi-channel absolute-value-based control and/or bus protocols.

Currently, the MIDIMonster supports the following protocols:

* MIDI (Linux, via ALSA)
* ArtNet
* Streaming ACN (sACN / E1.31)
* OpenSoundControl (OSC)
* evdev input devices (Linux)
* Open Lighting Architecture (OLA)

with additional flexibility provided by a Lua scripting environment.

The MIDIMonster allows the user to translate any channel on one protocol into channel(s)
on any other (or the same) supported protocol, for example to:

* Translate MIDI Control Changes into Notes ([Example configuration](configs/unifest-17.cfg))
* Translate MIDI Notes into ArtNet or sACN ([Example configuration](configs/launchctl-sacn.cfg))
* Translate OSC messages into MIDI ([Example configuration](configs/midi-osc.cfg))
* Dynamically route and modify events using the Lua programming language ([Example configuration](configs/lua.cfg) and [Script](configs/demo.lua)) to create your own lighting controller or run effects on TouchOSC (Flying faders demo [configuration](configs/flying-faders.cfg) and [script](configs/flying-faders.lua))
* Use an OSC app as a simple lighting controller via ArtNet or sACN
* Visualize ArtNet data using OSC tools
* Control lighting fixtures or DAWs using gamepad controllers, trackballs, etc ([Example configuration](configs/evdev.cfg))
* Play games, type, or control your mouse using MIDI controllers ([Example configuration](configs/midi-mouse.cfg))

[![Build Status](https://travis-ci.com/cbdevnet/midimonster.svg?branch=master)](https://travis-ci.com/cbdevnet/midimonster) [![Coverity Scan Build Status](https://scan.coverity.com/projects/15168/badge.svg)](https://scan.coverity.com/projects/15168)

# Table of Contents

  * [Usage](#usage)
  * [Configuration](#configuration)
  * [Backend documentation](#backend-documentation)
  * [Building](#building)
    + [Prerequisites](#prerequisites)
    + [Build](#build)
  * [Development](#development)

## Usage

The MIDImonster takes as it's first argument the name of an optional configuration file
to use (`monster.cfg` is used as default if none is specified). The configuration
file syntax is explained in the next section.

## Configuration

Each protocol supported by MIDIMonster is implemented by a *backend*, which takes
global protocol-specific options and provides *instance*s, which can be configured further.

The configuration is stored in a file with a format very similar to the common
INI file format. A section is started by a header in `[]` braces, followed by
lines of the form `option = value`.

Lines starting with a semicolon are treated as comments and ignored. Inline comments
are not currently supported.

Example configuration files may be found in [configs/](configs/).

### Backend and instance configuration

A configuration section may either be a *backend configuration* section, started by
`[backend <backend-name>]`, an *instance configuration* section, started by
`[<backend-name> <instance-name>]` or a *mapping* section started by `[map]`.

Backends document their global options in their [backend documentation](#backend-documentation).
Some backends may not require global configuration, in which case the configuration
section for that particular backend can be omitted.

To make an instance available for mapping channels, it requires at least the
`[<backend-name> <instance-name>]` configuration stanza. Most backends require
additional configuration for their instances.

### Channel mapping

The `[map]` section consists of lines of channel-to-channel assignments, reading like

```
instance.channel-a < instance.channel-b
instance.channel-a > instance.channel-b
instance.channel-c <> instance.channel-d
```

The first line above maps any event originating from `instance.channel-b` to be output
on `instance.channel-a` (right-to-left mapping).

The second line makes that mapping a bi-directional mapping, so both of those channels
output eachothers events.

The last line is a shorter way to create a bi-directional mapping.

### Multi-channel mapping

To make mapping large contiguous sets of channels easier, channel names may contain
expressions of the form `{<start>..<end>}`, with *start* and *end* being positive integers
delimiting a range of channels.  Multiple such expressions may be used in one channel
specification, with the rightmost expression being incremented (or decremented) first for
evaluation.

Both sides of a multi-channel assignment need to have the same number of channels, or one
side must have exactly one channel.

Example multi-channel mapping:

```
instance-a.channel{1..10} > instance-b.{10..1}
```

## Backend documentation

Every backend includes specific documentation, including the global and instance
configuration options, channel specification syntax and any known problems or other
special information. These documentation files are located in the `backends/` directory.

* [`midi` backend documentation](backends/midi.md)
* [`artnet` backend documentation](backends/artnet.md)
* [`sacn` backend documentation](backends/sacn.md)
* [`evdev` backend documentation](backends/evdev.md)
* [`loopback` backend documentation](backends/loopback.md)
* [`ola` backend documentation](backends/ola.md)
* [`osc` backend documentation](backends/osc.md)
* [`lua` backend documentation](backends/lua.md)

## Building

This section will explain how to build the provided sources to be able to run
`midimonster`.

### Prerequisites

In order to build the MIDIMonster, you'll need some libraries that provide
support for the protocols to translate.

* `libasound2-dev` (for the MIDI backend)
* `libevdev-dev` (for the evdev backend)
* `liblua5.3-dev` (for the lua backend)
* `libola-dev` (for the optional OLA backend)
* `pkg-config` (as some projects and systems like to spread their files around)
* `libssl-dev` (for the MA Web Remote backend)
* A C compiler
* GNUmake

### Build

Just running `make` in the source directory should do the trick.

Some backends have been marked as optional as they require rather large additional software to be installed,
for example the `ola` backend. To build these, run `make full` in the backends directory.

## Development

The architecture is split into the `midimonster` core, handling mapping
and resource management, and the backends, which are shared objects loaded
at start time, which provide a protocol mapping to instances / channels.

The API and structures are more-or-less documented in [midimonster.h](midimonster.h),
more detailed documentation may follow.

To build with `clang` sanitizers and even more warnings enabled, run `make sanitize`.
This is useful to check for common errors and oversights.

For runtime leak analysis with `valgrind`, you can use `make run`.
