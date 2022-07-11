# Golaborate

Golaborate is an application for integrating laboratory hardware into automated testbeds.  Example uses include:

1) Corongraphs operated inside a vacuum chamber remotely

2) Ground support equipment to align imaging spectrometers

3) Automated lifetime testing

Users should begin with this README.  Golaborate is divided into two
repositories, a suite of device drivers and server applications and clients for
those servers available in both python and matlab.  The repos are:

* Unified Documentation -
  [Golaborate-docs](https://github.com/nasa-jpl/golaborate-docs) (you are here)
* server - [Golaborate](https://github.com/nasa-jpl/golaborate)
* clients - [Golaborate-clients](https://github.com/nasa-jpl/golaborate-clients)

This documentation is a work in progress.  If you are at JPL, contact Brandon
Dube (383D) via email or phone for assistance.  If you are outside JPL, please
open an issue on this repo and assistance will be provided on an as-time-allows
basis.

## Installation

Precompiled releases of multiserver, andorhttp3, andorhttp2, and dacsrv are
available on the Github releases page.  When self-compile Multiserver is pure
Go.  The andorhttp binaries depend on the installation of andor's SDK version 2
(andorhttp2) or 3 (andorhttp3).  dacsrv uses Cgo but all C dependencies are
vendored.

Installing the Go tooling and compilation are outside the scope of this
documentation.

To install the clients, simply
```sh
$ python -m pip install https://github.com/nasa-jpl/golaborate-clients
```

In the future the clients will be available from PyPi.

## Getting Started

### Multiserver setup
Once multiserver is in hand, a configuration file must be made.  We'll set up a
Physik Instrument motion system and use the mocking feature for the purposes of
the demo.  On the command line, run:

```sh
$ multiserver mkconf
```
This will make a `multiserver.yml` file in the current directory, which is
essentially empty:
```sh
$ cat multiserver.yml
Addr: :8000
Mock: false
Nodes: []
```
Addr is where the program will listen for requests
Mock is a global flag for whether hardware is simulated or real.  Nodes is the
list of hardware that will be run through multiserver.  We'll add two PI
controllers:

```sh
$ cat multiserver.yml
Addr: :8000
Mock: true
Nodes:
  - Type: pi
    Endpoint: my-pi-controller1
    Addr: 192.168.100.1

  - Type: pi
    Endpoint: my-pi-controller2
    Addr: 192.168.100.2

```
The port used must be the default for each vendor and not specified in the
config file.  `Serial: True` can be added to any node to use the serial (RS-232
or RS-422) interface if it is supported.  Then addr would typically be
`/dev/ttyusbN`.

With a valid config, simply start multiserver:
```sh
$ multiserver run
```

The program can be left running idefinitely.  On linux a trailing ampersand can
be used to run it in the background, or a tmux or screen session used.  The logs
are written to stderr, which can be redirected to a file to save them.

### First moves

```python
import motion

c = motion.Controller('localhost:8000/my-pi-controller1', axes=['1', '2', '3'])

x_axis = getattr(c, '1')
x_axis.enable()
x_axis.home()
print(x_axis.pos())
>>> 0
x_axis.velocity(5) # set
print(x_axis.velocity()) # get
>>> 5
x_axis.move_abs(20)
x_axis.move_rel(-5)
print(x_axis.pos())
>>> 15.000000012101383
```
The axes being labeled 1, 2, 3 is a PI semantic; other motion vendors have
different axis naming schemes.

There is no state within the client, an arbitrary number of clients can come and
go without doing anything to the server.  Similarly, the client can be
initialized before the server is turned on.  Typically, the server is run as
part of the infrastructure for a testbed, and users have no concern for starting
or stopping it.

## Multiserver config

### Addr

before the colon a specific address (such as localhost) can be used to limit who
the server listens for.  A trailing slash can be used to modify the rool URL,
e.g. `Addr: localhost:8000/mytestbed` will place hardware at
`localhost:8000/mytestbed/<endpoint>`.
